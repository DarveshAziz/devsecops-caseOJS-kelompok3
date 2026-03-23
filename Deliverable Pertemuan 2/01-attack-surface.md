# 0.1 Attack Surface OJS

## Identitas Target

| Item | Nilai |
|---|---|
| Nama Tim | Kelompok 3 |
| Kelas | DevSecOps-A |
| Tanggal observasi | 23 Maret 2026 |
| Target utama | `http://10.34.100.180/` |
| Aplikasi | Open Journal Systems |
| Versi teramati | `3.3.0.8` |
| Web server | Apache `2.4.58` pada Ubuntu |
| Port yang terbuka | `22/tcp`, `80/tcp` |
| Port yang tertutup | `443/tcp`, `8080/tcp` |

## 1. Ringkasan Attack Surface

Berdasarkan observasi langsung ke target, aplikasi yang aktif di `10.34.100.180` memang OJS 3.3.0.8 dan berjalan pada port `80`. Dari luar, permukaan serangan yang terlihat saat ini masih berada di level site, belum di level jurnal. Hal ini terlihat dari halaman depan yang menampilkan pesan bahwa belum ada jurnal yang tersedia.

Implikasinya cukup penting. Attack surface OJS yang biasanya luas menjadi sedikit lebih sempit untuk saat ini karena route seperti artikel, issue, search jurnal, submission, dan workflow editorial belum muncul di UI publik. Meski begitu, permukaan serangan inti tetap ada, terutama pada login, reset password, register page, halaman about, static asset delivery, session cookie, dan beberapa endpoint aplikasi yang merespons walaupun belum sepenuhnya terbuka.

Jadi, laporan ini sengaja dibagi menjadi dua lapisan. Lapisan pertama adalah surface yang benar-benar teramati langsung dari target saat ini. Lapisan kedua adalah surface bawaan OJS 3.3.0.8 yang sangat mungkin aktif begitu jurnal mulai dikonfigurasi dan workflow digunakan oleh author, reviewer, atau editor.

## 2. Hasil Observasi Langsung

Berikut ringkasan temuan yang benar-benar terlihat saat pengamatan:

| Item | Hasil observasi | Makna untuk attack surface |
|---|---|---|
| Fingerprinting | `whatweb` mengidentifikasi `Open Journal Systems 3.3.0.8` | Versi aplikasi terekspos dengan jelas |
| Port scanning | `22/tcp` dan `80/tcp` terbuka, `443` dan `8080` tertutup | Surface jaringan utama saat ini ada pada SSH dan HTTP |
| Halaman depan | HTTP `200`, menampilkan "There are no journals available." | Belum ada konteks jurnal publik yang memperluas route |
| Halaman login | HTTP `200`, form mengarah ke `/index.php/index/login/signIn` | Entry point autentikasi aktif |
| Halaman reset password | HTTP `200`, form mengarah ke `/index.php/index/login/requestResetPassword` | Account recovery surface aktif |
| Halaman register | HTTP `200`, tetapi berisi pesan bahwa registrasi tidak diterima | Register page ada, namun fungsi pendaftaran sedang ditutup |
| Halaman about | HTTP `200`, menyebut OJS 3.3.0.8 secara eksplisit | Information disclosure versi aplikasi |
| Session cookie | Server mengirim cookie `OJSSID` | Session management sudah aktif sejak akses anonim |
| API context | `/index.php/index/api/v1/contexts` merespons `403` | Endpoint API ada, tetapi dibatasi |
| API root | `/api/v1/contexts/` berakhir `500` setelah redirect | Error handling pada jalur API layak diuji lebih lanjut |

Ada dua observasi yang cukup menonjol. Pertama, halaman publik saat ini masih sangat minimal karena belum ada jurnal yang dipublikasikan. Kedua, meskipun UI publik sempit, komponen autentikasi dan API sudah hidup, sehingga attacker tetap punya beberapa titik masuk yang layak diuji.

## 3. Diagram Attack Surface

Visualisasi attack surface utama dapat dilihat pada file diagram yang diunggah pada repositori GitHub kelompok. Diagram tersebut memetakan hubungan antara pengguna anonim, web server Apache, aplikasi OJS, route inti yang aktif, cookie sesi, dan backend data. Dengan kondisi target saat ini, node yang paling penting untuk muncul di diagram adalah halaman depan, login, reset password, register page, halaman about, static assets, serta jalur API yang merespons `403` dan `500`.

