# API and Integration Analysis - SIPPOL Reborn Kediri

## 1. REST Endpoints Documentation

### 1.1 AJAX Endpoints

#### **GET /admin/ajax-get-data.php**
- **Purpose**: Mengambil data user berdasarkan ID
- **Method**: GET
- **Parameters**: 
  - `id_user` (required) - ID user yang akan diambil datanya
- **Response**: JSON
```json
{
  "nama": "string",
  "npm": "string", 
  "nama_instansi": "string"
}
```
- **Authentication**: None (⚠️ **Security Issue**)
- **SQL Injection Risk**: HIGH - Parameter langsung dimasukkan ke query

#### **POST /admin/ajax-get-pemagang.php**
- **Purpose**: Mengambil daftar pemagang berdasarkan instansi
- **Method**: POST
- **Parameters**:
  - `asal` (required) - Nama instansi
- **Response**: HTML content dengan data pemagang dan tabel statistik
- **Authentication**: None (⚠️ **Security Issue**)
- **SQL Injection Risk**: HIGH

#### **POST /admin/handler/handler-show-table.php**
- **Purpose**: Filter dan tampilkan data presensi
- **Method**: POST
- **Parameters**:
  - `selected_year` (optional)
  - `selected_month` (optional)
  - `selected_date` (optional)
  - `selected_institution` (optional)
- **Response**: JSON
```json
{
  "success": true,
  "data": [
    {
      "id_data": "number",
      "data_nama": "string",
      "data_npm": "string",
      "data_asal": "string",
      "data_tanggal": "datetime",
      "data_status": "string",
      "data_deskripsi": "string"
    }
  ]
}
```
- **Authentication**: Session-based
- **SQL Injection Risk**: HIGH

### 1.2 Form Handler Endpoints

#### **POST /handler-presensi.php**
- **Purpose**: Submit presensi pemagang
- **Method**: POST
- **Parameters**:
  - `data_nama` (required)
  - `data_npm` (required)
  - `data_asal` (required)
  - `data_status` (required)
  - `data_deskripsi` (required)
- **Response**: Redirect dengan session message
- **Authentication**: Session-based (`$_SESSION['id_user']`)
- **Input Sanitization**: Basic `mysqli_real_escape_string()`

#### **POST /admin/handler/handler-add-pemagang.php**
- **Purpose**: Tambah user/pemagang baru
- **Method**: POST
- **Parameters**:
  - `username` (required)
  - `nama` (optional)
  - `npm` (required)
  - `password` (required)
  - `asal` (required)
  - `mulai_magang` (required)
  - `akhir_magang` (required)
  - `status` (required)
- **Response**: Redirect dengan session message
- **Authentication**: Session-based admin
- **Security**: Password hashing dengan `password_hash()`

#### **POST /admin/handler/handler-add-pengajuan.php**
- **Purpose**: Upload dan simpan pengajuan dengan file attachment
- **Method**: POST
- **Parameters**:
  - `nama_instansi` (required)
  - `tahun[]` (required array)
  - `keterangan[]` (required array)
  - `srt_pengajuan[]` (required files)
  - `srt_balasan[]` (required files)
- **Response**: Redirect dengan session message
- **Authentication**: Session-based admin
- **File Upload Validation**: 
  - Max size: 2MB
  - Allowed extensions: PDF only
  - File existence check

### 1.3 Export Endpoints

#### **POST /admin/handler/handler-export.php**
- **Purpose**: Export data presensi ke CSV
- **Method**: POST
- **Parameters**:
  - `selected_month` (optional)
  - `selected_year` (optional)
  - `selected_date` (optional)
  - `selected_institution` (optional)
- **Response**: CSV file download
- **Authentication**: Session-based admin
- **Headers**: 
  - `Content-Type: text/csv`
  - `Content-Disposition: attachment`

### 1.4 CRUD Handler Endpoints

| Endpoint | Method | Purpose | Auth Required |
|----------|--------|---------|---------------|
| `/admin/handler/handler-add-instansi.php` | POST | Tambah instansi | ✅ Admin |
| `/admin/handler/handler-edit-instansi.php` | POST | Edit instansi | ✅ Admin |
| `/admin/handler/handler-delete-instansi.php` | GET | Hapus instansi | ✅ Admin |
| `/admin/handler/handler-add-presensi.php` | POST | Tambah presensi | ✅ Admin |
| `/admin/handler/handler-edit-presensi.php` | POST | Edit presensi | ✅ Admin |
| `/admin/handler/handler-delete-presensi.php` | GET | Hapus presensi | ✅ Admin |
| `/admin/handler/handler-edit-pengajuan.php` | POST | Edit pengajuan | ✅ Admin |
| `/admin/handler/handler-delete-pengajuan.php` | GET | Hapus pengajuan | ✅ Admin |

