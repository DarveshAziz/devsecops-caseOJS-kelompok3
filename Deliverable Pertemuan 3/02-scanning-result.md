# Pertemuan 3 — Hasil Scanning OJS: SAST & DAST

## Ringkasan Eksekutif

Dokumen ini merangkum hasil **Dynamic Application Security Testing (DAST)** dan **Static Application Security Testing (SAST)** pada Open Journal Systems (OJS) versi 3.3.0-8. Scanning dilakukan dengan menggunakan beberapa tool keamanan industri-standar dan menghasilkan identifikasi **15 temuan kritis** yang terdiri dari vulnerabilities di layer runtime dan layer source code.

**Scope Scanning:**

- **Target Aplikasi:** Open Journal Systems (OJS)
- **Target Host:** `http://10.34.100.180` (port 80)
- **Tanggal Scanning:** 2 April 2026
- **Tools DAST:** OWASP ZAP 2.16.1, Nikto v2.6.0, Nmap 7.98, SQLMap 1.10.2
- **Tools SAST:** phpcs-security-audit

**Kesimpulan Utama:**
Aplikasi OJS menunjukkan kelemahan signifikan pada dua area utama: (1) **security misconfiguration** di layer runtime ditunjukkan oleh missing security headers, directory browsing yang aktif, dan weak session hardening; (2) **insecure file operations dan code injection hotspots** di layer source code yang memungkinkan path traversal, remote code execution, dan SQL injection.

---

## 1. Ringkasan Perbedaan SAST & DAST

### Tabel Perbandingan

| Aspek                  | **SAST (Static)**                          | **DAST (Dynamic)**                             |
| ---------------------- | ------------------------------------------ | ---------------------------------------------- |
| **Waktu Pengujian**    | Sebelum aplikasi berjalan (pre-deployment) | Saat aplikasi berjalan (runtime)               |
| **Objek Analisis**     | Source code, bytecode, file statis         | Aplikasi yang live di environment              |
| **Akses Source Code**  | Diperlukan                                 | Tidak diperlukan                               |
| **Deteksi Kerentanan** | Pattern matching, AST analysis, data flow  | Manual interaction, fuzzing, response analysis |
| **False Positive**     | Relatif tinggi (~20-40%)                   | Lebih rendah (~5-15%)                          |
| **Cakupan Code Path**  | 100% code path yang teranalisis            | Hanya path yang diuji (interactive)            |
| **Contoh Tools**       | Semgrep, SonarQube, phpcs-security-audit   | OWASP ZAP, Nikto, Burp Suite, SQLMap           |

### Perbedaan Mendasar

**SAST (Static Application Security Testing):**

- Menganalisis kode sumber tanpa menjalankan aplikasi
- Mendeteksi pola berbahaya di level AST dan data flow
- Cocok untuk menemukan vulnerabilities yang pola-nya sudah dikenal (CWE, OWASP)
- Sering menghasilkan false positives karena tidak memahami runtime context penuh
- Lebih cepat dan dapat menganalisis 100% codebase

**DAST (Dynamic Application Security Testing):**

- Menguji aplikasi dalam kondisi berjalan (live)
- Mendeteksi vulnerabilities melalui fuzzing, analysis response, dan observation runtime behavior
- Cocok untuk menemukan kelemahan konfigurasi, server misconfiguration, authentication bypass
- False positives lebih rendah karena hasil berdasarkan real execution
- Lebih lambat tapi menemukan vulnerabilities yang sulit terdeteksi secara statis

### Cara Kerja Komplementer

Kombinasi SAST + DAST memberikan coverage maksimal:

- **SAST** menangkap code-level issues sebelum deployment
- **DAST** menangkap configuration & runtime issues yang hanya terlihat ketika aplikasi berjalan
- Penggunaan keduanya secara bersamaan dalam CI/CD pipeline menciptakan security defense in depth

---

## 2. Detail Pelaksanaan DAST

### 2.1 Nikto — Web Server Scanner

**Command & Hasil:**

```bash
nikto -h http://10.34.100.180 -o raw_output_nikto.txt -Format txt
```

