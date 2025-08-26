# DATABASE SIMPPOL - DESIGN LENGKAP
## Sistem Informasi Magang dan Praktek Kerja Lapangan

*Berdasarkan analisis database SIPPOL dan SIMPOL yang sudah ada*

---

## **SIAPA SAJA USER SISTEMNYA?**

### **👥 4 LEVEL USER**
1. **ADMIN SISTEM** (Level 1) - Kelola seluruh sistem
2. **KEPALA BIDANG** (Level 2) - Approve pengajuan magang
3. **PEMBIMBING** (Level 3) - Bimbing peserta sehari-hari  
4. **PESERTA MAGANG** (Level 4) - Mahasiswa yang magang

---

## **ALUR KERJA SISTEM**

### **🔄 ALUR PESERTA MAGANG**
```
1. PENGAJUAN MAGANG
   - Daftar online melalui form pengajuan
   - Upload dokumen surat pengantar
   - Pilih bidang yang diinginkan
   - Tentukan durasi magang
   
2. PROSES PERSETUJUAN  
   - Admin review dokumen
   - Kepala bidang approve dan assign pembimbing
   - Sistem buat akun login peserta
   
3. AKTIVITAS MAGANG HARIAN
   - Check-in dan check-out presensi
   - Isi jurnal aktivitas harian
   - Kerjakan tugas dari pembimbing
   - Upload hasil tugas
   
4. EVALUASI DAN SELESAI
   - Pembimbing kasih nilai
   - Dapat sertifikat digital
```

### **🔄 ALUR PEMBIMBING** 
```
1. DAPAT PESERTA BARU
   - Terima notifikasi peserta baru
   - Review profil peserta
   
2. SUPERVISI HARIAN
   - Monitor kehadiran peserta
   - Kasih tugas mingguan
   - Review dan approve jurnal
   - Kasih feedback dan nilai
```

---

## **ERD (ENTITY RELATIONSHIP DIAGRAM)**

### **🏗️ DIAGRAM RELASI TABEL SIMPPOL**

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                         DATABASE SIMPPOL - ERD LENGKAP                                │
└────────────────────────────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────┐            ┌─────────────────────┐
                    │      configs        │            │       bidang        │ *NEW*
                    ├─────────────────────┤            ├─────────────────────┤
                    │ title               │            │ id_bidang (PK)      │
                    └─────────────────────┘            │ nama_bidang         │
                                                      │ deskripsi           │
                                                      │ kuota_maksimal      │
                                                      │ is_active           │
                                                      └─────────────────────┘
                                                               │
                                                               │1:M
                                                               ▼
┌─────────────────────┐         ┌─────────────────────┐      ┌─────────────────────┐
│      instansi       │         │    kepala_bidang    │ *NEW*│    pembimbing       │ *NEW*
├─────────────────────┤         ├─────────────────────┤      ├─────────────────────┤
│ id_instansi (PK)    │         │ id_kepala (PK)      │      │ id_pembimbing (PK)  │
│ nama_instansi       │         │ bidang_id (FK)      │      │ bidang_id (FK)      │
│ alamat_instansi     │         │ nama_kepala         │      │ kepala_id (FK)      │
└─────────────────────┘         │ nip                 │      │ nama_pembimbing     │
         │                      │ email               │      │ nip                 │
         │1:M                   │ phone               │      │ email               │
         │                      │ is_active           │      │ phone               │
         ▼                      └─────────────────────┘      │ spesialisasi        │
