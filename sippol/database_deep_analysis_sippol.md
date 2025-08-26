# Database Deep Analysis - SIPPOL Reborn Kediri

## Struktur Database dan ERD (Entity Relationship Diagram)

### 1. Identifikasi Tabel

Berdasarkan analisis kode PHP dan query SQL, sistem SIPPOL Reborn memiliki 5 tabel utama:

1. **configs** - Konfigurasi sistem
2. **instansi** - Master data instansi/universitas
3. **user** - Data pemagang/mahasiswa
4. **presensi** - Data kehadiran pemagang
5. **pengajuan** - Data pengajuan magang dari instansi

### 2. ERD (Entity Relationship Diagram) - Format ASCII

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│     configs     │       │     instansi    │       │      user       │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ title (VARCHAR) │       │ id_instansi (PK)│◄─────►│ id_user (PK)    │
└─────────────────┘       │ nama_instansi   │       │ username        │
                          │ alamat_instansi │       │ nama            │
                          └─────────────────┘       │ npm             │
                                   △               │ password        │
                                   │                │ id_instansi (FK)│
                                   │                │ mulai_magang    │
                                   │                │ akhir_magang    │
                          ┌─────────────────┐       │ status          │
                          │   pengajuan     │       └─────────────────┘
                          ├─────────────────┤                │
                          │ id_pengajuan(PK)│                │
                          │ id_instansi (FK)│                │
                          │ srt_pengajuan   │                │
                          │ srt_balasan     │                │
                          │ tahun           │                │
                          │ keterangan      │                │
                          │ timestamp       │                └───────┐
                          └─────────────────┘                        │
                                                                     ▼
                                                          ┌─────────────────┐
                                                          │    presensi     │
                                                          ├─────────────────┤
                                                          │ id_data (PK)    │
                                                          │ id_user (FK)    │
                                                          │ data_nama       │
                                                          │ data_npm        │
                                                          │ data_tanggal    │
                                                          │ data_status     │
                                                          │ data_deskripsi  │
                                                          │ data_asal       │
                                                          └─────────────────┘
```

### 3. Relasi Antar Tabel

1. **instansi → user** (One-to-Many)
   - `instansi.id_instansi` ← `user.id_instansi`

2. **user → presensi** (One-to-Many)
   - `user.id_user` ← `presensi.id_user`

3. **instansi → pengajuan** (One-to-Many)
   - `instansi.id_instansi` ← `pengajuan.id_instansi`

## Analisis Query Kompleks

### 1. Query di data-pemagang.php

```sql
SELECT u.id_user, u.username, u.nama, u.npm, u.password, 
       i.nama_instansi AS asal, u.mulai_magang, u.akhir_magang, u.status 
FROM user u
LEFT JOIN instansi i ON u.id_instansi = i.id_instansi
```

**Analisis:**
- Query dengan LEFT JOIN untuk menampilkan data user beserta nama instansi
- Menggunakan alias untuk memperjelas nama kolom
- Efficient karena hanya mengambil kolom yang dibutuhkan

### 2. Query di data-presensi.php

```sql
SELECT p.id_data, 
       IFNULL(u.nama, p.data_nama) AS nama,
       IFNULL(u.npm, p.data_npm) AS npm,
       IFNULL(i.nama_instansi, p.data_asal) AS nama_instansi,
       p.data_tanggal, p.data_status, p.data_deskripsi
FROM presensi p
LEFT JOIN user u ON p.id_user = u.id_user
LEFT JOIN instansi i ON u.id_instansi = i.id_instansi
```

**Analisis:**
- Query kompleks dengan double LEFT JOIN
- Menggunakan IFNULL untuk fallback data jika relasi tidak ada
- Desain memungkinkan presensi tanpa user (standalone)

### 3. Query di data-instansi.php

```sql
SELECT i.id_instansi as id_ins, i.nama_instansi as nama, 
       i.alamat_instansi as alamat, COUNT(DISTINCT u.id_user) AS jumlah 
