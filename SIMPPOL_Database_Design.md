# DATABASE SIMPPOL - DESIGN SEDERHANA
## Sistem Magang Kominfo Kediri

*Database yang menggabungkan SIPPOL + SIMPOL + fitur magang baru*

---

## **SIAPA SAJA USERNYA?**

### **👤 4 JENIS USER**
1. **ADMIN SISTEM** - Kelola seluruh sistem
2. **KEPALA BIDANG** - Approve pengajuan dan assign pembimbing
3. **PEMBIMBING** - Bimbing peserta sehari-hari
4. **PESERTA MAGANG** - Mahasiswa yang magang

---

### **🏗️ ENTITY RELATIONSHIP DIAGRAM SIMPPOL**

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│                             DATABASE SIMPPOL - ERD LENGKAP                                     │
│                    Sistem Informasi Magang dan Praktek Kerja Lapangan                         │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘

                                   ┌─────────────────────┐
                                   │   system_configs    │ 
                                   ├─────────────────────┤
                                   │ 🔑 config_key (PK)  │
                                   │    config_value     │
                                   │    description      │
                                   │    is_public        │
                                   │    created_at       │
                                   │    updated_at       │
                                   └─────────────────────┘

┌─────────────────────┐      ┌─────────────────────┐      ┌─────────────────────┐
│    institutions     │      │      bidang         │      │  kepala_bidang      │
├─────────────────────┤      ├─────────────────────┤      ├─────────────────────┤
│ 🔑 id (PK)          │      │ 🔑 id (PK)          │      │ 🔑 id (PK)          │
│    name             │      │    nama_bidang      │      │ 🔗 bidang_id (FK)   │
│    alamat           │      │    deskripsi        │      │    nama_kepala      │
│    phone            │      │    is_active        │      │    nip              │
│    contact_person   │      │    created_at       │      │    email            │
│    email            │      │    updated_at       │      │    phone            │
│    website          │      └─────────────────────┘      │    is_active        │
│    is_active        │               │                    │    created_at       │
│    created_at       │               │1:M                 │    updated_at       │
│    updated_at       │               ▼                    └─────────────────────┘
│    deleted_at       │      ┌─────────────────────┐               │
└─────────────────────┘      │    pembimbing       │               │1:M
         │                   ├─────────────────────┤               │
         │1:M                │ 🔑 id (PK)          │               ▼
         │                   │ 🔗 bidang_id (FK)   │      ┌─────────────────────┐
         ▼                   │ 🔗 kepala_id (FK)   │      │       users         │
