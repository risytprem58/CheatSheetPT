# 📝 Executive Summary — Template Laporan Pentest

> **Tujuan:** Template executive summary yang **tinggal copas** dan ganti bagian `<...>` sesuai target.

---

## 📌 Cara Pakai

1. Copy seluruh bagian **Template** di bawah.
2. Ganti semua `<...>` dengan informasi sesuai engagement kamu.
3. Sesuaikan jumlah baris tabel temuan & rekomendasi.
4. Hapus baris contoh yang tidak dipakai.

---

## 📋 Template Executive Summary

---

### 1. Ringkasan Pengujian

```text
Pengujian keamanan (penetration testing) ini dilakukan terhadap
<NAMA_APLIKASI/SISTEM> (<URL/IP_TARGET>) dengan tujuan untuk mengidentifikasi
celah keamanan yang dapat dimanfaatkan oleh pihak tidak berwenang.

Pengujian dilaksanakan berdasarkan ruang lingkup dan metodologi yang telah
disepakati pada Rules of Engagement (RoE), mencakup pendekatan <METODE_TESTING>.

Fokus utama pengujian meliputi <FOKUS_PENGUJIAN>.

Selama periode pengujian (<TANGGAL_MULAI> s/d <TANGGAL_SELESAI>), berhasil
diidentifikasi sebanyak <JUMLAH_TEMUAN> temuan kerentanan dengan rincian
tingkat keparahan (severity) sebagai berikut:
```

---

### 2. Ringkasan Temuan (Severity)

| Severity | Jumlah |
|----------|--------|
| 🔴 Critical | `<JUMLAH>` |
| 🟠 High | `<JUMLAH>` |
| 🟡 Medium | `<JUMLAH>` |
| 🟢 Low | `<JUMLAH>` |
| 🔵 Informational | `<JUMLAH>` |
| **Total** | **`<TOTAL>`** |

---

### 3. Daftar Temuan

| No | Kerentanan | Severity | Endpoint / Lokasi | CVSS |
|----|------------|----------|--------------------|------|
| 1 | `<NAMA_KERENTANAN>` | 🔴 Critical | `<ENDPOINT>` | `<SKOR>` |
| 2 | `<NAMA_KERENTANAN>` | 🟠 High | `<ENDPOINT>` | `<SKOR>` |
| 3 | `<NAMA_KERENTANAN>` | 🟡 Medium | `<ENDPOINT>` | `<SKOR>` |
| 4 | `<NAMA_KERENTANAN>` | 🟢 Low | `<ENDPOINT>` | `<SKOR>` |

---

### 4. Rekomendasi Utama

```text
Sebagian besar temuan disebabkan oleh <AKAR_MASALAH_UMUM>.
```

| Prioritas | Rekomendasi |
|-----------|-------------|
| 🥇 Pertama | `<REKOMENDASI_1>` |
| 🥈 Kedua | `<REKOMENDASI_2>` |
| 🥉 Ketiga | `<REKOMENDASI_3>` |
| 4️⃣ Keempat | `<REKOMENDASI_4>` |

---

### 5. Informasi Engagement

| Item | Detail |
|------|--------|
| **Nama Proyek** | `<NAMA_PROYEK>` |
| **Target** | `<URL/IP_TARGET>` |
| **Metode** | `<black-box / grey-box / white-box>` |
| **Periode** | `<TANGGAL_MULAI>` s/d `<TANGGAL_SELESAI>` |
| **Penguji** | `<NAMA_PENGUJI>` |
| **Versi Laporan** | `<v1.0>` |

---

---

## 🧪 Contoh Sudah Diisi

> Contoh di bawah menggunakan studi kasus target **Job Portal**.

### 1. Ringkasan Pengujian

Pengujian keamanan (penetration testing) ini dilakukan terhadap aplikasi web/API **Job Portal** (`https://jobportal.vulnapp.id`) dengan tujuan untuk mengidentifikasi celah keamanan yang dapat dimanfaatkan oleh pihak tidak berwenang. Pengujian dilaksanakan berdasarkan ruang lingkup dan metodologi yang telah disepakati pada Rules of Engagement (RoE), mencakup pendekatan **black-box testing**. Fokus utama pengujian meliputi evaluasi kontrol akses, verifikasi otorisasi di sisi server, serta ketahanan aplikasi terhadap manipulasi input pada berbagai endpoint kritis. Selama periode pengujian (**1 Sep 2026** s/d **7 Sep 2026**), berhasil diidentifikasi sebanyak **4 temuan** kerentanan dengan rincian tingkat keparahan (severity) sebagai berikut:

### 2. Ringkasan Temuan (Severity)

| Severity | Jumlah |
|----------|--------|
| 🔴 Critical | 1 |
| 🟠 High | 1 |
| 🟡 Medium | 1 |
| 🟢 Low | 1 |
| **Total** | **4** |

### 3. Daftar Temuan

| No | Kerentanan | Severity | Endpoint / Lokasi | CVSS |
|----|------------|----------|--------------------|------|
| 1 | SQL Injection | 🔴 Critical | `/api/search?q=` | 9.8 |
| 2 | IDOR (Insecure Direct Object Reference) | 🟠 High | `/api/users/{id}/profile` | 7.5 |
| 3 | Stored XSS | 🟡 Medium | `/jobs/post` (field deskripsi) | 6.1 |
| 4 | Unrestricted File Upload | 🟢 Low | `/api/resume/upload` | 4.3 |

### 4. Rekomendasi Utama

Sebagian besar temuan disebabkan oleh **minimnya sanitasi input** serta tidak tersedianya mekanisme **validasi otorisasi dan kontrol akses** yang ketat di sisi server.

| Prioritas | Rekomendasi |
|-----------|-------------|
| 🥇 Pertama | Terapkan **Prepared Statements / Parameterized Queries** pada seluruh fungsi kueri database untuk menutup kerentanan SQL Injection. |
| 🥈 Kedua | Validasi **kepemilikan objek dan hak akses** pengguna (access control check) di sisi server pada setiap pemanggilan ID/parameter API untuk mencegah IDOR. |
| 🥉 Ketiga | Lakukan **sanitasi input** yang masuk dan terapkan **HTML output encoding** sebelum menampilkan data pengguna ke aplikasi guna mencegah Stored XSS. |
| 4️⃣ Keempat | Batasi jenis ekstensi berkas (**whitelisting**), validasi tipe MIME/header di sisi server, dan simpan berkas unggahan di **direktori non-executable** untuk mengamankan fitur file upload. |

### 5. Informasi Engagement

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
