# Laporan Final: Analisis OWASP Top 10 & Penilaian Risiko (OJS 3.3.0-8)

Dokumen ini merupakan laporan komprehensif hasil konsolidasi scanning (DAST & SAST), manual code review, pemetaan OWASP Top 10 2021, dan perhitungan risiko menggunakan standar CVSS v3.1.

---

## 1. Ringkasan Eksekutif

Berdasarkan rangkaian pengujian keamanan yang dilakukan, ditemukan **20 kerentanan terpilih** yang merepresentasikan postur keamanan Open Journal Systems (OJS) versi 3.3.0-8. 

**Distribusi Keparahan Risiko:**
- **Critical:** 5 temuan (Apache RCE, Eval Injection, Stored XSS, Unserialize, Assert RCE)
- **High:** 5 temuan (OpenSSH RegreSSHion, File Deletion, File Read, Include/Require Variable, File Upload)
- **Medium:** 6 temuan (jQuery UI, Directory Browsing, CSP, Clickjacking, phpinfo, Error Disclosure)
- **Low:** 4 temuan (HSTS, Sniffing, CSRF, Referrer-Policy)

Aplikasi ini berada dalam kondisi **High-Risk** karena terdapat jalur eksploitasi Remote Code Execution (RCE) yang dapat diakses dari jaringan tanpa atau dengan hak akses minimal.

---

## 2. Pemetaan Temuan ke OWASP Top 10 (2021)

Setiap temuan telah dipetakan ke kategori OWASP Top 10 2021 untuk memahami klasifikasi ancaman secara global.

| ID | Deskripsi Kerentanan | Kategori OWASP | CWE | CVE / Note |
| :--- | :--- | :--- | :--- | :--- |
| **VUL-001** | Apache HTTP Server 2.4.58 RCE | A06:2021 | CWE-1395 | CVE-2024-38476 |
| **VUL-003** | Eval Injection (Installer) | A03:2021 | CWE-94 | Critical (SAST) |
| **VUL-004** | Stored XSS via `customHeaders` | A03:2021 | CWE-79 | Critical (Manual) |
| **VUL-005** | Unsafe `unserialize()` (DAO) | A08:2021 | CWE-502 | Critical (Manual) |
| **VUL-006** | Assert() Dynamic Parameter | A03:2021 | CWE-94 | Critical (SAST) |
| **VUL-007** | Dynamic File Deletion | A01:2021 | CWE-73 | Critical (SAST) |
| **VUL-002** | OpenSSH 9.6p1 RegreSSHion | A06:2021 | CWE-362 | CVE-2024-6387 |
| **VUL-008** | Dynamic File Read (`readfile`) | A01:2021 | CWE-73 | High (SAST) |
| **VUL-009** | Variable-based Include/Require | A03:2021 | CWE-98 | High (SAST) |
| **VUL-010** | File Upload Without Validation | A05:2021 | CWE-434 | High (Manual) |
| **VUL-011** | Vulnerable jQuery UI v1.12.1 | A06:2021 | CWE-1395 | CVE-2021-41184 |
| **VUL-012** | Directory Browsing Enabled | A05:2021 | CWE-548 | 88 directories |
| **VUL-013** | CSP Header Not Set | A05:2021 | CWE-693 | 573 instances |
| **VUL-014** | Missing Anti-clickjacking Header | A05:2021 | CWE-693 | 122 instances |
| **VUL-018** | phpinfo() Exposure | A05:2021 | CWE-200 | Semgrep |
| **VUL-015** | Application Error Disclosure | A05:2021 | CWE-209 | Info Leak |
| **VUL-016** | Missing Strict-Transport-Security | A05:2021 | CWE-319 | Nikto |
| **VUL-017** | Missing X-Content-Type-Options | A05:2021 | CWE-693 | Nikto |
| **VUL-019** | Possible CSRF on Search Form | A01:2021 | CWE-352 | Nmap Indication |
| **VUL-020** | Missing Referrer-Policy Header | A05:2021 | CWE-693 | Low Severity |

---

## 3. Kalkulasi Detail CVSS v3.1 (Top 5 Kerentanan)

Bagian ini mendetailkan perhitungan teknis untuk kerentanan dengan prioritas tertinggi.

