### Pertanyaan Diskusi 

**1. Mengapa sebuah kerentanan dengan CVSS 9.8 (Critical) bisa memiliki *actual risk* yang lebih rendah dari CVSS 6.5 (Medium) dalam konteks bisnis tertentu?**

**Jawaban:**
CVSS pada dasarnya dirancang untuk mengukur keparahan teknis (*technical severity*), bukan risiko bisnis (*business risk*). *Actual risk* atau risiko nyata sangat bergantung pada konteks di mana kerentanan tersebut berada (nilai aset dan dampak bisnis). 
* Sebagai contoh, sebuah kerentanan RCE dengan **CVSS 9.8** yang ditemukan di *server* pengujian (*sandbox*) yang terisolasi dari jaringan utama dan hanya berisi data *dummy*, memiliki dampak bisnis yang nyaris nol. 
* Sebaliknya, kerentanan seperti *Arbitrary File Read* atau IDOR dengan **CVSS 6.5** yang berada di *server* produksi dan berhasil membocorkan kredensial *database* atau data pribadi mahasiswa (PII), memiliki *actual risk* yang sangat tinggi. Kebocoran ini dapat memicu denda regulasi, kerugian operasional, dan hancurnya reputasi institusi.

---

**2. Jelaskan perbedaan CVSS Base Score, Temporal Score, dan Environmental Score! Mana yang paling relevan untuk laporan vulnerability assessment institusi pendidikan?**

**Jawaban:**
* **Base Score:** Mengukur karakteristik bawaan dari sebuah kerentanan yang tidak berubah seiring berjalannya waktu dan bersifat konstan di lingkungan apa pun (seberapa mudah dieksploitasi dan dampak teknis dasarnya).
* **Temporal Score:** Mengukur karakteristik kerentanan yang berubah seiring waktu. Faktor ini dipengaruhi oleh ketersediaan eksploit publik (apakah *tools* eksploitasinya sudah tersebar luas) dan ketersediaan *patch* atau solusi sementara (*workaround*).
* **Environmental Score:** Menyesuaikan skor berdasarkan lingkungan operasional unik milik organisasi. Ini memperhitungkan perlindungan yang sudah ada (misalnya *firewall*) dan seberapa kritis aset tersebut bagi organisasi (kebutuhan kerahasiaan, integritas, dan ketersediaan data).

**Yang Paling Relevan:** Untuk *vulnerability assessment* di institusi pendidikan, **Environmental Score** adalah yang paling relevan. Institusi pendidikan sering kali menyimpan data yang sangat sensitif (data induk mahasiswa, rekam medis, hak kekayaan intelektual riset). Menghitung skor *environmental* membantu manajemen melihat mana kerentanan yang benar-benar mengancam aset kritis kampus mereka, alih-alih hanya berpatokan pada skor teknis mentah dari vendor.

---

**3. Dalam kasus OJS, apakah A06 (Vulnerable & Outdated Components) seharusnya mendapatkan skor tinggi? Jelaskan argumen Anda!**

**Jawaban:**
Ya, sangat pantas mendapatkan skor risiko yang tinggi (Kritis/High).
**Argumen:** Laporan pemindaian menunjukkan bahwa infrastruktur *server* yang menopang aplikasi OJS tersebut menggunakan versi komponen yang sudah usang dan tidak lagi menerima *patch* keamanan, yaitu **Apache 2.4.58** dan **OpenSSH 9.6p1**. Kedua layanan ini memiliki kerentanan publik (CVE) tingkat kritikal yang memungkinkan *Remote Code Execution* (RCE) di tingkat jaringan (seperti CVE-2024-38476 dan celah *RegreSSHion*). 
Keamanan sistem bekerja seperti rantai; sekuat apa pun kode PHP aplikasi OJS diperbaiki, semuanya akan runtuh apabila penyerang dapat langsung mengambil alih sistem operasi dasar atau *web server*-nya (*Server Takeover*). Oleh karena itu, komponen usang di level infrastruktur ini adalah ancaman paling nyata.

---

**4. Seandainya Anda adalah CISO universitas yang menggunakan OJS, kerentanan mana (VUL-001 s/d VUL-010) yang akan Anda prioritaskan perbaikan pertama kali dan mengapa?**

**Jawaban:**
Sebagai CISO, **kami** akan memprioritaskan penyelesaian level *hotfix* pada infrastruktur terlebih dahulu, yaitu:
1. **VUL-001 (Apache RCE CVE-2024-38476)**
2. **VUL-002 (OpenSSH RegreSSHion CVE-2024-6387)**

**Alasan:** Kompromi pada layanan infrastruktur perantara (HTTP daemon) dan sistem operasi (SSH) akan mengakibatkan hilangnya kontrol universitas secara keseluruhan atas *server*. Memperbaiki celah di tingkat aplikasi menjadi tidak ada gunanya jika peladennya sendiri sudah dikuasai oleh penyerang. 

Setelah *server* di-*upgrade* ke versi terbaru yang aman, barulah **kami** akan mengalihkan sumber daya tim *developer* untuk menghapus *anti-pattern* pemrograman tingkat kritis di dalam kode aplikasi PHP, yaitu **VUL-003 (RCE via `assert()`)** dan **VUL-004 (Arbitrary File Deletion via `delete()`)**, untuk mencegah serangan injeksi dari sisi aplikasi web.