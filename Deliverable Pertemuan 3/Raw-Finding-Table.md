# Tabel Temuan Raw Scanning

Dokumen ini berisi 20 temuan raw terpilih yang sudah dikonsolidasikan dari seluruh proses scanning. Temuan yang sama antar-tool digabungkan dan hanya dipertahankan pada sumber bukti yang paling kuat.

## Bagian 1 - DAST (OWASP ZAP)

## Temuan #1

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Content Security Policy (CSP) Header Not Set |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | OWASP ZAP 2.16.1 |
| **URL / File** | http://10.34.100.180 (multiple endpoints) |
| **Parameter / Baris Kode** | HTTP Response Headers |
| **Method** | GET |
| **Payload** | N/A |
| **Response / Bukti** | Alert `Content Security Policy (CSP) Header Not Set` ditemukan pada 573 respons. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output ZAP mencatat `Content Security Policy (CSP) Header Not Set` dengan `count: 573`.

### Catatan

Header CSP belum diterapkan sehingga browser tidak memiliki pembatasan sumber script, style, dan resource lain. Kondisi ini memperbesar dampak XSS dan injeksi resource pihak ketiga.

---

## Temuan #2

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Directory Browsing Enabled |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | OWASP ZAP 2.16.1 |
| **URL / File** | http://10.34.100.180/cache/, /lib/, /plugins/, /templates/ |
| **Parameter / Baris Kode** | Directory paths |
| **Method** | GET |
| **Payload** | GET `/cache/`, GET `/lib/`, GET `/templates/` |
| **Response / Bukti** | Alert `Directory Browsing` muncul pada 88 direktori dengan respons 200 dan listing direktori. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output ZAP mencatat `Directory Browsing` dengan `count: 88`.

### Catatan

Listing direktori yang terbuka memudahkan penyerang melakukan reconnaissance, melihat struktur aplikasi, plugin, file cache, dan lokasi file yang sensitif.

---

## Temuan #3

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Missing Anti-clickjacking Header |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | OWASP ZAP 2.16.1 |
| **URL / File** | http://10.34.100.180 (multiple endpoints) |
| **Parameter / Baris Kode** | HTTP header `x-frame-options` |
| **Method** | GET, POST |
| **Payload** | N/A |
| **Response / Bukti** | Alert `Missing Anti-clickjacking Header` terdeteksi pada 122 respons. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output ZAP mencatat `Missing Anti-clickjacking Header` dengan `count: 122`.

### Catatan

Halaman dapat di-embed dalam iframe dari domain lain karena proteksi clickjacking belum diterapkan.

---

## Temuan #4

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Vulnerable JavaScript Library (jQuery UI v1.12.1) |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | OWASP ZAP 2.16.1 |
| **URL / File** | http://10.34.100.180/lib/pkp/lib/vendor/components/jqueryui/jquery-ui.min.js?v=3.3.0.8 |
| **Parameter / Baris Kode** | Versi library |
| **Method** | GET |
| **Payload** | N/A |
| **Response / Bukti** | Alert `Vulnerable JS Library` menunjukkan jQuery UI v1.12.1 digunakan pada aplikasi. |
| **OWASP Category** | A06:2021 - Vulnerable and Outdated Components |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output ZAP mencatat `Vulnerable JS Library` dengan `count: 1` pada file jQuery UI.

### Catatan

Versi jQuery UI tersebut sudah lama dan diketahui memiliki beberapa CVE publik. Risiko meningkat jika komponen ini digunakan bersama input yang dapat dikendalikan pengguna.

---

## Temuan #5

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Application Error Disclosure |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | OWASP ZAP 2.16.1 |
| **URL / File** | http://10.34.100.180/cache/t_compile/ (multiple compiled templates) |
| **Parameter / Baris Kode** | HTTP response error pages |
| **Method** | GET |
| **Payload** | GET `/cache/t_compile/[compiled-template].php` |
| **Response / Bukti** | Alert `Application Error Disclosure` muncul berulang pada file compiled template dan endpoint lain. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output ZAP mencatat `Application Error Disclosure` dengan dua grup alert, termasuk grup medium `count: 63`.

