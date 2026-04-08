# Pertemuan 3 — Hasil Scanning OJS: SAST & DAST

## Ringkasan Eksekutif

Dokumen ini merangkum hasil **Dynamic Application Security Testing (DAST)**, **Static Application Security Testing (SAST)**, dan **manual code review** pada Open Journal Systems (OJS) versi 3.3.0-8. Seluruh hasil scanning terbaru dikonsolidasikan menjadi **20 temuan raw terpilih** yang mewakili isu paling relevan setelah deduplikasi antar-tool.

**Scope Scanning:**

- **Target Aplikasi:** Open Journal Systems (OJS)
- **Target Host:** `http://10.34.100.180` (port 80)
- **Tools DAST:** OWASP ZAP 2.16.1, Nikto v2.6.0, Nmap 7.98, SQLMap 1.10.2
- **Tools SAST:** phpcs-security-audit, Semgrep (`p/php`, `p/owasp-top-ten`, custom rules)
- **Metode Tambahan:** Manual code review pada 5 file kritis

**Kesimpulan Utama:**
Aplikasi OJS menunjukkan kelemahan signifikan pada dua area utama: (1) **security misconfiguration dan komponen usang** di layer runtime, ditunjukkan oleh missing security headers, directory browsing, potensi CSRF, serta service version yang rentan; (2) **unsafe code pattern** di layer source code, ditunjukkan oleh eval/assert dinamis, operasi file tanpa kontrol ketat, include berbasis variabel, stored XSS, dan unserialize tanpa filter.

---

## 1. Ringkasan Perbedaan SAST & DAST

### Tabel Perbandingan

| Aspek                        | **SAST (Static)**                    | **DAST (Dynamic)**                       |
| ---------------------------- | ------------------------------------------ | ---------------------------------------------- |
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

| Service                 | CVE                            | Severity                    |
| ----------------------- | ------------------------------ | --------------------------- |
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

| Pattern                  | Count | Severity           |
| ------------------------ | ----- | ------------------ |
| assert() dynamic param   | 617   | **Critical** |
| SQL injection hotspot    | 174   | High               |
| array_map() callback     | 118   | Medium             |
| delete() dynamic param   | 37    | Critical           |
| basename() dynamic param | 28    | High               |
| readfile() dynamic param | 1     | High               |

**Interpretation:** PHPCS menghasilkan banyak hotspot warnings yang memerlukan manual verification. Tidak semua otomatis vulnerability; banyak yang merupakan false positives karena framework-level protections tidak terdeteksi static analysis.

---

### 3.2 Semgrep

**Ruleset yang Dijalankan:**

- `p/php`
- `p/owasp-top-ten`
- custom rules

**Ringkasan Hasil:**

| Ruleset | Raw Match | Temuan Utama |
| ------- | --------- | ------------ |
| `p/php` | 1 | `phpinfo()` exposure |
| `p/owasp-top-ten` | 1 | `phpinfo()` exposure |
| Custom rules | 107 | `include/require` berbasis variabel, `eval()` injection |

**Interpretation:** Semgrep menambah bukti baru yang tidak muncul eksplisit pada tabel PHPCS, terutama pola `eval()` dinamis, `include/require` dengan variabel, dan paparan `phpinfo()`. Hasil Semgrep kemudian dideduplikasi agar tidak menghitung dua kali temuan `phpinfo()` dari dua ruleset berbeda.

---

### 3.3 Manual Code Review

**Cakupan Review:**

- DAO / database layer
- Authorization policy
- File upload manager
- TinyMCE plugin
- Template manager

**Temuan Manual Paling Penting:**

| Area | Temuan | Severity |
| ---- | ------ | -------- |
| Template manager | Stored XSS via `customHeaders` tanpa sanitasi | Critical |
| DAO layer | `unserialize()` tanpa `allowed_classes` | Critical |
| File manager | Upload file tanpa validasi MIME/ekstensi di fungsi inti | High |

**Interpretation:** Manual review melengkapi SAST otomatis dengan menemukan isu yang lebih kontekstual, terutama pada jalur output HTML, pengolahan data persisten, dan desain upload core yang terlalu permisif.