┌─────────────────────┐                  │                   │ max_peserta         │
│     pengajuan       │                  │1:M                │ is_active           │
├─────────────────────┤                  │                   └─────────────────────┘
│ id_pengajuan (PK)   │                  ▼                            │
│ id_instansi (FK)    │         ┌─────────────────────┐               │1:M
│ nama_kelompok       │ *NEW*   │       users         │               │
│ nama_pemohon        │ *NEW*   ├─────────────────────┤               ▼
│ email_pemohon       │ *NEW*   │ id_user (PK)        │      ┌─────────────────────┐
│ bidang_pilihan      │ *NEW*   │ username            │      │       users         │
│ durasi_bulan        │ *NEW*   │ password            │      │   (extended)        │
│ date_mulai          │ *NEW*   │ nama_lengkap        │      ├─────────────────────┤
│ date_selesai        │ *NEW*   │ npm                 │      │ email               │
│ srt_pengajuan       │         │ phone               │      │ level_user          │
│ srt_balasan         │         │ photo_url           │      │ is_active           │
│ tahun               │         │ level_user:         │      │ id_instansi (FK)    │
│ keterangan          │         │   1 = Admin         │      │ bidang_id (FK)      │ *NEW*
│ timestamp           │         │   2 = Kepala Bidang │      │ pembimbing_id (FK)  │ *NEW*
│ status              │ *NEW*   │   3 = Pembimbing    │      │ mulai_magang        │
│ reviewed_by         │ *NEW*   │   4 = Peserta       │      │ akhir_magang        │
│ approved_by_kb      │ *NEW*   │ is_active           │      │ is_online           │
└─────────────────────┘         │ is_online           │      │ date_create         │
         │                      │ date_create         │      │ first_login         │
         │M:1                   │ first_login         │      │ last_login          │
         │                      └─────────────────────┘      └─────────────────────┘
         ▼                               │                            │
┌─────────────────────┐                 │1:M                         │1:M
│       files         │                 ▼                            ▼
├─────────────────────┤        ┌─────────────────────┐      ┌─────────────────────┐
│ id_file (PK)        │        │      presensi       │      │     jurnal          │
│ original_name       │        ├─────────────────────┤      ├─────────────────────┤
│ stored_name         │        │ id_data (PK)        │      │ id_jurnal (PK)      │
│ file_path           │        │ id_user (FK)        │      │ id_user (FK)        │
│ file_size           │        │ data_nama           │      │ datetime_jurnal     │
│ mime_type           │        │ data_npm            │      │ kelompok_jurnal     │
│ category            │        │ data_tanggal        │      │ deskripsi_jurnal    │
│ uploaded_by (FK)    │        │ data_status         │      │ mulai_jurnal        │
│ entity_type         │        │ data_deskripsi      │      │ selesai_jurnal      │
│ entity_id           │        │ data_asal           │      │ status_jurnal       │
│ uploaded_at         │        └─────────────────────┘      │ catatan_pembimbing  │ *NEW*
└─────────────────────┘                                     │ reviewed_by (FK)    │ *NEW*
                                                            │ reviewed_at         │ *NEW*
                              ┌─────────────────────┐       └─────────────────────┘
                              │       tugas         │
                              ├─────────────────────┤       ┌─────────────────────┐
                              │ id_tugas (PK)       │       │      nilai          │
                              │ judul_tugas         │ *NEW* ├─────────────────────┤
                              │ deskripsi_tugas     │       │ id_nilai (PK)       │
                              │ jadwal_tugas        │       │ id_user (FK)        │
                              │ deadline            │ *NEW* │ id_pembimbing (FK)  │ *NEW*
                              │ max_score           │ *NEW* │ kategori_nilai      │
                              │ created_by (FK)     │ *NEW* │ nilai               │
                              │ is_active           │ *NEW* │ justifikasi         │
                              └─────────────────────┘       │ semester            │ *NEW*
                                       │                    │ is_final            │ *NEW*
                                       │1:M                 │ created_at          │
                                       ▼                    └─────────────────────┘
                              ┌─────────────────────┐
                              │  submission_tugas   │       ┌─────────────────────┐
                              ├─────────────────────┤       │    sertifikat       │
                              │ id_submission (PK)  │       ├─────────────────────┤
                              │ id_user (FK)        │       │ id_sertifikat (PK)  │
                              │ id_tugas (FK)       │       │ id_user (FK)        │
                              │ submission_text     │ *NEW* │ judul_sertifikat    │ *NEW*
                              │ submission_files    │       │ link_sertifikat     │
                              │ dateline_tugas      │       │ qr_code             │ *NEW*
                              │ status              │ *NEW* │ verification_code   │ *NEW*
                              │ score               │ *NEW* │ issued_date         │ *NEW*
                              │ feedback            │ *NEW* │ issued_by (FK)      │ *NEW*
                              │ graded_by (FK)      │ *NEW* │ is_revoked          │ *NEW*
                              │ graded_at           │ *NEW* └─────────────────────┘
                              └─────────────────────┘