### Catatan

Pesan error yang terekspos membantu attacker memetakan struktur aplikasi, path internal, dan area yang gagal menangani error secara aman.

---

## Bagian 2 - DAST (Nikto)

## Temuan #6

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Missing Strict-Transport-Security Header |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nikto v2.6.0 |
| **URL / File** | http://10.34.100.180/ |
| **Parameter / Baris Kode** | HTTP header `strict-transport-security` |
| **Method** | GET |
| **Payload** | N/A |
| **Response / Bukti** | Nikto melaporkan `Suggested security header missing: strict-transport-security`. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output Nikto: `/: Suggested security header missing: strict-transport-security`.

### Catatan

Tanpa HSTS, browser tidak dipaksa menggunakan HTTPS pada koneksi berikutnya. Ini membuka peluang downgrade attack dan MITM pada jalur yang tidak terenkripsi.

---

## Temuan #7

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Missing X-Content-Type-Options Header |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nikto v2.6.0 |
| **URL / File** | http://10.34.100.180/ |
| **Parameter / Baris Kode** | HTTP header `x-content-type-options` |
| **Method** | GET |
| **Payload** | N/A |
| **Response / Bukti** | Nikto melaporkan `Suggested security header missing: x-content-type-options`. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output Nikto: `/: Suggested security header missing: x-content-type-options`.

### Catatan

Header `X-Content-Type-Options: nosniff` belum ada, sehingga browser dapat melakukan MIME sniffing yang meningkatkan risiko interpretasi konten yang tidak semestinya.

---

## Temuan #8

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Missing Referrer-Policy Header |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nikto v2.6.0 |
| **URL / File** | http://10.34.100.180/ |
| **Parameter / Baris Kode** | HTTP header `referrer-policy` |
| **Method** | GET |
| **Payload** | N/A |
| **Response / Bukti** | Nikto melaporkan `Suggested security header missing: referrer-policy`. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Low |

### Screenshot / Bukti

Raw output Nikto: `/: Suggested security header missing: referrer-policy`.

### Catatan

Ketiadaan `Referrer-Policy` dapat menyebabkan kebocoran URL asal atau parameter sensitif ke situs lain melalui header `Referer`.

---

## Bagian 3 - DAST (Nmap)

## Temuan #9

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | OpenSSH 9.6p1 Multiple Known Vulnerabilities |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nmap 7.98 (`--script vuln`) |
| **URL / File** | 10.34.100.180:22 |
| **Parameter / Baris Kode** | Service version |
| **Method** | N/A |
| **Payload** | N/A |
| **Response / Bukti** | Nmap mengidentifikasi OpenSSH 9.6p1 Ubuntu dan memetakan beberapa CVE, termasuk `CVE-2024-6387`. |
| **OWASP Category** | A06:2021 - Vulnerable and Outdated Components |
| **Severity (Raw)** | High |

### Screenshot / Bukti

Raw output Nmap pada port 22 menampilkan `OpenSSH 9.6p1 Ubuntu 3ubuntu13.15` dan daftar CVE seperti `CVE-2024-6387`, `CVE-2025-26465`, `CVE-2025-26466`.

### Catatan

Versi OpenSSH yang terdeteksi memiliki beberapa paparan kerentanan publik. Karena ini berada di perimeter host, dampaknya relevan untuk keamanan server secara keseluruhan.

---

## Temuan #10

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Apache HTTP Server 2.4.58 Multiple Known Vulnerabilities |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nmap 7.98 (`--script vuln`) |
| **URL / File** | 10.34.100.180:80 |
| **Parameter / Baris Kode** | Service version |
| **Method** | N/A |
| **Payload** | N/A |
| **Response / Bukti** | Nmap mengidentifikasi Apache 2.4.58 dan memetakan banyak CVE dengan skor tinggi, termasuk `CVE-2024-38476` dan `CVE-2024-38474`. |
| **OWASP Category** | A06:2021 - Vulnerable and Outdated Components |
| **Severity (Raw)** | High |