### 3.1 VUL-001 & VUL-003: Remote Code Execution (RCE)
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`
*   **Impact Score:** 5.87 (High impact pada C/I/A)
*   **Exploitability:** 3.89 (Network access, No Auth)
*   **Base Score:** **9.8 (Critical)**

### 3.2 VUL-004: Stored XSS via `customHeaders`
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H/A:N`
*   **Penjelasan:** Injeksi pada header kustom yang merubah scope (Changed) sehingga dapat mencuri session cookie admin lain yang membuka halaman pengaturan.
*   **Base Score:** **9.0 (Critical)**

### 3.3 VUL-005: Unsafe `unserialize()` (Object Injection)
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H`
*   **Penjelasan:** PHP object injection melalui data tersimpan di database yang memungkinkan RCE jika gadget chain terpenuhi.
*   **Base Score:** **9.0 (Critical)**

### 3.4 VUL-006: Assert() Dynamic Parameter
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`
*   **Base Score:** **8.8 (High)**

---

## 4. Visualisasi Risk Matrix (5x5)

Matriks di bawah ini memetakan 20 temuan berdasarkan **Likelihood** (Kemungkinan) dan **Business Impact** (Dampak).

```
         │  VERY    │         │         │         │  VERY   │
BUSN     │  LOW     │   LOW   │ MEDIUM  │  HIGH   │  HIGH   │
IMPACT   │   (1)    │   (2)   │   (3)   │   (4)   │   (5)   │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
CRITICAL │          │ VUL-010 │ VUL-004 │ VUL-001 │         │
   (5)   │          │         │ VUL-005 │ VUL-003 │         │
         │          │         │         │ VUL-006 │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
HIGH     │          │         │ VUL-009 │ VUL-007 │         │
   (4)   │          │         │         │ VUL-008 │         │
         │          │ VUL-002 │         │         │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
MEDIUM   │          │         │ VUL-011 │         │         │
   (3)   │          │         │ VUL-018 │         │         │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
LOW      │          │         │ VUL-019 │ VUL-015 │ VUL-012 │
   (2)   │          │         │         │         │ VUL-013 │
         │          │         │         │         │ VUL-014 │
─────────┼──────────┼─────────┼─────────┼─────────┼─────────┤
MINIMAL  │          │         │         │         │ VUL-016 │
   (1)   │          │         │         │         │ VUL-017 │
         │          │         │         │         │ VUL-020 │
─────────┴──────────┴─────────┴─────────┴─────────┴─────────┘
         LIKELIHOOD (1=Very Low → 5=Very High)

🔴 Critical (Merah)  🟠 High (Oranye)  🟡 Medium (Kuning)  🟢 Low (Hijau)
```

---

## 5. Analisis Berdasarkan Kategori OWASP

### A03:2021 — Injection (Fokus Utama)
Kerentanan injeksi mendominasi profil risiko aplikasi OJS. Selain pola injeksi kode PHP (`eval`, `assert`), ditemukan juga injeksi script (Stored XSS). 
- **Temuan Kritis:** Injeksi pada skrip installer (VUL-003) memungkinkan pengambilalihan server sebelum aplikasi benar-benar terpasang.
- **Rekomendasi:** Hapus fungsi `eval()` dan `assert()` pada lingkungan produksi. Gunakan parameterized queries dan sanitasi output (HTML escaping) secara menyeluruh.

### A06:2021 — Vulnerable and Outdated Components
Temuan pada middleware (Apache 2.4.58 dan OpenSSH 9.6p1) menunjukkan bahwa keamanan aplikasi tidak hanya bergantung pada kode PHP, tetapi juga pada patch management di level sistem operasi/kontainer.
- **Rekomendasi:** Segera lakukan pembaruan (patch) pada image Docker atau package manager sistem operasi untuk menutup CVE-2024-38476 dan CVE-2024-6387.

### A01:2021 — Broken Access Control
Penggunaan path dinamis pada fungsi `delete()` dan `readfile()` (VUL-007, VUL-008) memungkinkan penyerang untuk membaca atau menghapus file sensitif sistem seperti `/etc/passwd` atau `config.inc.php`.
- **Rekomendasi:** Terapkan whitelist direktori yang diperbolehkan dan validasi input path menggunakan fungsi `basename()` yang ketat.

---

## 6. Rekomendasi Strategis Berdasarkan Prioritas Risiko

