# Risk Register Final - Open Journal Systems (OJS)

Dokumen ini merupakan risk register final untuk deliverable pertemuan 5. Isinya dikonsolidasikan dari [Risk-Register pertemuan 4](../Deliverable%20Pertemuan%204/Risk-Register.md) dan prioritas risiko pada visual risk register final.

## Informasi Dokumen

| Field              | Nilai                                                                                                               |
| :----------------- | :------------------------------------------------------------------------------------------------------------------ |
| Nama Tim           | Kelompok 3                                                                                                          |
| Mata Kuliah        | DevSecOps                                                                                                           |
| Objek Assessment   | Open Journal Systems (OJS) 3.3.0-8                                                                                  |
| Tanggal Finalisasi | 09/04/2026                                                                                                          |
| Status             | Final                                                                                                               |
| Catatan            | Seluruh temuan pada register ini berstatus open sampai mitigasi diterapkan dan patch verification selesai dilakukan |

## Ringkasan Risiko

| Tingkat Risiko | Jumlah Temuan | ID                                                   |
| :------------- | :------------ | :--------------------------------------------------- |
| Critical       | 5             | VUL-001, VUL-003, VUL-004, VUL-005, VUL-006          |
| High           | 5             | VUL-007, VUL-002, VUL-008, VUL-009, VUL-010          |
| Medium         | 6             | VUL-011, VUL-012, VUL-013, VUL-014, VUL-018, VUL-015 |
| Low            | 4             | VUL-016, VUL-017, VUL-019, VUL-020                   |

## Risk Register

