# Requirements Document

## Introduction

This document defines the functional and non-functional requirements for the foundation stage of the **PAL-MRO Integrated Portal (SMART-HARKAN)** platform, built for the Pemeliharaan dan Perbaikan (Harkan) Division of PT PAL Indonesia. Stage 1 — System Foundation, RBAC & Master Data — covers the base application architecture, the maritime corporate visual identity, role-based authentication and authorization (RBAC), master data management for dock facilities and ship MRO projects, automated ship-vs-dock capacity validation, and an immutable audit trail.

This foundation stage establishes the shared infrastructure, security model, and master data upon which later stages (Dock Capacity Planner, Digital Repair List & E-Surveyor, S-Curve Monitoring, Client Portal, Material Readiness Tracker) will build.

## Glossary

- **System**: The SMART-HARKAN platform (PAL-MRO Integrated Portal) — the web application defined by this document. Used as the system name in EARS acceptance criteria ("THE SYSTEM SHALL ...").
- **SMART-HARKAN**: PAL-MRO Integrated Portal for PT PAL Indonesia's Harkan Division.
- **Harkan**: Pemeliharaan dan Perbaikan (Maintenance and Repair) Division of PT PAL Indonesia.
- **RBAC**: Role-Based Access Control — authorization model restricting access by operational role.
- **Role**: One of five operational user roles used by the System: `ADMIN`, `PROJECT_MANAGER`, `FIELD_ENGINEER`, `CLASS_SURVEYOR`, `CLIENT`.
  - `ADMIN`: System administrator with access to system configuration and master data management.
  - `PROJECT_MANAGER`: Project manager responsible for planning MRO projects and managing dock facilities and master data.
  - `FIELD_ENGINEER`: Field engineer performing on-site inspections (often in low-connectivity dock areas).
  - `CLASS_SURVEYOR`: Class surveyor (e.g., BKI) performing certification inspections.
  - `CLIENT`: External client/owner (e.g., Satgas TNI AL, government agency, commercial owner) owning one or more ships.
- **Dock Facility**: PT PAL docking infrastructure — either a Graving Dock or a Floating Dock — characterized by maximum dimensional capacity limits.
- **Graving Dock**: Dry dock built into the shoreline (`dock_type` = `GRAVING_DOCK`).
- **Floating Dock**: Floating docking structure (`dock_type` = `FLOATING_DOCK`).
- **MRO Project**: Maintenance, Repair, and Overhaul project for a specific ship, registered in the System with a unique project code.
- **LOA**: Length Overall — total length of a ship, measured in meters (m).
- **Beam**: Maximum width of a ship, measured in meters (m).
- **Draft**: Maximum depth of a ship's hull below the waterline, measured in meters (m).
- **Soft Delete**: Logical deletion that marks a record with a `deleted_at` timestamp without physically removing the record from the database.
- **AuditLog**: Immutable, append-only log table recording every `CREATE`, `UPDATE`, or `SOFT_DELETE` operation on audited entities.
- **Client Instance**: The external agency or organization (`client_id`) that owns a ship (e.g., a TNI AL unit, a government body, or a commercial company).

## Requirements

### Requirement 1: Setup Arsitektur & Standar Antarmuka Maritim

**User Story:** Sebagai pengembang dan pengguna sistem PT PAL Indonesia, saya membutuhkan kerangka aplikasi web yang responsif, aman, dan memiliki identitas visual maritim korporat agar siap digunakan di kantor manajemen maupun perangkat lapangan.

#### Acceptance Criteria