## 2. Authentication Mechanism Analysis

### 2.1 Session-Based Authentication

#### **User Authentication (Pemagang)**
```php
// File: login.php
session_start();
// Login process
$_SESSION['login_user'] = $username;
// Verification with password_verify()
```

**Session Variables:**
- `$_SESSION['login_user']` - Username pemagang yang login
- `$_SESSION['id_user']` - ID user untuk tracking presensi

#### **Admin Authentication** 
```php
// File: admin/login.php  
session_start();
// Login process
$_SESSION['isLogin'] = true;
// Query admin table with password verification
```

**Session Variables:**
- `$_SESSION['isLogin']` - Boolean flag untuk admin session

### 2.2 Authentication Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Login Form    │    │  Session Check  │    │  Access Control │
│                 │    │                 │    │                 │
│ - Username      │───►│ session_start() │───►│ Redirect if     │
│ - Password      │    │ Check session   │    │ not authorized  │
│ - Submit        │    │ variables       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2.3 Session Security Issues

❌ **Critical Issues:**
1. **No CSRF Protection** - Tidak ada token CSRF
2. **No Session Regeneration** - Session ID tidak di-regenerate setelah login
3. **No Session Timeout** - Tidak ada timeout otomatis
4. **No HTTPOnly Flag** - Session cookie tidak protected
5. **No Secure Flag** - Tidak ada SSL enforcement

## 3. Rate Limiting & Security Measures

### 3.1 Rate Limiting Analysis

❌ **NOT IMPLEMENTED**
- Tidak ada rate limiting pada endpoint manapun
- Semua endpoint dapat dipanggil tanpa batas
- Rentan terhadap brute force attacks

### 3.2 Security Measures Assessment

#### **Input Validation**

**✅ Implemented (Partial):**
```php
// File upload validation
$maxFileSize = 2 * 1024 * 1024; // 2MB
$allowedExtensions = ["pdf"];

// Basic SQL escape
$nama = mysqli_real_escape_string($koneksi, $nama);
```

**❌ Missing:**
- CSRF tokens
- Input sanitization untuk XSS
- Comprehensive input validation
- Request size limiting

#### **SQL Injection Protection**

**Status: VULNERABLE**

❌ **High Risk Examples:**
```php
// admin/ajax-get-data.php - Direct injection
$sql = "SELECT ... WHERE user.id_user = $id_user";

// admin/ajax-get-pemagang.php - String interpolation  
$query_mahasiswa = "... WHERE i.nama_instansi = '$asal'";
```

**✅ Partial Protection:**
```php
// Some handlers use mysqli_real_escape_string
$selected_institution = mysqli_real_escape_string($koneksi, $selected_institution);
```

#### **File Upload Security**

**✅ Implemented:**
- File size validation (2MB max)
- File extension validation (PDF only)
- File existence check to prevent overwrite
- Files stored outside web root (`/admin/files/`)

**❌ Missing:**
- MIME type validation
- Virus scanning
- File content validation
- Secure file naming (timestamping)

#### **Password Security**

**✅ Implemented:**
```php
// Strong password hashing
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Secure verification
password_verify($password, $row['password'])
```

## 4. Third-Party API Integrations

### 4.1 External Dependencies Analysis

**JavaScript Libraries (CDN):**
```html
<!-- SweetAlert2 -->
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

<!-- DataTables -->
<script src="https://cdn.datatables.net/1.11.5/js/jquery.dataTables.min.js"></script>
<script src="https://cdn.datatables.net/1.11.5/js/dataTables.bootstrap5.min.js"></script>

<!-- PDF.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.11.338/pdf.min.js"></script>

<!-- HTML2Canvas -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/0.4.1/html2canvas.min.js"></script>

<!-- jQuery -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
```

### 4.2 External API Calls

❌ **NO EXTERNAL API INTEGRATIONS FOUND**