### Screenshot / Bukti

Raw output Nmap pada port 80 menampilkan `Apache httpd 2.4.58 ((Ubuntu))` dengan daftar CVE seperti `CVE-2024-38476`, `CVE-2024-38474`, `CVE-2024-38475`, `CVE-2025-23048`.

### Catatan

Komponen server web berada di attack surface utama aplikasi. Versi yang usang meningkatkan risiko eksploitasi terhadap layanan HTTP.

---

## Temuan #11

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Possible CSRF on Search Form |
| **Tool Penemu** | DAST |
| **Tool Spesifik** | Nmap 7.98 (`http-csrf`) |
| **URL / File** | http://10.34.100.180/index.php/halo/search |
| **Parameter / Baris Kode** | Form `query` |
| **Method** | POST |
| **Payload** | Form action ke `http://10.34.100.180/index.php/halo/search/index` |
| **Response / Bukti** | Script `http-csrf` Nmap menandai form pencarian sebagai `possible CSRF vulnerabilities`. |
| **OWASP Category** | A01:2021 - Broken Access Control |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output Nmap: `Found the following possible CSRF vulnerabilities` pada path `/index.php/halo/search` dengan `Form id: query`.

### Catatan

Temuan ini masih berupa indikasi dan perlu verifikasi manual tambahan, tetapi tetap layak dicatat karena form diproses tanpa bukti kontrol CSRF dari output Nmap.

---

## Bagian 4 - DAST (SQLMap)

SQLMap sudah dijalankan pada endpoint login, search, dan hasil crawl. Tidak ada parameter yang terverifikasi injectable, sehingga tidak dimasukkan sebagai temuan pada tabel raw final.

---

## Bagian 5 - SAST (phpcs-security-audit)

## Temuan #12

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Assert Eval Function with Dynamic Parameter |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | phpcs-security-audit |
| **URL / File** | /pages/submission/SubmissionHandler.inc.php |
| **Parameter / Baris Kode** | Line 56 |
| **Method** | N/A |
| **Payload** | `assert($dynamicValue)` |
| **Response / Bukti** | PHPCS memberi warning `Assert eval function assert() detected with dynamic parameter`. |
| **OWASP Category** | A03:2021 - Injection |
| **Severity (Raw)** | Critical |

### Screenshot / Bukti

Raw output PHPCS pada `SubmissionHandler.inc.php` line 56 menandai penggunaan `assert()` dengan parameter dinamis; pola serupa juga muncul di banyak file lain.

### Catatan

Pemakaian `assert()` pada nilai dinamis dapat berujung code execution jika aliran datanya berasal dari input yang tidak dipercaya.

---

## Temuan #13

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Dynamic File Deletion Primitive |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | phpcs-security-audit |
| **URL / File** | /lib/pkp/controllers/api/file/PKPManageFileApiHandler.inc.php |
| **Parameter / Baris Kode** | Line 59 |
| **Method** | N/A |
| **Payload** | `delete($dynamicPath)` |
| **Response / Bukti** | PHPCS memberi warning `Filesystem function delete() detected with dynamic parameter`. |
| **OWASP Category** | A01:2021 - Broken Access Control |
| **Severity (Raw)** | Critical |

### Screenshot / Bukti

Raw output PHPCS pada `PKPManageFileApiHandler.inc.php` line 59 menandai operasi `delete()` dengan parameter dinamis.

### Catatan

Primitive penghapusan file yang menggunakan path dinamis perlu divalidasi sangat ketat karena dapat membuka peluang arbitrary file deletion.

---

