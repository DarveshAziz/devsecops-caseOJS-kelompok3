// delete later (Revisi klo salah yak :v)
// Narasi analisis per kategori OWASP yang relevan

### A01 — Broken Access Control

**Temuan 1 — Arbitrary File Deletion via `delete()`:**
Fungsi manajemen file sistem terekspos ke parameter dinamis tanpa melalui proses validasi sanitasi direktori yang aman, memungkinkan Path Traversal.

**Bukti:**
```text
Sumber: Hasil SAST (phpcs-security-audit)
Lokasi: /lib/pkp/controllers/api/file/PKPManageFileApiHandler.inc.php (Line 59)
Peringatan: WARNING: Filesystem function delete() detected with dynamic parameter

Skenario Eksploitasi:
Input manipulasi path: delete("../../config.inc.php")
Dampak: Penghapusan file kritis sistem atau konfigurasi aplikasi.
```

**Temuan 2 — Arbitrary File Read via `readfile()`:**
Penggunaan `readfile()` dengan input dinamis memungkinkan penyerang membaca berkas di luar root direktori web.
**Bukti:**
```text
Sumber: Hasil SAST (phpcs-security-audit)
Lokasi: /lib/pkp/controllers/page/PageHandler.inc.php (Line 155)
Skenario Eksploitasi: GET parameter dengan nilai ../../../.env atau ../../config.inc.php
```

### A03 — Injection
**Temuan 1 — Eksekusi Kode (RCE) via `assert():`**
Injeksi pada lapisan eksekusi interpreter PHP karena penyertaan parameter tanpa penyaringan (unsanitized input) ke dalam fungsi eval-like.

**Bukti:**
```text
Sumber: Hasil SAST (phpcs-security-audit)
Lokasi: /lib/pkp/pages/submission/PKPSubmissionHandler.inc.php (Line 60)
Peringatan: WARNING: Assert eval function assert() detected with dynamic parameter
```

**Temuan 2 — Stored XSS Candidate pada Section Names:**
Aplikasi menyimpan payload HTML/JS jahat yang diinputkan pada nama section dan me-render-nya ke dalam data bootstrap frontend yang terautentikasi.

**Bukti:**
```text
Payload: <svg/onload=alert('XSS_SUCCESS')>
Lokasi: [http://10.34.100.180/index.php/halo/submissions](http://10.34.100.180/index.php/halo/submissions) (Bootstrap JSON Data)
Target Render: {"param":"sectionIds","value":4,"title":"<svg/onload=alert('XSS_SUCCESS')>"}
```


### A05 — Security Misconfiguration
**Temuan 1 — Directory Browsing / Indexing Terbuka:**
Konfigurasi Apache gagal menutup akses pembacaan indeks direktori, mengakibatkan bocornya file internal dan cache

**Bukti:**
```text
GET /cache/ HTTP/1.1
Host: 10.34.100.180

Response: HTTP/1.1 200 OK
(Menampilkan halaman "Index of /cache" berisi folder t_compile, HTML, URI, dll)
```

**Temuan 2 — Missing Security Headers**
Aplikasi berjalan di web tanpa mendeklarasikan header pertahanan seperti Content Security Policy (CSP), HSTS, dan X-Frame-Options.

**Bukti:**
```text
Sumber: Pemindaian Nikto & ZAP
Target: [http://10.34.100.180](http://10.34.100.180) (Semua Endpoint)
Hasil: 
- Suggested security header missing: strict-transport-security
- Suggested security header missing: content-security-policy
- Suggested security header missing: x-frame-options
```

### A06 — Vulnerable & Outdated Components
**Temuan — Web Server & SSH Usang (Multiple RCE):**
Infrastruktur (web server Apache dan modul SSH) tidak menggunakan pembaruan keamanan terbaru, meninggalkannya rentan terhadap CVE tingkat kritikal.

**Bukti:**
```text
Sumber: Pemindaian Nmap 7.98
Target: 10.34.100.180
Hasil Deteksi Port:
- Port 80: Apache/2.4.58 (Ubuntu) -> Rentan CVE-2024-38476 (CVSS 9.8), CVE-2024-38474 (9.8)
- Port 22: OpenSSH 9.6p1 Ubuntu -> Rentan CVE-2024-6387 "RegreSSHion" (CVSS 8.1)
```