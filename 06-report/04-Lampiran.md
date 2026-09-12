# Lampiran

> **Tujuan:** Informasi pendukung pengujian meliputi ruang lingkup, tools, batasan, serta aktivitas reconnaissance dan enumeration.

---

## A. Ruang Lingkup & Metodologi Pengujian

| Item | Detail |
|------|--------|
| **Ruang Lingkup (In-Scope)** | Pengujian mencakup seluruh endpoint, parameter, dan fitur aplikasi target yang disepakati. |
| **Metodologi Pengujian** | Pengujian menggunakan pendekatan Black Box Testing dari sudut pandang penyerang eksternal tanpa pengetahuan awal sistem. |

---

## B. Tools yang Digunakan

| Nama Tools | Fungsi |
|------------|--------|
| **Kali Linux** | Sistem operasi utama pengujian keamanan. |
| **Burp Suite** | Intersepsi lalu lintas HTTP/HTTPS dan analisis manipulasi request. |
| **Nmap** | Pemindaian port dan identifikasi layanan target. |
| **Feroxbuster** | Pembongkaran direktori dan berkas tersembunyi web. |
| **Sqlmap** | Pengujian dan validasi celah SQL Injection secara otomatis. |

---

## C. Batasan Pengujian

| Item | Detail |
|------|--------|
| **Cakupan Pengujian** | Terbatas pada rentang waktu dan lingkungan target yang ditentukan. |

---

## D. Reconnaissance dan Enumeration

- **Pemindaian Port** Identifikasi port dan layanan yang terbuka pada server target.
- **Struktur Web** Pemetaan direktori, berkas, dan endpoint aplikasi target.