Sistem ini tidak mengintegrasikan dengan:
- Payment gateways
- Email services  
- SMS services
- Social media APIs
- Government APIs
- Cloud storage services

### 4.3 Vendor Dependencies

**PHP Dependencies:**
- Tidak menggunakan Composer
- Tidak ada vendor libraries
- Pure PHP implementation

**CSS/JS Frameworks:**
- Bootstrap (local copy)
- SB Admin 2 template
- FontAwesome icons

## 5. File Upload/Download Mechanisms

### 5.1 File Upload Implementation

#### **Upload Directory Structure**
```
/admin/files/
├── pengajuan/     # Surat pengajuan
└── balasan/       # Surat balasan
```

#### **Upload Process Flow**
```php
// File: admin/handler/handler-add-pengajuan.php
$target_dir_pengajuan = "../files/pengajuan/";
$target_dir_balasan = "../files/balasan/";

// Multi-file upload loop
for ($i = 0; $i < count($srt_pengajuan_files["name"]); $i++) {
    // Validation
    // File size check
    // Extension check  
    // Existence check
    // Move uploaded file
    move_uploaded_file($temp_file, $target_file);
}
```

### 5.2 File Upload Security Analysis

#### **✅ Security Measures Implemented:**

1. **File Size Validation:**
```php
$maxFileSize = 2 * 1024 * 1024; // 2MB limit
if ($srt_pengajuan_files["size"][$i] > $maxFileSize) {
    $_SESSION["error_message_admin"] = "Ukuran file tidak boleh melebihi 2MB.";
    exit();
}
```

2. **File Extension Validation:**
```php
$allowedExtensions = ["pdf"];
$extension = pathinfo($filename, PATHINFO_EXTENSION);
if (!in_array($extension, $allowedExtensions)) {
    $_SESSION["error_message_admin"] = "File harus berupa PDF.";
    exit();
}
```

3. **File Existence Check:**
```php
if (file_exists($target_file)) {
    $_SESSION["error_message_admin"] = "Nama file tidak boleh sama.";
    exit();
}
```

4. **Directory Separation:**
- Files stored in `/admin/files/` directory
- Separated by type (pengajuan/balasan)

#### **❌ Missing Security Measures:**

1. **MIME Type Validation** - Hanya check extension, tidak MIME type
2. **File Content Validation** - Tidak validasi isi file
3. **Virus Scanning** - Tidak ada antivirus check
4. **Secure File Naming** - Menggunakan nama original (rentan)
5. **File Access Control** - Tidak ada akses kontrol per file

### 5.3 File Download/Access Mechanism

#### **PDF Viewer Implementation:**
```html
<!-- Modal untuk view PDF -->
<div class="modal fade" id="pdfModal">
    <div class="modal-body">
        <embed id="pdfEmbed" src="" type="application/pdf" width="100%" height="600px">
    </div>
</div>

<script>
function openPdfModal(url) {
    $('#pdfEmbed').attr('src', 'admin/' + url + "#navpanes=0");
    $('#pdfModal').modal('show');
}
</script>
```

#### **Export/Download Endpoints:**

1. **CSV Export:**
```php
// File: admin/handler/handler-export.php
header('Content-Type: text/csv');
$filename = 'export-' . date('d-F-Y') . '.csv';
header('Content-Disposition: attachment; filename="' . $filename . '"');
```

2. **File Access:**
- Direct file access via URL path
- No access control validation
- No download logging

### 5.4 File Security Risk Assessment

| Risk Level | Issue | Impact |
|------------|-------|--------|
| 🔴 **HIGH** | No MIME validation | Malicious file upload |
| 🔴 **HIGH** | Direct file access | Unauthorized file access |
| 🔴 **HIGH** | Original filename usage | Directory traversal risk |
| 🟡 **MEDIUM** | No virus scanning | Malware distribution |
| 🟡 **MEDIUM** | No access logs | No audit trail |
| 🟢 **LOW** | File size limit | DoS via large files |

## 6. API Security Summary & Recommendations

### 6.1 Current Security Status

| Category | Status | Grade |
|----------|--------|-------|
| **Authentication** | Session-based | 🟡 C+ |
| **Authorization** | Basic | 🟡 C |
| **Input Validation** | Partial | 🔴 D+ |
| **SQL Injection** | Vulnerable | 🔴 F |
| **File Upload** | Basic protection | 🟡 C |
| **Rate Limiting** | Not implemented | 🔴 F |
| **CSRF Protection** | Not implemented | 🔴 F |