Kalau dilihat dari hasil observasi, titik serang yang paling nyata sekarang bukan submission atau workflow, melainkan autentikasi, reset password, error handling, session handling, dan informasi versi aplikasi.

## 4. Tabel Entry Points

### A. Authentication & Session

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| A1 | `/index.php/index/login` | GET | Form login utama site | Brute force, credential stuffing |
| A2 | `/index.php/index/login/signIn` | POST | Proses autentikasi pengguna | Auth bypass, input validation issue |
| A3 | `/index.php/index/login/lostPassword` | GET | Halaman reset password | User enumeration |
| A4 | `/index.php/index/login/requestResetPassword` | POST | Proses permintaan reset password berbasis email | Abuse reset workflow, email flooding |
| A5 | Cookie `OJSSID` | Set-Cookie | Identitas sesi dibuat sejak akses awal | Session fixation, cookie hardening lemah |

### B. File Upload

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| B1 | `api/v1/_uploadPublicFile` | POST | Upload file publik OJS, belum teramati aktif pada UI saat ini | File upload abuse |
| B2 | `api/v1/temporaryFiles` | POST | Upload file sementara pada workflow submission, belum teramati aktif pada target saat ini | File handling weakness |
| B3 | `/index.php/<journal>/submission/wizard/...` | GET/POST | Wizard submission bawaan OJS yang akan aktif setelah jurnal dan author digunakan | Malicious file upload |
| B4 | Dashboard plugin atau settings upload | POST | Upload aset atau plugin saat area admin aktif | Plugin abuse, file upload risk |

### C. User Input / Reflected Data

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| C1 | `/` | GET | Halaman utama site, saat ini menampilkan belum ada jurnal | Information disclosure |
| C2 | `/index.php/index/index` | GET | Site index OJS | Enumerasi struktur awal aplikasi |
| C3 | `/index.php/index/user/register` | GET | Halaman register masih dapat diakses walau registrasi ditutup | Logic flaw, information disclosure |
| C4 | `/index.php/index/about/aboutThisPublishingSystem` | GET | Halaman about yang menampilkan informasi platform | Version disclosure, fingerprinting |
| C5 | Static assets `/lib/pkp/...`, `/plugins/themes/...`, `/templates/...` | GET | Asset statis yang memperlihatkan struktur OJS dan nomor versi | Asset enumeration, version leakage |

### D. REST API

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| D1 | `/index.php/index/api/v1/contexts` | GET | Endpoint API context yang merespons `403` | Access control verification |
| D2 | `/api/v1/contexts/` | GET | Jalur API alternatif yang berakhir `500` | Error handling issue |
| D3 | `api/v1/submissions` | GET/POST | Surface submission API bawaan OJS, belum aktif di UI target saat ini | IDOR, broken access control |
| D4 | `api/v1/users` | GET | Surface user API bawaan OJS saat role privileged aktif | User data exposure |

### E. Admin Panel / Configuration

| No | URL / Endpoint | Method | Deskripsi | Risiko Potensial |
|---|---|---|---|---|
| E1 | Site settings OJS | GET/POST | Pengaturan site OJS, belum teramati dari sisi publik | Security misconfiguration |
| E2 | Plugin management | GET/POST | Pengelolaan plugin bawaan OJS | Plugin abuse, privilege misuse |
| E3 | Journal settings | GET/POST | Konfigurasi jurnal yang akan aktif setelah jurnal dibuat | Misconfiguration, access control issue |
| E4 | User management | GET/POST | Pengelolaan akun oleh admin atau editor | Privilege escalation |

### Catatan penting

- Halaman utama masih kosong dari sisi konten jurnal, sehingga surface publik yang terlihat memang belum penuh.
- Route login dan reset password sudah siap dipakai. Dari perspektif attacker, ini tetap lebih menarik daripada homepage.
- Halaman register masih bisa diakses walaupun pendaftaran ditutup. Ini berarti fitur registrasi belum hilang dari surface, hanya dibatasi oleh logika aplikasi.
- Jalur API belum bersih dari anomali. Satu endpoint memberi `403`, sedangkan pola lain berujung `500`.

## 5. Surface Lanjutan yang Belum Terbuka Penuh

