# 📡 Port Scanning — Memindai Port dan Service

> **Tujuan:** Mengidentifikasi port yang terbuka dan service yang berjalan pada target.
> **Tools utama:** `nmap`

---

## 📌 Penjelasan Singkat

- **-sV** → Cek versi service di port.
- **-sC** → Jalankan default script scan.
- **-p-** → Scan semua 65535 port (default hanya 1000).
- **--min-rate** → Minimal paket per detik (untuk speed up).

---

## ⚙️ Langkah-Langkah (dari yang mudah ke sulit)

### Langkah 1: Quick Scan (1000 Port Teratas)

```bash
nmap -sV -sC <TARGET>
# Tanpa -p-, Nmap hanya memindai 1000 port yang paling sering digunakan
```

**Contoh output:**

```text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh      OpenSSH 8.2p1
80/tcp   open  http     Apache httpd 2.4.41
3306/tcp open  mysql    MySQL 8.0.25
Service Info: OS: Linux
```

> **Penjelasan:** Ditemukan 3 port terbuka: SSH, HTTP, MySQL.

---

### Langkah 2: Full Scan (Semua 65535 Port)

```bash
nmap -sV -sC -p- <TARGET>
# -p-: Scan SEMUA port
```

**Contoh output:**

```text
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
443/tcp   open     https
3306/tcp  open     mysql
8080/tcp  open     http-proxy
Nmap done: 65535 ports scanned in 45.23 seconds
```

> **Catatan:** `filtered` = port diblokir firewall, bukan tertutup.

---

### Langkah 3: Fast Scan (CTF/Lab)

```bash
nmap -sV -p- --min-rate 5000 <TARGET>
# --min-rate 5000: Kirim 5000 paket/detik
```

> **Gunakan** saat yakin target reachable dan tidak ada IDS/IPS.

---

## 🔍 Port Penting yang Perlu Diperhatikan

| Port | Service | Aksi |
|------|---------|------|
| 21 | FTP | Cek login anonymous |
| 22 | SSH | Brute force / cari credentials |
| 80/443 | HTTP/HTTPS | Web enum, cari vulnerability |
| 3306 | MySQL | Cek default credentials |
| 3389 | RDP | Brute force |
| 8080 | HTTP Proxy | Web enum |

---

## ⚠️ Catatan Penting

- Scan full port lebih lambat tapi lebih komprehensif.
- `--min-rate` hanya jika tidak ada IDS/IPS.
- Catat semua port/service untuk enumeration selanjutnya.

---

## ✅ Checklist Setelah Port Scanning

- [ ] Port & service terbuka
- [ ] Version service
- [ ] OS detection
- [ ] Catat untuk enumeration

---

## 📚 Referensi

- [Nmap Documentation](https://nmap.org/book/man.html)
