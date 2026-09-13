# Reverse Shell — Post-Exploitation Cheat Sheet

> **Tujuan:** Panduan tindakan setelah reverse shell terhubung — stabilisasi shell interaktif, verifikasi akses, pencarian flag, hingga pengumpulan kredensial & data sensitif untuk eskalasi berikutnya.
> **Prasyarat:** Listener sudah aktif (`nc -lvnp 4444`) dan koneksi reverse shell sudah masuk → lihat `02-reverse-shell/Listener.md` dan `02-reverse-shell/Stabilize_TTY.md`.

---

## Penjelasan Singkat

| Item | Detail |
|------|--------|
| **Kondisi awal** | Shell mentah (non-interactive) dari payload reverse shell |
| **Langkah pertama** | Stabilkan shell menjadi interaktif (TTY) agar nyaman dikendalikan |
| **Target akhir** | Verifikasi user → cari flag → eskalasi ke root → ambil kredensial penting |

```text
Listener aktif (nc -lvnp 4444)
    ↓
Reverse shell masuk (non-interactive)
    ↓
Stabilisasi TTY (script / python pty)
    ↓
Verifikasi akses (id, whoami, sudo -l)
    ↓
Cari flag & kredensial → eskalasi ke root
```

---

## Langkah 1: Stabilisasi Shell (TTY)

### 1. Cara Cepat — `script`

Cara paling cepat mendapatkan shell interaktif tanpa Python:

```bash
script /dev/null -c /bin/bash
```

> **Penjelasan:** `script` membuat pseudo-terminal (PTY) baru dan menjalankan `/bin/bash` di dalamnya. `Ctrl+C` kembali berfungsi normal, dan prompt shell lebih responsif.

### 2. Python PTY

Jika `python3` tersedia di target:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

### 3. Python PTY Fully Interactive (Autocomplete & Ukuran Terminal Benar)

```bash
# Di shell target
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z                      # kembali ke terminal attacker (suspend)

# Di terminal attacker
stty raw -echo              # matikan echo & pemrosesan karakter
fg                          # kembali ke shell target
Enter  Enter                # tekan Enter dua kali

# Di shell target (lagi)
export TERM=xterm
export SHELL=/bin/bash
stty rows 40 cols 120       # sesuaikan ukuran terminal
```

> **Penjelasan:** `stty raw -echo` di sisi attacker membuat karakter dikirim mentah (Tab, panah, Ctrl+D berfungsi). `TERM=xterm` mengaktifkan fitur terminal seperti clear screen.

### 4. Alternatif Lain

```bash
/bin/sh -i                       # paksa interactive shell
script -qc /bin/bash /dev/null   # variasi script dengan -q (quiet)
```

---

## Langkah 2: Verifikasi Akses

```bash
# Identitas user saat ini
id
whoami

# Hostname & info sistem
hostname
uname -a
cat /etc/os-release

# Hak akses sudo
sudo -l

# Koneksi jaringan aktif (pivoting ke host internal)
ip a
ss -tulnp
netstat -tulnp
```

---

## Langkah 3: Pencarian Flag & Data Sensitif

### 1. Lokasi Flag yang Sering Ditemukan

```text
/var/www/html/          # web root — flag.txt, user.txt, .env, konfigurasi
/home/<user>/           # direktori home user — user.txt, flag.txt
/root/                 # flag root — root.txt, flag.txt
/tmp/                  # flag sementara lab
/opt/                  # aplikasi tambahan
/srv/                  # data service
```

### 2. Cari Flag via `find`

```bash
# Cari berkas flag dengan nama umum
find / -iname "flag.txt" 2>/dev/null
find / -iname "user.txt" -o -iname "root.txt" -o -iname "flag*" 2>/dev/null

# Cari berkas kecil bertipe teks di lokasi umum
find /var/www /home /root /opt /tmp -type f -iname "*flag*" 2>/dev/null

# Cari berkas .env (kredensial database sering tersimpan di sini)
find / -name ".env" -type f 2>/dev/null
```

### 3. Cari Kredensial di Dalam Berkas

```bash
# Cari string password/secret di web root
grep -rniE "(password|passwd|secret|api_key|token)\s*=" /var/www/ 2>/dev/null

# Cari koneksi database di berkas konfigurasi
grep -rniE "(db_|jdbc:|mysql:|database)" /var/www/ 2>/dev/null

# Cari kredensial di home directory
grep -rni "password" /home/ 2>/dev/null
```

> **Catatan:** Detail lengkap pencarian berkas & eksploitasi SUID via `find` ada di `07-suplement/find.md`.

---

## Langkah 4: Pengumpulan Kredensial Setelah Root