1. THE SYSTEM SHALL dibangun di atas Next.js (App Router) dengan TypeScript strict mode aktif dan Tailwind CSS, menerapkan styling melalui kelas Tailwind tanpa inline styles (atribut `style` inline pada elemen antarmuka).
2. WHILE ditampilkan pada lebar viewport antara 360px dan 1280px, THE SYSTEM SHALL merender tata letak (navigasi dan header) yang beradaptasi terhadap lebar viewport tersebut tanpa menimbulkan scroll horizontal.
3. WHEN pengguna membuka halaman operasional (yaitu setiap halaman selain panel konfigurasi admin), THE SYSTEM SHALL menampilkan label antarmuka dalam Bahasa Indonesia sebagai bahasa utama dan menerapkan palet warna maritim korporat (Deep Navy Blue `#0A192F`, Steel Slate `#334155`, Safety Orange `#F97316`) sebagai skema warna utama antarmuka.
4. WHEN pengguna mengaktifkan atau menonaktifkan Dark Mode melalui saklar pada antarmuka, THE SYSTEM SHALL mengganti skema warna antarmuka ke mode terang atau gelap sesuai pilihan dan mempertahankan preferensi tersebut antar sesi.
5. WHEN pengguna dengan peran `ADMIN` membuka panel konfigurasi sistem (halaman konfigurasi admin), THE SYSTEM SHALL menampilkan label antarmuka panel tersebut dalam Bahasa Inggris.

### Requirement 2: Autentikasi & Otorisasi Berbasis Peran (RBAC)

**User Story:** Sebagai Administrator Sistem PT PAL, saya ingin membatasi akses setiap pengguna berdasarkan peran operasional mereka agar kerahasiaan data proyek kapal perang (TNI AL) dan kapal komersial tetap terjaga.

#### Acceptance Criteria

1. WHEN pengguna mengirimkan kredensial login yang cocok dengan akun pengguna terdaftar, THE SYSTEM SHALL menerbitkan sesi terenkripsi (HTTP-only cookie / JWT) yang mengidentifikasi pengguna beserta perannya dan, untuk peran `CLIENT`, instansi klien pengguna, lalu mengarahkan pengguna ke dashboard sesuai perannya (`ADMIN`, `PROJECT_MANAGER`, `FIELD_ENGINEER`, `CLASS_SURVEYOR`, atau `CLIENT`).
2. IF pengguna tidak terautentikasi mencoba mengakses rute `/dashboard/*` atau `/api/*` (selain rute auth), THEN THE SYSTEM SHALL menolak akses dengan mengembalikan kode status HTTP `401 Unauthorized` untuk rute `/api/*` dan mengarahkan ke halaman `/login` untuk rute `/dashboard/*`.
3. IF pengguna dengan peran `CLIENT`, `FIELD_ENGINEER`, atau `CLASS_SURVEYOR` mencoba mengakses rute konfigurasi admin atau manajemen master fasilitas dock, THEN THE SYSTEM SHALL menolak permintaan dengan kode status HTTP `403 Forbidden`.
4. WHEN pengguna dengan peran `CLIENT` (misal: Satgas TNI AL) melihat daftar proyek, THE SYSTEM SHALL hanya menampilkan proyek kapal yang terhubung dengan ID instansi klien tersebut.
5. IF pengguna mengirimkan kredensial login yang tidak cocok dengan akun pengguna terdaftar, THEN THE SYSTEM SHALL menolak proses login, tidak menerbitkan sesi, dan menampilkan pesan kesalahan dalam Bahasa Indonesia yang menunjukkan bahwa kredensial tidak valid.

### Requirement 3: Manajemen Master Data Fasilitas Dock

**User Story:** Sebagai Project Manager Divisi Harkan, saya ingin mengelola data fasilitas graving dock dan floating dock milik PT PAL beserta batas dimensi maksimalnya agar penempatan kapal dapat direncanakan secara akurat.

#### Acceptance Criteria

