# SIPPOL Reborn - Business Logic Deep Dive

## 1. Complete User Journeys for Main Features

### 1.1 Student (Pemagang) User Journey

#### Registration & Setup Journey
1. **Account Creation** (Admin-managed)
   - Admin creates account via `admin/handler/handler-add-pemagang.php`
   - Required fields: username, password, nama, npm, asal, mulai_magang, akhir_magang, status
   - Password is hashed using `password_hash()` with `PASSWORD_DEFAULT`
   - Institution lookup via subquery: `(SELECT id_instansi FROM instansi WHERE nama_instansi = '$asal')`

2. **Initial Login**
   - User accesses `login.php`
   - Credentials validated against database
   - Password verification using `password_verify()`
   - Account status check (status = 0 → account disabled)
   - Session creation: `$_SESSION['login_user'] = $username`

3. **Profile Completion Check**
   - System verifies if nama, npm, or instansi fields are empty
   - If incomplete: User redirected to profile completion
   - Profile update via `handler-edit-profil.php`

#### Daily Attendance Journey
1. **Dashboard Access** (`index.php`)
   - Session validation
   - User status verification (disabled accounts logged out)
   - Profile completeness check
   - Attendance eligibility calculation

2. **Attendance Logic Calculation**
   ```php
   // Business rule: Maximum 2 attendances per day (Check In + Check Out)
   $query_presensi = "SELECT COUNT(*) as total_presensi, MAX(data_status) as last_status 
                      FROM presensi WHERE id_user = (SELECT id_user FROM user WHERE username = '$username') 
                      AND DATE(data_tanggal) = '$current_date'";
   ```

3. **Attendance State Machine**
   - **No attendance**: Show "Check In" option (status = 5)
   - **1 attendance with status 5**: Show "Check Out" option (status = 6)  
   - **1 attendance with status ≠ 5**: Show "Check In" option (error recovery)
   - **2+ attendances**: Block further attendance

4. **Attendance Submission** (`handler-presensi.php`)
   - Input sanitization using `mysqli_real_escape_string()`
   - Timestamp generation: `date("Y-m-d H:i:s")` with Asia/Jakarta timezone
   - Database insertion with user ID lookup

### 1.2 Admin User Journey

#### Authentication Flow
1. **Admin Login** (`admin/login.php`)
   - Separate authentication system from students
   - Session variable: `$_SESSION["isLogin"] = true`
   - Enhanced security check in all admin handlers

#### Institution Management Journey
1. **Institution Creation** (`admin/handler/handler-add-instansi.php`)
   - Required fields: nama_instansi, alamat_instansi
   - Basic validation for required fields
   - Direct database insertion

#### Student Management Journey
1. **Student Account Creation**
   - Form validation for all required fields
   - Institution lookup and validation
   - Password hashing before storage
   - Status assignment (1 = active, 0 = inactive)

2. **Student Account Modification** (`admin/handler/handler-edit-pemagang.php`)
   - Conditional password update (only if new password provided)
   - Institution ID resolution via name lookup
   - Status management for account activation/deactivation

#### Document Management Journey
1. **Pengajuan (Application) Management** (`admin/handler/handler-add-pengajuan.php`)
   - **File Upload Validation**:
     - Maximum file size: 2MB per file
     - Allowed extensions: PDF only
     - File existence check (no duplicates)
   - **Multi-file Processing**: Supports batch upload of pengajuan and balasan documents
   - **File Storage**: Separate directories for pengajuan and balasan files
   - **Database Storage**: File paths stored with metadata (year, description)

#### Reporting & Export Journey
1. **Data Export** (`admin/handler/handler-export.php`)
   - **Filter Options**:
     - Month, Year, Date, Institution
     - Dynamic SQL query building based on selected filters
   - **Export Format**: CSV with UTF-8 encoding
   - **Data Transformation**:
     - Status codes converted to human-readable format (5→"Presensi Masuk", 6→"Presensi Keluar")
     - Date formatting for readability
     - NPM and description padding for better CSV formatting

## 2. Validation Rules

### 2.1 Client-Side Validation

#### Attendance Form Validation
```html
<!-- All form fields marked as required -->
<input type="text" required>
<select required></select>
<textarea required></textarea>
```

