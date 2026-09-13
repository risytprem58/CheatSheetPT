# Kompetensi — Enam Unit Kompetensi yang Dipraktikkan Peserta

> **Tujuan:** Panduan urutan praktik uji penetrasi — peserta menyelesaikan enam unit kompetensi **secara berurutan**, mulai dari penentuan ruang lingkup hingga penyusunan laporan akhir.

---

## Enam Unit Kompetensi

| No | Unit Kompetensi | Praktiknya |
|----|-----------------|------------|
| 1 | **Menentukan Ruang Lingkup** | RoE (Rules of Engagement) membatasi pengujian hanya pada IP target yang telah disepakati. |
| 2 | **Menentukan Metode Penilaian** | Asesi wajib memberi skor CVSS v4.0 untuk setiap temuan dalam laporan. |
| 3 | **Mengumpulkan Informasi** | Port scan, enumerasi web, dan enumerasi pasca-foothold dilakukan secara berulang. |
| 4 | **Mencari Kerentanan** | XSS, LFI, upload bypass, password reuse — disertai pembedaan false positive. |
| 5 | **Menguji Kerentanan** | Eksploitasi aktual dilakukan hingga memperoleh flag di direktori user. |
| 6 | **Menyusun Laporan** | Deliverable akhir berupa laporan hasil pengujian dalam format PDF (eksekutif summary → summary temuan → POC/detail teknis → kesimpulan dan rekomendasi). |

---

## Peta ke Dokumen Referensi di Repo

| Unit Kompetensi | Referensi Terkait |
|-----------------|-------------------|
| 1. Menentukan Ruang Lingkup | `06-report/00-Informasi Engagement.md`, `06-report/04-Lampiran.md` |
| 2. Menentukan Metode Penilaian | `06-report/02-Daftar Temuan.md` |
| 3. Mengumpulkan Informasi | `00-recon/` (scan), `03-enumeration/` (pasca-foothold), `07-suplement/post-exploitation.md` |
| 4. Mencari Kerentanan | `01-initial-foothold/` (XSS, LFI, File Upload), `07-suplement/ssh.md` (password reuse) |
| 5. Menguji Kerentanan | `02-reverse-shell/`, `05-proof/Submission.md` |
| 6. Menyusun Laporan | `06-report/` (semua template laporan) |

---

## Checklist Unit Kompetensi

- [ ] RoE disepakati — pengujian hanya pada IP target yang ditentukan.
- [ ] Setiap temuan diberi skor CVSS v4.0.
- [ ] Port scan & enumerasi (web, sistem) dilakukan berulang setiap perubahan kondisi.
- [ ] Temuan divalidasi — false positive dibedakan dari kerentanan nyata.
- [ ] Eksploitasi berhasil — flag di direktori user diperoleh & didokumentasikan.
- [ ] Laporan PDF tersusun: eksekutif summary → summary temuan → POC/detail teknis → kesimpulan & rekomendasi.
