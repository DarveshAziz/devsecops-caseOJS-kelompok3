// delete later (Klo salah revisi aja yak :v)
// CVSS score calculation untuk minimal 5 temuan kritis

### Deliverable 2: CVSS Score Calculation untuk 5 Temuan Kritis

**1. VUL-001: Apache 2.4.58 Multiple RCE (CVE-2024-38476)**
* **Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`
* **Penjelasan:** Dapat dieksploitasi dari jarak jauh tanpa kondisi khusus, tanpa autentikasi, dan tanpa interaksi pengguna. Dampaknya adalah pengambilalihan kontrol penuh atas kerahasiaan, integritas, dan ketersediaan server.
* **Kalkulasi Manual:**
  $$ISS = 1 - [(1 - 0.56) \times (1 - 0.56) \times (1 - 0.56)] = 0.9148$$
  $$Impact = 6.42 \times 0.9148 = 5.873$$
  $$Exploitability = 8.22 \times 0.85 \times 0.77 \times 0.85 \times 0.85 = 3.887$$
  $$Base Score = \text{Roundup}(\min(5.873 + 3.887, 10)) = \text{Roundup}(9.76) = 9.8$$
* **Skor Akhir:** **9.8 (Critical)**

---

**2. VUL-003: RCE Risk via `assert()` (SAST)**
* **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`
* **Penjelasan:** Parameter dinamis pada `assert()` memicu eksekusi kode. Penyerang butuh login level dasar (PR:L) karena fungsi berada di modul *Submission*. Tidak butuh interaksi korban. Berdampak penuh (C/I/A High) pada aplikasi.
* **Kalkulasi Manual:**
  $$ISS = 1 - [(1 - 0.56) \times (1 - 0.56) \times (1 - 0.56)] = 0.9148$$
  $$Impact = 6.42 \times 0.9148 = 5.873$$
  $$Exploitability = 8.22 \times 0.85 \times 0.77 \times 0.62 \times 0.85 = 2.835$$
  $$Base Score = \text{Roundup}(\min(5.873 + 2.835, 10)) = \text{Roundup}(8.708) = 8.8$$
* **Skor Akhir:** **8.8 (High)**

---

**3. VUL-002: OpenSSH 9.6p1 RegreSSHion (CVE-2024-6387)**
* **Vector:** `CVSS:3.1/AV:N/AC:H/PR:N/UI:N/S:U/C:H/I:H/A:H`
* **Penjelasan:** Serangan RCE tingkat jaringan tanpa autentikasi, namun membutuhkan kondisi *race condition* yang sulit (AC:H). Jika berhasil, penyerang mendapatkan akses *root* secara penuh (C/I/A High).
* **Kalkulasi Manual:**
  $$ISS = 1 - [(1 - 0.56) \times (1 - 0.56) \times (1 - 0.56)] = 0.9148$$
  $$Impact = 6.42 \times 0.9148 = 5.873$$
  $$Exploitability = 8.22 \times 0.85 \times 0.44 \times 0.85 \times 0.85 = 2.221$$
  $$Base Score = \text{Roundup}(\min(5.873 + 2.221, 10)) = \text{Roundup}(8.094) = 8.1$$
* **Skor Akhir:** **8.1 (High)**

---

**4. VUL-004: Arbitrary File Deletion via `delete()` (SAST)**
* **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:H`
* **Penjelasan:** Melalui manipulasi input pada API manajemen file, file penting sistem dapat dihapus. Tidak berdampak pada kebocoran data (C:N), namun memiliki efek destruktif (I:H) dan melumpuhkan sistem (A:H).
* **Kalkulasi Manual:**
  $$ISS = 1 - [(1 - 0) \times (1 - 0.56) \times (1 - 0.56)] = 0.8064$$
  $$Impact = 6.42 \times 0.8064 = 5.177$$
  $$Exploitability = 8.22 \times 0.85 \times 0.77 \times 0.62 \times 0.85 = 2.835$$
  $$Base Score = \text{Roundup}(\min(5.177 + 2.835, 10)) = \text{Roundup}(8.012) = 8.1$$
* **Skor Akhir:** **8.1 (High)**

---

**5. VUL-005: Arbitrary File Read via `readfile()` (SAST)**
* **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N`
* **Penjelasan:** *Path traversal* yang memungkinkan penyerang membaca *source code* dan kredensial database. Ini hanya berdampak penuh pada kerahasiaan data (C:H) tanpa merusak integritas atau mematikan ketersediaan (I:N, A:N).
* **Kalkulasi Manual:**
  $$ISS = 1 - [(1 - 0.56) \times (1 - 0) \times (1 - 0)] = 0.56$$
  $$Impact = 6.42 \times 0.56 = 3.595$$
  $$Exploitability = 8.22 \times 0.85 \times 0.77 \times 0.62 \times 0.85 = 2.835$$
  $$Base Score = \text{Roundup}(\min(3.595 + 2.835, 10)) = \text{Roundup}(6.43) = 6.5$$
* **Skor Akhir:** **6.5 (Medium)**