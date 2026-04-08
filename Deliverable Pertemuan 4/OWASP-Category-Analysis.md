# Analisis Mendalam Per Kategori OWASP (2021) — OJS 3.3.0-8

Dokumen ini menyajikan narasi analisis teknis dan bisnis untuk setiap kategori OWASP Top 10 yang relevan berdasarkan 20 temuan keamanan pada sistem OJS.

---

## 1. A01:2021 — Broken Access Control

**Kondisi pada OJS:**
Ditemukan penggunaan parameter dinamis pada fungsi sistem file inti seperti `delete()` dan `readfile()` (VUL-007, VUL-008). Selain itu, terdapat indikasi CSRF pada form pencarian (VUL-019).

**Narasi Analisis:**
Kategori ini sangat kritis karena OJS mengelola dokumen riset yang bersifat privat (sebelum publikasi). Dengan adanya *primitive* penghapusan atau pembacaan file secara dinamis, penyerang yang memiliki akses minimal (seperti akun Author) berpotensi melampaui batasan aksesnya untuk membaca file konfigurasi `config.inc.php` yang berisi kredensial database plain-text. Kegagalan kontrol akses di sini berarti runtuhnya sekat privasi antar pengguna dan integritas data jurnal.

**Mitigasi:**
- Terapkan prinsip *Least Privilege* pada level aplikasi.
- Gunakan *Indirect Object References* (seperti ID file di database) alih-alih path file langsung dari input pengguna.

---

## 2. A03:2021 — Injection

**Kondisi pada OJS:**
Merupakan kategori dengan temuan paling berbahaya, mencakup `eval()` injection (VUL-003), Stored XSS (VUL-004), dan `assert()` dynamic parameter (VUL-006).

**Narasi Analisis:**
Injeksi pada OJS bermanifestasi dalam dua bentuk: *Code Injection* dan *Script Injection*. Penggunaan `eval()` pada skrip installer adalah "bom waktu" yang memungkinkan penyerang mengeksekusi perintah sistem operasi. Sementara itu, Stored XSS pada `customHeaders` memungkinkan serangan *sidestepping* terhadap admin; penyerang tidak perlu menyerang server secara langsung, cukup dengan menitipkan payload yang akan mencuri sesi admin saat mereka login.

**Mitigasi:**
- Larang penggunaan fungsi eksekusi kode dinamis (`eval`, `assert`, `preg_replace` dengan `/e`).
- Implementasikan *Context-Aware Output Escaping* untuk semua data yang dirender ke browser.

---

## 3. A05:2021 — Security Misconfiguration

**Kondisi pada OJS:**
Ditemukan directory browsing pada 88 direktori (VUL-012), ketiadaan CSP (VUL-013), dan paparan informasi melalui `phpinfo()` serta error disclosure (VUL-015, VUL-018).

**Narasi Analisis:**
Miskonfigurasi sering dianggap sebagai celah "Low", namun pada OJS, directory browsing pada folder `/cache/` atau `/plugins/` memberikan peta jalan (*roadmap*) bagi penyerang untuk mengidentifikasi versi plugin yang rentan atau mengunduh file sementara yang mungkin berisi data sensitif. Ketiadaan header keamanan seperti CSP juga membuat mitigasi terhadap XSS (A03) menjadi tidak ada sama sekali.

**Mitigasi:**
- Nonaktifkan `Options Indexes` pada konfigurasi Apache/.htaccess.
- Implementasikan security headers (CSP, HSTS, X-Frame-Options) secara konsisten di seluruh aplikasi.

---

## 4. A06:2021 — Vulnerable and Outdated Components

**Kondisi pada OJS:**
Penggunaan Apache 2.4.58 (VUL-001) dan OpenSSH 9.6p1 (VUL-002) yang memiliki CVE kritis, serta library jQuery UI usang (VUL-011).

**Narasi Analisis:**
OJS sangat bergantung pada ekosistem middleware-nya. Meskipun kode aplikasi OJS diperbaiki, jika web server (Apache) memiliki celah RCE seperti CVE-2024-38476, maka seluruh perlindungan di level aplikasi menjadi tidak relevan. Institusi pendidikan seringkali lambat dalam melakukan *patching* middleware karena kekhawatiran akan *downtime*, namun temuan ini menunjukkan bahwa menunda update komponen adalah risiko keamanan tertinggi saat ini.

**Mitigasi:**
- Lakukan pemindaian dependensi secara rutin.
- Gunakan kontainerisasi (Docker) untuk mempermudah proses update dan *rollback* versi middleware.

---

## 5. A08:2021 — Software and Data Integrity Failures

**Kondisi pada OJS:**
Ditemukan penggunaan `unserialize()` tanpa filter class yang aman pada layer DAO (VUL-005).

**Narasi Analisis:**
Kategori ini berkaitan dengan asumsi bahwa data yang tersimpan di database selalu aman. OJS melakukan deseralisasi data persisten yang, jika berhasil dimanipulasi melalui akses database atau celah injeksi lain, dapat memicu eksekusi objek PHP yang berbahaya. Ini adalah ancaman "Silent Killer" karena tidak terlihat pada interaksi user biasa namun dapat melumpuhkan sistem dari dalam.

**Mitigasi:**
- Ganti `unserialize()` dengan `json_decode()` jika memungkinkan.
- Jika harus menggunakan `unserialize()`, gunakan opsi `allowed_classes => false` atau whitelist kelas tertentu yang ketat.

---

**Analisis ini disusun untuk memberikan konteks naratif terhadap temuan teknis Deliverable Pertemuan 4.**
**Status:** FINAL