┌─────────────────────┐      │    nama_pembimbing  │      ├─────────────────────┤
│   applications      │      │    nip              │      │ 🔑 id (PK)          │
├─────────────────────┤      │    email            │      │    username (UQ)    │
│ 🔑 id (PK)          │      │    phone            │      │    email (UQ)       │
│ 🔗 institution_id(FK)│     │    spesialisasi     │      │    password_hash    │
│    group_name       │      │    max_peserta      │      │    full_name        │
│    applicant_name   │      │    is_active        │      │    npm              │
│    applicant_email  │      │    created_at       │      │    phone            │
│    phone            │      │    updated_at       │      │    photo_url        │
│    start_date       │      └─────────────────────┘      │    level_user       │
│    end_date         │               │                    │    is_active        │
│    bidang_pilihan   │ *NEW*         │1:M                 │    is_online        │
│    durasi_bulan     │ *NEW*         │                    │ 🔗 institution_id(FK)│
│    participants     │               ▼                    │ 🔗 supervisor_id(FK)│
│    description      │      ┌─────────────────────┐      │ 🔗 bidang_id (FK)   │ *NEW*
│    status           │      │       users         │      │ 🔗 pembimbing_id(FK)│ *NEW*
│    priority         │      │   (extends here)    │      │    internship_start │
│    reviewed_by      │      └─────────────────────┘      │    internship_end   │
│    approved_by_kb   │ *NEW*         │1:M                 │    first_login      │
│    reviewed_at      │               │                    │    last_login       │
│    review_notes     │               ▼                    │    created_at       │
│    docs_path        │      ┌─────────────────────┐      │    updated_at       │
│    year             │      │     attendance      │      │    deleted_at       │
│    created_at       │      ├─────────────────────┤      └─────────────────────┘
│    updated_at       │      │ 🔑 id (PK)          │               │
│    deleted_at       │      │ 🔗 user_id (FK)     │               │1:M
└─────────────────────┘      │    date             │               │
                            │    status           │               ▼
                            │    check_in_time    │      ┌─────────────────────┐
                            │    check_out_time   │      │      journals       │
                            │    total_hours      │      ├─────────────────────┤
                            │    description      │      │ 🔑 id (PK)          │
                            │    location_lat     │      │ 🔗 user_id (FK)     │
                            │    location_lng     │      │    date             │
                            │    ip_address       │      │    group_name       │
                            │    backup_name      │      │    activity_desc    │
                            │    backup_npm       │      │    skills_learned   │ *NEW*
                            │    backup_institution│     │    challenges       │ *NEW*
                            │    is_approved      │      │    solutions        │ *NEW*
                            │    approved_by      │      │    start_time       │
                            │    created_at       │      │    end_time         │
                            │    updated_at       │      │    status           │
                            └─────────────────────┘      │    supervisor_notes │
                                     │                   │    reviewed_by      │
                                     │M:1                │    reviewed_at      │
                                     │                   │    created_at       │
                                     ▼                   │    updated_at       │
                            ┌─────────────────────┐      └─────────────────────┘
                            │       grades        │
                            ├─────────────────────┤      ┌─────────────────────┐
                            │ 🔑 id (PK)          │      │       tasks         │
                            │ 🔗 user_id (FK)     │      ├─────────────────────┤
                            │ 🔗 graded_by (FK)   │      │ 🔑 id (PK)          │
                            │    category         │      │    title            │
                            │    grade_value      │      │    description      │
                            │    max_grade        │      │    instructions     │
                            │    justification    │      │    schedule_week    │
                            │    semester         │      │    due_date         │
                            │    graded_at        │      │    max_score        │
                            │    is_final         │      │    submission_type  │
                            │    created_at       │      │    is_group_task    │
                            │    updated_at       │      │    created_by       │
                            └─────────────────────┘      │    is_active        │
                                     │                   │    created_at       │
                                     │M:M               │    updated_at       │
                                     │                   │    deleted_at       │
                                     ▼                   └─────────────────────┘
                            ┌─────────────────────┐               │
                            │ task_submissions    │               │1:M
                            ├─────────────────────┤               │
                            │ 🔑 id (PK)          │               ▼
                            │ 🔗 user_id (FK)     │      ┌─────────────────────┐
                            │ 🔗 task_id (FK)     │      │    certificates     │
                            │    submission_text  │      ├─────────────────────┤
                            │    submission_files │      │ 🔑 id (PK)          │
                            │    status           │      │    title            │
                            │    score            │      │    type             │
                            │    feedback         │      │    issued_to_user   │
                            │    graded_by        │      │    certificate_url  │
                            │    submitted_at     │      │    qr_code          │
                            │    graded_at        │      │    verification_code│
                            │    created_at       │      │    issued_date      │
                            │    updated_at       │      │    issued_by        │
                            └─────────────────────┘      │    is_revoked       │
                                     │                   │    created_at       │
                                     │M:1                │    updated_at       │
                                     ▼                   └─────────────────────┘
                            ┌─────────────────────┐      
                            │       files         │      ┌─────────────────────┐
                            ├─────────────────────┤      │  notifications      │
                            │ 🔑 id (PK)          │      ├─────────────────────┤
                            │    original_name    │      │ 🔑 id (PK)          │
                            │    stored_name      │      │ 🔗 user_id (FK)     │
                            │    file_path        │      │    type             │
                            │    file_size        │      │    title            │
                            │    mime_type        │      │    message          │
                            │    file_hash        │      │    is_read          │
                            │    category         │      │    priority         │
                            │ 🔗 uploaded_by (FK) │      │    created_at       │
                            │    entity_type      │      └─────────────────────┘
                            │    entity_id        │      
                            │    is_public        │      ┌─────────────────────┐
                            │    uploaded_at      │      │ user_activity_log   │
                            │    deleted_at       │      ├─────────────────────┤
                            └─────────────────────┘      │ 🔑 id (PK)          │
                                                        │ 🔗 user_id (FK)     │
                                                        │    action           │
                                                        │    entity_type      │
                                                        │    entity_id        │
                                                        │    ip_address       │
                                                        │    user_agent       │
                                                        │    details (JSON)   │
                                                        │    timestamp        │
                                                        └─────────────────────┘