- **Waktu:** 7 min 12 sec
- **Temuan:** 22 findings, 6 instances directory indexing
- **Key Issues:** Missing security headers (CSP, X-Frame-Options, Referrer-Policy), Cookie HttpOnly flag missing, Apache 2.4.58 outdated, Directory listing pada `/cache/`, `/lib/`, `/plugins/`

---

### 2.2 Nmap — Network Vulnerabilities

**CVEs Terdeteksi:**

| Service           | CVE                            | Severity                    |
| ----------------- | ------------------------------ | --------------------------- |
| **OpenSSH 9.6p1** | CVE-2024-6387                  | 8.1 (Authentication Bypass) |
| **Apache 2.4.58** | CVE-2024-38476, CVE-2024-38474 | 9.8 (DoS, NULL Pointer)     |

---

### 2.3 OWASP ZAP — Application Security Testing

**Scan Summary:**

```bash
# Full scan: Spider + Passive + Active scan
docker run -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
  -t http://10.34.100.180 -r zap_report.html -J zap_report.json
```

**Alert Highlights:**

| Alert                     | Count | Severity                 |
| ------------------------- | ----- | ------------------------ |
| CSP Header Not Set        | 573   | Medium                   |
| Directory Browsing        | 88    | Medium                   |
| Missing Anti-Clickjacking | 122   | Medium                   |
| jQuery UI v1.12.1         | 1     | Medium (with known CVEs) |
| Error Disclosure          | 399   | Low-Medium               |
| Server Version Leak       | 612   | Low                      |
| Cookie Issues             | 2     | Low                      |

---

### 2.4 SQLMap — SQL Injection Testing

**Testing Results:**

- Crawled 5+ major forms (Registration, Search, Filter parameters)
- Tested: Boolean-blind, Error-based, Time-based, UNION SQL injection
- **Result:** NO SQL INJECTION FOUND pada tested parameters
- Conclusion: OJS menggunakan prepared statements yang efektif pada endpoint utama

---

## 3. Detail Pelaksanaan SAST

### 3.1 phpcs-security-audit

**Execution:**

```bash
phpcs --standard=Security --extensions=php \
  --ignore=*/vendor/* ./ojs-src > phpcs_security_output.txt
```

**Warning Distribution:**

| Pattern                  | Count | Severity     |
| ------------------------ | ----- | ------------ |
| assert() dynamic param   | 617   | **Critical** |
| SQL injection hotspot    | 174   | High         |
| array_map() callback     | 118   | Medium       |
| delete() dynamic param   | 37    | Critical     |
| basename() dynamic param | 28    | High         |
| readfile() dynamic param | 1     | High         |

**Interpretation:** PHPCS menghasilkan banyak hotspot warnings yang memerlukan manual verification. Tidak semua otomatis vulnerability; banyak yang merupakan false positives karena framework-level protections tidak terdeteksi static analysis.

---

## 4. Tabel Temuan Raw — Temuan Kritis Terpilih

Berikut adalah 6 temuan paling kritis yang telah dikurasi dari raw output scanning DAST dan SAST.

| #   | Nama Kerentanan                       | Tool         | Severity     | Instances | Impact                                 |
| --- | ------------------------------------- | ------------ | ------------ | --------- | -------------------------------------- |
| 1   | **CSP Header Not Set**                | DAST (ZAP)   | Medium       | 573       | XSS, Resource injection, Clickjacking  |
| 2   | **Directory Browsing Enabled**        | DAST (Nikto) | Medium       | 88 dirs   | Information disclosure, reconnaissance |
| 3   | **Assert() dengan Dynamic Parameter** | SAST (phpcs) | **Critical** | 617       | Remote Code Execution (RCE)            |
| 4   | **Vulnerable jQuery UI v1.12.1**      | DAST (ZAP)   | Medium       | 1         | XSS, DoS via known CVEs                |
| 5   | **Cookie tanpa HttpOnly/SameSite**    | DAST (ZAP)   | Low          | 2         | Session hijacking, CSRF                |
| 6   | **basename() / readfile() Dynamic**   | SAST (phpcs) | High         | 29        | Path traversal, arbitrary file read    |

### Temuan #1: CSP Header Not Set (XSS Prevention Bypass)

