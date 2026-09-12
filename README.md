# Pentesting CheatSheet — Lab Reference

> **Referensi lengkap untuk penetration testing:**  
> **Recon** → **Foothold** → **Reverse Shell** → **Enumeration** → **Privilege Escalation** → **Proof** → **Report**

---

## Struktur Direktori

```
.
├──  00-recon/                  # Reconnaissance: pengenalan target
│   ├──  Network_Discovery.md     # Temukan host aktif di jaringan
│   ├──  Port_Scanning.md         # Identifikasi layanan terbuka
│   ├──  Wapiti Vulnerability Scanner.md  # Audit otomatis kerentanan web
│   └──  Web_Directory_Bruteforce.md     # Cari direktori tersembunyi
├──  01-initial-foothold/       # Mendapatkan akses awal
│   ├──  Command_Injection.md     # Eksekusi command arbitrer
│   ├──  easy-simple-php-webshell.php   # File webshell PHP siap pakai
│   ├──  File_Upload.md           # Unggah webshell
│   ├──  LFI.md                   # Include file lokal
│   ├──  SQL_Injection.md         # Eksploitasi kerentanan SQL
│   ├──  SSTI.md                  # Injeksi template sisi server
│   └──  XSS.md                   # Koleksi payload Cross-Site Scripting (XSS)
├──  02-reverse-shell/          # Membangun koneksi balik ke attacker
│   ├──  Command_Injection.md     # Reverse shell via command injection
│   ├──  Listener.md              # Siapkan penerima di attacker
│   ├──  Msfvenom.md              # Generate payload dengan Metasploit
│   ├──  one-liner.md             # Buat file PHP one-liner di target
│   ├──  php-reverse-shell.php   # Script reverse shell PHP (pentestmonkey)
│   └──  Stabilize_TTY.md         # Jadikan shell interaktif
├──  03-enumeration/            # Gathering informasi sistem
│   ├──  LinPEAS.md               # Otomatisasi enumeration Linux
│   ├──  Manual_Enumeration.md    # Teknik enumeration manual
│   └──  GTFOBins.md              # Eksploitasi binary SUID/sudo
├──  04-privilege-escalation/   # Naikkan privilege ke root
│   ├──  00-Kernel_LPE.md          # Ringkasan kernel exploits
│   ├──  01-LPE_Oneliner.md        # Oneliner enumerasi SUID/sudo/capabilities
│   ├──  02-Sudo.md                # Konfigurasi sudo yang tidak aman
│   ├──  03-suid.md                # Manipulasi binary SUID
│   ├──  04-Capabilities.md        # Eksploitasi Linux capabilities
│   ├──  05-Writable_Cron.md       # Cron job yang dapat dimodifikasi
│   ├──  06-DirtyFrag.md           # Kernel LPE — CVE-2026-43284/43500
│   ├──  07-CopyFail.md            # Kernel LPE — CVE-2026-31431
│   └──  08-Weak_Permission.md     # File dengan permission longgar
├──  05-proof/                  # Bukti bahwa akses root sudah diperoleh
│   └──  Submission.md           # Tampilkan id dan hostname
├──  06-report/                 # Laporan pentest & detail temuan
    ├──  00-Informasi Engagement.md    # Informasi umum pelaksanaan engagement
    ├──  01-Executive Summary.md        # Ringkasan eksekutif laporan pentest
    ├──  02-Daftar Temuan.md           # Rekap temuan diurutkan per keparahan
    ├──  03-Detail Temuan.md           # Kerangka detail per temuan
    ├──  03.1-SQL Injection.md         # Detail temuan SQL Injection
    ├──  03.2-IDOR.md                  # Detail temuan IDOR
    ├──  03.3-Unrestricted File Upload.md  # Detail temuan File Upload RCE
    ├──  03.4-Stored XSS.md            # Detail temuan Stored XSS
    ├──  03.5-Local File Inclusion.md  # Detail temuan Local File Inclusion (LFI)
    ├──  03.6-Command Injection.md      # Detail temuan Command Injection
    ├──  03.7-Sensitive Data Exposure Information Disclosure.md  # Detail temuan Sensitive Data Exposure
    ├──  03.8-Directory Listing.md     # Detail temuan Directory Listing
    ├──  03.9-Unnecessary Open Ports and Exposed Services.md  # Detail temuan Open Ports & Exposed Services
    └──  04-Lampiran.md                # Ruang lingkup, tools, & batasan pengujian
└──  07-suplement/              # Suplemen tools & services pendukung
    ├──  find.md                  # Pencarian berkas & eksploitasi find SUID
    ├──  ftp.md                   # Enumerasi & anonymous login FTP
    ├──  mysql.md                 # Enumerasi & eksploitasi MySQL
    └──  ssh.md                   # Autentikasi, transfer berkas, tunneling SSH
```

---

## Daftar Isi Lengkap

| No | Folder | Topik | Deskripsi Singkat |
|----|--------|-------|-------------------|
| 1 | `00-recon/` | **Pengawalan** | Network discovery, port scanning, web directory bruteforce, Wapiti |
| 2 | `01-initial-foothold/` | **Mendapatkan Akses Awal** | SQL injection, file upload, SSTI, LFI, command injection, XSS |
| 3 | `02-reverse-shell/` | **Reverse Shell** | Listener, payload generation, stabilization, command injection |
| 4 | `03-enumeration/` | **Enumerasi Sistem** | LinPEAS, manual enumeration, GTFOBins |
| 5 | `04-privilege-escalation/` | **Privilege Escalation** | Kernel exploits (DirtyFrag, CopyFail), SUID, sudo, capabilities, weak permissions, cron, oneliner enumerasi |
| 6 | `05-proof/` | **Bukti Akses** | Verifikasi `uid=0` dan hostname |
| 7 | `06-report/` | **Laporan Pentest** | Informasi engagement, executive summary, daftar temuan, detail per temuan (SQL Injection, IDOR, dll), lampiran |
| 8 | `07-suplement/` | **Suplemen Tools & Services** | Cheat sheet per-tool/per-service: find, ftp, mysql, ssh |

---

## Tips Penggunaan CheatSheet

1. **Ikuti alur**: Mulai dari recon hingga proof secara berurutan.
2. **Sesuaikan dengan target**: Tidak semua teknik perlu digunakan.
3. **Validasi hasil**: Selalu verifikasi sebelum lanjut ke tahap berikutnya.
4. **Privilege escalation**: Gunakan **LES (Linux Exploit Suggester)** untuk mencocokkan kernel dengan CVE yang tepat (DirtyFrag, CopyFail, DirtyPipe, dll).
5. **Bukti (proof)**: Selalu dokumentasikan `id` → `uid=0(root)` dan `hostname` sebagai bukti.
6. **Catatan penting**: Teknik ini hanya untuk digunakan pada sistem yang Anda miliki atau dengan izin eksplisit.

---

## Referensi & Sumber

- [w4h4z / Pentest-Cheat-Sheet](https://github.com/w4h4z/Pentest-Cheat-Sheet/tree/main/)

---

## Legal Disclaimer

Seluruh teknik dalam cheat sheet ini hanya untuk **keperluan edukasi** dan **uji penetrasi resmi** pada sistem yang Anda miliki atau sistem dengan **izin tertulis**. Penyalahgunaan terhadap sistem yang tidak berizin adalah **tindakan ilegal** dan melanggar hukum yang berlaku.