Walaupun saat observasi belum ada jurnal aktif di halaman depan, OJS 3.3.0.8 secara arsitektur masih menyimpan surface berikut yang akan relevan begitu site mulai dipakai penuh:

| Area | Pola route | Kapan relevan | Risiko utama |
|---|---|---|---|
| Search jurnal | `/index.php/<journal>/search` | Setelah jurnal tersedia | Reflected XSS, input validation |
| Halaman artikel | `/index.php/<journal>/article/view/<id>` | Setelah ada artikel | Stored XSS, file access, ID guessing |
| Halaman issue | `/index.php/<journal>/issue/view/<id>` | Setelah ada issue | Metadata exposure, file disclosure |
| Submission wizard | `/index.php/<journal>/submission/wizard/...` | Setelah author aktif | Upload abuse, metadata injection |
| Temporary files API | `api/v1/temporaryFiles` | Saat workflow upload dipakai | File handling weakness |
| Submission API | `api/v1/submissions` | Saat workflow editorial aktif | IDOR, broken access control |
| User API | `api/v1/users` | Saat role admin/editor dipakai | User data exposure |
| Public file upload | `api/v1/_uploadPublicFile` | Saat pengelola upload aset | File upload risk |
| Workflow editorial | route workflow dan review | Setelah ada submission | Stored XSS, privilege escalation |
| Plugin dan settings | dashboard admin dan pengaturan site | Setelah akun web privileged aktif | Misconfiguration, plugin abuse |

Bagian ini tetap penting dicantumkan karena attack surface aplikasi tidak hanya ditentukan oleh apa yang sedang muncul di homepage hari ini, tetapi juga oleh fitur yang sudah tersedia di codebase dan bisa aktif kapan saja saat konfigurasi berubah.

## 6. Data Flow Diagram Dua Alur Kritis

### 6.1 Alur Login

Visualisasi alur login dapat dilihat pada file diagram DFD login yang diunggah pada repositori GitHub kelompok. Secara proses, user anonim mengirim username dan password ke form login, aplikasi memvalidasi input lalu memeriksa data akun ke backend OJS, kemudian server membuat session dan mengirim cookie `OJSSID` jika autentikasi berhasil. Boundary paling sensitif pada alur ini ada di proses validasi kredensial dan pembuatan sesi.

Komponen utama alur login adalah:

| Komponen | Peran |
|---|---|
| User anonim | Mengirim kredensial login |
| Form login OJS | Menerima input `username` dan `password` |
| Handler autentikasi | Memproses permintaan sign in |
| Backend database OJS | Menyimpan data akun dan hash password |
| Session management | Membuat sesi dan cookie pengguna |

### 6.2 Alur Reset Password

Visualisasi alur reset password dapat dilihat pada file diagram DFD reset password yang diunggah pada repositori GitHub kelompok. Pada alur ini, user memasukkan alamat email terdaftar ke halaman reset password, aplikasi memeriksa apakah email terkait dengan akun OJS, lalu sistem menyiapkan proses lanjutan untuk pengiriman instruksi reset. Boundary penting pada alur ini ada di validasi email, respons aplikasi, dan pengendalian agar proses ini tidak dipakai untuk enumerasi akun atau spam.

Komponen utama alur reset password adalah:

| Komponen | Peran |
|---|---|
| User anonim | Mengirim email untuk reset password |
| Form reset password | Menerima input email |
| Handler request reset | Memproses permintaan reset password |
| Backend database OJS | Memeriksa kecocokan email dengan akun |
| Mekanisme email OJS | Mengirim instruksi reset bila konfigurasi aktif |

## 7. Trust Boundary

Untuk kondisi target saat ini, boundary yang paling relevan adalah:

| Boundary | Sumber | Tujuan | Kenapa penting |
|---|---|---|---|
| TB-01 | Pengguna anonim | Login dan reset password | Titik masuk utama sebelum ada akun web sah |
| TB-02 | Browser | Session cookie `OJSSID` | Semua state login bergantung pada cookie ini |
| TB-03 | OJS web layer | Backend database dan session store | Semua autentikasi dan reset password bermuara ke sini |
| TB-04 | OJS web layer | Static asset dan template delivery | Menjadi sumber version leakage dan enumerasi struktur |
| TB-05 | Site context kosong | Konteks jurnal yang belum dibuat | Surface akan melebar signifikan setelah jurnal aktif |