## Temuan #14

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Dynamic File Read via `readfile()` |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | phpcs-security-audit |
| **URL / File** | /lib/pkp/controllers/page/PageHandler.inc.php |
| **Parameter / Baris Kode** | Line 155 |
| **Method** | N/A |
| **Payload** | `readfile($dynamicPath)` |
| **Response / Bukti** | PHPCS memberi warning `Filesystem function readfile() detected with dynamic parameter`. |
| **OWASP Category** | A01:2021 - Broken Access Control |
| **Severity (Raw)** | High |

### Screenshot / Bukti

Raw output PHPCS pada `PageHandler.inc.php` line 155 menandai pemanggilan `readfile()` dengan parameter dinamis.

### Catatan

Jika jalur file dipengaruhi input user, pola ini dapat mengarah ke arbitrary file read atau information disclosure.

---

## Bagian 6 - SAST (Semgrep)

## Temuan #15

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Variable-based Include/Require |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | Semgrep Custom Rules |
| **URL / File** | /var/www/html/ojs/index.php |
| **Parameter / Baris Kode** | Line 65 |
| **Method** | N/A |
| **Payload** | `include/require $variable` |
| **Response / Bukti** | Semgrep rule `php-file-include-variable` menemukan 104 instance include/require berbasis variabel. |
| **OWASP Category** | A03:2021 - Injection |
| **Severity (Raw)** | High |

### Screenshot / Bukti

Raw output Semgrep custom menunjukkan `php-file-include-variable` dengan 104 hasil; salah satu instance ada di `/var/www/html/ojs/index.php` line 65.

### Catatan

Pola include/require dengan path yang tidak di-whitelist dapat berkembang menjadi local file inclusion, remote file inclusion, atau path traversal tergantung sumber variabelnya.

---

## Temuan #16

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Eval Injection Pattern |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | Semgrep Custom Rules |
| **URL / File** | /var/www/html/ojs/lib/pkp/classes/install/Installer.inc.php |
| **Parameter / Baris Kode** | Line 383 |
| **Method** | N/A |
| **Payload** | `eval($dynamicCode)` |
| **Response / Bukti** | Semgrep rule `php-eval-injection` menemukan 3 instance penggunaan `eval()` berbasis variabel. |
| **OWASP Category** | A03:2021 - Injection |
| **Severity (Raw)** | Critical |

### Screenshot / Bukti

Raw output Semgrep custom menunjukkan `php-eval-injection` pada `Installer.inc.php` line 383.

### Catatan

`eval()` pada data dinamis adalah pola berisiko sangat tinggi karena dapat berubah menjadi code execution jika nilai dapat dikendalikan atau dimanipulasi.

---

## Temuan #17

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | phpinfo Exposure |
| **Tool Penemu** | SAST |
| **Tool Spesifik** | Semgrep (`p/php`, `p/owasp-top-ten`) |
| **URL / File** | /var/www/html/ojs/lib/pkp/pages/admin/AdminHandler.inc.php |
| **Parameter / Baris Kode** | Line 375 |
| **Method** | N/A |
| **Payload** | `phpinfo()` |
| **Response / Bukti** | Ruleset Semgrep `p/php` dan `p/owasp-top-ten` sama-sama menandai penggunaan `phpinfo()`. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | Medium |

### Screenshot / Bukti

Raw output Semgrep pada kedua ruleset memuat rule `php.lang.security.phpinfo-use.phpinfo-use` di `AdminHandler.inc.php` line 375.

### Catatan

Pemanggilan `phpinfo()` berisiko membocorkan konfigurasi PHP, ekstensi aktif, environment, dan detail server bila dapat diakses di lingkungan produksi.

---

## Bagian 7 - Manual Code Review

## Temuan #18

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Stored XSS via `customHeaders` |
| **Tool Penemu** | Manual |
| **Tool Spesifik** | Manual Code Review |
| **URL / File** | /lib/pkp/classes/template/PKPTemplateManager.inc.php |
| **Parameter / Baris Kode** | Lines 223-227 |
| **Method** | N/A |
| **Payload** | `<script>alert(1)</script>` |
| **Response / Bukti** | Nilai `customHeaders` dari database diteruskan ke `addHeader()` tanpa sanitasi atau escaping. |
| **OWASP Category** | A03:2021 - Injection |
| **Severity (Raw)** | Critical |