KETERANGAN:
- PK = Primary Key
- FK = Foreign Key  
- *NEW* = Field/fitur baru atau enhancement
- 1:M = One to Many (Satu ke Banyak)
```

---

## **DETAIL TABEL DATABASE**

### **📊 TABEL DARI SIPPOL (DIPERTAHANKAN)**

#### **1. configs**
```
Konfigurasi sistem
- title (TEXT) - Setting sistem
```

#### **2. instansi**
```  
Master institusi/universitas
- id_instansi (PK)
- nama_instansi
- alamat_instansi
```

#### **3. users** (Gabungan dari `user` SIPPOL + `db_user` SIMPOL)
```
Data semua pengguna sistem
- id_user (PK)
- username (UNIQUE)
- password (HASHED)
- nama_lengkap
- npm (untuk peserta)
- email (UNIQUE)
- phone
- photo_url
- level_user:
  * 1 = Admin Sistem
  * 2 = Kepala Bidang  
  * 3 = Pembimbing
  * 4 = Peserta Magang
- is_active (1=aktif, 0=non-aktif)
- is_online (1=online, 0=offline)
- id_instansi (FK ke instansi)
- bidang_id (FK ke bidang) *NEW*
- pembimbing_id (FK ke pembimbing) *NEW*
- mulai_magang
- akhir_magang
- date_create
- first_login
- last_login
```

#### **4. presensi** 
```
Data kehadiran peserta (dari SIPPOL)
- id_data (PK)
- id_user (FK)
- data_nama (backup nama)
- data_npm (backup npm)
- data_tanggal
- data_status ('5'=check in, lainnya=check out)
- data_deskripsi
- data_asal (backup institusi)
```

#### **5. pengajuan** (Enhanced dari SIPPOL)
```
Pengajuan magang
- id_pengajuan (PK)
- id_instansi (FK)
- nama_kelompok *NEW*
- nama_pemohon *NEW*
- email_pemohon *NEW*
- bidang_pilihan *NEW*
- durasi_bulan *NEW*
- date_mulai *NEW*
- date_selesai *NEW*
- srt_pengajuan
- srt_balasan
- tahun
- keterangan
- timestamp
- status (draft/submitted/approved/rejected) *NEW*
- reviewed_by (FK ke users) *NEW*
- approved_by_kb (FK ke kepala bidang) *NEW*
```

### **📊 TABEL DARI SIMPOL (DIPERTAHANKAN)**

#### **6. jurnal** (dari `db_jurnal` SIMPOL)
```
Jurnal aktivitas harian
- id_jurnal (PK)
- id_user (FK)
- datetime_jurnal
- kelompok_jurnal
- deskripsi_jurnal
- mulai_jurnal
- selesai_jurnal
- status_jurnal (0=draft, 1=submitted, 2=approved, 3=rejected)
- catatan_pembimbing *NEW*
- reviewed_by (FK) *NEW*
- reviewed_at *NEW*
```

#### **7. tugas** (dari `db_tugas` SIMPOL)
```
Tugas dari pembimbing
- id_tugas (PK)
- judul_tugas *NEW*
- deskripsi_tugas
- jadwal_tugas (minggu 1,2,3,4)
- deadline *NEW*
- max_score *NEW*
- created_by (FK ke users) *NEW*
- is_active *NEW*
```

#### **8. submission_tugas** (dari `db_galery_tugas` SIMPOL)
```
Pengumpulan tugas peserta
- id_submission (PK)
- id_user (FK)
- id_tugas (FK)
- submission_text *NEW*
- submission_files (gambar_tugas)
- dateline_tugas
- status (submitted/graded/revision) *NEW*
- score *NEW*
- feedback *NEW*
- graded_by (FK) *NEW*
- graded_at *NEW*
```

#### **9. nilai** (dari `db_nilai` SIMPOL)
```
Penilaian peserta
- id_nilai (PK)
- id_user (FK)
- id_pembimbing (FK) *NEW*
- kategori_nilai (justifikasi_mhs → kategori_mhs)
- nilai (nilai_mhs)
- justifikasi
- semester *NEW*
- is_final *NEW*
- created_at *NEW*
```

#### **10. sertifikat** (dari `db_sertifikat` SIMPOL)
```
Sertifikat digital
- id_sertifikat (PK)
- id_user (FK)
- judul_sertifikat *NEW*
- link_sertifikat
- qr_code *NEW*
- verification_code *NEW*
- issued_date *NEW*
- issued_by (FK) *NEW*
- is_revoked *NEW*
```

### **📊 TABEL BARU (FITUR MAGANG)**

#### **11. bidang** *(TABEL BARU)*
```
Master bidang Kominfo
- id_bidang (PK)
- nama_bidang (Aptika, Komunikasi, dll)
- deskripsi
- kuota_maksimal
- is_active
```

#### **12. kepala_bidang** *(TABEL BARU)*
```
Data kepala bidang
- id_kepala (PK)
- bidang_id (FK)
- nama_kepala
- nip
- email
- phone
- is_active
```

#### **13. pembimbing** *(TABEL BARU)*
```
Data pembimbing
- id_pembimbing (PK)
- bidang_id (FK)
- kepala_id (FK)
- nama_pembimbing
- nip
- email
- phone
- spesialisasi
- max_peserta
- is_active
```

#### **14. files** (Unified dari berbagai tabel galery SIMPOL)
```
Unified file management
- id_file (PK)
- original_name
- stored_name
- file_path
- file_size
- mime_type
- category (dokumen_pengajuan/tugas/profil)
- uploaded_by (FK)
- entity_type (pengajuan/submission_tugas/users)
- entity_id
- uploaded_at
```

---

## **HUBUNGAN ANTAR TABEL**

### **🔗 RELASI UTAMA**
```
instansi → pengajuan (1:M)
instansi → users (1:M)