### 6.2 Critical Security Recommendations

#### **🔴 URGENT (Critical)**

1. **Fix SQL Injection Vulnerabilities:**
```php
// Bad (Current)
$sql = "SELECT * FROM user WHERE id_user = $id_user";

// Good (Recommended) 
$stmt = $koneksi->prepare("SELECT * FROM user WHERE id_user = ?");
$stmt->bind_param("i", $id_user);
```

2. **Implement CSRF Protection:**
```php
// Generate CSRF token
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));

// Validate CSRF token
if (!hash_equals($_SESSION['csrf_token'], $_POST['csrf_token'])) {
    die('CSRF token mismatch');
}
```

3. **Add Authentication to AJAX Endpoints:**
```php
// Add to all AJAX endpoints
session_start();
if (!isset($_SESSION['isLogin']) || $_SESSION['isLogin'] === false) {
    http_response_code(401);
    echo json_encode(['error' => 'Unauthorized']);
    exit();
}
```

#### **🟡 HIGH PRIORITY**

4. **Implement Rate Limiting:**
```php
// Simple rate limiting
$_SESSION['requests'] = $_SESSION['requests'] ?? [];
$current_time = time();
$_SESSION['requests'] = array_filter($_SESSION['requests'], function($time) use ($current_time) {
    return ($current_time - $time) < 60; // 1 minute window
});

if (count($_SESSION['requests']) >= 30) { // 30 requests per minute
    http_response_code(429);
    die('Rate limit exceeded');
}
$_SESSION['requests'][] = $current_time;
```

5. **Enhance File Upload Security:**
```php
// Add MIME type validation
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
if ($mime_type !== 'application/pdf') {
    die('Invalid file type');
}

// Secure filename generation
$secure_filename = hash('sha256', uniqid() . time()) . '.pdf';
```

6. **Session Security Enhancements:**
```php
// Secure session configuration
ini_set('session.cookie_httponly', 1);
ini_set('session.cookie_secure', 1);
ini_set('session.use_strict_mode', 1);
session_regenerate_id(true); // After successful login
```

#### **🟢 MEDIUM PRIORITY**

7. **Input Sanitization for XSS:**
```php
function clean_input($data) {
    $data = trim($data);
    $data = stripslashes($data);
    $data = htmlspecialchars($data, ENT_QUOTES, 'UTF-8');
    return $data;
}
```

8. **API Response Standardization:**
```php
function api_response($success, $data = null, $message = '') {
    header('Content-Type: application/json');
    echo json_encode([
        'success' => $success,
        'data' => $data,
        'message' => $message,
        'timestamp' => date('Y-m-d H:i:s')
    ]);
    exit();
}
```

9. **Add Request Logging:**
```php
function log_request($endpoint, $method, $user_id = null) {
    $log_data = [
        'timestamp' => date('Y-m-d H:i:s'),
        'endpoint' => $endpoint,
        'method' => $method,
        'user_id' => $user_id,
        'ip' => $_SERVER['REMOTE_ADDR'],
        'user_agent' => $_SERVER['HTTP_USER_AGENT']
    ];
    error_log(json_encode($log_data), 3, '/path/to/api.log');
}
```

## 7. Kesimpulan

SIPPOL Reborn memiliki arsitektur API yang sederhana namun **rentan terhadap berbagai serangan keamanan**. Sistem menggunakan pendekatan tradisional dengan session-based authentication dan form handlers, tetapi **tidak mengimplementasikan praktik keamanan modern**.

**Strengths:**
- Session-based authentication yang functional
- Password hashing yang secure
- Basic file upload validation
- Clean separation of concerns

**Critical Weaknesses:**
- Multiple SQL injection vulnerabilities
- No CSRF protection
- Missing rate limiting
- Insufficient input validation
- No API versioning or documentation

**Immediate Actions Required:**
1. Fix all SQL injection vulnerabilities (CRITICAL)
2. Implement CSRF protection (CRITICAL)
3. Add authentication to public endpoints (CRITICAL)
4. Implement rate limiting (HIGH)
5. Enhance file upload security (HIGH)

---
*Analisis ini dibuat berdasarkan review kode dan dapat berubah seiring dengan perkembangan aplikasi. Implementasi perbaikan keamanan harus dilakukan sesegera mungkin.*
