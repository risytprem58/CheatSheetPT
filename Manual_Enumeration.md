# Manual Enumeration (Quick Wins)

Enumerasi manual digunakan untuk mencari konfigurasi, permission, service,
dan informasi lain yang berpotensi membantu **Linux Privilege Escalation**.

```bash
id; sudo -l                                  # privilege + sudo rights
uname -a; cat /etc/os-release                # kernel + OS
find / -perm -4000 -type f 2>/dev/null       # SUID
find / -perm -2000 -type f 2>/dev/null       # SGID
getcap -r / 2>/dev/null                      # capabilities
cat /etc/crontab; ls -la /etc/cron.d/        # cron jobs
find / -writable -type f 2>/dev/null | grep -v proc  # writable files
ls -la /home/*; cat ~/.bash_history          # info bocor
ss -tlnp                                     # service internal
```

**Keterangan singkat:**

```text
id / sudo -l       → Cek privilege dan hak sudo
uname / os-release → Cek kernel dan versi OS
SUID / SGID        → Cari binary dengan permission khusus
getcap             → Cari Linux capabilities
cron               → Cek scheduled jobs
writable files     → Cari file yang dapat dimodifikasi
home/history       → Cari informasi atau credential yang bocor
ss                 → Cek service/port yang sedang listening
```

## Basic Information

```bash
id
# Menampilkan user ID, group, dan privilege yang dimiliki

sudo -l
# Menampilkan command yang dapat dijalankan menggunakan sudo
```

## Kernel & OS

```bash
uname -a
# Menampilkan informasi kernel dan arsitektur sistem

cat /etc/os-release
# Menampilkan informasi distribusi dan versi Linux
```

> Gunakan informasi kernel/OS untuk menentukan apakah terdapat vulnerability
> atau exploit yang relevan dengan sistem.

## SUID

```bash
find / -perm -4000 -type f 2>/dev/null
# Mencari file dengan SUID bit
# Binary yang ditemukan dapat diperiksa di GTFOBins
```

## SGID

```bash
find / -perm -2000 -type f 2>/dev/null
# Mencari file dengan SGID bit
# Periksa binary yang tidak biasa atau memiliki permission berisiko
```

## Linux Capabilities

```bash
getcap -r / 2>/dev/null
# Mencari file yang memiliki Linux capabilities
# Capability tertentu dapat memberikan privilege tambahan
```

## Cron Jobs

```bash
cat /etc/crontab
# Melihat scheduled task pada sistem

ls -la /etc/cron.d/
# Melihat konfigurasi cron tambahan
```

**Fokus pemeriksaan:**

```text
Cron Job
   ↓
Script / Binary
   ↓
Permission
   ↓
Apakah dapat dimodifikasi oleh user?
   ↓
Potensi Privilege Escalation
```

## Writable Files

```bash
find / -writable -type f 2>/dev/null | grep -v proc
# Mencari file yang dapat ditulis oleh user
# Mengecualikan filesystem /proc dari hasil
```

## Home Directory & History

```bash
ls -la /home/*
# Melihat direktori dan file milik user lain

cat ~/.bash_history
# Melihat command history user saat ini
# Dapat menemukan informasi konfigurasi atau credential yang tertinggal
```

## Internal Services

```bash
ss -tlnp
# Menampilkan TCP port yang sedang listening
# Membantu menemukan service yang hanya tersedia secara lokal
```

## Alur Enumeration

```text
System Information
       ↓
Privilege & Sudo
       ↓
SUID / SGID / Capabilities
       ↓
Cron & Writable Files
       ↓
Credential / History
       ↓
Internal Services
       ↓
Validasi Temuan
       ↓
Potential Privilege Escalation
```

> **Catatan:** hasil enumeration bukan otomatis vulnerability. Setiap temuan
> perlu dianalisis berdasarkan permission, konfigurasi, versi software, dan
> konteks sistem.
