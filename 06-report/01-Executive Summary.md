# Executive Summary

> **Tujuan:** Ringkasan eksekutif hasil pengujian keamanan (penetration testing).

---

## 1. Ringkasan Pengujian

Pengujian keamanan (penetration testing) ini dilakukan terhadap aplikasi web TeknoBantu (`http://192.168.56.105:8083`) dan KlaimKu (`http://192.168.56.106:8084`) dengan tujuan untuk mengidentifikasi celah keamanan yang dapat dimanfaatkan oleh pihak tidak berwenang. Pengujian dilaksanakan berdasarkan ruang lingkup dan metodologi yang telah disepakati pada Rules of Engagement (RoE), mencakup pendekatan **black-box testing**. Fokus utama pengujian meliputi evaluasi kontrol akses, verifikasi otorisasi di sisi server, serta ketahanan aplikasi terhadap manipulasi input pada berbagai endpoint kritis. Selama periode pengujian **11 September 2025**, berhasil diidentifikasi rincian tingkat keparahan (severity) sebagai berikut:

| Tingkat Keparahan | Jumlah Temuan |
|-------------------|---------------|
| Critical | 3 |
| High | 3 |
| Medium | 2 |
| Low | 1 |
| Informational | 0 |

---

## 2. Rekomendasi Utama

Celah keamanan yang ditemukan pada aplikasi target bersumber dari tiga aspek utama: kelemahan manajemen kredensial dan eksposur direktori/port, belum optimalnya validasi dan penanganan input di sisi server, serta ketiadaan verifikasi otorisasi pengguna. Berikut adalah rekomendasi langkah mitigasi dan perbaikan berdasarkan skala prioritas keamanan:

- **Credential Management & Sensitive File Exposure:** Mengganti atau menghapus informasi sensitive, serta menutup akses langsung ke file konfigurasi atau repositori sensitif
- **Directory Listing:** Nonaktifkan fitur pengindeksan direktori (*Directory Indexing / Directory Browsing*) pada konfigurasi web server  agar daftar file dan folder internal tidak dapat dilihat secara terbuka oleh publik.
- **Unnecessary Open Ports & Exposed Services:** Batasi akses publik ke port sensitif dan layanan administratif menggunakan firewall (*IP Whitelisting*), batasi akses ssh /database hanya ke interface lokal (`bind-address = 127.0.0.1`), serta nonaktifkan akses remote langsung untuk akun administrator/root.
- **SQL Injection (SQLi):** Terapkan **Prepared Statements / Parameterized Queries** pada seluruh fungsi kueri database untuk menutup kerentanan SQL Injection, serta jalankan akun database dengan prinsip *least privilege*.
- **Unrestricted File Upload:** Batasi jenis ekstensi berkas (*whitelisting*), validasi tipe MIME/header di sisi server, ubah nama file (*randomize filename*), dan simpan berkas unggahan di **direktori non-executable** atau penyimpanan terpisah.
- **Local File Inclusion (LFI):** **Command Injection:** Hindari meneruskan input pengguna secara langsung ke fungsi eksekusi sistem (shell) atau inklusi file; gunakan allowlist statis serta fungsi/library bawaan aplikasi yang aman.
- **IDOR (Insecure Direct Object Reference):** Terapkan mekanisme kontrol akses dan verifikasi otorisasi (*access control check*) di sisi server pada setiap pemanggilan ID atau parameter objek untuk memastikan pengguna hanya dapat mengakses data milik mereka sendiri.
- **Cross-Site Scripting (XSS):** Terapkan sanitasi input yang masuk serta lakukan **context-aware HTML output encoding** sebelum menampilkan data masukan pengguna ke halaman aplikasi web.






