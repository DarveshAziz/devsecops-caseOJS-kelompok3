# Visualisasi Risk Matrix (5x5)

Diagram ini memvisualisasikan sebaran 20 temuan risiko berdasarkan **Likelihood** (Kemungkinan) dan **Business Impact** (Dampak).

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

Legend Warna (Prioritas):
🔴 VUL-001, 003, 004, 005, 006 (Critical)
🟠 VUL-007, 002, 008, 009, 010 (High)
🟡 VUL-011, 012, 013, 014, 018, 015 (Medium)
🟢 VUL-016, 017, 019, 020 (Low)
```

---

### Analisis Sebaran Risiko

1.  **Zona Merah (Kanan Atas):** Berisi kerentanan RCE (Apache, Eval, Assert) yang memiliki dampak kritis dan kemungkinan eksploitasi tinggi karena kemudahan serangan atau ketersediaan exploit publik.
2.  **Zona Oranye (Tengah Atas/Kiri Atas):** Berisi kerentanan seperti File Deletion dan File Upload. Meskipun dampaknya kritis, faktor seperti *Privilege Required* (OpenSSH AC:H atau PR:L) menurunkan tingkat risikonya ke kategori High.
3.  **Zona Kuning (Tengah/Kanan Bawah):** Berisi masalah miskonfigurasi (Directory Browsing, Missing Headers). Meskipun kemungkinan dieksploitasi sangat tinggi (Very High Likelihood), dampak teknisnya terbatas pada kebocoran informasi awal.
4.  **Zona Hijau (Bawah):** Berisi kerentanan dengan dampak minimal atau yang bersifat indikatif (Possible CSRF).