1. WHEN `ADMIN` atau `PROJECT_MANAGER` menambahkan atau mengubah data fasilitas dock, THE SYSTEM SHALL memvalidasi bahwa nilai dimensi (`max_loa`, `max_beam`, `max_draft`) adalah angka positif lebih besar dari `0,00` hingga maksimal `999,99` dalam satuan meter (`m`) dengan presisi maksimal 2 angka di belakang koma, dan bahwa kapasitas angkat (`max_tonnage`) adalah angka positif lebih besar dari `0,00` hingga maksimal `999.999,99` dalam satuan metrik ton.
2. WHEN fasilitas dock disimpan, THE SYSTEM SHALL menyimpan atribut: nama fasilitas (contoh: Graving Dock Semarang, Graving Dock Surabaya, Floating Dock) dengan panjang 1 hingga 100 karakter, tipe dok (`GRAVING_DOCK` | `FLOATING_DOCK`), `max_loa`, `max_beam`, `max_draft`, kapasitas angkat (`max_tonnage`) dalam satuan metrik ton, dan status operasional (`ACTIVE` | `MAINTENANCE` | `INACTIVE`).
3. IF fasilitas dock sedang memiliki proyek aktif dengan status `IN_DOCK` dan pengguna dengan peran `ADMIN` atau `PROJECT_MANAGER` mencoba menghapusnya, THEN THE SYSTEM SHALL memblokir penghapusan, mempertahankan catatan fasilitas dock tanpa perubahan, dan menampilkan pesan kesalahan dalam Bahasa Indonesia yang menunjukkan bahwa fasilitas sedang digunakan oleh proyek aktif.
4. IF validasi dimensi atau kapasitas angkat gagal saat penambahan atau perubahan data fasilitas dock, THEN THE SYSTEM SHALL menolak penyimpanan, mempertahankan nilai input pengguna, dan menampilkan pesan kesalahan dalam Bahasa Indonesia yang menunjukkan field mana yang tidak valid.
5. WHEN pengguna dengan peran `ADMIN` atau `PROJECT_MANAGER` menghapus fasilitas dock yang tidak memiliki proyek aktif dengan status `IN_DOCK`, THE SYSTEM SHALL melakukan soft delete dengan mengisi `deleted_at` menggunakan timestamp saat ini tanpa menghapus catatan dari database.

### Requirement 4: Manajemen Master Proyek Kapal (MRO Project) & Validasi Kapasitas Dock

**User Story:** Sebagai Project Manager Divisi Harkan, saya ingin mendaftarkan proyek pemeliharaan dan perbaikan kapal baru serta mendapatkan validasi otomatis apakah dimensi kapal sesuai dengan kapasitas fasilitas dock yang dipilih.

#### Acceptance Criteria

1. WHEN `PROJECT_MANAGER` membuat proyek MRO baru, THE SYSTEM SHALL mewajibkan input: kode proyek unik (format `HKN-YYYY-XXX`), nama kapal, kategori kapal (`WARSHIP_TNI_AL` | `GOVERNMENT` | `COMMERCIAL`), instansi pemilik (`client_id`), dimensi kapal (`loa`, `beam`, `draft` berupa angka positif dalam meter dengan presisi maksimal 2 desimal), fasilitas dock tujuan (`dock_id`), tanggal rencana masuk dok (`planned_docking_date`), dan tanggal rencana keluar dok (`planned_undocking_date`).
2. WHEN `loa`, `beam`, atau `draft` kapal melebihi `max_loa`, `max_beam`, atau `max_draft` pada fasilitas dock yang dipilih, THE SYSTEM SHALL menggagalkan penyimpanan proyek dan mengembalikan pesan peringatan spesifik dimensi mana yang melebihi batas kapasitas dok.
3. IF `planned_undocking_date` lebih awal atau sama dengan `planned_docking_date`, THEN THE SYSTEM SHALL menolak penyimpanan dan mengembalikan pesan validasi yang menyatakan bahwa tanggal rencana keluar dok harus lebih akhir dari tanggal rencana masuk dok.
4. WHEN seorang `ADMIN` atau `PROJECT_MANAGER` menghapus proyek MRO, THE SYSTEM SHALL melakukan soft delete dengan mengisi `deleted_at` menggunakan timestamp, mengeluarkan proyek tersebut dari daftar proyek aktif, dan mempertahankan catatan historis dalam database.
5. IF pengajuan proyek MRO baru memiliki field wajib yang kosong, kode proyek yang tidak sesuai format `HKN-YYYY-XXX`, kode proyek yang sudah ada pada sistem, dimensi kapal (`loa`, `beam`, atau `draft`) yang bukan angka positif dengan maksimal 2 desimal, atau `dock_id` yang merujuk pada fasilitas dock dengan status operasional bukan `ACTIVE`, THEN THE SYSTEM SHALL menolak penyimpanan dan mengembalikan pesan validasi yang menunjukkan field dan aturan spesifik yang gagal.

