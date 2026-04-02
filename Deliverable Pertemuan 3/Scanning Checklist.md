## 5. Checklist Scanning

Checklist ini digunakan untuk memastikan kelengkapan scanning:

### DAST Checklist
- [ ] Nikto dijalankan dan output tersimpan
- [ ] ZAP Spider dijalankan (minimal 100 URL terindeks)
- [ ] ZAP Active Scan dijalankan (unauthenticated)
- [ ] ZAP Active Scan dijalankan (authenticated sebagai Author)
- [ ] ZAP Active Scan dijalankan (authenticated sebagai Admin)
- [ ] SQLMap dijalankan pada minimal 3 parameter
- [ ] SSRF test dilakukan pada CVE-2021-27188
- [ ] Manual XSS test pada form search, profil, abstrak

### SAST Checklist
- [ ] Semgrep dengan ruleset `p/php` selesai dijalankan
- [ ] Semgrep dengan ruleset `p/owasp-top-ten` selesai dijalankan
- [ ] Custom rules Semgrep dijalankan
- [ ] Manual review pada 5 file kritis
- [ ] Temuan deduplikasi (hapus false positive)