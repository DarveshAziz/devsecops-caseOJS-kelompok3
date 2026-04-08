# Pemetaan Temuan ke OWASP Top 10 (2021)

Dokumen ini memetakan 20 temuan raw dari proses scanning dan manual review ke dalam kategori risiko OWASP Top 10 2021.

| ID | Nama Kerentanan | OWASP Category | CWE | CVE / Note |
| :--- | :--- | :--- | :--- | :--- |
| **VUL-001** | Apache HTTP Server 2.4.58 Multiple CVEs | A06:2021 | CWE-1395 | CVE-2024-38476, dll |
| **VUL-002** | OpenSSH 9.6p1 RegreSSHion (RCE) | A06:2021 | CWE-362 | CVE-2024-6387 |
| **VUL-003** | Eval Injection Pattern (Installer) | A03:2021 | CWE-94 | Critical (SAST) |
| **VUL-004** | Stored XSS via `customHeaders` | A03:2021 | CWE-79 | Critical (Manual) |
| **VUL-005** | Unsafe `unserialize()` on Persisted Data | A08:2021 | CWE-502 | Critical (Manual) |
| **VUL-006** | Assert() dengan Parameter Dinamis | A03:2021 | CWE-94 | Critical (SAST) |
| **VUL-007** | Dynamic File Deletion Primitive | A01:2021 | CWE-73 | Critical (SAST) |
| **VUL-008** | Dynamic File Read via `readfile()` | A01:2021 | CWE-73 | High (SAST) |
| **VUL-009** | Variable-based Include/Require | A03:2021 | CWE-98 | High (SAST) |
| **VUL-010** | File Upload Without Validation | A05:2021 | CWE-434 | High (Manual) |
| **VUL-011** | Vulnerable jQuery UI v1.12.1 | A06:2021 | CWE-1395 | CVE-2021-41184 |
| **VUL-012** | Directory Browsing Enabled | A05:2021 | CWE-548 | 88 directories |
| **VUL-013** | Content Security Policy (CSP) Not Set | A05:2021 | CWE-693 | 573 instances |
| **VUL-014** | Missing Anti-clickjacking Header | A05:2021 | CWE-693 | 122 instances |
| **VUL-015** | Application Error Disclosure | A05:2021 | CWE-209 | Info Leak |
| **VUL-016** | Missing Strict-Transport-Security (HSTS) | A05:2021 | CWE-319 | Nikto |
| **VUL-017** | Missing X-Content-Type-Options | A05:2021 | CWE-693 | Nikto |
| **VUL-018** | phpinfo() Exposure (Admin) | A05:2021 | CWE-200 | Semgrep |
| **VUL-019** | Possible CSRF on Search Form | A01:2021 | CWE-352 | Nmap Indication |
| **VUL-020** | Missing Referrer-Policy Header | A05:2021 | CWE-693 | Low Severity |

---

### Ringkasan Distribusi Kategori

1. **A01:2021 - Broken Access Control:** 3 temuan (File Deletion, File Read, CSRF Indication)
2. **A03:2021 - Injection:** 5 temuan (Eval, XSS, Assert, Include, Callback)
3. **A05:2021 - Security Misconfiguration:** 8 temuan (Directory Browsing, CSP, Clickjacking, Error Disclosure, HSTS, Sniffing, phpinfo, Referrer)
4. **A06:2021 - Vulnerable and Outdated Components:** 3 temuan (Apache, OpenSSH, jQuery UI)
5. **A08:2021 - Software and Data Integrity Failures:** 1 temuan (Unsafe Unserialize)
