# 🪟 Stabilkan Shell (TTY) — Membuat Reverse Shell Interaktif

> **Tujuan:** Mengubah shell non‑interaktif menjadi **shell interaktif** (dengan autocomplete, history, dll.) setelah berhasil mendapatkan reverse shell.

---

## 📌 Penjelasan Singkat

Setelah reverse shell terhubung, biasanya berupa **non‑interactive shell** (hanya mengeksekusi satu perintah). Dengan **pseudo‑terminal (PTY)**, kita dapat mendapatkan **shell interaktif** seperti sesi SSH.

---

## ⚙️ Metode TTY (dari yang paling umum ke yang paling sederhana)

### 1️⃣ Python (Jika tersedia)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

> **Penjelasan:** `pty.spawn` membuat PTY baru, memberi Anda prompt bash lengkap.

---

### 2️⃣ Python via `os.system`

```bash
echo os.system('/bin/bash')
```

> **Catatan:** Kadang‑kadang diperlukan trik injection untuk mengeksekusi perintah `os.system`.

---

### 3️⃣ Interactive Shell (Bash)

```bash
/bin/sh -i
```

> **Penjelasan:** `-i` memaksa shell menjadi interactive.

---

### 4️⃣ `script` Command (Jika tersedia)

```bash
script -qc /bin/bash /dev/null
```

> **Catatan:** `script` menciptakan pseudo‑terminal dan mengeksekusi perintah dalamnya.

---

### 5️⃣ Perl (Jika tersedia)

```bash
perl -e 'exec "/bin/sh";'
```

---

## 📋 Checklist TTY

- [ ] Cek program yang tersedia pada target (`which python3`, `which script`, `which perl`, `which bash`).
- [ ] Coba `python3 -c 'import pty;pty.spawn("/bin/bash")'`.
- [ ] Jika gagal, coba `script -qc /bin/bash /dev/null`.
- [ ] Jika masih gagal, gunakan `perl -e 'exec "/bin/sh";'` atau `/bin/sh -i`.
- [ ] Verifikasi interaktifitas (`Ctrl‑C`, `Tab` autocomplete, `history`).

---

## ✅ Contoh Penggunaan

```bash
# 1. Jalankan listener di attacker
nc -lvnp 4444

# 2. Trigger reverse shell di target (contoh payload PHP)
# 3. Saat koneksi masuk, coba stabilkan TTY
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

---

## 📚 Referensi

- [PentestMonkey – Stabilize Reverse Shell](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [OWASP – TTY Injection](https://owasp.org/www-community/attacks/TTY_Injection)