| Prioritas | Target Perbaikan | Deskripsi Tindakan |
| :--- | :--- | :--- |
| **P1 (Hotfix)** | VUL-001, 003, 004, 005, 006 | Patch Apache, hapus `eval`/`assert`, sanitasi `customHeaders`, dan amankan `unserialize()`. |
| **P2 (Critical)** | VUL-002, 007, 008, 009, 010 | Update OpenSSH, audit parameter operasi file, dan tambahkan validasi upload. |
| **P3 (Hardening)** | VUL-012, 013, 014, 016, 017 | Tutup directory browsing dan implementasikan security headers (CSP, HSTS). |
| **P4 (Clean-up)** | VUL-011, 018, 019, 020 | Update jQuery UI, hapus sisa debug (phpinfo), dan perbaiki proteksi CSRF. |

---

## 7. Jawaban Pertanyaan Diskusi

### 1. Mengapa kerentanan CVSS 9.8 (Critical) bisa memiliki *actual risk* yang lebih rendah dari CVSS 6.5 (Medium) dalam konteks bisnis?
CVSS adalah skor keparahan teknis (*technical severity*), bukan skor risiko bisnis (*business risk*). Risiko sebenarnya dipengaruhi oleh **eksposur** dan **nilai aset**. 
- **Contoh:** Kerentanan RCE (9.8) pada server internal yang terisolasi dan tidak berisi data sensitif memiliki risiko bisnis lebih rendah dibandingkan kerentanan IDOR (6.5) pada portal publik yang mengekspos ribuan data pribadi (PII) dosen dan mahasiswa. Jika aset tidak dapat dijangkau penyerang atau tidak memiliki nilai strategis, dampak bisnisnya menjadi rendah meskipun keparahan teknisnya kritis.

### 2. Perbedaan CVSS Base, Temporal, dan Environmental Score. Mana yang paling relevan untuk institusi pendidikan?
- **Base Score:** Karakteristik intrinsik kerentanan yang bersifat statis dan tidak berubah (misal: Attack Vector, Complexity).
- **Temporal Score:** Karakteristik yang berubah seiring waktu (misal: ketersediaan exploit publik, status patch).
- **Environmental Score:** Karakteristik yang spesifik pada lingkungan pengguna (misal: tingkat kepentingan data bagi universitas, mitigasi yang sudah ada di firewall).
- **Relevansi:** **Environmental Score** adalah yang paling relevan untuk laporan internal universitas karena skor ini mencerminkan dampak nyata pada infrastruktur spesifik institusi tersebut, membantu manajemen memprioritaskan aset mana yang paling krusial untuk dilindungi.

### 3. Dalam kasus OJS, apakah A06 (Vulnerable & Outdated Components) seharusnya mendapatkan skor tinggi?
**Ya.** OJS adalah platform berbasis web yang seringkali berjalan di atas middleware (Apache/PHP) yang terpapar internet secara langsung. Kerentanan pada komponen luar (seperti VUL-001 pada Apache 2.4.58) sering kali menjadi pintu masuk utama ("pintu depan") yang memungkinkan penyerang melakukan RCE tanpa perlu mengetahui celah di dalam kode OJS itu sendiri. Dalam arsitektur *Defense in Depth*, kegagalan pada layer middleware dianggap berisiko sangat tinggi karena dapat melumpuhkan seluruh aplikasi di atasnya.

### 4. Sebagai CISO Universitas, kerentanan mana yang akan diprioritaskan pertama kali (VUL-001 s/d VUL-010)?
Saya akan memprioritaskan **VUL-001 (Apache HTTP RCE)** dan **VUL-003 (Eval Injection)**.
- **Alasan:** Kedua kerentanan ini memiliki skor 9.8 (Critical) dan bersifat "Unauthenticated RCE". Penyerang tidak perlu memiliki akun untuk menguasai server. Sebagai CISO, memprioritaskan "perimeter defense" (Apache) dan "installer security" (Eval Injection) adalah langkah krusial untuk mencegah kompromi total server universitas yang dapat berujung pada pencurian database riset, jurnal, dan kredensial pengguna secara masif.

---

**Laporan ini disusun sebagai bagian dari Deliverable Pertemuan 4 mata kuliah DevSecOps.**
**Status:** FINAL REVISED & COMPLETED
