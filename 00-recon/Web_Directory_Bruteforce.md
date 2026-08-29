# 🌐 Web Directory Brute Force — Mencari Direktori Tersembunyi

> **Tujuan:** Menemukan direktori, file, dan endpoint tersembunyi pada web server.
> **Tools utama:** `dirsearch`, `feroxbuster`, `dirb`, `ffuf`

---

## 📌 Penjelasan Singkat

| Tool | Kelebihan | Penggunaan |
|------|-----------|------------|
| **Dirsearch** | Mudah digunakan | Enumeration dasar |
| **Feroxbuster** | Cepat & recursive | Scan deep directories |
| **Dirb** | Bawaan Kali Linux | Quick scan |
| **Ffuf** | Fleksibel & cepat | Fuzzing advanced |

---

## ⚙️ Langkah-Langkah (dari yang mudah ke sulit)

### Langkah 1: Dirsearch (Enumeration Dasar)

```bash
# Scan dengan wordlist default
dirsearch -u http://<TARGET>:PORT/

# Scan dengan custom wordlist
dirsearch -u http://<TARGET>:PORT/ -w /usr/share/wordlists/dirb/common.txt
```

**Contoh output:**

```text
[17:30:45] Starting:
http://10.0.2.10:80/
http://10.0.2.10:80/admin/      [CODE:200|SIZE:5120]
http://10.0.2.10:80/login.php   [CODE:200|SIZE:1024]
http://10.0.2.10:80/uploads/    [CODE:301|REDIRECT]
http://10.0.2.10:80/config.php  [CODE:200|SIZE:2048]
```

> **Catatan:** `/admin/` dan `/uploads/` menarik untuk investigasi.

---

### Langkah 2: Feroxbuster (Recursive Scan)

```bash
feroxbuster -u http://<TARGET>:PORT/ -w /usr/share/wordlists/dirb/common.txt
```

**Contoh output:**

```text
200   512l   1024w  12288c http://10.0.2.10/index.php
200   128l    256w   4096c http://10.0.2.10/admin/
200    64l    128w   2048c http://10.0.2.10/api/users
200    32l     64w   1024c http://10.0.2.10/config.php
```

---

### Langkah 3: Dirb (Quick Scan)

```bash
# Default wordlist
dirb http://<TARGET>:PORT/

# Custom wordlist (TANPA flag -w, taruh setelah URL)
dirb http://<TARGET>:PORT/ /usr/share/wordlists/dirb/common.txt
```

**Scan dengan extension tertentu:**

```bash
dirb http://10.10.10.6:8000/ /usr/share/dirb/wordlists/common.txt -X .php,.html,.aspx,.jsp,.js,.txt,.zip -r
# -X: Extension yang dicari
# -r: Recursive
```

**Scan dengan cookie (untuk halaman login):**

```bash
dirb http://10.10.10.6:8000/admin/ -c "PHPSESSID=a1b2c3d4e5f6..."
# -c: Kirim HTTP Cookie
```

---

### Langkah 4: Ffuf (Fuzzing Fleksibel)

```bash
ffuf -u http://<TARGET>:PORT/FUZZ -w /usr/share/wordlists/dirb/common.txt
# FUZZ: Posisi diganti isi wordlist
```

**Filter berdasarkan status code:**

```bash
ffuf -u http://<TARGET>:PORT/FUZZ \
    -w /usr/share/wordlists/dirb/common.txt \
    -mc 200,300-399
# -mc 200: hanya tampilkan 200
# -mc 300-399: hanya tampilkan redirect
```

**Contoh output:**

```text
admin       [Status: 200, Size: 5120]
login.php   [Status: 200, Size: 1024]
uploads     [Status: 301, Size: 0]
```

---

## 📋 Status Code yang Perlu Diperhatikan

| Code | Arti | Aksi |
|------|------|------|
| **200** | OK - Resource ditemukan | ✅ Investigasi |
| **301/302** | Redirect | ⚠️ Cek target redirect |
| **401** | Unauthorized | 🔐 Butuh autentikasi |
| **403** | Forbidden | ⚠️ Ada tapi ditolak |
| **404** | Not Found | ❌ Skip |
| **500** | Internal Server Error | ⚠️ Potensi bug |

---

## 📚 Wordlist Default Kali Linux

### Dirb

```bash
/usr/share/wordlists/dirb/common.txt
# Paling sering dipakai

/usr/share/wordlists/dirb/big.txt
# Lebih lengkap
```

### Dirbuster

```bash
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
# Sedang

/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
# Sangat lengkap
```

**Jika file masih .gz, ekstrak dulu:**

```bash
sudo gzip -d /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt.gz
```

---

## ⚠️ Catatan Penting

- Response `403` tetap perlu diperhatikan — bisa berarti resource ada tapi ditolak.
- Response `3xx` bisa mengungkap endpoint lain.
- Gunakan wordlist kecil untuk awal, besar untuk deep scan.

---

## ✅ Checklist Setelah Directory Bruteforce

- [ ] Catat semua endpoint yang ditemukan
- [ ] Identifikasi page menarik (admin, upload, config)
- [ ] Cek file sensitif (.git, .env, backup)
- [ ] Coba akses manual setiap endpoint
- [ ] Catat untuk tahap exploitation

---

## 📚 Referensi

- [Dirsearch GitHub](https://github.com/maurosoria/dirsearch)
- [Feroxbuster](https://github.com/epi052/feroxbuster)
- [SecLists Wordlists](https://github.com/danielmiessler/SecLists)