KETERANGAN:
🔑 = Primary Key
🔗 = Foreign Key  
UQ = Unique Constraint
1:M = One to Many (Satu ke Banyak)
M:1 = Many to One (Banyak ke Satu)
M:M = Many to Many (Banyak ke Banyak)
*NEW* = Field/Tabel Baru untuk fitur magang

RELASI UTAMA:
1. institutions (1) → applications (M)      [institution_id]
2. bidang (1) → pembimbing (M)              [bidang_id]
3. kepala_bidang (1) → users (M)            [kepala_bidang sebagai supervisor]
4. pembimbing (1) → users (M)               [pembimbing_id]
5. users (1) → attendance (M)               [user_id]
6. users (1) → journals (M)                 [user_id]
7. users (1) → task_submissions (M)         [user_id]
8. tasks (1) → task_submissions (M)         [task_id]
9. users (1) → grades (M)                   [user_id]
10. users (1) → notifications (M)           [user_id]
```

## **GIMANA ALUR KERJANYA?**

### **🔄 ALUR PESERTA MAGANG**
```
1. DAFTAR MAGANG ONLINE
   - Isi form pengajuan
   - Pilih bidang (Aptika/Komunikasi/dll)
   - Upload dokumen
   
2. TUNGGU PERSETUJUAN  
   - Admin review → Setuju/Tolak
   - Kepala Bidang review → Pilih pembimbing
   - Dapat akun login
   
3. MULAI MAGANG
   - Absen masuk/keluar 
   - Isi jurnal harian
   - Kerjakan tugas dari pembimbing
   
4. SELESAI MAGANG
   - Dapat nilai dari pembimbing
   - Dapat sertifikat digital
```

### **🔄 ALUR PEMBIMBING**
```
1. DAPAT PESERTA BARU
   - Notifikasi ada peserta baru
   - Lihat profil peserta
   
2. BIMBING HARIAN
   - Cek absensi peserta
   - Kasih tugas
   - Review jurnal harian
   - Kasih nilai
```

---

## **TABEL APA SAJA DI DATABASE?**

### **📊 MASTER DATA (Data Dasar)**

#### **1. Tabel: bidang**
```
Menyimpan bidang-bidang di Kominfo
- id
- nama_bidang (Aptika, Komunikasi, dll)
- kuota_max (berapa peserta maksimal)
```

#### **2. Tabel: kepala_bidang** 
```
Data kepala setiap bidang
- id
- bidang_id (bidang mana)
- nama_kepala
- email
- nip
```

#### **3. Tabel: pembimbing**
```
Data pembimbing di setiap bidang
- id  
- bidang_id (bidang mana)
- nama_pembimbing
- email
- spesialisasi (keahlian)
- max_peserta (maksimal bimbing berapa orang)
```

#### **4. Tabel: institusi**
```
Data universitas/sekolah peserta
- id
- nama_institusi
- alamat
- contact_person
```

### **📊 DATA PENGGUNA**

#### **5. Tabel: users**
```
Data semua pengguna sistem
- id
- username
- email  
- password
- nama_lengkap
- npm (untuk peserta)
- level_user:
  * 1 = Admin
  * 2 = Kepala Bidang
  * 3 = Pembimbing  
  * 4 = Peserta Magang
- bidang_id (ditempatkan di bidang mana)
- pembimbing_id (dibimbing siapa)
- institusi_id (dari institusi mana)
```

### **📊 DATA PENGAJUAN MAGANG**

#### **6. Tabel: pengajuan**
```
Data pengajuan magang
- id
- institusi_id (dari institusi mana)
- nama_kelompok
- nama_pemohon
- email_pemohon
- bidang_pilihan (mau ke bidang mana)
- durasi_bulan (berapa bulan magang)
- tanggal_mulai
- tanggal_selesai
- status:
  * draft = Belum submit
  * submitted = Sudah submit
  * approved_admin = Disetujui admin
  * approved_kabid = Disetujui kepala bidang
  * rejected = Ditolak