Setelah berhasil menjadi root (`uid=0`), kredensial penting biasanya tersebar di lokasi-lokasi berikut:

### 1. Berkas Konfigurasi & Environment

```bash
# Environment aplikasi (kredensial database/API)
cat /var/www/html/.env
cat /var/www/*/.env

# Konfigurasi database aplikasi web
cat /var/www/html/wp-config.php          # WordPress
cat /var/www/html/config.php             # aplikasi PHP umum
cat /var/www/html/configuration.php      # Joomla
```

### 2. Hash Password Sistem

```bash
# Hash password seluruh user (root dkk.) — crack offline dengan hashcat/john
cat /etc/shadow

# Informasi user & home directory
cat /etc/passwd
```

### 3. Kredensial Jaringan & Service

```bash
# Kredensial database (plain text di file konfigurasi)
cat /etc/mysql/debian.cnf
grep -rni "password" /etc/mysql/ 2>/dev/null

# Private key SSH target — bisa dipakai login langsung
cat /root/.ssh/id_rsa
cat /home/*/.ssh/id_rsa

# Riwayat perintah user (sering berisi password yang pernah diketik)
cat /root/.bash_history
cat /home/*/.bash_history

# Riwayat MySQL client
cat /root/.mysql_history
```

### 4. Memory & Process

```bash
# Environment variable process yang berjalan (bisa berisi secret)
cat /proc/*/environ 2>/dev/null | tr '\0' '\n' | grep -iE "pass|secret|key"

# Process berjalan — bisa berisi kredensial di command line
ps aux | grep -iE "pass|user|login"
```

---

## Langkah 5: Eskalasi ke Root (Setelah Shell Stabil)

```text
1. Enumerasi sistem:  linPEAS / LES / manual (lihat 03-enumeration/)
2. Cek SUID binary:   find / -perm -4000 -type f 2>/dev/null (lihat 07-suplement/find.md)
3. Cek sudo misconfig: sudo -l (lihat 04-privilege-escalation/02-Sudo.md)
4. Kernel exploit:     LES suggester → DirtyFrag / CopyFail (lihat 04-privilege-escalation/)
5. Ambil bukti root:  id → uid=0(root) (lihat 05-proof/Submission.md)
```

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Minimalisir PTY**: Batasi service web (Apache, nginx, PHP-FPM) agar tidak bisa membuat pseudo-terminal — gunakan `noexec`, seccomp, atau isolasi container.
- **Nonaktifkan Fungsi Berbahaya PHP**: Matikan `system`, `exec`, `shell_exec`, `proc_open`, `fsockopen` bila tidak diperlukan via `disable_functions` di `php.ini`.
- **Amankan `/etc/shadow`**: Pastikan permission `640` dan gunakan hash kuat (`sha512crypt`/`yescrypt`); pindahkan kredensial ke secret manager.
- **Bersihkan History**: Atur `HISTSIZE=0` atau `export HISTFILE=/dev/null` untuk root dan user service agar kredensial tidak tercecer di `.bash_history`.
- **Berkas `.env` di Luar Web Root**: Simpan `.env` di luar direktori yang bisa diakses web dan set permission `600`.
- **Isolasi Container**: Jalankan aplikasi web dalam container dengan `--cap-drop=ALL` dan `--security-opt=no-new-privileges` untuk menghambat stabilisasi TTY dan eskalasi.

---

## Checklist Post-Exploitation Reverse Shell

- [ ] Stabilkan shell interaktif (`script /dev/null -c /bin/bash` atau `python3 pty.spawn`).
- [ ] Verifikasi user & hostname (`id`, `whoami`, `hostname`).
- [ ] Cek hak sudo (`sudo -l`).
- [ ] Cari flag di lokasi umum (`/var/www/html`, `/home/`, `/root/`).
- [ ] Cari berkas `.env` & kredensial database di web root.
- [ ] Dump `/etc/shadow` setelah root (untuk crack offline).
- [ ] Ambil SSH private key (`/root/.ssh/id_rsa`, `/home/*/.ssh/id_rsa`).
- [ ] Periksa history (`~/.bash_history`, `~/.mysql_history`).
- [ ] Lanjut enumerasi privilege escalation (lihat `03-enumeration/` dan `04-privilege-escalation/`).
- [ ] Dokumentasikan temuan & kredensial untuk laporan (`06-report/`).

---

## Referensi

- [PentestMonkey — Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [revshells.com — Generator Reverse Shell](https://revshells.com)
- [GTFOBins — Binary SUID Exploitation](https://gtfobins.github.io/)
- [PayloadsAllTheThings — Post Exploitation](https://swisskyrepo.github.io/PayloadsAllTheThings/)