Boundary paling penting saat ini ada pada peralihan dari user anonim ke user terautentikasi. Karena surface jurnal belum aktif, kontrol di login, reset password, dan session cookie menjadi gerbang utama yang menentukan seberapa jauh seorang attacker bisa maju.

## 8. Aset Kritis

| Aset | Confidentiality | Integrity | Availability | Nilai Kritis | Alasan |
|---|---|---|---|---|---|
| Kredensial admin web OJS | High | High | Medium | Kritis | Jika bocor, seluruh site settings dapat diambil alih |
| Session cookie `OJSSID` | High | High | Medium | Tinggi | Menjadi kunci state autentikasi pengguna |
| Data user OJS | High | High | Medium | Kritis | Akan berisi akun admin, editor, reviewer, dan author |
| Konfigurasi site OJS | Medium | High | Medium | Tinggi | Mengontrol perilaku registrasi, jurnal, plugin, dan email |
| Endpoint reset password | High | High | Medium | Tinggi | Bisa menjadi jalur takeover jika kontrolnya lemah |
| Informasi versi aplikasi | Medium | Low | Low | Sedang | Mempermudah pencocokan dengan advisory atau CVE yang dikenal |
| Struktur route dan asset | Medium | Low | Low | Sedang | Membantu attacker melakukan enumerasi lebih cepat |
| Backend database OJS | High | High | High | Kritis | Menyimpan akun, konfigurasi, token, dan data operasional site |

## 9. Matriks STRIDE

| Threat | Singkatan | Contoh pada target | Entry Point / Aset Terkait | Dampak Potensial |
|---|---|---|---|---|
| Spoofing | S | Upaya login memakai kredensial curian atau hasil brute force | `/index.php/index/login`, `signIn`, cookie `OJSSID` | Pengambilalihan akun |
| Tampering | T | Perubahan parameter request login, reset password, atau konfigurasi saat panel admin aktif | `signIn`, `requestResetPassword`, site settings | Perubahan data atau state aplikasi tanpa izin |
| Repudiation | R | Pengguna menyangkal permintaan login atau reset password bila logging tidak memadai | Login flow, reset password flow, log aplikasi | Sulit melakukan audit insiden |
| Information Disclosure | I | Versi OJS terekspos di halaman about, static assets, dan generator meta tag | Halaman about, asset path, header aplikasi | Mempermudah fingerprinting dan pencarian CVE |
| Denial of Service | D | Penyalahgunaan login atau reset password secara berulang hingga membebani aplikasi | Login page, lost password, request reset | Gangguan layanan dan pemborosan resource |
| Elevation of Privilege | E | Penyalahgunaan endpoint admin atau API bila kontrol akses lemah setelah jurnal aktif | API OJS, panel admin, user management | User biasa mendapat hak akses lebih tinggi |

## 10. Prioritas Pengujian

Berdasarkan kondisi target yang sekarang, urutan prioritas yang paling masuk akal adalah:

1. Login, reset password, dan session cookie
   Ini adalah surface paling nyata dan paling bernilai saat jurnal belum aktif.

2. Error handling dan akses kontrol pada API
   Respons `403` dan `500` di area API menandakan jalur ini layak dipetakan lebih detail.

3. Information disclosure
   Versi OJS, header server, static asset path, dan route bawaan sangat membantu fingerprinting.

4. Register page dan logic boundary
   Pendaftaran memang ditutup, tetapi halaman register tetap hidup. Ini tetap perlu diuji sebagai logic surface.

5. Surface lanjutan setelah jurnal aktif
   Begitu jurnal dibuat, fokus harus bergeser ke submission, file upload, workflow review, dan akses lintas role.

## 11. Kesimpulan

Attack surface target `10.34.100.180` saat ini lebih sempit dibanding instalasi OJS yang sudah benar-benar dipakai untuk operasional jurnal. Dari observasi live, permukaan serangan yang aktif sekarang berpusat pada site index, login, reset password, register page, about page, static assets, session cookie, dan sebagian jalur API.

Namun, kesempitan surface ini sifatnya sementara. Begitu jurnal mulai tersedia dan workflow editorial dipakai, surface OJS akan melebar ke area artikel, issue, submission, upload file, API internal, dan dashboard pengelola. Karena itu, laporan ini tidak hanya mencatat apa yang terbuka hari ini, tetapi juga memberi peta transisi ke surface yang kemungkinan besar akan muncul di tahap berikutnya.
