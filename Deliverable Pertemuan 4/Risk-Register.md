// delete later
// Risk Register lengkap (semua temuan dari Pertemuan 3)


### 5.2 Risk Register OJS — Template Terisi

| ID      | Kerentanan                               | OWASP | CVSS Score | Rating   | Likelihood    | Business Impact               | Risk       | Prioritas |
| ------- | ---------------------------------------- | ----- | ---------- | -------- | ------------- | ----------------------------- | ---------- | --------- |
| VUL-001 | Apache RCE (CVE-2024-38476)              | A06   | 9.8        | Critical | 4 (High)      | 5 (Kritis — Kompromi Server)  | **Critical** | 1         |
| VUL-003 | RCE via `assert()`                       | A03   | 8.8        | High     | 4 (High)      | 5 (Kritis — Kompromi Aplikasi)| **Critical** | 2         |
| VUL-002 | OpenSSH RegreSSHion (CVE-2024-6387)      | A06   | 8.1        | High     | 2 (Low)       | 5 (Kritis — Root Access)      | **High** | 3         |
| VUL-004 | Arbitrary File Deletion via `delete()`   | A01   | 8.1        | High     | 4 (High)      | 4 (Tinggi — Data Loss/DoS)    | **High** | 4         |
| VUL-005 | Arbitrary File Read via `readfile()`     | A01   | 6.5        | Medium   | 4 (High)      | 4 (Tinggi — Credential Leak)  | **High** | 5         |
| VUL-006 | Path Traversal via `basename()`          | A01   | 6.5        | Medium   | 4 (High)      | 4 (Tinggi — Bypass akses)     | **High** | 6         |
| VUL-007 | Callback Injection via `array_map`       | A03   | 6.5        | Medium   | 3 (Medium)    | 4 (Tinggi — Code execution)   | **High** | 7         |
| VUL-008 | Vulnerable JS Library (jQuery UI 1.12.1) | A06   | 6.1        | Medium   | 3 (Medium)    | 3 (Sedang — Klien terdampak)  | **Medium** | 8         |
| VUL-009 | Directory Browsing pada direktori kritis | A05   | 5.3        | Medium   | 5 (Very High) | 2 (Rendah — Info Leak)        | **Medium** | 9         |
| VUL-010 | Missing Security Headers                 | A05   | 4.8        | Medium   | 5 (Very High) | 1 (Minimal — Info)            | **Low** | 10        |