---

## 4. Tabel Temuan Raw — Temuan Kritis Terpilih

Berikut adalah 20 temuan raw final yang telah dikonsolidasikan dari ZAP, Nikto, Nmap, SQLMap, PHPCS, Semgrep, dan manual review. Temuan yang sama pada beberapa tools hanya dihitung satu kali pada sumber bukti yang paling representatif.

| # | Nama Kerentanan | Tool / Metode | Severity | Bukti Ringkas |
| - | --------------- | ------------- | -------- | ------------- |
| 1 | CSP Header Not Set | DAST (ZAP) | Medium | 573 respons tanpa CSP |
| 2 | Directory Browsing Enabled | DAST (ZAP) | Medium | 88 direktori terbuka |
| 3 | Missing Anti-clickjacking Header | DAST (ZAP) | Medium | 122 respons tanpa proteksi iframe |
| 4 | Vulnerable jQuery UI v1.12.1 | DAST (ZAP) | Medium | 1 library usang terdeteksi |
| 5 | Application Error Disclosure | DAST (ZAP) | Medium | Error page terekspos pada compiled templates |
| 6 | Missing HSTS Header | DAST (Nikto) | Medium | Header `strict-transport-security` tidak ada |
| 7 | Missing X-Content-Type-Options | DAST (Nikto) | Medium | Header `x-content-type-options` tidak ada |
| 8 | Missing Referrer-Policy | DAST (Nikto) | Low | Header `referrer-policy` tidak ada |
| 9 | OpenSSH 9.6p1 Multiple CVEs | DAST (Nmap) | High | `CVE-2024-6387` dan CVE lain terpetakan |
| 10 | Apache 2.4.58 Multiple CVEs | DAST (Nmap) | High | `CVE-2024-38476`, `CVE-2024-38474`, dst |
| 11 | Possible CSRF on Search Form | DAST (Nmap) | Medium | Script `http-csrf` menandai form `query` |
| 12 | Assert Dynamic Parameter | SAST (PHPCS) | Critical | `assert()` dinamis pada banyak file |
| 13 | Dynamic File Deletion Primitive | SAST (PHPCS) | Critical | `delete()` dinamis di API file |
| 14 | Dynamic File Read via `readfile()` | SAST (PHPCS) | High | `readfile()` dinamis di `PageHandler` |
| 15 | Variable-based Include/Require | SAST (Semgrep) | High | 104 instance `include/require` dinamis |
| 16 | Eval Injection Pattern | SAST (Semgrep) | Critical | 3 instance `eval()` dinamis |
| 17 | phpinfo Exposure | SAST (Semgrep) | Medium | `phpinfo()` terdeteksi di `AdminHandler` |
| 18 | Stored XSS via `customHeaders` | Manual Review | Critical | Output HTML tanpa sanitasi |
| 19 | File Upload Without Validation | Manual Review | High | Upload core tanpa whitelist MIME/ekstensi |
| 20 | Unsafe `unserialize()` on Persisted Data | Manual Review | Critical | `unserialize()` tanpa `allowed_classes` |

Enam temuan berikut dibahas lebih rinci sebagai sorotan yang mewakili risiko utama dari seluruh proses scanning.

### Temuan #1: CSP Header Not Set

**Content Security Policy (CSP)** adalah HTTP response header yang membatasi sumber resource yang boleh dimuat browser. ZAP menunjukkan 573 respons tanpa CSP, sehingga dampak XSS dan resource injection menjadi lebih besar.

**Impact:** Script tidak terpercaya lebih mudah dieksekusi ketika ada celah injeksi di sisi aplikasi atau konten.

**Remediation:** Tambahkan kebijakan CSP yang ketat pada level web server atau aplikasi.

---

### Temuan #2: Directory Browsing Enabled

**Directory indexing** masih aktif pada puluhan direktori seperti `/cache/`, `/lib/`, dan `/templates/`. Kondisi ini memudahkan attacker melakukan reconnaissance terhadap struktur aplikasi.

**Impact:** Penyerang lebih mudah menemukan file, plugin, dan area sensitif yang bisa dipakai untuk pivot serangan.

