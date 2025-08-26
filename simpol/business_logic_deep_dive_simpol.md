# SIMPOL Business Logic Deep Dive

> **Analisis Mendalam Business Logic Application SIMPOL**  
> 2025-08-26  
> Application: SIMPOL - Sistem Pkl Kominfo  
> Framework: CodeIgniter 4 + MySQL  

---

## 📋 Table of Contents

1. [Complete User Journey](#1-complete-user-journey-untuk-fitur-fitur-utama)
2. [Validation Rules Analysis](#2-validation-rules-analysis)
3. [Permission System & Role Management](#3-permission-system--role-management)
4. [Error Handling Patterns](#4-error-handling-patterns)
5. [Business Calculations & Formulas](#5-business-calculations--formulas)
6. [Security Analysis](#6-security-analysis)
7. [Business Rules Summary](#7-business-rules-summary)

---

## 1. Complete User Journey untuk Fitur-fitur Utama

### 🔐 **Authentication Flow**

#### **Login Process:**
```mermaid
graph in Form]
    B --> C[Submit Credentials]
    C --> D{Validation Pass?}TD
    A[Landing Page] --> B[Log
    D -->|No| E[Show Error Messages]
    E --> B
    D -->|Yes| F[Check User in Database]
    F --> G{User Exists?}
    G -->|No| H[Invalid Credentials Error]
    H --> B
    G -->|Yes| I{Check Password}
    I -->|Invalid| H
    I -->|Valid| J{Check User Level}
    J -->|Level 1| K[Admin Dashboard]
    J -->|Level 2| L[User Dashboard]
    K --> M[Update is_active = 1]
    L --> M
```

**Business Rules:**
- Level 1 = Admin (Kominfo Staff)
- Level 2 = User (PKL Participants)
- `is_active` flag tracks online status
- Password hashing using `PASSWORD_DEFAULT`
- Session stores: `username`, `nama_user`, `level_user`, `id_user`, `id_nilai_user`

#### **Registration Process:**
```php
// Registration Flow (Admin-Only)
1. Admin Login Required ✓
2. Form Validation ✓
3. Duplicate Check (username + email) ✓
4. Password Confirmation Match ✓
5. Password Hashing ✓
6. Default level_user = 2, is_active = 1 ✓
```

### 📝 **Pengajuan (Application) Flow**

#### **Public Application Process:**
```
1. Public Form Access (No Auth Required)
   ↓
2. Fill Application Form:
   - Nama Kelompok (Group Name)
   - Nama Pemohon (Applicant Name) 
   - Email Pemohon
   - Deskripsi Berkas (Description)
   - Date Range (Start - End)
   - File Upload (Multiple files)
   ↓
3. Server-side Validation
   ↓
4. Generate Random ID (000-999)
   ↓
5. Save to db_pengajuan + db_galery_berkas
   ↓
6. Files moved to /public/folder_bio
   ↓
7. Success Message + Redirect to Login
```

**Business Logic:**
- **Random ID**: `rand(000, 999)` - potential collision risk!
- **Required Documents**: Berkas Rekomendasi Bakesbangpol + Form Penilaian
- **File Storage**: Physical files in `/public/folder_bio`
- **No Admin Approval**: Auto-accepted submissions

### 👤 **User Journey (PKL Participants)**

#### **Dashboard Access:**
```
Login (level_user = 2) → User Dashboard → Main Features:

├── 📋 Lihat Tugas (View Tasks)
│   ├── Display all tasks from db_tugas
│   └── Upload Task Files (Multiple)
│       ├── Generate random upload_id
│       ├── Update user.id_upload
│       ├── Save to db_galery_tugas
│       └── Move files to /public/folder_upload
│
├── 📊 Lihat Nilai (View Grades)
│   ├── JOIN db_user + db_nilai
│   ├── Filter by current user session
│   └── Display grade categories + justifications
│
├── 📓 Jurnal Kegiatan (Activity Journal)
│   ├── View Own Journals (Filtered by session.id_user)
│   ├── Status Display: Menunggu/Diterima/Ditolak
│   └── Add New Journal Entry:
│       ├── Date & Time Input
│       ├── Group Name
│       ├── Activity Description
│       ├── Start/End Time
│       └── Default status_jurnal = 0 (Pending)
│
└── 🎓 Cetak Sertifikat (Certificate)
    └── View/Download certificate link from db_sertifikat
```

### 👨‍💼 **Admin Journey (Kominfo Staff)**

#### **Admin Dashboard Features:**
```
Login (level_user = 1) → Admin Dashboard → Management Features:

├── 👥 Kelola User (User Management)
│   ├── View All PKL Participants (level_user = 2)
│   ├── Create New Accounts
│   ├── Edit User Data
│   └── Delete Users (Hard Delete - No Soft Delete!)
│
├── 📋 Kelola Tugas (Task Management)
│   ├── Create Tasks (deskripsi + jadwal)
│   ├── Edit Tasks 
│   ├── Delete Tasks
│   └── Schedule Options: Minggu 1/2/3/4
│
├── 📓 Kelola Jurnal (Journal Management)
│   ├── View All User Journals (JOIN db_user + db_jurnal)
│   ├── Filter by Individual Users
│   ├── Approve Journals (status_jurnal = 1)
│   ├── Reject Journals (status_jurnal = 2)
│   └── Delete Journals (Hard Delete)
│
├── 📊 Report Pengajuan (Application Reports)
│   ├── View All Applications
│   └── Print Reports
│
└── 🎓 Upload Sertifikat (Certificate Management)
    └── Add Certificate Links (Google Drive URLs)
```

---

## 2. Validation Rules Analysis

### 🛡️ **Server-Side Validation (CodeIgniter)**

#### **Login Validation:**
```php
'login' => [
    'username' => 'required|max_length[100]|min_length[3]',
    'password' => 'required|max_length[100]|min_length[3]'
]

// Error Messages:
- "Nama Kelompok tidak boleh kosong!" (Username)
- "Password tidak boleh kosong!"
- Min 3, Max 100 characters
```

#### **Registration Validation:**
```php
'register' => [
    'username'   => 'required|max_length[100]|min_length[3]',
    'nama_user'  => 'required|max_length[100]|min_length[3]', 
    'password'   => 'required|max_length[255]|min_length[3]',
    'email_user' => 'required|valid_email'
]

// Custom Business Logic:
✓ Duplicate Check: username + email combination
✓ Password Confirmation Match (client-side only)
✓ Password Hashing with cost=10
```

#### **Pengajuan (Application) Validation:**
```php
'pengajuan' => [
    'nama_kelompok'     => 'required|max_length[100]|min_length[3]',
    'nama_pemohon'      => 'required|max_length[100]|min_length[3]',
    'deskripsi_berkas'  => 'required|max_length[1000]|min_length[3]',
    'date_mulai'        => 'required',
    'date_selesai'      => 'required', 
    'email_pemohon'     => 'required|valid_email'
]

// File Upload Validation:
⚠️ MISSING: File type validation
⚠️ MISSING: File size limits  
⚠️ MISSING: File count limits
```

#### **Task Management Validation:**
```php
'tugas' => [
    'deskripsi_tugas' => 'required|max_length[255]',
    'jadwal_tugas'    => 'required|max_length[255]'
]

// Hardcoded Schedule Options:
- Minggu Pertama, Kedua, Ketiga, Keempat
```

#### **Grade Validation (Commented Out):**
```php
// Unused validation rules for grade system
'nilai' => [
    'nilai_1-10' => 'required|max_length[3]|integer'
]
// Implies grade range: 1-100 (3 digit max)
```

### 🔍 **Client-Side Validation**

#### **Minimal Client-Side Checks:**
```javascript
// Date Range Picker (Pengajuan form)
setDateRangePicker(".startdate", ".enddate")

// HTML5 Validation:
- required attributes on form fields
- type="email" for email validation
- type="password" for password fields

⚠️ MISSING: JavaScript validation for most forms
⚠️ MISSING: Real-time validation feedback
⚠️ MISSING: File upload validation
```

### ❌ **Critical Validation Gaps:**

1. **File Upload Security:**
   - No file type whitelist
   - No file size limits
   - No malware scanning
   - Directory traversal potential

2. **Date Validation:**
   - No date range validation (start < end)
   - No future date restrictions
   - VARCHAR storage instead of DATE type

3. **Business Logic Validation:**
   - No duplicate journal entry checks
   - No maximum file upload per user
   - No task deadline validation

---

## 3. Permission System & Role Management

### 🔐 **Role-Based Access Control (RBAC)**

#### **User Levels:**
```php
// Database: db_user.level_user
LEVEL_1 = 1  // Admin (Kominfo Staff)
LEVEL_2 = 2  // User (PKL Participants) 

// User Status:
is_active = 1  // Online/Active
is_active = 0  // Offline/Inactive
```

#### **Authorization Implementation:**

**⚠️ CRITICAL SECURITY FLAW:**
```php
// All controllers use basic session check:
if (session()->get('username') == '') {
    session()->setFlashData('gagal', 'Lakukan login terlebih dahulu');
    return redirect()->to(base_url('Login'));
}

// ❌ MISSING: Role-based authorization
// ❌ MISSING: Permission granularity  
// ❌ MISSING: Resource-level access control
```

#### **Authentication Flow:**
```php
// Login.php - Level-based routing
if (level_user == 2) {
    // PKL Participants
    redirect()->to('User/user_home')
} else if (level_user == 1) {
    // Admin
    redirect()->to('Admin/admin_home')
}

// Session Data Stored:
- username, nama_user, level_user
- id_user, id_upload, id_nilai_user
```

#### **Access Control Matrix:**

| Feature | Public | User (L2) | Admin (L1) |
|---------|--------|-----------|------------|
| **Landing Page** | ✅ | ✅ | ✅ |
| **Pengajuan Form** | ✅ | ❌ | ❌ |
| **Login** | ✅ | ✅ | ✅ |
| **User Dashboard** | ❌ | ✅ | ❌ |
| **Admin Dashboard** | ❌ | ❌ | ✅ |
| **Task Management** | ❌ | ❌ | ✅ |
| **User Registration** | ❌ | ❌ | ✅ |
| **Journal Approval** | ❌ | ❌ | ✅ |
| **View Own Grades** | ❌ | ✅ | ❌ |
| **Submit Journals** | ❌ | ✅ | ❌ |
| **Upload Task Files** | ❌ | ✅ | ❌ |

#### **Session Management:**
```php
// Login sets session + updates is_active = 1
session()->set([
    'username' => $user['username'],
    'nama_user' => $user['nama_user'], 
    'level_user' => $user['level_user'],
    'id_user' => $user['id_user']
]);

// Logout updates is_active = 0 + destroys session
$data = ['is_active' => 0];
$this->LoginModel->update_user($data, session()->get('id_user'));
session()->setTempdata('username');  // ⚠️ Wrong method usage!
```

### 🚨 **Security Vulnerabilities:**

1. **Horizontal Privilege Escalation:**
   - Users can access other users' data by changing URL parameters
   - No ownership verification in controllers

2. **Vertical Privilege Escalation:**
   - Session hijacking can grant admin access
   - No additional admin authentication checks

3. **Session Security:**
   - No session timeout
   - No concurrent session limits
   - Wrong session destruction method

4. **Authorization Bypass:**
   - Direct URL access not properly restricted
   - CSRF tokens missing from forms

---

## 4. Error Handling Patterns

### 📢 **FlashData Message System**

#### **Success Messages:**
```php
// Standard Success Pattern:
session()->setFlashdata('success', 'Message Text');
return redirect()->to(base_url('ControllerName/method'));

// Examples:
- "Data Berhasil di tambahkan"
- "Berhasil membuat akun baru" 
- "Jurnal berhasil ditambahkan"
- "User berhasil di update"
- "Jurnal diterima/ditolak"
```

#### **Error Messages:**
```php
// Validation Errors:
session()->setFlashdata('inputs', $this->request->getPost());
session()->setFlashdata('errors', $validation->getErrors());

// Business Logic Errors:
session()->setFlashdata('gagal', 'Error Message');
session()->setFlashdata('gagal_register', 'Registration Error');

// Examples:
- "Username and Password salah!"
- "Username and Email sudah pernah digunakan!"
- "Password not matched"
- "Lakukan login terlebih dahulu"
```

#### **Special Messages:**
```php
// Application Success:
session()->setFlashdata('success_pengajuan', 
    'Pengajuan Sedang Proses,Silahkan Menunggu Pengumuman di Grup Pkl');
```

### 🎨 **Frontend Error Display:**

#### **Error Message Template:**
```php
// Consistent error display pattern across all views:
<?php
$errors = session()->getFlashdata('errors');
if (!empty($errors)) { ?>
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        <ul>
            <?php foreach ($errors as $error) { ?>
                <li><?= esc($error) ?></li>
            <?php } ?>
        </ul>
        <button type="button" class="close" data-dismiss="alert">
            <span>&times;</span>
        </button>
    </div>
<?php } ?>
```

#### **Success Message Template:**
```php
<?php if (!empty(session()->getFlashdata('success'))) { ?>
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        <?= session()->getFlashdata('success'); ?>
        <button type="button" class="close" data-dismiss="alert">
            <span>&times;</span>
        </button>
    </div>
<?php } ?>
```

### ⚠️ **Error Handling Issues:**

#### **1. Inconsistent Error Types:**
```php
// Multiple error flashdata keys:
- 'gagal' (general failures)
- 'gagal_register' (registration specific)
- 'errors' (validation errors)
- 'success_pengajuan' (application specific)

// ❌ Should be standardized
```

#### **2. Missing Error Handling:**
```php
// Database Errors:
❌ No try-catch blocks in controllers
❌ No database connection error handling
❌ No foreign key constraint error handling

// File Upload Errors:
❌ No file size limit errors
❌ No invalid file type errors  
❌ No disk space errors
❌ No permission errors
```

#### **3. Error Information Leakage:**
```php
// Debug information in production:
⚠️ Database errors may expose schema info
⚠️ File path errors may expose server structure
⚠️ No custom error pages (404, 500)
```

#### **4. User Experience Issues:**
```php
// Poor error UX:
❌ No field-specific error highlighting
❌ No inline validation feedback
❌ No loading states during form submission
❌ No confirmation dialogs for destructive actions
```

---

## 5. Business Calculations & Formulas

### 📊 **Grading System (Commented Out)**

#### **Grade Categories (Historical):**
```php
// From commented code in AdminModel:
$grade_categories = [
    'Disiplin',
    'Kejujuran', 
    'Inisiatif',
    'Adaptasi',
    'Kerjasama',
    'Komunikasi',
    'Tanggung Jawab',
    'Kesopanan',
    'Kehadiran',
    'Pengerjaan Tugas'
];

// Grade Range: 1-100 (integer validation)
// Each category has: nilai_mhs + justifikasi_mhs
```

#### **Grade Storage Logic:**
```php
// Batch insert for efficiency:
$random = rand(000, 999);  // Grade set ID

// Update user with grade reference:
$data_nilai = ['id_nilai_user' => $random];
$this->AdminModel->update_nilai_user($data_nilai, $id_user);

// Insert all 10 grades in single batch:
$data_upload = [
    ['id_nilai_user' => $random, 'nilai_mhs' => $nilai1, 
     'kategori_mhs' => 'Disiplin', 'justifikasi_mhs' => $justifikasi1],
    // ... repeat for all 10 categories
];
$this->AdminModel->insert_nilai($data_upload);
```

### 🔢 **ID Generation Formulas**

#### **Random ID Pattern:**
```php
// Used across multiple features:
$random = rand(000, 999);

// Usage:
- Pengajuan ID (db_pengajuan.id_pengajuan)
- Upload Batch ID (db_pengajuan.id_upload_berkas)
- User Grade Set ID (db_user.id_nilai_user)
- Task Upload ID (db_user.id_upload)

// ⚠️ COLLISION RISK: Only 1000 possible values!
```

#### **File Naming Formula:**
```php
// Random file names to prevent conflicts:
$filename = $img->getRandomName();

// CodeIgniter generates: [random].[original_extension]
// Example: "1a2b3c4d5e6f.jpg"
```

### ⏰ **Time/Date Calculations**

#### **Default Timestamps:**
```sql
-- Database auto-timestamps:
date_create DEFAULT CURRENT_TIMESTAMP
dateline_tugas DEFAULT CURRENT_TIMESTAMP

-- Manual timestamp storage (⚠️ VARCHAR format):
datetime_jurnal VARCHAR(255)  -- Should be DATETIME
```

#### **Session Duration:**
```php
// No explicit session timeout calculation
// Relies on PHP default session settings
// is_active flag tracks online status manually
```

### 📈 **Status Calculations**

#### **Journal Status Logic:**
```php
// Status enumeration:
status_jurnal = 0  // "Menunggu" (Pending)
status_jurnal = 1  // "Diterima" (Approved) 
status_jurnal = 2  // "Ditolak" (Rejected)

// Display logic in views:
if ($status == 0) echo 'Menunggu';
else if ($status == 1) echo 'Diterima';
else echo 'Ditolak';
```

#### **User Activity Status:**
```php
// Login/Logout calculations:
Login:  is_active = 1, update timestamp
Logout: is_active = 0, clear session

// No automatic timeout calculation
```

### 🚨 **Calculation Issues:**

#### **1. Random ID Collisions:**
```php
// High collision probability:
rand(000, 999) = only 1000 possibilities
// Should use: uniqid(), UUID, or auto-increment
```

#### **2. Missing Business Calculations:**
```php
❌ No PKL duration calculation (start_date - end_date)
❌ No attendance percentage 
❌ No task completion rate
❌ No average grade calculation
❌ No progress tracking metrics
```

#### **3. Data Type Issues:**
```php
// Incorrect data types affect calculations:
datetime_jurnal VARCHAR(255)  // Can't do date math
date_mulai VARCHAR(100)       // Can't calculate duration
date_selesai VARCHAR(100)     // Can't validate ranges
```

---

## 6. Security Analysis

### 🔒 **Password Security**

#### **✅ Secure Practices:**
```php
// Strong password hashing:
$option = ['cost' => 10];
$hash_pass = password_hash($password, PASSWORD_DEFAULT, $option);

// Proper verification:
password_verify($password, $stored_hash)
```

#### **⚠️ Security Gaps:**
```php
// Password policy issues:
❌ Minimum length only 3 characters
❌ No complexity requirements
❌ No password rotation policy
❌ No account lockout after failed attempts
```

### 🛡️ **Input Validation Security**

#### **✅ Some Protection:**
```php
// HTML entity escaping in views:
<?= esc($error) ?>

// CodeIgniter validation rules prevent basic injection
```

#### **❌ Major Vulnerabilities:**
```php
// SQL Injection potential:
⚠️ Using Query Builder reduces risk but not eliminated
❌ No prepared statements for complex queries

// File Upload Vulnerabilities:
❌ No file type validation
❌ No malware scanning  
❌ Executable files can be uploaded
❌ Directory traversal possible

// XSS Vulnerabilities:
❌ User input not always escaped
❌ No Content Security Policy headers
❌ Flash messages may contain unescaped content
```

### 🚪 **Session Security**

#### **⚠️ Session Issues:**
```php
// Weak session management:
❌ No session regeneration after login
❌ No session timeout implementation
❌ Wrong session destruction method: setTempdata()
❌ Session hijacking possible
❌ No HTTPS enforcement
```

### 🔐 **Authorization Security**

#### **❌ Critical Authorization Flaws:**
```php
// Broken access control:
❌ No resource ownership verification
❌ Direct object references not protected
❌ Horizontal privilege escalation possible
❌ No CSRF protection
❌ Missing role-based middleware
```

---

## 7. Business Rules Summary

### 📋 **Core Business Rules**

#### **User Management Rules:**
1. **Registration**: Admin-only user creation
2. **User Levels**: Level 1 = Admin, Level 2 = PKL Participant
3. **Unique Constraints**: Username + Email combination must be unique
4. **Default Status**: New users created as active (is_active = 1)
5. **Session Tracking**: is_active flag tracks online status

#### **Application (Pengajuan) Rules:**
1. **Public Access**: Anyone can submit PKL applications
2. **Required Documents**: Berkas Rekomendasi Bakesbangpol + Form Penilaian
3. **Date Range**: Must specify start and end dates
4. **File Storage**: Files stored in `/public/folder_bio`
5. **Auto-Processing**: No manual approval required

#### **Task Management Rules:**
1. **Admin Creation**: Only admins can create/edit/delete tasks
2. **Schedule System**: Tasks assigned to weekly periods (Minggu 1-4)
3. **User Uploads**: PKL participants can upload task completion files
4. **File Storage**: Task files stored in `/public/folder_upload`
5. **Multiple Files**: Users can upload multiple files per task

#### **Journal Management Rules:**
1. **User Creation**: PKL participants create daily activity journals
2. **Default Status**: New journals start as "Menunggu" (status = 0)
3. **Admin Approval**: Admins can approve (1) or reject (2) journals
4. **Time Tracking**: Journals include start/end times for activities
5. **Group Activity**: Journals associated with group names

#### **Grading Rules (Inactive):**
1. **10 Categories**: Disiplin, Kejujuran, Inisiatif, etc.
2. **Grade Range**: 1-100 integer values
3. **Justification Required**: Each grade must have justification text
4. **Batch Processing**: All grades for a user saved together
5. **Admin Only**: Only admins can assign grades

#### **Certificate Rules:**
1. **Link-Based**: Certificates are Google Drive links
2. **Admin Upload**: Only admins can add certificate links
3. **Public Access**: PKL participants can view/download certificates
4. **Single Certificate**: One certificate link per batch

### ⚠️ **Business Rule Violations:**

#### **Data Integrity Issues:**
1. **Random ID Collisions**: rand(000,999) has high collision probability
2. **No Foreign Key Constraints**: Orphaned data possible
3. **Hard Deletes**: No soft deletion for audit trails
4. **Missing Validation**: No business rule validation at database level

#### **Security Rule Violations:**
1. **Authorization Bypass**: Users can access others' data via URL manipulation
2. **File Upload Risks**: No file type or size validation
3. **Session Issues**: Improper session management allows hijacking
4. **Password Policy**: Weak password requirements (min 3 chars)

#### **Business Logic Gaps:**
1. **No Workflow States**: Missing application approval workflow
2. **No Deadlines**: Tasks have no due dates or enforcement
3. **No Capacity Limits**: No maximum participants or file sizes
4. **No Audit Trail**: Changes not tracked for compliance

#### **User Experience Issues:**
1. **Poor Error Messages**: Generic error messages not user-friendly
2. **No Confirmation**: Destructive actions lack confirmation dialogs
3. **Limited Feedback**: No progress indicators or status updates
4. **Mobile Unfriendly**: No responsive design considerations

---

## 🎯 **RECOMMENDATIONS FOR IMPROVEMENT**

### 🔥 **Critical Fixes:**

1. **Fix ID Generation:**
   ```php
   // Replace rand(000,999) with:
   $id = uniqid() or use AUTO_INCREMENT primary keys
   ```

2. **Implement Proper Authorization:**
   ```php
   // Add resource ownership checks:
   if ($data['user_id'] !== session()->get('id_user')) {
       throw new UnauthorizedException();
   }
   ```

3. **Secure File Uploads:**
   ```php
   // Add file validation:
   $allowedTypes = ['pdf', 'doc', 'docx', 'jpg', 'png'];
   $maxSize = 5 * 1024 * 1024; // 5MB
   ```

4. **Fix Session Management:**
   ```php
   // Proper session destruction:
   session()->destroy();
   // Add session timeout and regeneration
   ```

### ⚡ **High Priority Improvements:**

1. **Add CSRF Protection**
2. **Implement Input Sanitization**
3. **Add Database Foreign Keys**
4. **Create Proper Error Pages**
5. **Add Audit Logging**

### 💡 **Future Enhancements:**

1. **Workflow Management System**
2. **Real-time Notifications**  
3. **Mobile-Responsive Design**
4. **Advanced Reporting Dashboard**
5. **API for Mobile App Integration**

---

*🔧 Analysis method: Manual code review + business logic tracing*  
*📊 Application: SIMPOL PKL Management System*