bidang → kepala_bidang (1:1)
bidang → pembimbing (1:M)
bidang → users (1:M)

pembimbing → users (1:M)
users → presensi (1:M)
users → jurnal (1:M)
users → submission_tugas (1:M)
users → nilai (1:M)
users → sertifikat (1:M)

tugas → submission_tugas (1:M)
```

---

## **MIGRASI DATA DARI SISTEM LAMA**

### **📤 DARI SIPPOL**

#### **Data Instansi**
```
instansi (SIPPOL) → instansi (SIMPPOL)
- Langsung copy semua data
```

#### **Data User** 
```
user (SIPPOL) → users (SIMPPOL)
- username → username
- password → password (sudah hashed)
- nama → nama_lengkap
- npm → npm
- level_user = 4 (semua jadi peserta)
- id_instansi → id_instansi
- mulai_magang → mulai_magang
- akhir_magang → akhir_magang
- status → is_active
```

#### **Data Presensi**
```
presensi (SIPPOL) → presensi (SIMPPOL)
- Langsung copy semua data
- Tidak ada perubahan struktur
```

#### **Data Pengajuan**
```
pengajuan (SIPPOL) → pengajuan (SIMPPOL)
- Tambah field baru yang kosong
- Set status = 'approved' untuk data lama
```

### **📤 DARI SIMPOL**

#### **Data User SIMPOL**
```
db_user (SIMPOL) → users (SIMPPOL)
- username → username
- password → password
- nama_user → nama_lengkap
- email_user → email
- level_user → level_user (1=admin, 2=user → perlu mapping)
- is_active → is_active
- date_create → date_create
```

#### **Data Jurnal**
```
db_jurnal (SIMPOL) → jurnal (SIMPPOL)
- Langsung copy dengan tambahan field kosong
```

#### **Data Tugas & Submission**
```
db_tugas (SIMPOL) → tugas (SIMPPOL)
db_galery_tugas (SIMPOL) → submission_tugas (SIMPPOL)
- Tambah field enhancement
```

#### **Data Nilai**
```
db_nilai (SIMPOL) → nilai (SIMPPOL)
- kategori_mhs → kategori_nilai
- nilai_mhs → nilai
- justifikasi_mhs → justifikasi
```

### **📝 DATA BARU HARUS DIISI**

#### **Master Bidang**
```
INSERT INTO bidang:
1. Aplikasi Informatika
2. Komunikasi dan Informasi Publik
3. Statistik dan Persandian
4. E-Government
```

#### **Master Kepala Bidang & Pembimbing**
```
Sesuai struktur organisasi Kominfo Kediri saat ini
```

---

## **FITUR ENHANCEMENT**

### **✨ FITUR YANG DITINGKATKAN**

1. **User Management** - 4 level user vs 2 level sebelumnya
2. **Pengajuan** - Approval berlapis (admin → kepala bidang)
3. **Jurnal** - Tambah review dari pembimbing
4. **Tugas** - Sistem feedback dan grading
5. **Nilai** - Multi-kategori assessment  
6. **Sertifikat** - Verifikasi dengan QR code
7. **File Management** - Unified system
8. **Master Data** - Bidang, kepala bidang, pembimbing

### **🎯 BUSINESS RULES YANG DIPERTAHANKAN**

#### **Dari SIPPOL:**
- Status user: 0=disabled, 1=enabled
- Status presensi: '5'=check in, lainnya=check out
- Password hashing dengan PHP password_hash()
- Backup data di tabel presensi

#### **Dari SIMPOL:**
- Level user: 1=admin, 2=user (diperluas jadi 1,2,3,4)
- Status jurnal: 0=draft, 1=submitted, 2=approved, 3=rejected
- Auto timestamp pada record creation
- File upload dengan validation

---

## **ESTIMASI IMPLEMENTASI**

### **📅 TIMELINE 5 MINGGU**

**Minggu 1:**
- Setup database SIMPPOL
- Buat semua tabel dengan structure lengkap
- Buat foreign key constraints

**Minggu 2:**
- Migrasi data instansi dari SIPPOL
- Migrasi data user dari SIPPOL & SIMPOL
- Setup master bidang, kepala bidang, pembimbing

**Minggu 3:**
- Migrasi data presensi dari SIPPOL
- Migrasi data pengajuan dari SIPPOL
- Handle konflik dan duplikasi data

**Minggu 4:**
- Migrasi jurnal, tugas, nilai dari SIMPOL
- Migrasi sertifikat dan files dari SIMPOL
- Testing integritas data

**Minggu 5:**
- Full system testing
- Performance tuning
- User acceptance testing

### **📊 ESTIMASI DATA**

| Tabel | Estimasi Records |
|-------|------------------|
| instansi | ~30 |
| users | ~400 |
| pengajuan | ~200 |
| presensi | ~8.000 |
| jurnal | ~3.000 |
| tugas | ~75 |
| submission_tugas | ~300 |
| nilai | ~750 |
| sertifikat | ~200 |

---

## **KEUNGGULAN DATABASE INI**

### **✅ KELEBIHAN DESIGN**

1. **Unified Structure** - Menggabungkan kekuatan SIPPOL dan SIMPOL
2. **Enhanced Features** - Fitur magang yang lebih lengkap
3. **Data Integrity** - Foreign keys dan constraints yang proper
4. **Scalable** - Mudah dikembangkan untuk fitur masa depan
5. **Backward Compatible** - Data lama tetap bisa digunakan

### **🎯 HASIL YANG DIHARAPKAN**

- **Sistem terintegrasi** - Satu database untuk semua kebutuhan magang
- **Approval workflow** - Proses persetujuan yang terstruktur  
- **Better monitoring** - Tracking aktivitas peserta yang lebih baik
- **Digital certificate** - Sertifikat dengan verifikasi QR code
- **Reduced redundancy** - Eliminasi duplikasi data dan proses

**Database SIMPPOL siap mendukung sistem magang modern Kominfo Kediri!**

---

*Database Design berdasarkan analisis SIPPOL & SIMPOL*  
*Generated: 26 Agustus 2025*
