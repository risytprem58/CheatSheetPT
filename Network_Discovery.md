# 🔍 Network Discovery — Menemukan Target di Jaringan

> **Tujuan:** Menemukan host aktif di dalam network (terutama di lingkungan NAT/CTF).
> **Tools utama:** `netdiscover`, `nmap`

---

## 📌 Penjelasan Singkat

- **netdiscover** → Mengirim ARP request untuk mendeteksi semua host aktif di LAN.
- **nmap -sn** → Melakukan ping scan (tidak memindai port, lebih cepat).

---

## ⚙️ Langkah-Langkah

### Langkah 1: Deteksi Host Aktif dengan netdiscover

```bash
sudo netdiscover -r 10.0.2.0/24
# -r (Range): Menentukan rentang IP yang akan dipindai
```

**Contoh output:**

```text
   IP            At MAC Address     Count  Len  MAC Vendor
   ----------------------------------------------------------
   10.0.2.4      08:00:27:a5:3c:11      1    60  PCS Systemtechnik
   10.0.2.10     08:00:27:aa:bb:cc      2   120  Oracle Corporation
```

> **Penjelasan:** Setiap baris menunjukkan IP dan MAC address. Cari host mencurigakan (MAC "QEMU", "Oracle", dll.) — biasanya mesin target.

---

### Langkah 2: Deteksi Host Aktif dengan nmap

```bash
nmap -sn 10.0.2.0/24
# -sn: Deteksi host aktif saja (tanpa scan port)
```

**Contoh output:**

```text
Starting Nmap 7.94
Nmap scan report for 10.0.2.4
Host is up (0.00025s latency).
MAC Address: 08:00:27:AA:BB:CC (PCS Systemtechnik)
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.13 seconds
```

> **Catatan:** Hasil `2 hosts up` = 2 host aktif di network.

---

### Langkah 3: Cek IP Kali (Attacker)

```bash
ip a | grep 10.0.
# Cek IP Kali untuk LHOST reverse shell
```

**Contoh output:**

```text
inet 10.0.2.5/24 brd 10.0.2.255 scope global dynamic eth0
```

> **Penjelasan:** IP Kali = `10.0.2.5`. Ini adalah **LHOST** untuk reverse shell.

---

## ⚠️ Catatan Penting

- Gunakan `sudo` untuk netdiscover (butuh akses raw socket).
- Pastikan network interface aktif dan terhubung.
- IP Kali (`LHOST`) harus tetap sama dari awal sampai reverse shell.

---

## ✅ Checklist Setelah Network Discovery

- [ ] IP Target (misal: 10.0.2.10)
- [ ] IP Kali/LHOST (misal: 10.0.2.5)
- [ ] Network range (misal: 10.0.2.0/24)

---

## 📚 Referensi

- [Nmap Official](https://nmap.org/book/man.html)