FROM instansi i 
LEFT JOIN user u ON i.id_instansi = u.id_instansi 
GROUP BY i.id_instansi, i.nama_instansi, i.alamat_instansi 
ORDER BY nama_instansi asc
```

**Analisis:**
- Query dengan agregasi COUNT untuk menghitung jumlah user per instansi
- Menggunakan DISTINCT untuk menghindari duplikasi
- GROUP BY yang proper untuk agregasi

### 4. Query di data-pengajuan.php

```sql
SELECT p.*, i.nama_instansi
FROM pengajuan p
LEFT JOIN instansi i ON p.id_instansi = i.id_instansi 
ORDER BY id_pengajuan desc
```

**Analisis:**
- Simple JOIN dengan ordering descending
- Menggunakan SELECT * yang sebaiknya dihindari untuk performa

## Potential Performance Issues

### 1. Missing Indexes

**Rekomendasi Index:**

```sql
-- Primary Keys (sudah ada secara default)
-- Untuk Foreign Keys
CREATE INDEX idx_user_id_instansi ON user(id_instansi);
CREATE INDEX idx_presensi_id_user ON presensi(id_user);
CREATE INDEX idx_pengajuan_id_instansi ON pengajuan(id_instansi);

-- Untuk Sorting/Filtering
CREATE INDEX idx_user_status ON user(status);
CREATE INDEX idx_presensi_tanggal ON presensi(data_tanggal);
CREATE INDEX idx_pengajuan_tahun ON pengajuan(tahun);
CREATE INDEX idx_instansi_nama ON instansi(nama_instansi);

-- Composite Indexes
CREATE INDEX idx_user_status_instansi ON user(status, id_instansi);
CREATE INDEX idx_presensi_user_tanggal ON presensi(id_user, data_tanggal);
```

### 2. Query Optimization Issues

**Issue 1: SELECT * dalam pengajuan**
```sql
-- Problematic
SELECT p.*, i.nama_instansi FROM pengajuan p...

-- Better
SELECT p.id_pengajuan, p.srt_pengajuan, p.srt_balasan, 
       p.tahun, p.keterangan, p.timestamp, i.nama_instansi 
FROM pengajuan p...
```

**Issue 2: Lack of LIMIT in data display**
- Tidak ada LIMIT pada query, berpotensi slow pada data besar
- Recommend: Implementasi pagination

**Issue 3: String concatenation dalam WHERE clause**
```sql
-- Dalam handler files, berpotensi SQL Injection
WHERE u.id_user = '$id_user'