| ID          | Kerentanan                          | OWASP | CVSS | Rating   | Likelihood | Impact | Risk         | Prioritas | Mitigasi Utama                                                                                          | Target Waktu | Status                               |
| :---------- | :---------------------------------- | :---- | :--- | :------- | :--------- | :----- | :----------- | :-------- | :------------------------------------------------------------------------------------------------------ | :----------- | :----------------------------------- |
| **VUL-001** | Apache HTTP RCE (CVE-2024-38476)    | A06   | 9.8  | Critical | 4          | 5      | **Critical** | 1         | Upgrade Apache HTTP Server ke versi yang sudah patched dan pasang WAF sebagai kontrol sementara         | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-003** | Eval Injection (Installer)          | A03   | 9.8  | Critical | 4          | 5      | **Critical** | 2         | Nonaktifkan atau hapus installer dari production dan refactor kode untuk menghilangkan `eval()`         | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-004** | Stored XSS via `customHeaders`      | A03   | 9.0  | Critical | 3          | 5      | **Critical** | 3         | Escape output secara konsisten, sanitasi input HTML, dan aktifkan Content-Security-Policy               | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-005** | Unsafe `unserialize()` (DAO)        | A08   | 9.0  | Critical | 3          | 5      | **Critical** | 4         | Ganti `unserialize()` dengan format aman seperti JSON atau whitelist kelas yang diizinkan               | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-006** | Assert() Dynamic Parameter          | A03   | 8.8  | High     | 4          | 5      | **Critical** | 5         | Hapus pemakaian `assert()` untuk input dinamis dan gunakan validasi eksplisit di sisi aplikasi          | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-007** | Dynamic File Deletion               | A01   | 8.1  | High     | 4          | 4      | **High**     | 6         | Terapkan whitelist path, validasi `realpath()`, dan batasi operasi hapus hanya pada direktori aman      | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-002** | OpenSSH RegreSSHion (CVE-2024-6387) | A06   | 8.1  | High     | 2          | 5      | **High**     | 7         | Upgrade OpenSSH ke versi patched dan batasi akses SSH dengan allowlist IP atau VPN                      | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-008** | Dynamic File Read (`readfile`)      | A01   | 7.5  | High     | 4          | 4      | **High**     | 8         | Validasi path file dengan whitelist direktori dan cegah akses ke file sensitif di luar upload directory | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-009** | Variable-based Include/Require      | A03   | 7.5  | High     | 3          | 4      | **High**     | 9         | Ganti include dinamis dengan mapping statis atau whitelist file yang diperbolehkan                      | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-010** | File Upload Without Validation      | A05   | 7.3  | High     | 2          | 5      | **High**     | 10        | Terapkan allowlist ekstensi dan MIME type, rename file upload, dan simpan file di luar web root         | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-011** | Vulnerable jQuery UI v1.12.1        | A06   | 6.1  | Medium   | 3          | 3      | **Medium**   | 11        | Upgrade jQuery UI ke versi yang didukung atau hapus komponen yang sudah tidak digunakan                 | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-012** | Directory Browsing Enabled          | A05   | 5.3  | Medium   | 5          | 2      | **Medium**   | 12        | Nonaktifkan directory listing dengan `Options -Indexes` atau konfigurasi setara di web server           | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-013** | CSP Header Not Set                  | A05   | 4.7  | Medium   | 5          | 2      | **Medium**   | 13        | Tambahkan header Content-Security-Policy yang membatasi sumber script, style, dan frame                 | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-014** | Missing Anti-clickjacking Header    | A05   | 4.7  | Medium   | 5          | 2      | **Medium**   | 14        | Tambahkan `X-Frame-Options` atau `frame-ancestors` pada CSP                                             | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-018** | phpinfo() Exposure                  | A05   | 4.3  | Medium   | 3          | 3      | **Medium**   | 15        | Hapus halaman `phpinfo()` dari production atau batasi akses hanya untuk administrator internal          | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-015** | Application Error Disclosure        | A05   | 4.3  | Medium   | 4          | 2      | **Medium**   | 16        | Matikan debug mode, sembunyikan stack trace, dan gunakan custom error page                              | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-016** | Missing HSTS Header                 | A05   | 4.3  | Medium   | 5          | 1      | **Low**      | 17        | Aktifkan `Strict-Transport-Security` setelah seluruh trafik HTTPS tervalidasi                           | 7-30 hari    | Open - patch verification diperlukan |
| **VUL-017** | Missing X-Content-Type-Options      | A05   | 4.3  | Medium   | 5          | 1      | **Low**      | 18        | Tambahkan header `X-Content-Type-Options: nosniff` pada konfigurasi web server                          | 0-7 hari     | Open - patch verification diperlukan |
| **VUL-019** | Possible CSRF on Search Form        | A01   | 4.3  | Medium   | 3          | 2      | **Low**      | 19        | Verifikasi kebutuhan CSRF token pada endpoint terkait dan perkuat cookie dengan `SameSite`              | 30-90 hari   | Open - patch verification diperlukan |
| **VUL-020** | Missing Referrer-Policy Header      | A05   | 3.2  | Low      | 5          | 1      | **Low**      | 20        | Tambahkan header `Referrer-Policy: strict-origin-when-cross-origin`                                     | 7-30 hari    | Open - patch verification diperlukan |

## Prioritas Mitigasi

| Kelompok | Fokus                                                        | ID Temuan                                                                       | Target     |
| :------- | :----------------------------------------------------------- | :------------------------------------------------------------------------------ | :--------- |
| P1       | Hotfix untuk mencegah RCE dan code execution                 | VUL-001, VUL-003, VUL-004, VUL-005, VUL-006                                     | 0-7 hari   |
| P2       | Perbaikan kontrol file operation dan komponen sistem         | VUL-002, VUL-007, VUL-008, VUL-009, VUL-010                                     | 7-30 hari  |
| P3       | Hardening konfigurasi server dan browser security controls   | VUL-011, VUL-012, VUL-013, VUL-014, VUL-015, VUL-016, VUL-017, VUL-018, VUL-020 | 0-30 hari  |
| P4       | Penyempurnaan kontrol preventif dan verifikasi residual risk | VUL-019                                                                         | 30-90 hari |