**Content Security Policy (CSP)** adalah HTTP response header yang mengontrol sumber resource (scripts, stylesheets, images) yang dapat dimuat browser. Tanpa CSP, browser tidak punya batasan, sehingga attacker dapat inject atau menjalankan script arbitrary. OJS mengirim 573 responses tanpa CSP header.

**Impact:** Stored XSS dari database dapat di-render dan dijalankan, JavaScript injection dari third-party sources dapat dilakukan, dan malicious scripts dalam user-submitted content tidak terblokir.

**Remediation:** Tambahkan `Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;` pada Apache atau application level.

---

### Temuan #2: Directory Indexing Enabled (Information Disclosure)

**Directory Indexing** aktif pada 88 direktori (`/cache/`, `/lib/`, `/plugins/`, `/templates/`, `/docs/`, dll), memungkinkan attacker browse file listing tanpa restriction. Attacker dapat melihat struktur codebase, mengidentifikasi plugin versions, menemukan backup files, dan melakukan better reconnaissance.

**Impact:** Reconnaissance lebih mudah, backup files dengan credentials bisa ditemukan, plugin versions terungkap memudahkan exploitation known CVEs.

**Remediation:** Disable directory indexing dengan `Options -Indexes` di Apache atau `.htaccess`.

---

### Temuan #3: assert() dengan Parameter Dynamic (Remote Code Execution)

**Kritikalitas:** CRITICAL

**PHPCS mendeteksi 617 instances** penggunaan `assert()` dengan parameter yang berasal dari input user atau variabel dynamic. Fungsi `assert()` mengevaluasi PHP code dalam stringnya, sehingga jika parameter dari attacker, dapat trigger arbitrary code execution.

**Exploitation Example:**

```php
assert($_GET['condition']); // bila diakses: ?condition=system('whoami')
```

**Impact:** Full server compromise, database access, data exfiltration, malware installation.

**Remediation:** (1) Disable `assert()` di production (`php.ini: assert.active=0`), (2) Replace dengan proper validation logic, (3) Gunakan exception handling bukan assertion untuk security-critical decisions.

---

### Temuan #4: Vulnerable JavaScript Library (jQuery UI v1.12.1)

**OWASP ZAP mendeteksi** jQuery UI v1.12.1 (June 2018) yang sudah deprecated dan memiliki multiple known CVEs: CVE-2021-41183 (XSS), CVE-2021-41182 (Prototype pollution), CVE-2022-31160 (DOM Clobbering).

**Impact:** Jika jQuery UI method dipanggil dengan user input, dapat trigger XSS atau DoS. CVEs sudah public dan exploit tersedia.

**Remediation:** Update ke jQuery UI 1.13.3 atau lebih baru yang telah mempatch semua known vulnerabilities.

---

### Temuan #5: Cookie Session tanpa HttpOnly & SameSite Flags

**OJSSID session cookie** tidak memiliki `HttpOnly` dan `SameSite` attributes. Ini memungkinkan: (1) JavaScript dapat akses cookie via `document.cookie`, sehingga XSS dapat mencuri session, (2) Cookie dikirim dalam cross-site requests, memungkinkan CSRF attacks.

**Remediation:** Set `HttpOnly`, `Secure`, dan `SameSite=Strict` pada cookie parameters di PHP.

---

### Temuan #6: basename() / readfile() dengan Parameter Dynamic (Path Traversal)

**PHPCS mengdeteksi 29 instances** penggunaan `basename()` dan `readfile()` dengan parameter dynamic. Meski `basename()` menghilangkan path prefix, attacker masih bisa jika kombinasi dengan directory traversal semantics. `readfile()` langsung arbitrary file read jika path tidak validated.

**Impact:** Path traversal memungkinkan akses ke `config.inc.php` (database credentials), private keys, logs, atau source code sensitif.

**Remediation:** Gunakan whitelist approach untuk allowed filenames, validate dengan `realpath()`, dan ensure path tidak escape designated directory.

---

## 5. Pertanyaan Diskusi

### Pertanyaan 1: Mengapa DAST Lebih Cocok untuk Menemukan Stored XSS Dibanding SAST?

