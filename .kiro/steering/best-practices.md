# Praktik Terbaik Rekayasa Perangkat Lunak — SMART-HARKAN (PT PAL Indonesia)

Aturan berikut wajib dipatuhi secara konsisten di sepanjang pengembangan platform **PAL-MRO Integrated Portal (SMART-HARKAN)** untuk Divisi Pemeliharaan dan Perbaikan PT PAL Indonesia:

## Praktik Git (Git Practices)

- Setelah menyelesaikan setiap tugas dari `tasks.md`, segera jalankan `git add .` dan `git commit -m "[pesan deskriptif]"` meskipun belum ada repositori jarak jauh (*remote repository*) yang dikonfigurasi.

- Gunakan format *conventional commit* dengan konteks modul proyek: `feat(dock-scheduler): ...`, `fix(s-curve): ...`, `chore(setup): ...`, `docs(readme): ...`.

- Jangan pernah menyisakan perubahan yang belum di-*commit* saat berpindah ke tugas berikutnya.

## Pengujian (Testing)

- Tulis setidaknya satu *unit test* per fungsi utilitas (*utility function*) — seperti kalkulasi bobot progres *S-Curve*, validasi dimensi kapal terhadap kapasitas *Graving/Floating Dock*, dan perhitungan deviasi jadwal — serta satu *integration test* per rute API (*API route*).

- Jaga agar file pengujian tetap ringkas — maksimal 3 kasus uji (*test cases*) per file untuk menghemat penggunaan token.

- Gunakan Jest untuk *unit test* dan Playwright atau Supertest untuk pengujian API (termasuk pengujian alur persetujuan *Additional Work / E-Surveyor*).

- File pengujian ditempatkan di dalam folder `tests/` yang sejajar dengan file yang sedang diuji.

## Keamanan (Security)

- Di akhir seluruh sesi pengembangan (*vibe coding*), lakukan tinjauan keamanan (*security review*): periksa apakah ada rahasia yang di-*hardcode* (*hardcoded secrets*), validasi input yang terlewat, rute API yang tidak terlindungi, dan *middleware* autentikasi serta otorisasi berbasis peran (*RBAC* untuk Klien/TNI AL, Project Manager, Field Engineer, Class Surveyor BKI, dan Admin) yang hilang.

- Jangan pernah melakukan *commit* pada file `.env`.

- Selalu validasi dan sanitasi input pengguna di sisi server (*server side*), terutama pada unggahan foto inspeksi lapangan, pengukuran *ultrasonic thickness*, dan dokumen sertifikasi kapal.

## Dokumentasi (Documentation)

- Kelola file `README.md` di *root* proyek SMART-HARKAN dan perbarui setiap kali selesai menambahkan modul utama (seperti *Dock Capacity Planner*, *Digital Repair List & E-Surveyor*, *S-Curve Monitoring*, *Client Portal*, atau *Material Readiness Tracker*).

- `README.md` harus mencakup: deskripsi proyek SMART-HARKAN, *tech stack*, cara menjalankan secara lokal, *environment variables* yang dibutuhkan, dan gambaran struktur folder.

## Kualitas Kode (Code Quality)

- Selalu gunakan *strict mode* TypeScript di seluruh lapisan kode.

- Semua komponen harus berupa komponen fungsional React (*functional React components*).

- Gunakan Tailwind CSS untuk semua pengaturan gaya (*styling*) dengan palet maritim korporat (*Deep Navy Blue*, *Steel Slate*, *Safety Orange*, dan dukungan *Dark Mode*) — dilarang menggunakan *inline styles*.

- Bahasa utama antarmuka pengguna (UI) adalah Bahasa Indonesia (menyesuaikan terminologi standar galangan kapal dan MRO PT PAL); panel admin dan konfigurasi sistem menggunakan label Bahasa Inggris.

## Manajemen State & Sinkronisasi Offline (Offline-First / Field Mode)

- **Local Storage / IndexedDB Queue:** Karena *Field Engineer* sering melakukan inspeksi di dalam area *graving dock* atau kompartemen lambung kapal dengan sinyal internet yang lemah, setiap pengisian formulir *E-Surveyor* dan unggahan foto kerusakan harus disimpan terlebih dahulu secara lokal dan otomatis disinkronkan saat koneksi kembali stabil.

- **Optimistic UI:** Gunakan pembaruan antarmuka optimistik pada *checklist* pekerjaan harian agar aplikasi terasa responsif di perangkat tablet maupun ponsel lapangan.

## Integritas Data & Jejak Audit (Audit Trail)

- **Immutable Audit Log:** Setiap perubahan pada jadwal *docking*, revisi bobot *S-Curve*, dan persetujuan *Additional Work* oleh Klien atau *Class Surveyor* (BKI) wajib mencatat `user_id`, `role`, `timestamp`, dan nilai sebelum/sesudah perubahan (*history log*). Data persetujuan yang sudah dikunci tidak boleh dihapus secara permanen (*soft delete only*).

- **Standarisasi Satuan Maritim:** Seluruh kalkulasi dimensi kapal (LOA, *Beam*, *Draft*) wajib menggunakan satuan meter (`m`) dengan presisi 2 angka di belakang koma, dan bobot pekerjaan proyek menggunakan persentase (`%`) dengan total kumulatif tepat `100.00%`.

## Penanganan Media & Dokumen Inspeksi

- **Kompresi Gambar Otomatis di Sisi Klien:** Foto bukti kerusakan lapangan (*Before-After* atau hasil ukur *ultrasonic thickness*) wajib dikompresi di sisi *frontend* maksimal menjadi `1MB` per file sebelum diunggah agar tidak membebani *bandwidth* lapangan dan penyimpanan server.

- **Penamaan File Terstruktur:** Setiap dokumen sertifikasi atau laporan inspeksi yang diunduh dari sistem harus mengikuti format baku: `[KODE_PROYEK]_[NAMA_KAPAL]_[JENIS_DOKUMEN]_[YYYYMMDD].pdf`.

## Kinerja Visualisasi Data (Chart & Gantt Performance)

- **Virtualisasi Daftar Pekerjaan:** Mengingat satu proyek *overhaul* kapal dapat memiliki ribuan item *Master Repair List*, tabel daftar pekerjaan wajib menggunakan teknik *pagination* atau *virtual scrolling* agar peramban (*browser*) tidak mengalami *lag*.

- **Memoization pada Komponen Grafik:** Komponen *Gantt Chart* jadwal *dock* dan grafik *S-Curve* wajib dibungkus dengan `React.memo` atau `useMemo` untuk mencegah *re-render* yang tidak perlu saat filter departemen diubah.