### Requirement 5: Jejak Audit Tidak Terubah (Immutable Audit Trail)

**User Story:** Sebagai Manajemen PT PAL dan Auditor, saya ingin setiap perubahan pada data fasilitas dock dan proyek kapal tercatat secara permanen untuk keperluan akuntabilitas proyek pertahanan dan komersial.

#### Acceptance Criteria

1. WHEN terjadi operasi `CREATE`, `UPDATE`, atau `SOFT_DELETE` pada entitas `DockFacility` atau `MroProject`, THE SYSTEM SHALL secara atomik menyimpan entri baru ke dalam tabel `AuditLog` yang berisi `user_id`, `user_role`, `entity_type`, `entity_id`, `action`, `old_values` (JSON berisi nilai entitas sebelum operasi, kosong untuk operasi `CREATE`), `new_values` (JSON berisi nilai entitas setelah operasi), dan `created_at`.
2. THE SYSTEM SHALL NOT menyediakan antarmuka maupun rute API untuk mengubah (`UPDATE`) atau menghapus (`DELETE`) entri di dalam tabel `AuditLog`.
3. IF penyimpanan entri `AuditLog` gagal, THEN THE SYSTEM SHALL membatalkan operasi yang memicu pencatatan audit, mengembalikan indikasi kesalahan kepada pengguna, dan tidak menyimpan perubahan pada entitas yang terkait.

## Correctness Properties

The following properties capture the testable invariants of the foundation stage's validation and audit logic. These are strong candidates for property-based testing because they exercise the System's own logic over varied input at low cost (pure validation functions, in-memory state).

1. **Dock Capacity Validation (Invariant / Model-Based):** For all ship dimension sets `(loa, beam, draft)` and dock capacity limits `(max_loa, max_beam, max_draft)`, a project passes the capacity check if and only if `loa <= max_loa` AND `beam <= max_beam` AND `draft <= max_draft`. When any dimension exceeds its limit, the rejection message SHALL identify the specific dimension(s) that failed.

2. **Date Ordering (Invariant):** For all pairs `(planned_docking_date, planned_undocking_date)`, the project date validation accepts if and only if `planned_undocking_date > planned_docking_date`.

3. **Dimension Precision and Positivity (Invariant):** For all dimension values, the value is accepted if and only if it is a positive number with at most 2 decimal places expressed in meters.

4. **Audit Log Append-Only (Idempotence / Invariant):** After any sequence of `CREATE`, `UPDATE`, or `SOFT_DELETE` operations on `DockFacility` or `MroProject`, the count of `AuditLog` entries SHALL equal the number of audited operations performed, and no existing `AuditLog` entry SHALL ever be modified or removed.

5. **Client Data Isolation (Metamorphic):** For a `CLIENT`-role user, the set of projects returned is always a subset of projects whose `client_id` matches that user's instance; the visible project set can never include a project belonging to another client instance.

## Out of Scope

Stage 1 covers system foundation, RBAC, and master data only. The following capabilities are deferred to later stages and are explicitly out of scope for this document:

- **Dock Capacity Planner** — scheduling and visualization of dock occupancy over time, including the Gantt chart.
- **Digital Repair List & E-Surveyor** — field inspection forms, offline-first queue, photo upload and client-side compression, and ultrasonic thickness measurement recording.
- **S-Curve Monitoring** — progress weight calculation, schedule deviation, and cumulative-weight-to-100% tracking.
- **Client Portal** — client-facing self-service views beyond the read-only project list covered by Requirement 2.4.
- **Material Readiness Tracker** — spare parts and material availability management.
- **Multi-language internationalization framework** beyond the Indonesia-primary / English-admin convention defined in Requirement 1.
- **Email and notification delivery infrastructure**, except where needed to support the authentication session issuance in Requirement 2.1.