#### Export Form Validation
- Date input: `min="1" max="31"`
- Year dropdown: Dynamically generated (current year - 4 years)
- Month selection: Dropdown with Indonesian month names

### 2.2 Server-Side Validation

#### Authentication Validation
```php
// Password verification
if (password_verify($password, $row['password'])) {
    // Login successful
}

// Account status validation
if ($row['status'] == 0) {
    $error = "Akun anda dinonaktifkan, hubungi Administrator!";
}
```

#### File Upload Validation
```php
// File size validation (2MB limit)
if ($srt_pengajuan_files["size"][$i] > $maxFileSize || 
    $srt_balasan_files["size"][$i] > $maxFileSize) {
    $_SESSION["error_message_admin"] = "Ukuran file tidak boleh melebihi 2MB.";
}

// File extension validation
$allowedExtensions = ["pdf"];
if (!in_array($extension, $allowedExtensions)) {
    $_SESSION["error_message_admin"] = "File harus berupa PDF.";
}

// File existence validation
if (file_exists($target_file)) {
    $_SESSION["error_message_admin"] = "Nama file tidak boleh sama.";
}
```

#### Data Sanitization
```php
// SQL injection prevention
$nama = mysqli_real_escape_string($koneksi, $nama);
$npm = mysqli_real_escape_string($koneksi, $npm);
$asal = mysqli_real_escape_string($koneksi, $asal);
```

#### Profile Update Validation
```php
// Password confirmation validation
if (!empty($password_baru) && $password_baru !== $konfirmasi_password) {
    $_SESSION['error_message'] = "Konfirmasi password tidak sesuai.";
}
```

## 3. Permission System and Role Management

### 3.1 Role Definitions
- **Student Role**: Regular users (pemagang)
  - Session variable: `$_SESSION['login_user']`
  - Access: Personal attendance, profile management
  
- **Admin Role**: Administrative users
  - Session variable: `$_SESSION["isLogin"] = true`
  - Access: All management functions, reports, exports

### 3.2 Authentication Mechanisms

#### Student Authentication
```php
// Session check in index.php
if (!isset($_SESSION['login_user'])) {
    header("Location: login.php");
    exit();
}

// Account status enforcement
$sql = "SELECT status FROM user WHERE username = '$username'";
if ($status == 0) {
    unset($_SESSION['login_user']); // Force logout
}
```

#### Admin Authentication
```php
// Admin session check (used in all admin handlers)
if (!isset($_SESSION["isLogin"]) || $_SESSION["isLogin"] === false) {
    header("Location: ../login.php");
    exit();
}
```

### 3.3 Authorization Matrix

| Feature | Student | Admin |
|---------|---------|-------|
| Personal Attendance | ✓ | ✗ |
| View Own Profile | ✓ | ✗ |
| Edit Own Profile | ✓ | ✗ |
| Manage Students | ✗ | ✓ |
| Manage Institutions | ✗ | ✓ |
| Manage Applications | ✗ | ✓ |
| Export Reports | ✗ | ✓ |
| System Configuration | ✗ | ✓ |

## 4. Error Handling Patterns

### 4.1 Session-Based Error Management
The application uses PHP sessions to manage error and success messages across page redirects.

#### Error Message Structure
```php
// Error messages
$_SESSION['error_message'] = "Error message for students";
$_SESSION['error_message_admin'] = "Error message for admins";

// Success messages  
$_SESSION['success_message'] = "Success message for students";
$_SESSION['success_message_admin'] = "Success message for admins";
```

#### Template-Based Display
```php
// template-alert.php and admin/template-alert.php
<?php if(isset($_SESSION['error_message'])): ?>
    <div class="alert alert-danger alert-dismissible fade show border-0" role="alert">
        <b>Error!</b> <?php echo $_SESSION['error_message']; ?>
    </div>
    <?php unset($_SESSION['error_message']); ?>
<?php endif; ?>
```

### 4.2 Database Error Handling

#### Connection Error Handling
```php
// Basic error reporting for database queries
if (mysqli_query($koneksi, $query)) {
    $_SESSION["success_message"] = "Operation successful";
} else {
    $_SESSION["error_message"] = "Operation failed";
    // Some handlers include: . mysqli_error($koneksi)
}
```

