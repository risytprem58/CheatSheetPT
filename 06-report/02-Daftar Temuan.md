# Daftar Temuan Keamanan

> **Tujuan:** Daftar rekapitulasi seluruh temuan celah keamanan yang teridentifikasi selama pengujian, diurutkan berdasarkan tingkat keparahan (CVSS Score) tertinggi.

---

| No. | Kode Temuan | Nama Temuan / Kerentanan | Severity | Skor CVSS | Target / Endpoint |
|---|---|---|---|---|---|
| 1 | `FIND-01` | Arbitrary File Upload to Remote Code Execution (RCE) | Critical | 9.3 | `POST /submit.php` (`http://192.168.56.105:8083`) |
| 2 | `FIND-02` | Command Injection (OS Command Injection) | Critical | 9.8 | `GET /api/tools/ping?target=...` |
| 3 | `FIND-03` | SQL Injection (SQLi) | Critical | 9.3 | `POST /api/auth/login`, `GET /api/jobs/search` |
| 4 | `FIND-04` | Insecure Direct Object Reference (IDOR) | High | 8.6 | `/api/users/{id}/profile`, `/api/users/{id}` |
| 5 | `FIND-05` | Local File Inclusion (LFI) | High | 7.5 | `GET /api/download?file=...` |
| 6 | `FIND-06` | Sensitive Data Exposure / Information Disclosure | High | 7.5 | `GET /.env`, `GET /robots.txt`, `GET /.git/` |
| 7 | `FIND-07` | Unnecessary Open Ports & Exposed Services | Medium | 6.9 | `Port 22/tcp (SSH)`, `Port 3306/tcp (MySQL)` |
| 8 | `FIND-08` | Stored Cross-Site Scripting (Stored XSS) | Medium | 6.1 | `POST /api/profile/update` |
| 9 | `FIND-09` | Directory Listing | Low | 3.7 | `GET /uploads/`, `GET /backup/` |


