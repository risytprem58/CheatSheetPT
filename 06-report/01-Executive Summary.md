# 📝 Executive Summary

> **Tujuan:** Ringkasan eksekutif hasil penetration testing — tinggal copas, ganti yang di-**bold** sesuai engagement kamu.

---

## 1. Ringkasan Pengujian

Pengujian keamanan (penetration testing) ini dilakukan terhadap aplikasi web/API **Job Portal** (`https://jobportal.vulnapp.id`) dengan tujuan untuk mengidentifikasi celah keamanan yang dapat dimanfaatkan oleh pihak tidak berwenang. Pengujian dilaksanakan berdasarkan ruang lingkup dan metodologi yang telah disepakati pada Rules of Engagement (RoE), mencakup pendekatan **black-box testing**. Fokus utama pengujian meliputi evaluasi kontrol akses, verifikasi otorisasi di sisi server, serta ketahanan aplikasi terhadap manipulasi input pada berbagai endpoint kritis. Selama periode pengujian (**1 Sep 2026** s/d **7 Sep 2026**), berhasil diidentifikasi sebanyak **4 temuan** kerentanan dengan rincian tingkat keparahan (severity) sebagai berikut:

---

## 2. Ringkasan Temuan (Severity)

| Severity | Jumlah |
|----------|--------|
| 🔴 Critical | 2 |
| 🟠 High | 1 |
| 🟡 Medium | 1 |
| **Total** | **4** |

---

## 3. Daftar Temuan

| No | Kerentanan | Severity | Endpoint / Lokasi | CVSS |
|----|------------|----------|--------------------|------|
| 1 | SQL Injection — Input tidak disanitasi, penyerang dapat dump seluruh database | 🔴 Critical | `/api/auth/login`, `/api/jobs/search?keyword=` | 9.3 |
| 2 | Unrestricted File Upload — Upload webshell `.php` berhasil dieksekusi sebagai script di server (RCE) | 🔴 Critical | `/api/resume/upload` → `/uploads/resume/shell.php?cmd=id` | 9.8 |
| 3 | IDOR (Insecure Direct Object Reference) — Akses & modifikasi data user lain via manipulasi ID | 🟠 High | `/api/users/{id}/profile` | 8.6 |
| 4 | Stored XSS (Cross-Site Scripting) — Payload JavaScript tersimpan dan tereksekusi di browser korban | 🟡 Medium | `/api/profile/update` (field profil) | 5.1 |

---

## 4. Rekomendasi Utama

Sebagian besar temuan disebabkan oleh **minimnya sanitasi input**, **tidak adanya validasi file upload**, serta **tidak tersedianya mekanisme kontrol akses** yang ketat di sisi server.

| Prioritas | Rekomendasi |
|-----------|-------------|
| 🥇 Pertama | Terapkan **Prepared Statements / Parameterized Queries** pada seluruh fungsi kueri database untuk menutup kerentanan SQL Injection. |
| 🥈 Kedua | Batasi jenis ekstensi berkas (**whitelisting**), validasi tipe MIME/header di sisi server, rename file hasil upload, dan simpan berkas unggahan di **direktori non-executable** agar file `.php` tidak bisa dieksekusi sebagai webshell. |
| 🥉 Ketiga | Validasi **kepemilikan objek dan hak akses** pengguna (access control check) di sisi server pada setiap pemanggilan ID/parameter API untuk mencegah IDOR. |
| 4️⃣ Keempat | Lakukan **sanitasi input** yang masuk dan terapkan **HTML output encoding** sebelum menampilkan data pengguna ke aplikasi guna mencegah Stored XSS. |

---

## 5. Informasi Engagement

| Item | Detail |
|------|--------|
| **Nama Proyek** | Pentest Aplikasi Job Portal |
| **Target** | `https://jobportal.vulnapp.id` |
| **Metode** | Black-box |
| **Periode** | 1 Sep 2026 s/d 7 Sep 2026 |
| **Penguji** | Tim Security |
| **Versi Laporan** | v1.0 |

---

## ✅ Checklist Executive Summary

- [ ] Ringkasan pengujian lengkap (target, metode, periode).
- [ ] Tabel severity terisi sesuai jumlah temuan.
- [ ] Daftar temuan lengkap dengan endpoint dan CVSS.
- [ ] Rekomendasi utama sesuai prioritas.
- [ ] Informasi engagement (proyek, penguji, versi laporan).

---

## 📚 Referensi

- [OWASP – Pentest Reporting Guidelines](https://owasp.org/www-project-pentest-reporting/)
- [CVSS Calculator — FIRST](https://www.first.org/cvss/calculator/3.1)
- [PTES – Reporting](http://www.pentest-standard.org/index.php/Reporting)
