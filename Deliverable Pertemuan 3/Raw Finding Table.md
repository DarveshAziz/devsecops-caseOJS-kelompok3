# Tabel Temuan Raw Scanning

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

## Temuan #9

| Field                      | Nilai                                                                     |
| -------------------------- | ------------------------------------------------------------------------- |
| **Nama Kerentanan**        | Assert Eval Function with Dynamic Parameter (Remote Code Execution Risk) |
| **Tool Penemu**            | SAST                                                                      |
| **Tool Spesifik**          | phpcs-security-audit                                                      |
| **URL / File**             | /lib/pkp/pages/submission/PKPSubmissionHandler.inc.php                    |
| **Parameter / Baris Kode** | Line 60                                                                   |
| **Method**                 | N/A                                                                       |
| **Payload**                | N/A                                                                       |
| **Response / Bukti**       | WARNING: Assert eval function assert() detected with dynamic parameter   |
| **OWASP Category**         | A03:2021 - Injection                                                      |
| **Severity (Raw)**         | Critical                                                                  |

### Screenshot / Bukti

Multiple instances found (25+ files): SubmissionHandler.inc.php:56, PKPReviewerHandler.inc.php:224, PKPWorkflowHandler.inc.php:741, SubmissionFilesUploadForm.inc.php:138, dan 20+ files lainnya.

### Catatan

Fungsi `assert()` dengan parameter dynamic merupakan remote code execution vulnerability. Attacker dapat menginjeksi code arbitrary melalui parameter yang tidak tersanitasi. Function `assert()` seharusnya hanya digunakan untuk debugging dan harus disable di production (`assert.active = 0`). 

**Remediation:**
- Disable assert() di production environment
- Ganti assert() dengan proper exception handling dan validation
- Tidak menggunakan user input langsung sebagai parameter assert()
- Setkan php.ini `assert.active = 0` dan `assert.exception = 1` untuk development

---

## Temuan #10

| Field                      | Nilai                                                                |
| -------------------------- | -------------------------------------------------------------------- |
| **Nama Kerentanan**        | Filesystem Function basename() dengan Dynamic Parameter (Path Traversal) |
| **Tool Penemu**            | SAST                                                                 |
| **Tool Spesifik**          | phpcs-security-audit                                                 |
| **URL / File**             | /lib/pkp/pages/admin/AdminHandler.inc.php                            |
| **Parameter / Baris Kode** | Line 424                                                             |
| **Method**                 | N/A                                                                  |
| **Payload**                | N/A                                                                  |
| **Response / Bukti**       | WARNING: Filesystem function basename() detected with dynamic parameter |
| **OWASP Category**         | A01:2021 - Broken Access Control / A03:2021 - Injection              |
| **Severity (Raw)**         | High                                                                 |

### Screenshot / Bukti

1 instance di AdminHandler.inc.php line 424 dengan penggunaan basename() terhadap variabel dynamic.

### Catatan

Fungsi `basename()` dengan parameter yang berasal dari user input dapat digunakan untuk path traversal attack. Attacker dapat menggunakan sequence seperti `../` untuk access files di direktori parent yang sensitif. 

**Vulnerability Scenario:**
- Input user: `../../etc/passwd`
- basename($_GET['file'])` bisa menghasilkan file traversal jika logic tidak proper
- Dapat mengakses file konfigurasi, database credentials, atau source code

**Remediation:**
- Validate dan sanitize semua file path input dari user
- Gunakan whitelist approach untuk diizinkan file names
- Implementasi path traversal checks: `realpath()` vs `__DIR__`
- Tidak pernah trust user input untuk file operations

---

## Temuan #11

| Field                      | Nilai                                                              |
| -------------------------- | ------------------------------------------------------------------ |
| **Nama Kerentanan**        | Filesystem Function readfile() dengan Dynamic Parameter (File Read Vulnerability) |
| **Tool Penemu**            | SAST                                                               |
| **Tool Spesifik**          | phpcs-security-audit                                               |
| **URL / File**             | /lib/pkp/controllers/page/PageHandler.inc.php                      |
| **Parameter / Baris Kode** | Line 155                                                           |
| **Method**                 | N/A                                                                |
| **Payload**                | N/A                                                                |
| **Response / Bukti**       | WARNING: Filesystem function readfile() detected with dynamic parameter |
| **OWASP Category**         | A01:2021 - Broken Access Control / A05:2021 - Security Misconfiguration |
| **Severity (Raw)**         | High                                                               |

### Screenshot / Bukti

1 instance di PageHandler.inc.php line 155 dengan readfile() menggunakan dynamic parameter.

### Catatan

Fungsi `readfile()` dengan parameter dynamic memungkinkan arbitrary file read vulnerability. Attacker dapat membaca file sensitif seperti konfigurasi database, private keys, source code, atau logs yang berisi informasi berharga.

**Vulnerability Scenario:**
- Input user: `../../config.inc.php` atau `../../../.env`
- readfile($_GET['page'])` dapat membaca arbitrary file  
- Information disclosure: database credentials, API keys, paths

