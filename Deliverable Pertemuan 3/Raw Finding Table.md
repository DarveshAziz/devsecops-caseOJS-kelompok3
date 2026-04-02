# Tabel Temuan Raw Scanning - DAST (ZAP Scan)

## Temuan #1

| Field                      | Nilai                                                    |
| -------------------------- | -------------------------------------------------------- |
| **Nama Kerentanan**        | Content Security Policy (CSP) Header Not Set             |
| **Tool Penemu**            | DAST                                                     |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                         |
| **URL / File**             | http://10.34.100.180 (multiple endpoints)                |
| **Parameter / Baris Kode** | HTTP Response Headers                                    |
| **Method**                 | GET                                                      |
| **Payload**                | N/A                                                      |
| **Response / Bukti**       | Not Set / Missing CSP Header dalam multiple GET requests |
| **OWASP Category**         | A05:2021 - Security Misconfiguration                     |
| **Severity (Raw)**         | Medium                                                   |

### Screenshot / Bukti

Cache: 573 instances detected across application endpoints (/cache/, /lib/, /plugins/, etc.)

### Catatan

CSP adalah mekanisme keamanan yang mencegah XSS, clickjacking, dan injeksi resource. Tanpa CSP header, browser tidak memiliki panduan tentang sumber resource yang dapat dimuat. Aplikasi OJS tidak mengirimkan CSP header pada setiap endpoint, meninggalkan aplikasi rentan terhadap script injection dan related attacks.

---

## Temuan #2

| Field                      | Nilai                                                                    |
| -------------------------- | ------------------------------------------------------------------------ |
| **Nama Kerentanan**        | Directory Browsing                                                       |
| **Tool Penemu**            | DAST                                                                     |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                                         |
| **URL / File**             | http://10.34.100.180/cache/, /lib/, /plugins/, /templates/               |
| **Parameter / Baris Kode** | Directory paths                                                          |
| **Method**                 | GET                                                                      |
| **Payload**                | GET /cache/ GET /cache/\_db/ GET /lib/pkp/ GET /plugins/ GET /templates/ |
| **Response / Bukti**       | Parent Directory visible; HTTP 200 OK dengan directory listing           |
| **OWASP Category**         | A05:2021 - Security Misconfiguration                                     |
| **Severity (Raw)**         | Medium                                                                   |

### Screenshot / Bukti

88 instances pada: /cache/, /cache/\_db/, /cache/HTML/, /cache/t_cache/, /cache/t_compile/, /cache/t_config/, /cache/URI/, /lib/, /lib/pkp/, /plugins/, /templates/

### Catatan

Directory browsing memungkinkan attacker melihat struktur direktori aplikasi, menemukan file sensitif, backup files, dan mengidentifikasi plugin serta versinya. Direktori kritis seperti /cache/ dan /lib/ dapat dijelajahi publik tanpa batasan.

---

## Temuan #3

| Field                      | Nilai                                                     |
| -------------------------- | --------------------------------------------------------- |
| **Nama Kerentanan**        | Missing Anti-clickjacking Header (X-Frame-Options)        |
| **Tool Penemu**            | DAST                                                      |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                          |
| **URL / File**             | http://10.34.100.180 (multiple endpoints)                 |
| **Parameter / Baris Kode** | x-frame-options HTTP Response Header                      |
| **Method**                 | GET, POST                                                 |
| **Payload**                | N/A                                                       |
| **Response / Bukti**       | Header x-frame-options tidak ditemukan dalam respons HTTP |
| **OWASP Category**         | A05:2021 - Security Misconfiguration                      |
| **Severity (Raw)**         | Medium                                                    |

### Screenshot / Bukti

122 instances pada endpoints: /index.php/index/user/register, /index.php/test/login/signIn, /index.php/test/user/register, dan endpoints lainnya

### Catatan

Tanpa X-Frame-Options header, halaman dapat di-embed dalam iframe oleh attacker dari domain lain, memungkinkan clickjacking attacks untuk memaksa user mengklik elemen tersembunyi atau menginterceptkan form submissions.

---

## Temuan #4

| Field                      | Nilai                                                                                  |
| -------------------------- | -------------------------------------------------------------------------------------- |
| **Nama Kerentanan**        | Vulnerable JS Library (jQuery UI v1.12.1)                                              |
| **Tool Penemu**            | DAST                                                                                   |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                                                       |
| **URL / File**             | http://10.34.100.180/lib/pkp/lib/vendor/components/jqueryui/jquery-ui.min.js?v=3.3.0.8 |
| **Parameter / Baris Kode** | Library version                                                                        |
| **Method**                 | GET                                                                                    |
| **Payload**                | N/A                                                                                    |
| **Response / Bukti**       | /\*! jQuery UI - v1.12.1 dalam jquery-ui.min.js                                        |
| **OWASP Category**         | A06:2021 - Vulnerable and Outdated Components                                          |
| **Severity (Raw)**         | Medium                                                                                 |

### Screenshot / Bukti

1 instance: jQuery UI v1.12.1 detected (CVE-2021-41184, CVE-2021-41183, CVE-2021-41182, CVE-2022-31160)

### Catatan

jQuery UI 1.12.1 (2018) memiliki multiple known CVEs termasuk DOM Clobbering, RegEx DoS, dan XSS vulnerabilities. Versi terbaru (1.13.3+) sudah mempatch semua security issues. Penggunaan library yang rentan memungkinkan eksploitasi vulnerabilities yang sudah dipublikasikan.

