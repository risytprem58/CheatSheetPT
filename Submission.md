# ✅ Submission / Proof of Compromise (WAJIB untuk Submission)

> **Tujuan:** Mengumpulkan **bukti sah** bahwa target berhasil dikompromikan hingga **root**. Bukti biasanya diminta dalam bentuk **screenshot** atau **teks output** saat submission.

---

## 📌 Penjelasan Singkat

Submission **Proof** adalah tahap akhir dari pentest. File ini memuat **perintah verifikasi** yang dijalankan setelah privilege escalation berhasil, untuk membuktikan bahwa kita memiliki akses **root** pada target.

---

## 🛡️ Bukti Standar (Wajib)

```bash
id          # Tunjukkan uid=0(root) euid=0(root)
hostname    # Tunjukkan nama host/box target
```

Contoh output yang valid:

```text
$ id
uid=0(root) gid=0(root) groups=0(root)

$ hostname
htb-staff-001
```

---

## 🧪 Bukti Tambahan (Opsional tapi Direkomendasikan)

```bash
# 1. Akses /etc/shadow
head -n 1 /etc/shadow

# 2. Lihat user root entry di /etc/passwd
grep '^root:' /etc/passwd

# 3. Tulis marker file
echo "PWNED $(date)" > /root/pwned.txt
cat /root/pwned.txt

# 4. Lihat /root directory
ls -la /root
```

---

## 📋 Checklist Submission

- [ ] Tunjukkan `id` → `uid=0(root)`.
- [ ] Tunjukkan `hostname` → nama box target.
- [ ] (Opsional) Baca `/etc/shadow`.
- [ ] (Opsional) Tulis marker file di `/root`.
- [ ] Simpan **screenshot** atau **log output** sebagai lampiran submission.
- [ ] Pastikan output jelas, tidak terpotong, dan dapat diverifikasi.

---

## 📚 Referensi

- [OWASP – Pentest Reporting](https://owasp.org/www-project-pentest-reporting/)
- [HackTheBox – Submission Guidelines](https://www.hackthebox.com/)