-- Should use prepared statements
WHERE u.id_user = ?
```

### 3. N+1 Query Problem

Tidak terdeteksi N+1 query problem karena sistem menggunakan JOIN dengan baik.

## Business Rules yang Diimplementasi di Database Level

### 1. User Status Rules

```php
// File: data-pemagang.php line 83-87
if ($row['status'] == 0) {
    echo "<span class='badge badge-danger'>Disabled</span>";
} elseif ($row['status'] == 1) {
    echo "<span class='badge badge-success'>Enabled</span>";
}
```

**Business Rule:**
- `status = 0` → User nonaktif/disabled
- `status = 1` → User aktif/enabled

### 2. Presensi Status Rules

```php
// File: data-presensi.php line 89-93
if ($row['data_status'] == '5') {
    echo "<td><span class='badge bg-success text-white'>IN</span></td>";
} else {
    echo "<td><span class='badge bg-danger text-white'>OUT</span></td>";
}
```

**Business Rule:**
- `data_status = '5'` → Check IN (masuk)
- `data_status ≠ '5'` → Check OUT (keluar)

### 3. Password Security

```php
// File: handler-add-pemagang.php line 20
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// File: login.php line 32
if (password_verify($password, $row['password'])) {
```

**Security Rule:**
- Password di-hash menggunakan PHP `password_hash()`
- Verifikasi menggunakan `password_verify()`

### 4. Data Redundancy Rules

**Presensi Table Design:**
- Menyimpan data_nama, data_npm, data_asal sebagai backup
- Fallback mechanism jika relasi user/instansi hilang
- Trade-off: Storage vs Data Integrity

## Analisis Index dan Penggunaannya

### 1. Index yang Sudah Ada (Diasumsikan)

```sql
-- Primary Key Indexes (otomatis)
PK_configs_implicit
PK_instansi_id_instansi  
PK_user_id_user
PK_presensi_id_data
PK_pengajuan_id_pengajuan
```

### 2. Index yang Direkomendasikan

**High Priority:**
```sql
-- Foreign Key Indexes (wajib untuk JOIN performance)
CREATE INDEX idx_user_id_instansi ON user(id_instansi);
CREATE INDEX idx_presensi_id_user ON presensi(id_user);
CREATE INDEX idx_pengajuan_id_instansi ON pengajuan(id_instansi);
```

**Medium Priority:**
```sql
-- Filtering & Sorting Indexes
CREATE INDEX idx_user_status ON user(status);
CREATE INDEX idx_presensi_tanggal ON presensi(data_tanggal);
CREATE INDEX idx_instansi_nama ON instansi(nama_instansi);
```

**Low Priority:**
```sql
-- Composite Indexes untuk query kompleks
CREATE INDEX idx_user_status_instansi ON user(status, id_instansi);
CREATE INDEX idx_presensi_user_status ON presensi(id_user, data_status);
```

### 3. Index Usage Analysis

**Most Frequently Used:**
1. `user.id_instansi` - Digunakan di hampir semua JOIN
2. `presensi.id_user` - Untuk relasi presensi-user
3. `user.status` - Untuk filtering user aktif
4. `presensi.data_tanggal` - Untuk sorting presensi

**Monitoring Recommendation:**
```sql
-- MySQL: Untuk monitoring index usage
SHOW INDEX FROM user;
EXPLAIN SELECT ... FROM user WHERE status = 1;
```

## Rekomendasi Optimisasi

### 1. Database Schema

1. **Add proper constraints:**
```sql
ALTER TABLE user ADD CONSTRAINT chk_status CHECK (status IN (0,1));
ALTER TABLE presensi ADD CONSTRAINT chk_data_status CHECK (data_status IN ('5', 'other_valid_values'));
```

2. **Add NOT NULL constraints where appropriate:**
```sql
ALTER TABLE user MODIFY COLUMN username VARCHAR(255) NOT NULL;
ALTER TABLE user MODIFY COLUMN npm VARCHAR(255) NOT NULL;
```

### 2. Query Optimization

1. **Implement pagination:**
```sql
SELECT ... FROM table ORDER BY id DESC LIMIT 20 OFFSET 0;
```

2. **Use prepared statements:**
```php
$stmt = $koneksi->prepare("SELECT * FROM user WHERE id_user = ?");
$stmt->bind_param("i", $id_user);
```

3. **Add query caching for configs:**
```php
// Cache config values instead of querying every page load
```

### 3. Application Level

1. **Connection pooling**
2. **Query result caching**
3. **Lazy loading untuk data besar**
4. **Batch operations untuk INSERT/UPDATE**

## Kesimpulan

Database SIPPOL Reborn memiliki struktur yang cukup baik dengan relasi yang jelas. Namun ada beberapa area yang perlu diperbaiki:

**Strengths:**
- ERD yang logis dan mudah dipahami
- Penggunaan JOIN yang eficient
- Password security yang baik
- Fallback mechanism pada presensi

**Weaknesses:**
- Missing indexes untuk foreign keys
- Potential SQL injection pada beberapa query
- Tidak ada pagination
- SELECT * statements
- Tidak ada proper constraints

**Priority Improvements:**
1. Add missing indexes (Critical)
2. Implement prepared statements (Security)
3. Add pagination (Performance)
4. Add database constraints (Data Integrity)

---
*Analisis ini dibuat berdasarkan review kode pada tanggal analisis dan dapat berubah seiring dengan perkembangan aplikasi.*