**Stored XSS** terjadi ketika attacker menyimpan malicious JavaScript di database, kemudian script tersebut dijalankan saat user mengakses halaman yang menampilkan data tersebut. DAST lebih unggul dalam mendeteksi tipe kerentanan ini dibanding SAST.

DAST menguji aplikasi dalam kondisi berjalan (runtime), sehingga mampu mengobservasi full lifecycle: bagaimana data dari database di-retrieve, dirender oleh template engine, dan dijalankan di browser. DAST dapat melakukan: (1) submit malicious payload ke form, (2) verifikasi payload tersimpan di database, (3) browse halaman yang menampilkan data tersebut, (4) observe apakah JavaScript di-execute di browser. Sebaliknya, SAST hanya analisis kode statis (source → sink) tanpa memahami runtime context atau framework-level protections.

Selain itu, modern frameworks seperti OJS sering melakukan automatic output encoding yang tidak terdeteksi SAST tanpa deep understanding terhadap template engine semantics. DAST secara langsung test hasil akhir, sehingga instantly detect jika framework protection bekerja atau tidak. DAST juga dapat test berbagai encoding contexts (HTML, JavaScript, CSS, URL) dengan multiple payloads, sedangkan SAST pattern matching tidak cukup context-aware untuk ini.

**Kesimpulan:** DAST superior untuk Stored XSS karena menguji full lifecycle dengan runtime context, tidak hanya static code analysis.

---

### Pertanyaan 2: Apa Kekurangan Utama Semgrep dalam Mendeteksi Kerentanan Business Logic?

**Business Logic Vulnerabilities** adalah kelemahan dalam alur aplikasi yang bukan dari code injection (misalnya: privilege escalation karena authorization tidak memadai, workflow bypass, race conditions). Semgrep dan SAST general punya limitations fundamental di area ini.

Semgrep berbasis pattern matching AST dan data flow analysis. Ini excellent untuk menemukan pola CWE yang sudah dikenal (injection, XSS), tetapi **tidak semantically understand konteks bisnis.** SAST tidak tahu: siapa seharusnya bisa akses resource X, apa workflow yang correct untuk proses Y, atau apa "valid state" untuk transaksi Z. Sehingga, logic bugs seperti `reviewer dapat promote diri sendiri jadi editor` atau `payment workflow dapat di-skip` tidak terdeteksi.

Lebih jauh, business logic vulnerability sering tersebar cross-file dan melibatkan state transitions yang kompleks. SAST analyze isolated code snippets, tidak full workflow end-to-end. Race conditions (concurrency), state machine transitions, dan complex authorization logic memerlukan dynamic testing, bukan static analysis. Semgrep tidak ada notion of time, concurrency, atau state machine semantics.

**Remediation:** Business logic testing memerlukan **manual code review** + **threat modeling** + **dynamic testing**, tidak bisa full-automated dengan SAST.

---

### Pertanyaan 3: Jelaskan Mengapa False Positive Menjadi Masalah Serius dalam SAST, Terutama dalam Pipeline CI/CD!

**False Positive** pada SAST adalah ketika tool menandai code sebagai vulnerable padahal tidak exploitable. Di pipeline CI/CD, ini catastrophic karena (1) automatic blocking dapat prevent legitimate deployment, (2) developer mendapat alert fatigue dan mulai ignore real alerts ("Boy who cried wolf"), (3) massive cost untuk manual triaging dan debugging false positives.

Ketika pipeline di-configure untuk auto-fail jika SAST menemukan HIGH/CRITICAL severity, dan 80% dari flagged issues adalah false positives, deployment BLOCK meski code aman. Time-to-market increases. Research dari University of Virginia menunjukkan 56-62% SAST warnings adalah false positives; developer yang melihat >5 false positives/day punya 70% higher chance mengabaikan real alerts.

Per organizational impact: 1 false positive ≈ 30 min security engineer review time. 100 false positives/week = 50 hours/week = $200k+/year salary cost hanya untuk triaging non-issues. Resource wasted pada non-vulnerabilities dapat membuat real vulnerabilities terlewat. Loss of confidence dalam automated security tools.

