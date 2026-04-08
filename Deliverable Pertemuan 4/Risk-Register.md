# Risk Register - Open Journal Systems (OJS)

Dokumen ini berisi daftar risiko keamanan yang teridentifikasi pada OJS versi 3.3.0-8, dihitung berdasarkan metodologi CVSS v3.1 dan OWASP Risk Rating.

| ID | Kerentanan | OWASP | CVSS | Rating | Likelihood | Impact | Risk | Prioritas |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **VUL-001** | Apache HTTP RCE (CVE-2024-38476) | A06 | 9.8 | Critical | 4 | 5 | **Critical** | 1 |
| **VUL-003** | Eval Injection (Installer) | A03 | 9.8 | Critical | 4 | 5 | **Critical** | 2 |
| **VUL-004** | Stored XSS via `customHeaders` | A03 | 9.0 | Critical | 3 | 5 | **Critical** | 3 |
| **VUL-005** | Unsafe `unserialize()` (DAO) | A08 | 9.0 | Critical | 3 | 5 | **Critical** | 4 |
| **VUL-006** | Assert() Dynamic Parameter | A03 | 8.8 | High | 4 | 5 | **Critical** | 5 |
| **VUL-007** | Dynamic File Deletion | A01 | 8.1 | High | 4 | 4 | **High** | 6 |
| **VUL-002** | OpenSSH RegreSSHion (CVE-2024-6387) | A06 | 8.1 | High | 2 | 5 | **High** | 7 |
| **VUL-008** | Dynamic File Read (`readfile`) | A01 | 7.5 | High | 4 | 4 | **High** | 8 |
| **VUL-009** | Variable-based Include/Require | A03 | 7.5 | High | 3 | 4 | **High** | 9 |
| **VUL-010** | File Upload Without Validation | A05 | 7.3 | High | 2 | 5 | **High** | 10 |
| **VUL-011** | Vulnerable jQuery UI v1.12.1 | A06 | 6.1 | Medium | 3 | 3 | **Medium** | 11 |
| **VUL-012** | Directory Browsing Enabled | A05 | 5.3 | Medium | 5 | 2 | **Medium** | 12 |
| **VUL-013** | CSP Header Not Set | A05 | 4.7 | Medium | 5 | 2 | **Medium** | 13 |
| **VUL-014** | Missing Anti-clickjacking Header | A05 | 4.7 | Medium | 5 | 2 | **Medium** | 14 |
| **VUL-018** | phpinfo() Exposure | A05 | 4.3 | Medium | 3 | 3 | **Medium** | 15 |
| **VUL-015** | Application Error Disclosure | A05 | 4.3 | Medium | 4 | 2 | **Medium** | 16 |
| **VUL-016** | Missing HSTS Header | A05 | 4.3 | Medium | 5 | 1 | **Low** | 17 |
| **VUL-017** | Missing X-Content-Type-Options | A05 | 4.3 | Medium | 5 | 1 | **Low** | 18 |
| **VUL-019** | Possible CSRF on Search Form | A01 | 4.3 | Medium | 3 | 2 | **Low** | 19 |
| **VUL-020** | Missing Referrer-Policy Header | A05 | 3.2 | Low | 5 | 1 | **Low** | 20 |

---

### Metodologi Penilaian

1. **Likelihood (1-5):** Kemungkinan eksploitasi (1=Sangat Rendah, 5=Sangat Tinggi).
2. **Business Impact (1-5):** Dampak terhadap bisnis (1=Minimal, 5=Kritis/Kompromi Penuh).
3. **Risk Level:**
   - **Critical:** Likelihood/Impact sangat tinggi, butuh hotfix segera.
   - **High:** Dampak signifikan, perbaiki dalam 7 hari.
   - **Medium:** Dampak sedang, perbaiki dalam 30 hari.
   - **Low:** Dampak minimal, perbaiki dalam siklus rutin.
