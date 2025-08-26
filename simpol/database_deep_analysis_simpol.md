# SIMPOL Database Deep Analysis

> **Analisis Mendalam Database Project SIMPOL**  
> 2025-08-26  
> Database: MySQL/MariaDB (db_ci)  
> Framework: CodeIgniter 4  

---

## 📋 Table of Contents

1. [ERD (Entity Relationship Diagram)](#1-erd-entity-relationship-diagram)
2. [Analisa Query Kompleks](#2-analisa-query-kompleks-yang-ada-di-model-model)
3. [Performance Issues](#3-identifikasi-potential-performance-issues)
4. [Business Rules](#4-business-rules-yang-diimplementasi-di-database-level)
5. [Index Analysis](#5-catalog-semua-index-dan-penggunaannya)
6. [Rekomendasi Prioritas](#-rekomendasi-prioritas)

---

## 1. ERD (Entity Relationship Diagram)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        SIMPOL Database - Entity Relationship Diagram                │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────┐           ┌─────────────────────────┐           ┌─────────────────────────┐
│       db_user           │ 1    *    │       db_jurnal         │           │       db_nilai          │
├─────────────────────────┤───────────├─────────────────────────┤           ├─────────────────────────┤
│ PK id_user (int)        │           │ PK id_jurnal (int)      │           │ PK id_nilai_mhs (int)   │
│    username (varchar)   │           │ FK id_user (int)        │     ┌─────┤    justifikasi_mhs      │
│    password (varchar)   │           │    datetime_jurnal      │     │     │    kategori_mhs         │
│    nama_user (varchar)  │           │    kelompok_jurnal      │     │     │    nilai_mhs (int)      │
│    email_user (varchar) │           │    deskripsi_jurnal     │     │     │ FK id_nilai_user (int)  │
│    date_create (datetime)│          │    mulai_jurnal         │     │     └─────────────────────────┘
│    level_user (int)     │           │    selesai_jurnal       │     │               │
│    is_active (int)      │           │    status_jurnal (int)  │     │               │ *
│ FK id_upload (int)      │           └─────────────────────────┘     │               │
│ FK id_data_bio (int)    │                                           │               │ 1
│ FK id_jurnal_kelompok   │                                           │               │
│ FK id_nilai_user (int)  │───────────────────────────────────────────┘               │
└─────────────────────────┘                                                           │
          │ 1                                                                         │
          │                                                                           │
          │ *                                                                         │
┌─────────────────────────┐           ┌─────────────────────────┐                     │
│   db_galery_tugas       │           │       db_tugas          │                     │
├─────────────────────────┤           ├─────────────────────────┤                     │
│ PK id_galery_tugas      │     *     │ PK id_tugas (int)       │                     │
│ FK id_upload (int)      │───────────│    deskripsi_tugas      │                     │
│    gambar_tugas         │           │    jadwal_tugas         │                     │
│    dateline_tugas       │           └─────────────────────────┘                     │
└─────────────────────────┘                                                           │
                                                                                      │
┌─────────────────────────┐                                                           │
│     db_galery           │                                                           │
├─────────────────────────┤                                                           │
│ PK id_galery (int)      │                                                           │
│ FK id_img (int)         │                                                           │
│    gambar_mhs (varchar) │                                                           │
└─────────────────────────┘                                                           │
          │ *                                                                         │
          │                                                                           │
          │ 1 (implied)                                                               │
┌─────────────────────────┐                                                           │
│   db_galery_berkas      │                                                           │
├─────────────────────────┤                                                           │
│ PK id_berkas (int)      │                                                           │
│    data_berkas          │                                                           │
│ FK id_upload_berkas     │───────────┐                                               │
└─────────────────────────┘           │                                               │
                                      │ 1                                             │
                                      │                                               │
                                      │ *                                             │
┌─────────────────────────┐           │           ┌─────────────────────────┐         │
│    db_pengajuan         │───────────┘           │    db_sertifikat        │         │
├─────────────────────────┤                       ├─────────────────────────┤         │
│ PK id_pengajuan (int)   │                       │ PK id_sertifikat (int)  │         │
│    nama_kelompok        │                       │    link_sertifikat      │         │
│    nama_pemohon         │                       └─────────────────────────┘         │
│    email_pemohon        │                                                           │
│    deskripsi_berkas     │                                                           │
│    date_mulai           │                                                           │
│    date_selesai         │                                                           │
│ FK id_upload_berkas     │                                                           │
└─────────────────────────┘                                                           │
                                                                                      │
                                                                                      │
                                  Relationship Type Legend:                           │
                                  ────────── : Direct Foreign Key                     │
                                  ┌─────────┐: Implied/Weak Relationship              │
                                                                                      │
                                  Notes:                                              │
                                  - Some FK relationships exist in schema but         │
                                    no explicit constraints defined                   │
                                  - db_user has multiple FK fields that point         │
                                    to other entities                                 │
                                  - Missing tables referenced in JoinMahasiswaModel   │
                                    (db_mhs, db_fakultas, db_jurusan)                 │
```

---

## 2. Analisa Query Kompleks yang Ada di Model-model

### Query Kompleks dalam Model:

#### 🔍 **AdminModel.php**

**JOIN Query untuk Nilai:**
```php
// Baris 47-52: JOIN antara db_user dan db_nilai
public function get_join_nilai()
{
    return $this->db->table('db_user')
        ->join('db_nilai', 'db_nilai.id_nilai_user = db_user.id_nilai_user')
        ->get()->getResultArray();
}
```

**JOIN Query untuk Jurnal:**
```php  
// Baris 78-83: JOIN antara db_user dan db_jurnal
public function get_join_jurnal()
{
    return $this->db->table('db_user')
        ->join('db_jurnal', 'db_jurnal.id_user = db_user.id_user')
        ->get()->getResultArray();
}
```

**Batch Insert untuk Nilai:**
```php
// Baris 37-40: Insert batch untuk efisiensi
public function insert_nilai($data)
{
    return $this->db->table('db_nilai')->insertBatch($data);
}
```

#### 🔍 **JoinMahasiswaModel.php**

**⚠️ Triple JOIN Query (BERMASALAH!):**
```php
// Baris 11-17: JOIN 3 tabel sekaligus (MASALAH: tabel tidak ada di schema!)
public function get_mahasiswa()
{
    return $this->db->table('db_mhs')
        ->join('db_fakultas', 'db_fakultas.id_fakultas = db_mhs.id_fakultas')
        ->join('db_jurusan', 'db_jurusan.id_jurusan = db_mhs.id_jurusan')
        ->get()->getResultArray();
}
```
> ❌ **CRITICAL ERROR**: Tabel `db_mhs`, `db_fakultas`, dan `db_jurusan` tidak ada dalam schema database!

#### 🔍 **UserModel.php**

**Duplikasi JOIN Query:**
```php
// Baris 47-52 & 55-60: Query JOIN yang sama seperti di AdminModel
public function get_join_jurnal() // ❌ Duplikasi dari AdminModel
public function get_join_nilai()  // ❌ Duplikasi dari AdminModel
```

#### 🔍 **RegisterModel.php**

**Multi-condition WHERE Query:**
```php
// Baris 11-16: Query dengan multiple WHERE conditions
public function cek_register($username, $email_user)
{
    return $this->db->table('db_user')
        ->where(array('username' => $username, 'email_user' => $email_user))
        ->get()->getRowArray();
}
```

---

## 3. Identifikasi Potential Performance Issues

### 🚨 **CRITICAL Performance Issues:**

#### 1. **Missing Indexes untuk JOIN Operations**
```sql
❌ TIDAK ADA INDEX pada kolom JOIN:
- db_nilai.id_nilai_user (untuk JOIN dengan db_user)
- db_jurnal.id_user (untuk JOIN dengan db_user)  
- db_galery.id_img (untuk JOIN operations)
```

#### 2. **No Foreign Key Constraints**
```sql
❌ Tidak ada FOREIGN KEY constraints sama sekali!
- Dapat menyebabkan data inconsistency
- JOIN queries tidak dioptimasi oleh MySQL
- Risiko orphaned records tinggi
```

#### 3. **Inefficient Query Patterns**
```sql
❌ SELECT * tanpa WHERE clause:
- get_data_user() → SELECT semua user tanpa limit
- get_tambah_tugas() → SELECT semua tugas tanpa filter
- get_data_nilai() → SELECT semua nilai tanpa pagination

❌ Potensi N+1 Query Problem:
- Model methods tidak menggunakan eager loading
- Setiap record bisa trigger additional query
```

#### 4. **Data Type Issues**
```sql
❌ Inefficient data types:
- datetime_jurnal: VARCHAR(255) seharusnya DATETIME
- date_mulai/date_selesai: VARCHAR(100) seharusnya DATE
- Wasted storage space dan slow date operations
```

#### 5. **Missing Table Issues**
```sql
❌ JoinMahasiswaModel references non-existent tables:
- db_mhs (tidak ada di schema)
- db_fakultas (tidak ada di schema)  
- db_jurusan (tidak ada di schema)
- Akan error saat dijalankan!
```

#### 6. **Duplicate Code & Logic**
```sql
❌ Code duplication across models:
- get_join_jurnal() ada di AdminModel & UserModel
- get_join_nilai() ada di AdminModel & UserModel
- Maintenance nightmare & inconsistency risk
```

---

## 4. Business Rules yang Diimplementasi di Database Level

### ✅ **Implemented Rules:**

#### 1. **Auto-increment Primary Keys**
```sql
- Semua tabel menggunakan auto-increment PK
- Menjamin uniqueness untuk setiap record
```

#### 2. **Default Values**
```sql
- db_user.date_create: DEFAULT CURRENT_TIMESTAMP
- db_galery_tugas.dateline_tugas: DEFAULT CURRENT_TIMESTAMP  
- Automatic timestamp tracking
```

#### 3. **Engine Configuration**
```sql
- Semua tabel menggunakan ENGINE=InnoDB
- Mendukung transactions & foreign keys
- CHARSET=latin1 (tidak optimal untuk internasionalization)
```

#### 4. **Data Length Constraints**
```sql
- username: varchar(100) - reasonable limit
- email_user: varchar(100) - mungkin perlu lebih besar
- password: varchar(255) - cukup untuk hashed passwords
```

### ❌ **Missing Critical Business Rules:**

#### 1. **No Data Integrity Constraints**
```sql
❌ Tidak ada FOREIGN KEY constraints
❌ Tidak ada CHECK constraints
❌ Tidak ada UNIQUE constraints (kecuali PK)
```

#### 2. **No Data Validation**
```sql
❌ Email format validation
❌ Status value constraints (level_user, is_active, status_jurnal)
❌ Date range validations
```

#### 3. **No Cascade Rules**
```sql
❌ Tidak ada ON DELETE/UPDATE CASCADE
❌ Risk of orphaned records saat delete
```

#### 4. **No Triggers for Business Logic**
```sql
❌ Tidak ada triggers untuk audit trail
❌ Tidak ada auto-calculation triggers
❌ Tidak ada validation triggers
```

---

## 5. Catalog Semua Index dan Penggunaannya

### ✅ **Existing Indexes (Primary Keys Only):**

| Table | Index Name | Type | Columns | Usage | Status |
|-------|------------|------|---------|--------|---------|
| `db_galery` | PRIMARY | PK | `id_galery` | Unique identification | ✅ Active |
| `db_galery_berkas` | PRIMARY | PK | `id_berkas` | Unique identification | ✅ Active |  
| `db_galery_tugas` | PRIMARY | PK | `id_galery_tugas` | Unique identification | ✅ Active |
| `db_jurnal` | PRIMARY | PK | `id_jurnal` | Unique identification | ✅ Active |
| `db_nilai` | PRIMARY | PK | `id_nilai_mhs` | Unique identification | ✅ Active |
| `db_pengajuan` | PRIMARY | PK | `id_pengajuan` | Unique identification | ✅ Active |
| `db_sertifikat` | PRIMARY | PK | `id_sertifikat` | Unique identification | ✅ Active |
| `db_tugas` | PRIMARY | PK | `id_tugas` | Unique identification | ✅ Active |
| `db_user` | PRIMARY | PK | `id_user` | Unique identification | ✅ Active |

### 🔍 **Partial Index (Non-Unique):**
| Table | Index Name | Type | Columns | Status | Issue |
|-------|------------|------|---------|---------|--------|
| `db_galery` | `id_img` | KEY | `id_img` | ⚠️ Incomplete | Tidak ada tabel referensi |

### 🚨 **CRITICAL Missing Indexes:**

#### **Join Performance Indexes (URGENT!):**
```sql
-- Untuk AdminModel.get_join_nilai() & UserModel.get_join_nilai()
CREATE INDEX idx_nilai_user ON db_nilai(id_nilai_user);

-- Untuk AdminModel.get_join_jurnal() & UserModel.get_join_jurnal()  
CREATE INDEX idx_jurnal_user ON db_jurnal(id_user);

-- Untuk upload references
CREATE INDEX idx_user_upload ON db_user(id_upload);
CREATE INDEX idx_galery_tugas_upload ON db_galery_tugas(id_upload);
```

#### **Search/Filter Performance Indexes:**
```sql
-- Untuk LoginModel.Login()
CREATE UNIQUE INDEX idx_username ON db_user(username);

-- Untuk RegisterModel.cek_register()
CREATE INDEX idx_user_email ON db_user(email_user);
CREATE INDEX idx_user_credentials ON db_user(username, email_user);

-- Untuk status filtering
CREATE INDEX idx_user_level ON db_user(level_user);
CREATE INDEX idx_user_active ON db_user(is_active);
CREATE INDEX idx_jurnal_status ON db_jurnal(status_jurnal);
```

#### **Timestamp/Date Indexes:**
```sql
-- Untuk date range queries
CREATE INDEX idx_user_created ON db_user(date_create);
CREATE INDEX idx_tugas_datetime ON db_galery_tugas(dateline_tugas);
```

### 📊 **Index Usage Analysis:**

#### **High Usage (Critical for Performance):**
- `db_user.username` - Used in login authentication (very frequent)
- `db_jurnal.id_user` - Used in JOIN operations (frequent)
- `db_nilai.id_nilai_user` - Used in JOIN operations (frequent)

#### **Medium Usage (Important for Scalability):**
- `db_user.level_user` - Used for role-based filtering
- `db_user.is_active` - Used for user status filtering
- `db_user.date_create` - Used for reporting/analytics

#### **Low Usage (Nice to Have):**
- `db_galery_tugas.dateline_tugas` - Used for chronological sorting
- `db_pengajuan.date_mulai/date_selesai` - Used for date range filtering

---

## 📋 **REKOMENDASI PRIORITAS**

### 🔥 **URGENT (Implement Immediately):**

1. **🚨 Add Foreign Key Constraints**
   ```sql
   ALTER TABLE db_jurnal ADD CONSTRAINT fk_jurnal_user 
       FOREIGN KEY (id_user) REFERENCES db_user(id_user) ON DELETE CASCADE;
   
   ALTER TABLE db_nilai ADD CONSTRAINT fk_nilai_user 
       FOREIGN KEY (id_nilai_user) REFERENCES db_user(id_nilai_user) ON DELETE CASCADE;
   
   ALTER TABLE db_galery_tugas ADD CONSTRAINT fk_galery_upload 
       FOREIGN KEY (id_upload) REFERENCES db_user(id_upload) ON DELETE CASCADE;
   ```

2. **⚡ Create Critical JOIN Indexes**
   ```sql
   CREATE INDEX idx_jurnal_user ON db_jurnal(id_user);
   CREATE INDEX idx_nilai_user ON db_nilai(id_nilai_user);
   CREATE UNIQUE INDEX idx_username ON db_user(username);
   ```

3. **🗑️ Fix or Remove Dead Code**
   ```php
   // Either create missing tables or remove JoinMahasiswaModel entirely
   // Tables needed: db_mhs, db_fakultas, db_jurusan
   ```

### ⚡ **HIGH PRIORITY:**

1. **🔧 Optimize Data Types**
   ```sql
   ALTER TABLE db_jurnal MODIFY datetime_jurnal DATETIME NOT NULL;
   ALTER TABLE db_pengajuan MODIFY date_mulai DATE NOT NULL;
   ALTER TABLE db_pengajuan MODIFY date_selesai DATE NOT NULL;
   ```

2. **📖 Add Composite Indexes**
   ```sql
   CREATE INDEX idx_user_credentials ON db_user(username, email_user);
   CREATE INDEX idx_user_status ON db_user(level_user, is_active);
   ```

3. **🧹 Remove Code Duplication**
   ```php
   // Consolidate duplicate methods across AdminModel & UserModel
   // Create a base service class for common operations
   ```

4. **📄 Implement Query Pagination**
   ```php
   // Add LIMIT and OFFSET to large result set queries
   // Implement pagination in get_data_user(), get_tambah_tugas(), etc.
   ```

### 💡 **MEDIUM PRIORITY:**

1. **✅ Add CHECK Constraints**
   ```sql
   ALTER TABLE db_user ADD CONSTRAINT chk_level 
       CHECK (level_user IN (0, 1, 2));
   
   ALTER TABLE db_user ADD CONSTRAINT chk_active 
       CHECK (is_active IN (0, 1));
   ```

2. **📊 Implement Audit Triggers**
   ```sql
   CREATE TRIGGER tr_user_audit 
       AFTER UPDATE ON db_user 
       FOR EACH ROW ...
   ```

3. **🌐 Optimize Character Set**
   ```sql
   ALTER TABLE db_user CONVERT TO CHARACTER SET utf8mb4 
       COLLATE utf8mb4_unicode_ci;
   ```

4. **📈 Add Reporting Indexes**
   ```sql
   CREATE INDEX idx_user_created ON db_user(date_create);
   CREATE INDEX idx_tugas_datetime ON db_galery_tugas(dateline_tugas);
   ```

---

## 🎯 **KESIMPULAN**

Database SIMPOL memerlukan **optimisasi mendesak** terutama pada:

1. **🔐 Data Integrity**: Tidak ada foreign key constraints sama sekali
2. **⚡ Performance**: Missing indexes untuk JOIN operations  
3. **🗂️ Data Types**: Penggunaan VARCHAR untuk date/datetime fields
4. **🔍 Code Quality**: Duplikasi code dan dead references

**Impact jika tidak ditangani:**
- 🐌 Query performance sangat lambat pada data besar
- 💥 Risk data corruption dan inconsistency tinggi  
- 🚫 Aplikasi error karena missing tables
- 🔄 Maintenance sulit karena code duplication

**Estimated Implementation Time:**
- 🔥 Urgent fixes: **2-3 hari**
- ⚡ High priority: **1 minggu** 
- 💡 Medium priority: **2-3 minggu**

---

*🔧 Analysis tools: Manual code review, SQL schema analysis*  
*📊 Database: MySQL/MariaDB (db_ci)*