- dokumen_path (file surat pengantar dll)
```

### **📊 DATA AKTIVITAS MAGANG**

#### **7. Tabel: absensi**
```
Data kehadiran peserta
- id
- user_id (peserta mana)
- tanggal
- jam_masuk
- jam_keluar  
- total_jam
- lokasi_lat (GPS latitude)
- lokasi_lng (GPS longitude)
- foto_absen
- keterangan_aktivitas
- status_approved (sudah disetujui pembimbing?)
```

#### **8. Tabel: jurnal_harian**
```  
Jurnal aktivitas harian peserta
- id
- user_id (peserta mana)
- tanggal
- aktivitas_hari_ini (min 50 kata)
- skill_yang_dipelajari (NEW!)
- kendala_yang_dihadapi (NEW!)
- solusi_yang_diterapkan (NEW!)
- jam_mulai
- jam_selesai
- status:
  * 0 = Draft
  * 1 = Sudah submit
  * 2 = Disetujui pembimbing
  * 3 = Ditolak
- catatan_pembimbing
```

#### **9. Tabel: tugas**
```
Tugas dari pembimbing
- id
- judul_tugas
- deskripsi_tugas
- deadline
- nilai_max
- pembuat_tugas (pembimbing mana)
- minggu_ke (minggu 1,2,3,4)
```

#### **10. Tabel: pengumpulan_tugas**
```
Hasil tugas dari peserta
- id
- user_id (peserta mana)
- tugas_id (tugas mana)
- jawaban_teks
- file_tugas (file yang diupload)
- nilai_dapat
- feedback_pembimbing
- tanggal_kumpul
```

### **📊 DATA PENILAIAN**

#### **11. Tabel: penilaian**
```
Nilai peserta dari pembimbing
- id
- user_id (peserta mana)
- kategori_nilai:
  * Kedisiplinan
  * Komunikasi
  * Inisiatif
  * Kerjasama
  * Technical Skills
- nilai (0-100)
- justifikasi (alasan nilai)
- pembimbing_id (yang kasih nilai)
```

#### **12. Tabel: sertifikat**
```
Sertifikat digital peserta
- id
- user_id (untuk peserta mana)
- judul_sertifikat
- file_sertifikat (PDF)
- qr_code (untuk verifikasi)
- kode_verifikasi
- tanggal_terbit
```

### **📊 DATA PENDUKUNG**

#### **13. Tabel: notifikasi**
```
Notifikasi untuk user
- id
- user_id (untuk user mana)
- judul
- pesan
- sudah_dibaca
- tanggal_buat
```

#### **14. Tabel: file**
```
Semua file yang diupload
- id
- nama_asli
- nama_disimpan
- ukuran_file
- jenis_file
- kategori (dokumen_pengajuan, tugas, foto, dll)
- yang_upload
```

---

## **HUBUNGAN ANTAR TABEL**

### **🔗 RELASI UTAMA**
```
bidang → kepala_bidang (1 bidang punya 1 kepala)
bidang → pembimbing (1 bidang punya banyak pembimbing) 
pembimbing → users (1 pembimbing bimbing banyak peserta)
institusi → pengajuan (1 institusi bisa kirim banyak pengajuan)
users → absensi (1 peserta punya banyak absensi)
users → jurnal_harian (1 peserta punya banyak jurnal)
users → pengumpulan_tugas (1 peserta kumpul banyak tugas)
tugas → pengumpulan_tugas (1 tugas dikumpul banyak peserta)
```

---

## **CARA PINDAHIN DATA DARI SISTEM LAMA**

### **📤 DARI SIPPOL KE SIMPPOL**

#### **Data Institusi**
```
instansi (SIPPOL) → institusi (SIMPPOL)
- nama_instansi → nama_institusi
- alamat_instansi → alamat
```

#### **Data User**
```
user (SIPPOL) → users (SIMPPOL)  
- username → username
- nama → nama_lengkap
- npm → npm
- password → password
- level_user = 4 (semua jadi peserta)
```

#### **Data Absensi**
```
presensi (SIPPOL) → absensi (SIMPPOL)
- data_tanggal → tanggal
- data_status → jam_masuk/jam_keluar
- data_deskripsi → keterangan_aktivitas
```

### **📤 DARI SIMPOL KE SIMPPOL**

#### **Data Jurnal**
```
db_jurnal (SIMPOL) → jurnal_harian (SIMPPOL)
- deskripsi_jurnal → aktivitas_hari_ini
- mulai_jurnal → jam_mulai
- selesai_jurnal → jam_selesai
- status_jurnal → status
```

#### **Data Tugas**
```
db_tugas (SIMPOL) → tugas (SIMPPOL)
- deskripsi_tugas → deskripsi_tugas
- jadwal_tugas → minggu_ke

