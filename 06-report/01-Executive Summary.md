# Executive Summary

> **Tujuan:** Ringkasan eksekutif hasil pengujian keamanan (penetration testing).

---

## 1. Ringkasan Pengujian

Pengujian keamanan (penetration testing) ini dilakukan terhadap aplikasi web TeknoBantu (`http://192.168.56.105:8083`) dan KlaimKu (`http://192.168.56.106:8084`) dengan tujuan untuk mengidentifikasi celah keamanan yang dapat dimanfaatkan oleh pihak tidak berwenang. Pengujian dilaksanakan berdasarkan ruang lingkup dan metodologi yang telah disepakati pada Rules of Engagement (RoE), mencakup pendekatan **black-box testing**. Fokus utama pengujian meliputi evaluasi kontrol akses, verifikasi otorisasi di sisi server, serta ketahanan aplikasi terhadap manipulasi input pada berbagai endpoint kritis. Selama periode pengujian **11 September 2025**, berhasil diidentifikasi rincian tingkat keparahan (severity) sebagai berikut:

| Tingkat Keparahan | Jumlah Temuan |
|-------------------|---------------|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |
| Informational | 0 |

---

## 2. Rekomendasi Utama

Celah keamanan yang ditemukan pada aplikasi target bersumber dari tiga aspek utama: kelemahan manajemen kredensial dan eksposur direktori/port, belum optimalnya validasi dan penanganan input di sisi server, serta ketiadaan verifikasi otorisasi pengguna. Berikut adalah rekomendasi langkah mitigasi dan perbaikan berdasarkan masing-masing kategori kerentanan:

- **Credential Management & Sensitive File Exposure:** Mengganti atau menghapus default credential, menghapus informasi kredensial yang tersimpan di tempat umum, serta menutup akses langsung ke file konfigurasi atau repositori sensitif (seperti `.env`, `.git`, atau backup file).
- **Directory Listing:** Nonaktifkan fitur pengindeksan direktori (*Directory Indexing / Directory Browsing*) pada konfigurasi web server (misalnya setel `Options -Indexes` pada Apache atau hapus directive `autoindex on` pada Nginx) agar daftar file dan folder internal tidak dapat dilihat secara terbuka oleh publik.
- **Unnecessary Open Ports & Exposed Services:** Batasi akses publik ke port sensitif dan layanan administratif (seperti SSH port 22 atau Database MySQL port 3306) menggunakan aturan firewall (*IP Whitelisting*), hubungkan layanan database hanya ke interface lokal (`bind-address = 127.0.0.1`), serta nonaktifkan akses remote langsung untuk akun administrator/root.
- **SQL Injection (SQLi):** Terapkan **Prepared Statements / Parameterized Queries** pada seluruh fungsi kueri database untuk menutup kerentanan SQL Injection, serta jalankan akun database dengan prinsip *least privilege*.
- **Unrestricted File Upload:** Batasi jenis ekstensi berkas (*whitelisting*), validasi tipe MIME/header di sisi server, ubah nama file (*randomize filename*), dan simpan berkas unggahan di **direktori non-executable** atau penyimpanan terpisah.
- **Command Injection:** Hindari pemanggilan perintah shell sistem operasi secara langsung. Gunakan API/library bawaan bahasa pemrograman, atau terapkan *allowlist* input yang sangat ketat jika eksekusi sistem tidak terhindarkan.
- **Local File Inclusion (LFI):** Hindari meneruskan input pengguna secara langsung ke fungsi inklusi atau pembacaan file (`include`, `require`, `file_get_contents`). Gunakan pemetaan statis (*allowlist mapping*) untuk memilih file yang diizinkan.
- **IDOR (Insecure Direct Object Reference):** Terapkan mekanisme kontrol akses dan verifikasi otorisasi (*access control check*) di sisi server pada setiap pemanggilan ID atau parameter objek untuk memastikan pengguna hanya dapat mengakses data milik mereka sendiri.
- **Cross-Site Scripting (XSS):** Terapkan sanitasi input yang masuk serta lakukan **context-aware HTML output encoding** sebelum menampilkan data masukan pengguna ke halaman aplikasi web.