**Remediation:** Nonaktifkan listing direktori dengan `Options -Indexes` dan pastikan direktori non-public tidak dapat diakses langsung.

---

### Temuan #3: Assert() dengan Parameter Dynamic

**PHPCS** mendeteksi banyak penggunaan `assert()` dengan parameter dinamis. Walaupun tidak semua otomatis exploitable, pola ini tetap berbahaya karena `assert()` dapat berperan sebagai sink eksekusi kode.

**Impact:** Jika aliran datanya bisa dipengaruhi attacker, risiko paling buruk adalah remote code execution.

**Remediation:** Hapus penggunaan `assert()` pada jalur produksi dan ganti dengan validation atau exception handling yang eksplisit.

---

### Temuan #4: Apache 2.4.58 Multiple CVEs

**Nmap** memetakan service HTTP ke Apache 2.4.58 dan mengaitkannya dengan beberapa CVE berdampak tinggi. Ini memperlihatkan bahwa hardening aplikasi perlu dibarengi patching host dan middleware.

**Impact:** Attack surface tidak hanya berada di kode aplikasi, tetapi juga di web server yang melayani aplikasi.

**Remediation:** Upgrade Apache ke versi yang sudah menerima patch keamanan terbaru.

---

### Temuan #5: Eval Injection Pattern

**Semgrep custom rules** menemukan penggunaan `eval()` dinamis pada `Installer.inc.php`. Ini adalah pola klasik code injection yang tidak seharusnya muncul pada codebase produksi.

**Impact:** Nilai yang dapat dimanipulasi attacker berpotensi berubah menjadi kode PHP yang dieksekusi.

**Remediation:** Hindari `eval()` sepenuhnya; gunakan parser, mapping, atau dispatch yang eksplisit.

---

### Temuan #6: Stored XSS via `customHeaders`

**Manual review** menunjukkan bahwa `customHeaders` diteruskan ke output HTML tanpa sanitasi. Berbeda dengan SAST generik, temuan ini muncul dari pemahaman konteks template dan jalur render OJS.

**Impact:** Payload berbahaya dapat tersimpan dan dieksekusi ketika halaman dimuat oleh pengguna lain.

**Remediation:** Sanitasi `customHeaders` sebelum dirender dan terapkan escaping yang sesuai konteks output.

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

Dari scanning DAST, SAST, dan manual review terhadap OJS, dikonsolidasikan **20 temuan raw terpilih** dengan distribusi:

- **Critical:** 5 (`assert()` dinamis, `delete()` dinamis, `eval()` dinamis, stored XSS, `unserialize()` tanpa filter)
- **High:** 5 (OpenSSH usang, Apache usang, `readfile()` dinamis, variable include/require, upload tanpa validasi)
- **Medium:** 9 (CSP, directory browsing, clickjacking, jQuery UI usang, error disclosure, HSTS/X-Content-Type-Options hilang, CSRF indikatif, `phpinfo()`)
- **Low:** 1 (`Referrer-Policy` belum diterapkan)

### Prioritas Perbaikan

1. **Immediate (Week 1):**

   - Patch Apache dan OpenSSH ke versi yang aman
   - Disable `assert()` dan hilangkan pola `eval()` pada codebase
   - Add security headers (CSP, HSTS, X-Frame-Options atau `frame-ancestors`, X-Content-Type-Options)
   - Disable directory indexing di Apache
   - Sanitasi output `customHeaders` dan audit area render HTML lain
2. **Short-term (Week 2-4):**

   - Update jQuery UI v1.12.1 ke v1.13.3+
   - Audit dan remediate file operation vulnerabilities (`readfile()`, `delete()`, include/require dinamis)
   - Tambahkan whitelist MIME/ekstensi pada alur upload inti
   - Custom error pages (hide stack traces)
3. **Medium-term (Month 2-3):**

   - Implement comprehensive input validation/sanitization
   - Review dan harden authorization logic
   - Review proteksi CSRF pada form publik dan area autentikasi
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
**Dibuat Oleh:** DevSecOps Team
**Status:** FINAL