### Screenshot / Bukti

Manual review mencatat:
`$customHeaders = $currentContext->getLocalizedData('customHeaders');`
`$this->addHeader('customHeaders', $customHeaders);`

### Catatan

Jika field ini dapat diubah oleh admin jurnal yang berbahaya atau akun yang dikompromi, payload JavaScript dapat tersimpan dan dieksekusi di browser pengunjung.

---

## Temuan #19

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | File Upload Without MIME/Extension Validation |
| **Tool Penemu** | Manual |
| **Tool Spesifik** | Manual Code Review |
| **URL / File** | /lib/pkp/classes/file/FileManager.inc.php |
| **Parameter / Baris Kode** | Lines 136-146 |
| **Method** | N/A |
| **Payload** | Upload file berbahaya dengan ekstensi atau MIME yang dipalsukan |
| **Response / Bukti** | Fungsi `uploadFile()` memindahkan file upload tanpa validasi tipe, ekstensi, atau ukuran pada level fungsi inti. |
| **OWASP Category** | A05:2021 - Security Misconfiguration |
| **Severity (Raw)** | High |

### Screenshot / Bukti

Manual review mencatat bahwa `move_uploaded_file($_FILES[$fileName]['tmp_name'], $destFileName)` dipanggil tanpa whitelist MIME atau ekstensi di fungsi `uploadFile()`.

### Catatan

Validasi sepenuhnya diserahkan ke caller. Bila ada caller yang lupa memvalidasi, pola ini dapat menjadi jalur unrestricted file upload.

---

## Temuan #20

| Field | Nilai |
|---|---|
| **Nama Kerentanan** | Unsafe `unserialize()` on Persisted Data |
| **Tool Penemu** | Manual |
| **Tool Spesifik** | Manual Code Review |
| **URL / File** | /lib/pkp/classes/db/DAO.inc.php |
| **Parameter / Baris Kode** | Lines 245-253 |
| **Method** | N/A |
| **Payload** | Serialized object payload |
| **Response / Bukti** | Cabang fallback memanggil `unserialize($value)` tanpa `allowed_classes`. |
| **OWASP Category** | A08:2021 - Software and Data Integrity Failures |
| **Severity (Raw)** | Critical |

### Screenshot / Bukti

Manual review mencatat pola:
`$value = unserialize($value);`
setelah `json_decode()` gagal.

### Catatan

Apabila data tersimpan dapat dimanipulasi, pola ini dapat berkembang menjadi PHP object injection dan berpotensi memicu gadget chain yang berbahaya.

---

## Summary Findings

**Total Temuan Terpilih: 20**

| Tool / Metode | Medium | High | Critical | Low | Total |
| --- | --- | --- | --- | --- | --- |
| OWASP ZAP | 5 | 0 | 0 | 0 | 5 |
| Nikto | 2 | 0 | 0 | 1 | 3 |
| Nmap | 1 | 2 | 0 | 0 | 3 |
| SQLMap | 0 | 0 | 0 | 0 | 0 |
| phpcs-security-audit | 0 | 1 | 2 | 0 | 3 |
| Semgrep | 1 | 1 | 1 | 0 | 3 |
| Manual Review | 0 | 1 | 2 | 0 | 3 |
| **TOTAL** | **9** | **5** | **5** | **1** | **20** |

**Ringkasan Prioritas:**

1. Komponen server yang usang pada OpenSSH dan Apache perlu diprioritaskan karena berada di attack surface utama.
2. Pola code execution dan file access berisiko tinggi muncul pada hasil PHPCS, Semgrep, dan manual review.
3. Hardening HTTP headers dan penutupan directory browsing masih menjadi pekerjaan penting pada sisi runtime.
