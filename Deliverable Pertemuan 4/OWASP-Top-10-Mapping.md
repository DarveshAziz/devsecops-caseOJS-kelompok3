// Pemetaan temuan ke OWASP Top 10 (Revisi aja kalau ada yang salah :v)

Dari 22 temuan, berikut adalah 10 kerentanan dengan prioritas tertinggi yang dipetakan ke dalam kategori OWASP Top 10:

| ID Temuan | Deskripsi | OWASP | CWE | CVE |
| :--- | :--- | :--- | :--- | :--- |
| **VUL-001** | Apache 2.4.58 Multiple RCE Vulnerabilities | A06 | CWE-119 | CVE-2024-38476, dll |
| **VUL-002** | OpenSSH 9.6p1 RegreSSHion (RCE) | A06 | CWE-362 | CVE-2024-6387 |
| **VUL-003** | RCE Risk via `assert()` dengan parameter dinamis | A03 | CWE-94 | - |
| **VUL-004** | Arbitrary File Deletion via `delete()` | A01 | CWE-73 | - |
| **VUL-005** | Arbitrary File Read via `readfile()` | A01 | CWE-73 | - |
| **VUL-006** | Path Traversal via `basename()` | A01 | CWE-22 | - |
| **VUL-007** | Callback Injection via `array_map` & `array_udiff` | A03 | CWE-94 | - |
| **VUL-008** | Vulnerable JS Library (jQuery UI 1.12.1) | A06 | CWE-1395 | CVE-2021-41184 |
| **VUL-009** | Directory Browsing pada direktori kritis | A05 | CWE-548 | - |
| **VUL-010** | Missing Security Headers (CSP, HSTS, X-Frame, dll) | A05 | CWE-693 | - |
