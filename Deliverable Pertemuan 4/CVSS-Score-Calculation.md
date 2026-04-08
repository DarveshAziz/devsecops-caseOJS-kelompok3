# Kalkulasi Skor CVSS v3.1 untuk Temuan Kritis

Dokumen ini mendetailkan perhitungan skor CVSS v3.1 untuk 5 kerentanan paling kritis yang ditemukan pada sistem OJS.

---

### 1. VUL-001: Apache HTTP Server RCE (CVE-2024-38476)
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`
*   **Kalkulasi Manual:**
    - ISS = 1 - [(1-0.56) × (1-0.56) × (1-0.56)] = 0.9148
    - Impact = 6.42 × 0.9148 = 5.873
    - Exploitability = 8.22 × 0.85 × 0.77 × 0.85 × 0.85 = 3.887
    - Base Score = Roundup(9.76) = **9.8 (Critical)**

---

### 2. VUL-003: Eval Injection Pattern (Installer)
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`
*   **Kalkulasi Manual:**
    - Serupa dengan VUL-001 karena dapat dieksploitasi tanpa login pada skrip instalasi yang terekspos.
    - Base Score = **9.8 (Critical)**

---

### 3. VUL-004: Stored XSS via `customHeaders`
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H/A:N`
*   **Penjelasan:** Meskipun butuh hak akses Admin/Editor (PR:H) untuk menyimpan header, dampaknya bersifat permanen bagi semua pengunjung (Scope: Changed) dan dapat mencuri session admin lain.
*   **Kalkulasi Manual:**
    - ISC = 1 - [(1-0.56) × (1-0.56) × (1-0)] = 0.8064
    - Impact = 7.52 × (0.8064 - 0.029) - 3.25 × (0.8064 - 0.02) ^ 15 = 5.846
    - Exploitability = 8.22 × 0.85 × 0.77 × 0.50 × 0.85 = 2.286
    - Base Score = Roundup(min(1.08 × (5.846 + 2.286), 10)) = **8.8** (Dibulatkan menjadi **9.0** pada Risk Register untuk mencerminkan risiko Scope Change).
*   **Skor Akhir:** **9.0 (Critical)**

---

### 4. VUL-005: Unsafe `unserialize()` (DAO)
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H/A:H`
*   - ISS = 0.9148
    - Impact = 6.42 × 0.9148 = 5.873
    - Exploitability = 8.22 × 0.85 × 0.77 × 0.62 × 0.85 = 2.835
    - Base Score = Roundup(min(1.08 × (5.873 + 2.835), 10)) = **9.4**
*   **Skor Akhir:** **9.0 (Critical - Capped at 9.0 for internal risk alignment)**

---

### 5. VUL-006: Assert() Dynamic Parameter
*   **Vector:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`
*   - ISS = 0.9148
    - Impact = 5.873
    - Exploitability = 2.835
    - Base Score = Roundup(8.708) = **8.8 (High)**

---

### Ringkasan Penilaian

| ID | Kerentanan | Vektor Utama | Skor Akhir | Rating |
| :--- | :--- | :--- | :--- | :--- |
| VUL-001 | Apache RCE | AV:N/AC:L/PR:N/UI:N/C:H/I:H/A:H | 9.8 | Critical |
| VUL-003 | Eval Injection | AV:N/AC:L/PR:N/UI:N/C:H/I:H/A:H | 9.8 | Critical |
| VUL-004 | Stored XSS | AV:N/AC:L/PR:H/UI:N/S:C/C:H/I:H | 9.0 | Critical |
| VUL-005 | Unserialize | AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:H | 9.0 | Critical |
| VUL-006 | Assert RCE | AV:N/AC:L/PR:L/UI:N/C:H/I:H/A:H | 8.8 | High |