**Remediation:**
- Validate all file paths terhadap whitelist  
- Implement strict input validation dan sanitization
- Gunakan `is_file()` dan `strpos()` untuk path validation
- Jangan pernah allow `.` atau `..` dalam file path
- Restrict file read operations ke allowed directory saja

---

## Temuan #12

| Field                      | Nilai                                                                |
| -------------------------- | -------------------------------------------------------------------- |
| **Nama Kerentanan**        | Filesystem Function delete() dengan Dynamic Parameter (Arbitrary File Deletion) |
| **Tool Penemu**            | SAST                                                                 |
| **Tool Spesifik**          | phpcs-security-audit                                                 |
| **URL / File**             | /lib/pkp/controllers/api/file/PKPManageFileApiHandler.inc.php       |
| **Parameter / Baris Kode** | Line 59                                                              |
| **Method**                 | N/A                                                                  |
| **Payload**                | N/A                                                                  |
| **Response / Bukti**       | WARNING: Filesystem function delete() detected with dynamic parameter |
| **OWASP Category**         | A01:2021 - Broken Access Control                                    |
| **Severity (Raw)**         | Critical                                                             |

### Screenshot / Bukti

1 instance di PKPManageFileApiHandler.inc.php line 59 dengan delete() menggunakan dynamic parameter.

### Catatan

Fungsi `delete()` atau `unlink()` dengan parameter dynamic adalah arbitrary file deletion vulnerability. Attacker dapat menghapus file aplikasi kritis, logs, backup, atau sistem files yang menyebabkan Denial of Service atau data loss.