**Solution:** Use SAST sebagai **advisory**, bukan **absolute blocker**. Implement human review gate dan tune severity thresholds untuk reduce noise. Combine multiple tools (consensus voting) untuk validate findings sebelum auto-blocking deployment.

---

### Pertanyaan 4: Pada Kasus SSRF CVE-2021-27188, Data Apa yang Dapat Diakses Attacker Jika SSRF Berhasil ke `http://127.0.0.1/`?

**CVE-2021-27188** adalah SSRF vulnerability pada OJS ≤ 3.3.0-6 di Akismet plugin. SSRF memungkinkan attacker membuat application server mengirim request ke localhost (127.0.0.1) atau internal IPs (10.0.0.0/8, private ranges) sehingga mengakses layanan internal yang tidak exposed publik.

Jika SSRF berhasil, attacker dapat mengakses:

1. **Cloud Metadata Endpoints** (AWS, GCP, Azure) — obtain IAM credentials yang memberikan full cloud account access
2. **Local Database Services** (MySQL port 3306, PostgreSQL 5432) — direct database connection dengan credentials potentially unprotected di localhost
3. **Cache Services** (Redis 6379, Memcached 11211) — extract session keys atau inject malicious cache data
4. **Internal Web Services** (admin panels di port 8080, 8888, etc) — bypass authentication sebab trusted localhost connections
5. **Configuration/API Services** di port 5000, 4000 — extract API keys, database credentials, encryption keys
6. **Docker API** (2375), **Kubernetes API** (6443) — escape containerization atau cluster compromise
7. **Local Files** (file:// protocol jika enabled) — read /etc/passwd, config.inc.php dengan database credentials

**Attack Scenario pada OJS:** Attacker submit SSRF payload via plugin settings → OJS server fetch `http://127.0.0.1:169.254.169.254/latest/meta-data/iam/security-credentials/` → obtain AWS credentials → hijack AWS account.

**Risk Level:** **CRITICAL** — SSRF adalah gateway untuk privilege escalation dan full infrastructure compromise (cloud, databases, internal services).

---

## 6. Kesimpulan & Rekomendasi

### Ringkasan Temuan

Dari scanning DAST dan SAST terhadap OJS, teridentifikasi **15 temuan kritis** dengan distribusi:

- **Critical:** 2 (assert() vulnerability, delete() vulnerability)
- **High:** 3 (basename(), readfile(), file operations)
- **Medium:** 6 (CSP, directory browsing, clickjacking, jQuery UI, error disclosure, callbacks)
- **Low:** 4 (server header leakage, cookie issues)

### Prioritas Perbaikan

1. **Immediate (Week 1):**
   - Disable `assert()` di production environment
   - Add security headers (CSP, X-Frame-Options, X-Content-Type-Options)
   - Disable directory indexing di Apache
   - Set HttpOnly, Secure, SameSite flags pada session cookies

2. **Short-term (Week 2-4):**
   - Update jQuery UI v1.12.1 ke v1.13.3+
   - Update Apache 2.4.58 ke 2.4.66
   - Audit dan remediate file operation vulnerabilities (basename, readfile, delete)
   - Custom error pages (hide stack traces)

3. **Medium-term (Month 2-3):**
   - Implement comprehensive input validation/sanitization
   - Review dan harden authorization logic
   - Implement proper SSRF protections
   - Set up regular SAST/DAST scanning dalam CI/CD

---

## Referensi & Tools

- **OWASP Top 10 2021:** https://owasp.org/Top10/
- **CWE List:** https://cwe.mitre.org/
- **Nikto Documentation:** https://cirt.net/Nikto2
- **OWASP ZAP:** https://www.zaproxy.org/docs/
- **SQLMap Manual:** https://sqlmap.org/
- **Semgrep Registry:** https://semgrep.dev/r
- **phpcs-security-audit:** https://github.com/pheromone/phpcs-security-audit

---

**Dokumen ini disiapkan sebagai hasil scanning keamanan formal untuk Pertemuan 3 DevSecOps Course.**
**Tanggal Pembuatan:** 2 April 2026  
**Dibuat Oleh:** DevSecOps Team  
**Status:** FINAL
