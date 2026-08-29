# 🎧 Listener — Menerima Koneksi Reverse Shell

> **Tujuan:** Menyiapkan listener di mesin attacker agar koneksi reverse shell dari target bisa diterima dan di-interact.

---

## 📌 Penjelasan Singkat

Listener adalah **server kecil** yang dijalankan di Kali untuk menerima koneksi masuk dari target. **Harus dijalankan sebelum payload dieksekusi** di target.

---

## ⚙️ Cara Menjalankan

### 1️⃣ Netcat Standar

```bash
nc -lvnp 4444
```

| Flag | Fungsi |
|------|--------|
| `-l` | Listen mode |
| `-v` | Verbose (tampilkan info koneksi) |
| `-n` | Tanpa DNS resolution (lebih cepat) |
| `-p 4444` | Port yang di-listen |

### 2️⃣ Netcat + rlwrap (Recommended)

```bash
rlwrap nc -lvnp 4444
```

> **Penjelasan:** `rlwrap` menambahkan **history** dan **navigasi panah** saat mengetik di shell yang diterima, sehingga lebih nyaman.

---

## 🔄 Alur Reverse Shell

```text
Target (victim)
   │
   │ Reverse Connection (outbound)
   ▼
Kali Linux (attacker)
   │
   └── nc -lvnp 4444
          ↓
      Interactive Shell
```

---

## 💡 Tips Penggunaan

- **Jalankan listener dulu** sebelum trigger payload di target.
- **Gunakan port yang konsisten** di listener dan payload (mis. 4444).
- **Gunakan rlwrap** untuk pengalaman shell interaktif yang lebih baik.
- **Catat IP Kali (LHOST)** dengan `ip a | grep 10.0.` sebelum membuat payload.

---

## ✅ Checklist Listener

- [ ] Tentukan **LHOST** (IP Kali).
- [ ] Tentukan **LPORT** (port listener, mis. 4444).
- [ ] Jalankan `nc -lvnp <port>` (atau dengan rlwrap).
- [ ] Trigger payload di target.
- [ ] Verifikasi shell interaktif (coba `whoami`).

---

## 📚 Referensi

- [PentestMonkey Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [revshells.com — Generator Reverse Shell](https://revshells.com)