#### Query Error Patterns
- **File Operations**: Specific error messages for upload failures, file size limits, extension restrictions
- **Data Validation**: Clear messages for validation failures (password mismatch, required fields, etc.)
- **Business Logic**: Custom error messages for business rule violations (account disabled, attendance limits, etc.)

### 4.3 Error Recovery Mechanisms

#### Authentication Recovery
- Automatic logout for disabled accounts
- Session cleanup on authentication failures
- Redirect to appropriate login page based on user type

#### File Upload Recovery
- Validation before file operations to prevent partial uploads
- Rollback mechanisms for batch operations
- Clear error messages indicating specific failure points

## 5. Business Calculations and Formulas

### 5.1 Attendance Calculations

#### Daily Attendance Limit Logic
```php
// Maximum 2 attendances per day calculation
$query_presensi = "SELECT COUNT(*) as total_presensi, MAX(data_status) as last_status 
                   FROM presensi 
                   WHERE id_user = (SELECT id_user FROM user WHERE username = '$username') 
                   AND DATE(data_tanggal) = '$current_date'";

// State determination logic
if ($total_presensi < 1) {
    $options = "<option value='5'>Check In</option>";
} elseif ($total_presensi == 1) {
    if ($last_status == 5) {
        $options = "<option value='6'>Check Out</option>";
    } else {
        $options = "<option value='5'>Check In</option>"; // Recovery option
    }
} else {
    // Block further attendance - business rule enforced
}
```

#### Status Code Mapping
- **Status 5**: Check In (Presensi Masuk)  
- **Status 6**: Check Out (Presensi Keluar)

### 5.2 Export Data Processing

#### Date Range Filtering
```php
// Dynamic SQL building for export filters
$sql = "SELECT * FROM presensi WHERE 1 = 1";

if (!empty($selected_month)) {
    $sql .= " AND MONTH(data_tanggal) = $selected_month";
}
if (!empty($selected_year)) {
    $sql .= " AND YEAR(data_tanggal) = $selected_year";
}
if (!empty($selected_date)) {
    $sql .= " AND DAY(data_tanggal) = $selected_date";
}
```

#### Data Transformation for Export
```php
// Status code to human readable conversion
if ($row['data_status'] == 5) {
    $jenis_presensi = 'Presensi Masuk';
} elseif ($row['data_status'] == 6) {
    $jenis_presensi = 'Presensi Keluar';
}

// Date formatting for export
$date_obj = DateTime::createFromFormat('Y-m-d H:i:s', $row['data_tanggal']);
$formatted_date = $date_obj->format(' F d Y H:i:s');

// Data padding for better CSV formatting
str_pad($row['data_npm'], 50, " ");
str_pad($row['data_deskripsi'], 50, " ");
```

### 5.3 File Size Calculations

#### Upload Validation
```php
$maxFileSize = 2 * 1024 * 1024; // 2MB in bytes
if ($file_size > $maxFileSize) {
    // Reject upload
}
```

### 5.4 Time-based Calculations

#### Timezone Management
```php
date_default_timezone_set('Asia/Jakarta');
$data_tanggal = date("Y-m-d H:i:s"); // Current timestamp in Jakarta timezone
```

#### Year Range Generation for Reports
```javascript
// Generate year dropdown for last 5 years including current
var currentYear = currentDate.getFullYear();
for (var i = currentYear; i >= currentYear - 4; i--) {
    // Add year option
}
```

## 6. Key Business Rules Summary

1. **Attendance Rules**:
   - Maximum 2 attendances per day per user
   - Must complete profile before attendance
   - Sequential Check In → Check Out flow enforced
   - Disabled accounts automatically logged out

2. **File Management Rules**:
   - PDF files only for document uploads
   - 2MB maximum file size
   - No duplicate file names allowed
   - Batch processing for multiple documents

3. **User Management Rules**:
   - Password hashing required for all accounts
   - Institution must exist before assigning to users
   - Account status controls access (0=disabled, 1=active)

4. **Data Export Rules**:
   - Admin-only access to export functionality
   - Multiple filter criteria supported
   - Data formatting for readability in reports

5. **Security Rules**:
   - Session-based authentication
   - SQL injection prevention through sanitization
   - Role-based access control
   - Automatic session cleanup for compromised accounts

---

*Generated on: 2025-01-26*  
*System: SIPPOL Reborn - Sistem Informasi Presensi Politik Reborn Kediri*