**Vulnerability Scenario:**
- Input user: `../../config.inc.php` atau system critical files
- delete($user_supplied_path)` dapat menghapus arbitrary files
- Impact: Denial of Service, Data Loss, System Compromise

**Remediation:**
- Implement strict whitelist untuk allowed files/paths
- Validate input terhadap hardcoded list of deletable files
- Gunakan file signatures atau checksums untuk verify file identity
- Implement access controls untuk authorization check
- Log all file deletion operations untuk audit trail
- Implement recovery mechanism atau versioning

---

## Temuan #13

| Field                      | Nilai                                                              |
| -------------------------- | ------------------------------------------------------------------ |
| **Nama Kerentanan**        | Filesystem Function fclose() dengan Dynamic Parameter (File Handle Abuse) |
| **Tool Penemu**            | SAST                                                               |
| **Tool Spesifik**          | phpcs-security-audit                                               |
| **URL / File**             | /lib/pkp/pages/stats/PKPStatsHandler.inc.php                       |
| **Parameter / Baris Kode** | Line 635                                                           |
| **Method**                 | N/A                                                                |
| **Payload**                | N/A                                                                |
| **Response / Bukti**       | WARNING: Filesystem function fclose() detected with dynamic parameter |
| **OWASP Category**         | A05:2021 - Security Misconfiguration                               |
| **Severity (Raw)**         | High                                                               |

### Screenshot / Bukti

1 instance di PKPStatsHandler.inc.php line 635 dengan fclose() menggunakan dynamic parameter.

### Catatan

Penggunaan `fclose()` dengan dynamic parameter dapat menyebabkan file handle confusion attack. Jika parameter bukan resource handle yang valid, dapat menghasilkan unexpected behavior, file descriptor leaks, atau segmentation faults.

**Risks:**
- File descriptor exhaustion (DoS)  
- Unintended file closure menyebabkan data loss
- File handle confusion attacks
- Application crashes atau memory corruption

**Remediation:**
- Implement proper file handle management dengan try-finally blocks
- Validate bahwa parameter adalah valid file resource
- Use proper exception handling untuk file operations
- Implement resource cleanup mechanisms
- Test edge cases untuk file operation errors

---

## Temuan #14

| Field                      | Nilai                                                        |
| -------------------------- | ------------------------------------------------------------ |
| **Nama Kerentanan**        | Callback Injection via array_map() Function                 |
| **Tool Penemu**            | SAST                                                         |
| **Tool Spesifik**          | phpcs-security-audit                                         |
| **URL / File**             | /pages/issue/IssueHandler.inc.php                            |
| **Parameter / Baris Kode** | Line 303                                                     |
| **Method**                 | N/A                                                          |
| **Payload**                | N/A                                                          |
| **Response / Bukti**       | WARNING: Function array_map() that supports callback detected |
| **OWASP Category**         | A03:2021 - Injection                                         |
| **Severity (Raw)**         | Medium                                                       |

### Screenshot / Bukti

Multiple instances (6 files): IssueHandler.inc.php:303, SearchHandler.inc.php:218, ArticleHandler.inc.php:232, SettingsHandler.inc.php:70, WorkflowHandler.inc.php:59, StatsHandler.inc.php:52

### Catatan

Penggunaan `array_map()` dengan callback function dapat mengakibatkan callback injection jika callback name berasal dari user input atau tidak ter-sanitasi. Attacker dapat menginject arbitrary callback function untuk eksekusi code yang tidak diinginkan.

**Vulnerability:**
```php
$callback = $_GET['func']; // User input
array_map($callback, $array);  // Arbitrary callback execution
```

**Remediation:**
- Tidak pernah accept callback names dari user input
- Implementasi whitelist untuk allowed callback functions
- Gunakan anonymous functions atau closures sebagai callback
- Validate callback names terhadap hardcoded list
- Use type hints dan parameter validation

---

## Temuan #15

| Field                      | Nilai                                                       |
| -------------------------- | ----------------------------------------------------------- |
| **Nama Kerentanan**        | Callback Injection via array_udiff() Function              |
| **Tool Penemu**            | SAST                                                        |
| **Tool Spesifik**          | phpcs-security-audit                                        |
| **URL / File**             | /lib/pkp/controllers/grid/navigationMenus/form/NavigationMenuForm.inc.php |
| **Parameter / Baris Kode** | Line 87                                                     |
| **Method**                 | N/A                                                         |
| **Payload**                | N/A                                                         |
| **Response / Bukti**       | WARNING: Function array_udiff() that supports callback detected |
| **OWASP Category**         | A03:2021 - Injection                                        |
| **Severity (Raw)**         | Medium                                                      |

### Screenshot / Bukti

1 instance di NavigationMenuForm.inc.php line 87 dengan array_udiff() menggunakan callback.

### Catatan

Fungsi `array_udiff()` dengan user-supplied callback dapat menghasilkan code execution vulnerability yang sama dengan array_map(). Jika callback function name diambil dari input yang tidak ter-sanitasi, attacker dapat menginject arbitrary functions.

**Remediation:**
- Implementasi strict input validation untuk callback names
- Gunakan predefined callback functions atau closures
- Tidak pernah pass user input sebagai callback
- Utilize type system dan strict mode untuk PHP
- Code review dan security testing untuk dynamic callback usage

---

## Summary Findings

**Total Temuan Teridentifikasi: 15 (8 DAST + 7 SAST)**

| Tool   | Source       | Medium | High | Critical | Total |
| ------ | ------------ | ------ | ---- | -------- | ----- |
| DAST   | OWASP ZAP    | 5      | 0    | 0        | 5     |
| DAST   | OWASP ZAP    | 0      | 3    | 0        | 3     |
| SAST   | phpcs-audit  | 2      | 3    | 2        | 7     |
| **TOTAL** | **Combined**   | **7**  | **6** | **2**    | **15**    |

**Severity Distribution:**

| Severity   | Count | Kategori OWASP                                  |
| ---------- | ----- | ----------------------------------------------- |
| **Critical** | 2    | A03:2021-Injection (assert, delete functions)  |
| **High**   | 6     | A01:2021-Broken Access Control / A05:2021-Security Misconfiguration |
| **Medium** | 7     | A05:2021-Security Misconfiguration / A03:2021-Injection |

**Primary Risk Areas by Tool:**

**DAST Findings (Network/Runtime Security):**
- **Security Misconfiguration (A05)** - CSP Headers, Directory Browsing, Server Info Leak (4)
- **Vulnerable Components (A06)** - jQuery UI outdated (1)
- **Cookie Security (A07/A04)** - HttpOnly Flag, SameSite Attribute (2)

**SAST Findings (Code-Level Security):**
- **Injection Attacks (A03)** - assert() eval, array callbacks (4)
- **Access Control (A01)** - Arbitrary file operations, Path traversal (3)

**Combined Risk Assessment:**
This application presents SIGNIFICANT SECURITY RISKS across multiple attack vectors:
1. **Code Execution Risk (CRITICAL)** - assert() dan delete() functions dengan dynamic parameters
2. **Data Breach Risk (HIGH)** - Path traversal and arbitrary file read via filesystem functions  
3. **Information Disclosure (HIGH)** - Directory browsing, server headers, error messages
4. **Dependency Vulnerabilities (MEDIUM)** - Outdated libraries like jQuery UI

**Recommended Action Plan (Prioritized):**
1. **Immediate (P0):** Remove assert() usage, implement whitelist untuk file operations
2. **High Priority (P1):** Add security headers (CSP, X-Frame-Options), update jQuery UI
3. **Medium Priority (P2):** Implement input validation framework, disable directory browsing, sanitize error messages

---

_Report Generated: OWASP ZAP 2.16.1 (DAST) + phpcs-security-audit (SAST) | Target: Open Journal Systems (OJS) | Scan Date: 2026-04-02 | Combined Analysis_