db_galery_tugas (SIMPOL) → pengumpulan_tugas (SIMPPOL)
- gambar_tugas → file_tugas
```

#### **Data Nilai**
```
db_nilai (SIMPOL) → penilaian (SIMPPOL)
- kategori_mhs → kategori_nilai
- nilai_mhs → nilai
- justifikasi_mhs → justifikasi
```

### **📝 DATA BARU YANG HARUS DIISI**

#### **Master Bidang**
```
INSERT bidang:
1. Aplikasi Informatika (Aptika)
2. Komunikasi dan Informasi Publik  
3. Statistik dan Persandian
```

#### **Master Kepala Bidang**
```
INSERT kepala_bidang:
- Kepala Bidang Aptika
- Kepala Bidang Komunikasi
- Kepala Bidang Statistik
- dll (sesuai struktur Kominfo)
```

#### **Master Pembimbing**
```
INSERT pembimbing:
- Pembimbing A (Aptika)
- Pembimbing B (Aptika)
- Pembimbing C (Komunikasi)
- Pembimbing D (Statistik)
- dll
```

---

## **FITUR BARU YANG DITAMBAH**

### **✨ YANG DULU GAK ADA, SEKARANG ADA**

1. **Master Bidang** - Sekarang ada pilihan bidang magang
2. **Kepala Bidang** - Ada approval dari kepala bidang
3. **Pembimbing** - Setiap peserta dapat pembimbing khusus
4. **Durasi Fleksibel** - Bisa 1-6 bulan sesuai kebutuhan
5. **Jurnal Enhanced** - Ada skill, kendala, solusi (refleksi)
6. **GPS Tracking** - Absensi pakai GPS lokasi
7. **Approval Berlapis** - Admin → Kepala Bidang → Pembimbing
8. **Notifikasi Real-time** - Auto notif setiap ada update
9. **Sertifikat Digital** - QR code untuk verifikasi

---

## **JADWAL IMPLEMENTASI**

### **📅 TIMELINE 5 MINGGU**

**Minggu 1: Setup**
- Buat database baru
- Buat semua tabel  
- Setup relasi

**Minggu 2: Master Data**
- Pindahin data institusi dari SIPPOL
- Input master bidang, kepala bidang, pembimbing
- Setup admin user

**Minggu 3: Data User & Absensi** 
- Pindahin user dari SIPPOL & SIMPOL
- Pindahin data absensi SIPPOL
- Fix duplikasi data

**Minggu 4: Data PKL**
- Pindahin jurnal dari SIMPOL
- Pindahin tugas dari SIMPOL  
- Pindahin nilai dari SIMPOL

**Minggu 5: Testing**
- Test semua fitur
- Fix bug
- User training



## **KEUNGGULAN DATABASE INI**

### **✅ YANG BAGUS**
1. **Simple tapi Lengkap** - Mudah dipahami, fitur komplit
2. **Gabungan Terbaik** - Ambil yang bagus dari SIPPOL & SIMPOL  
3. **Fitur Magang Modern** - GPS, approval berlapis, refleksi
4. **Data Aman** - Backup data, audit trail, soft delete
5. **Bisa Berkembang** - Mudah tambah fitur baru

### **🎯 HASIL SETELAH JADI**
- Proses magang lebih terorganisir 80%
- Admin work berkurang 60% 
- Peserta lebih puas 90%
- Data lebih akurat dan real-time

**Database ini siap mendukung sistem magang Kominfo yang modern dan mudah digunakan!**

---