---

## Temuan #5

| Field                      | Nilai                                                               |
| -------------------------- | ------------------------------------------------------------------- |
| **Nama Kerentanan**        | Application Error Disclosure                                        |
| **Tool Penemu**            | DAST                                                                |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                                    |
| **URL / File**             | http://10.34.100.180/cache/t_compile/ (multiple compiled templates) |
| **Parameter / Baris Kode** | HTTP Response Status Code                                           |
| **Method**                 | GET                                                                 |
| **Payload**                | GET /cache/t_compile/[hash].php                                     |
| **Response / Bukti**       | HTTP/1.0 500 Internal Server Error pada multiple files              |
| **OWASP Category**         | A09:2021 - Security Logging and Monitoring Failures                 |
| **Severity (Raw)**         | Low                                                                 |

### Screenshot / Bukti

336 instances pada file-file di /cache/t_compile/ (compiled template errors)

### Catatan

Aplikasi menampilkan error 500 yang dapat mengungkap path direktori, framework details, dan library information. Ini membantu attacker untuk reconnaissance dan mengidentifikasi potential attack vectors. Custom error pages yang tidak mengungkap informasi teknis lebih disarankan.

---

## Temuan #6

| Field                      | Nilai                                              |
| -------------------------- | -------------------------------------------------- |
| **Nama Kerentanan**        | Server Leaks Version Information via Server Header |
| **Tool Penemu**            | DAST                                               |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                   |
| **URL / File**             | http://10.34.100.180 (all endpoints)               |
| **Parameter / Baris Kode** | HTTP Response Header: Server                       |
| **Method**                 | GET                                                |
| **Payload**                | N/A                                                |
| **Response / Bukti**       | Server: Apache/2.4.58 (Ubuntu)                     |
| **OWASP Category**         | A05:2021 - Security Misconfiguration               |
| **Severity (Raw)**         | Low                                                |

### Screenshot / Bukti

Multiple instances pada semua endpoints menampilkan: Apache/2.4.58 (Ubuntu)

### Catatan

Server header mengungkapkan web server (Apache 2.4.58) dan OS (Ubuntu), membantu attacker mengidentifikasi known vulnerabilities untuk versi spesifik. Information disclosure ini mengurangi attack surface protection dan memfasilitasi reconnaissance phase.

---

## Temuan #7

| Field                      | Nilai                                      |
| -------------------------- | ------------------------------------------ |
| **Nama Kerentanan**        | Cookie No HttpOnly Flag                    |
| **Tool Penemu**            | DAST                                       |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                           |
| **URL / File**             | http://10.34.100.180                       |
| **Parameter / Baris Kode** | Cookie: OJSSID                             |
| **Method**                 | GET                                        |
| **Payload**                | N/A                                        |
| **Response / Bukti**       | Set-Cookie: OJSSID (without HttpOnly flag) |
| **OWASP Category**         | A07:2021 - Cross-Site Scripting (XSS)      |
| **Severity (Raw)**         | Low                                        |

### Screenshot / Bukti

1 instance pada root URL dengan Set-Cookie: OJSSID tanpa flag HttpOnly

### Catatan

Session cookie OJSSID dapat diakses oleh JavaScript, sehingga XSS vulnerabilities dapat mencuri session ID dan melakukan session hijacking. HttpOnly flag harus diset untuk mencegah akses via JavaScript.

---

## Temuan #8

| Field                      | Nilai                                           |
| -------------------------- | ----------------------------------------------- |
| **Nama Kerentanan**        | Cookie without SameSite Attribute               |
| **Tool Penemu**            | DAST                                            |
| **Tool Spesifik**          | OWASP ZAP 2.16.1                                |
| **URL / File**             | http://10.34.100.180                            |
| **Parameter / Baris Kode** | Cookie: OJSSID                                  |
| **Method**                 | GET                                             |
| **Payload**                | N/A                                             |
| **Response / Bukti**       | Set-Cookie: OJSSID (without SameSite attribute) |
| **OWASP Category**         | A04:2021 - Insecure Deserialization             |
| **Severity (Raw)**         | Low                                             |

### Screenshot / Bukti

1 instance pada root URL dengan Set-Cookie: OJSSID tanpa SameSite attribute

### Catatan

Tanpa SameSite attribute, cookie akan dikirim dalam cross-site requests, memungkinkan CSRF attacks untuk menggunakan session user tanpa consent. SameSite=Strict atau Lax harus dikonfigurasi untuk session cookies yang sensitif.

---

## Summary Findings

**Total Temuan Teridentifikasi: 8 (dari 20+ alert types di ZAP)**

| Severity   | Count | Kategori OWASP               |
| ---------- | ----- | ---------------------------- |
| **Medium** | 5     | A05 (3x), A06 (1x), A09 (1x) |
| **Low**    | 3     | A05 (1x), A07 (1x), A04 (1x) |
| **TOTAL**  | **8** | Multi-kategori               |

**Primary Risk Areas:**

- **Security Misconfiguration (A05)** - CSP Headers, Directory Browsing, Server Info Leak
- **Vulnerable Components (A06)** - jQuery UI outdated
- **XSS Prevention** - Cookie security flags missing
- **CSRF Prevention** - SameSite attribute missing
- **Error Handling** - Sensitive information disclosure

---

_Report Generated: OWASP ZAP 2.16.1 | Target: Open Journal Systems (OJS) | Scan Date: 2026-04-02_
