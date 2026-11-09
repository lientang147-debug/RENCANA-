### SPESIFIKASI BACKEND GATEOPS

GATE Operations & Personnel Monitoring System

Dokumen spesifikasi teknis — Database, Backend, dan Logika Bisnis

### 1\. Gambaran Umum

Backend GATEOPS merupakan pusat pengolahan data dan proses bisnis yang menghubungkan frontend website dengan database. Backend bertanggung jawab atas autentikasi pengguna, pengaturan hak akses, pengajuan keluar, persetujuan otomatis, transaksi QR Code, GPS monitoring, pencatatan waktu, pengelolaan sesi, dan penyusunan laporan.

Backend menggunakan Next.js API, sedangkan penyimpanan data menggunakan PostgreSQL. Seluruh proses transaksi penting divalidasi oleh server untuk menjaga keamanan, konsistensi, dan keakuratan data.

Spesifikasi ini menggunakan ketentuan operasional yang telah disepakati:

* Satu NRP memiliki maksimal satu akun Personel dan satu akun Pasangan/Persit.

* Kedua jenis akun menggunakan role User dengan PIN masing-masing.

* Seluruh pengguna menggunakan satu halaman login terpusat.

* Satu QR Code pos digunakan untuk scan keluar maupun kembali.

* Batas waktu scan keluar maksimal 2 jam sejak pengajuan.

* GPS mengirimkan pembaruan lokasi setiap 60 detik.

* GPS dimulai setelah scan keluar berhasil dan berhenti setelah scan kembali berhasil.

### 2\. Spesifikasi Database

Database menggunakan PostgreSQL sebagai penyimpanan terpusat untuk seluruh data GATEOPS.

### 2.1 Struktur tabel database

Database dikelompokkan berdasarkan fungsi agar mudah dikelola dan dikembangkan.

| No. | Nama tabel            | Fungsi                                                                  |
| --- | --------------------- | ----------------------------------------------------------------------- |
| 1   | users                 | Menyimpan akun login, NRP, PIN hash, jenis akun, role, dan status akun. |
| 2   | personnel_profiles    | Menyimpan identitas dan informasi personel.                             |
| 3   | family_profiles       | Menyimpan identitas pasangan/Persit serta relasinya dengan personel.    |
| 4   | ranks                 | Menyimpan referensi pangkat.                                            |
| 5   | positions             | Menyimpan referensi jabatan.                                            |
| 6   | units                 | Menyimpan referensi satuan atau unit kerja.                             |
| 7   | exit_requests         | Menyimpan pengajuan rencana keluar dan status persetujuannya.           |
| 8   | activity_sessions     | Menyimpan sesi aktivitas keluar dan kembali.                            |
| 9   | qr_codes              | Menyimpan identitas, konfigurasi, dan status QR Code pos.               |
| 10  | gps_tracking_sessions | Menyimpan periode pemantauan GPS.                                       |
| 11  | gps_locations         | Menyimpan koordinat dan waktu penerimaan lokasi.                        |
| 12  | notifications         | Menyimpan notifikasi kepada pengguna.                                   |
| 13  | audit_logs            | Menyimpan catatan aktivitas dan perubahan data penting.                 |
| 14  | system_settings       | Menyimpan konfigurasi sistem, termasuk batas scan dan interval GPS.     |

### 2.2 Relasi akun berdasarkan NRP

Satu NRP dapat memiliki maksimal dua akun berbeda.

| Jenis akun      | Identitas login           | Role |
| --------------- | ------------------------- | ---- |
| Personel        | NRP + PIN Personel        | User |
| Pasangan/Persit | NRP + PIN Pasangan/Persit | User |

Ketentuan database:

* Satu NRP hanya boleh memiliki satu akun Personel.

* Satu NRP hanya boleh memiliki satu akun Pasangan/Persit.

* Kombinasi NRP dan jenis akun wajib unik.

* PIN setiap akun berbeda dan disimpan dalam bentuk hash.

* Kedua akun terhubung melalui identitas personel yang sama.

* Kedua akun memiliki hak akses operasional yang sama.

* Setiap transaksi tetap mencatat ID akun yang benar-benar melakukan aktivitas.

Pada tabel `users`, struktur identitas minimal meliputi:

| Field        | Fungsi                                    |
| ------------ | ----------------------------------------- |
| id           | Primary key akun                          |
| nrp          | Nomor Registrasi Pokok                    |
| account_type | PERSONNEL atau SPOUSE                     |
| pin_hash     | Hash PIN login                            |
| role         | SUPER_ADMIN, ADMIN_OPERATIONAL, atau USER |
| is_active    | Status aktif akun                         |
| created_at   | Waktu pembuatan akun                      |
| updated_at   | Waktu pembaruan akun                      |

Constraint database harus memastikan kombinasi `nrp + account_type` bersifat unik.

### 2.3 Relasi antar tabel

Hubungan data utama dirancang sebagai berikut:

* Satu personel dapat memiliki maksimal satu akun Personel dan satu akun Pasangan/Persit.

* Satu akun dapat membuat banyak pengajuan keluar.

* Satu pengajuan yang diproses dapat menghasilkan satu sesi aktivitas.

* Satu sesi aktivitas memiliki satu transaksi keluar dan maksimal satu transaksi kembali.

* Satu sesi aktivitas dapat memiliki banyak catatan GPS.

* Satu sesi GPS terhubung dengan satu sesi aktivitas.

* Setiap aktivitas penting dapat memiliki catatan audit.

* Satu QR Code pos dapat digunakan untuk transaksi keluar maupun kembali.

Seluruh hubungan menggunakan primary key dan foreign key untuk menjaga integritas data.

### 2.4 Ketentuan penyimpanan database

* Seluruh PIN disimpan menggunakan hash Argon2id atau bcrypt.

* Waktu transaksi dicatat berdasarkan waktu server.

* Penyimpanan waktu menggunakan UTC dan ditampilkan dalam zona waktu Asia/Jakarta.

* Kolom NRP, status, tanggal, dan relasi sesi dilengkapi indeks sesuai kebutuhan pencarian.

* Data transaksi penting tidak dihapus secara permanen tanpa prosedur yang sah.

* Database memiliki mekanisme backup berkala.

* Data GPS dihubungkan dengan ID sesi agar riwayat perjalanan dapat ditelusuri.

### 3\. Spesifikasi Backend

Backend menggunakan Next.js API sebagai pusat pemrosesan permintaan dari frontend. Setiap proses yang mengubah data harus melewati validasi backend sebelum disimpan ke database.

### 3.1 Komponen backend

| Komponen           | Fungsi                                                                        |
| ------------------ | ----------------------------------------------------------------------------- |
| Authentication     | Mengelola login, logout, dan sesi autentikasi.                                |
| Authorization      | Memvalidasi role dan hak akses pengguna.                                      |
| User Management    | Mengelola akun Personel, Pasangan/Persit, Admin Operasional, dan Super Admin. |
| Exit Request       | Mengelola pengajuan keluar dan persetujuan otomatis.                          |
| QR Transaction     | Memvalidasi scan QR untuk transaksi keluar dan kembali.                       |
| GPS Tracking       | Menerima, memvalidasi, dan menyimpan koordinat GPS.                           |
| Session Management | Mengelola status dan siklus sesi aktivitas keluar.                            |
| Reporting          | Mengolah data transaksi, durasi keluar, dan riwayat perjalanan.               |
| Notification       | Mengelola pemberitahuan sistem kepada pengguna.                               |
| Audit Logging      | Merekam aktivitas penting dan perubahan data.                                 |
| Scheduler          | Menjalankan proses otomatis, termasuk pemeriksaan pengajuan kedaluwarsa.      |

### 3.2 Autentikasi dan hak akses

Seluruh pengguna menggunakan satu halaman login terpusat dengan input NRP dan PIN. Tidak ada pilihan role pada halaman login.

Alur autentikasi:

* Pengguna memasukkan NRP dan PIN.

* Backend mencari akun aktif berdasarkan NRP.

* Backend memverifikasi PIN terhadap akun yang terdaftar.

* Sistem mengidentifikasi jenis akun dan role.

* Backend membuat sesi autentikasi.

* Pengguna diarahkan ke dashboard berdasarkan role.

Jika satu NRP memiliki dua akun, backend memeriksa kecocokan PIN terhadap akun Personel dan Pasangan/Persit. Sistem hanya mengizinkan login apabila ditemukan satu akun yang cocok.

Pembagian hak akses:

| Fitur             | Super Admin      | Admin Operasional   | User            |
| ----------------- | ---------------- | ------------------- | --------------- |
| Dashboard         | Seluruh sistem   | Operasional         | Pribadi         |
| Pengelolaan akun  | Penuh            | Sesuai kewenangan   | Tidak           |
| Pengajuan keluar  | Melihat          | Memantau            | Membuat sendiri |
| Scan QR           | Mengelola QR     | Memantau transaksi  | Melakukan scan  |
| GPS Monitoring    | Seluruh pengguna | Seluruh pengguna    | Lokasi sendiri  |
| Laporan           | Seluruh laporan  | Laporan operasional | Riwayat pribadi |
| Pengaturan sistem | Penuh            | Tidak               | Tidak           |
| Audit trail       | Akses penuh      | Tidak               | Tidak           |

Seluruh pembatasan akses diterapkan pada backend, bukan hanya dengan menyembunyikan menu di frontend.

### 3.3 Pengelolaan sesi login

Sesi login dan sesi aktivitas keluar merupakan dua hal yang berbeda.

Backend harus memastikan:

* Login tetap valid selama sesi autentikasi masih berlaku.

* Refresh browser tidak membuat pengajuan baru.

* Status aktivitas diambil dari database.

* Pengguna yang masih memiliki sesi keluar aktif diarahkan ke halaman monitoring atau scan QR sesuai status terakhir.

* Logout tidak otomatis mengakhiri sesi aktivitas keluar.

* Login kembali akan memulihkan status aktivitas sebelumnya.

### 4\. Logika Bisnis GATEOPS

### 4.1 Pengajuan rencana keluar

Formulir pengajuan keluar terdiri dari:

| Field               | Ketentuan                        |
| ------------------- | -------------------------------- |
| Tujuan              | Wajib diisi                      |
| Keperluan           | Wajib diisi                      |
| Keterangan tambahan | Opsional                         |
| Identitas pengguna  | Diambil otomatis dari akun login |
| Waktu pengajuan     | Dicatat otomatis oleh server     |

Proses bisnis:

* User mengisi formulir rencana keluar.

* Frontend mengirimkan data ke backend.

* Backend memvalidasi kelengkapan data dan memastikan tidak ada sesi keluar aktif.

* Backend membuat pengajuan dengan status `APPROVED`.

* Sistem mencatat waktu pengajuan berdasarkan waktu server.

* Sistem membuat sesi aktivitas dengan status `WAITING_EXIT_SCAN`.

* Frontend otomatis diarahkan ke halaman scan QR.

Pengajuan menggunakan mekanisme Auto Approval, sehingga tidak membutuhkan persetujuan manual Admin Operasional atau Super Admin.

### 4.2 Batas waktu scan QR keluar

Setelah pengajuan berhasil, User diberikan waktu maksimal 2 jam untuk melakukan scan QR di pos.

Ketentuan:

* Hitung waktu dimulai dari waktu pengajuan yang dicatat server.

* Batas waktu dihitung menggunakan waktu pengajuan ditambah 2 jam.

* Refresh halaman, logout, atau login kembali tidak mengulang batas waktu.

* Jika belum melakukan scan sampai batas waktu, sesi otomatis kedaluwarsa.

* Scan setelah kedaluwarsa ditolak oleh backend.

* User harus membuat pengajuan baru.

* GPS belum aktif selama menunggu scan keluar.

Status sesi:

| Status            | Keterangan                                    |
| ----------------- | --------------------------------------------- |
| WAITING_EXIT_SCAN | Menunggu scan QR keluar.                      |
| EXPIRED           | Pengajuan kedaluwarsa setelah 2 jam.          |
| OUTSIDE           | Scan keluar berhasil dan User berada di luar. |
| COMPLETED         | User sudah kembali dan sesi selesai.          |
| CANCELLED         | Sesi dibatalkan melalui proses yang sah.      |

### 4.3 Transaksi QR Code keluar dan kembali

GATEOPS menggunakan satu QR Code yang sama untuk transaksi keluar maupun kembali.

Backend menentukan jenis transaksi berdasarkan status sesi aktivitas pengguna.

| Status sesi saat scan | Jenis transaksi | Tindakan sistem                              |
| --------------------- | --------------- | -------------------------------------------- |
| WAITING_EXIT_SCAN     | Keluar          | Mencatat waktu keluar dan mengaktifkan GPS.  |
| OUTSIDE               | Kembali         | Mencatat waktu kembali dan menghentikan GPS. |
| EXPIRED               | Tidak valid     | Menolak transaksi.                           |
| COMPLETED             | Tidak valid     | Meminta pengajuan baru.                      |
| CANCELLED             | Tidak valid     | Menolak transaksi.                           |

Alur scan keluar:

* User membuka fitur scan QR.

* Kamera browser membaca QR Code pos.

* Data QR dikirim ke backend.

* Backend memvalidasi QR Code dan status sesi.

* Backend memastikan pengajuan belum melewati batas 2 jam.

* Backend mencatat waktu keluar aktual.

* Status sesi berubah menjadi `OUTSIDE`.

* Sistem membuat sesi GPS monitoring.

* Frontend menampilkan status User sedang berada di luar.

Alur scan kembali:

* User melakukan scan QR Code yang sama saat kembali ke pos.

* Backend memvalidasi QR Code dan sesi aktif.

* Backend mencatat waktu kembali aktual.

* Backend menghitung durasi aktivitas di luar.

* Status sesi berubah menjadi `COMPLETED`.

* Sesi GPS monitoring diakhiri.

* Frontend diarahkan kembali ke dashboard User.

Validasi transaksi harus menggunakan proses database yang atomik agar scan berulang atau bersamaan tidak menghasilkan pencatatan ganda.

### 4.4 GPS Monitoring

GPS monitoring dimulai setelah transaksi scan keluar berhasil.

Interval GPS

60 detik

### 1 menit

Satu kali pengiriman lokasi setiap 60 detik selama kondisi perangkat dan browser mendukung.

Alur GPS:

* Setelah scan keluar berhasil, frontend meminta izin lokasi perangkat.

* Jika izin diberikan, browser mengambil koordinat GPS.

* Frontend mengirimkan lokasi setiap 60 detik.

* Backend memvalidasi identitas akun dan sesi aktivitas.

* Backend menyimpan koordinat dan waktu penerimaan.

* Admin Operasional dan Super Admin dapat melihat posisi pengguna pada peta.

* Riwayat koordinat digunakan untuk menampilkan perjalanan pengguna.

* Setelah scan kembali berhasil, backend mengakhiri sesi GPS.

Data lokasi yang disimpan:

| Field       | Keterangan                      |
| ----------- | ------------------------------- |
| id          | ID data lokasi                  |
| user_id     | ID akun pengguna                |
| session_id  | ID sesi aktivitas               |
| latitude    | Koordinat lintang               |
| longitude   | Koordinat bujur                 |
| accuracy    | Tingkat akurasi GPS dalam meter |
| recorded_at | Waktu pengambilan koordinat     |
| received_at | Waktu data diterima server      |

Penanganan kondisi GPS:

* Jika izin lokasi ditolak, sistem menampilkan status GPS tidak aktif.

* Jika internet terputus, sistem menampilkan waktu terakhir lokasi diterima.

* Jika pengiriman gagal, frontend dapat mencoba mengirim ulang data yang belum terkirim.

* Data yang dikirim ulang harus mempertahankan waktu pengambilan aslinya.

* Backend menolak pengiriman koordinat untuk sesi yang sudah selesai.

* Lokasi terakhir tidak boleh dianggap sebagai lokasi terkini jika sudah lama tidak diperbarui.

Interval 60 detik merupakan interval pengiriman yang ditargetkan, bukan jaminan perangkat selalu dapat mengirim lokasi tepat setiap menit.

### 4.5 Penghitungan durasi keluar

Durasi aktivitas di luar dihitung otomatis oleh backend.

\\text{Durasi Keluar} = \\text{Waktu Kembali} - \\text{Waktu Keluar}

Waktu keluar dan kembali tidak dapat diinput atau diubah secara manual oleh User.

Hasil penghitungan disimpan pada sesi aktivitas dan digunakan untuk kebutuhan dashboard serta laporan.

### 4.6 Pemulihan status aktivitas

Sistem harus dapat memulihkan status aktivitas ketika:

* Browser direfresh.

* Pengguna logout kemudian login kembali.

* Browser ditutup dan dibuka kembali.

* Koneksi internet terputus sementara.

Backend mengambil status terakhir dari database dan mengarahkan User ke halaman yang sesuai.

| Status terakhir                | Halaman yang ditampilkan                      |
| ------------------------------ | --------------------------------------------- |
| Belum memiliki pengajuan aktif | Dashboard User                                |
| WAITING_EXIT_SCAN              | Halaman scan QR dan countdown                 |
| OUTSIDE                        | Halaman monitoring dan instruksi scan kembali |
| EXPIRED                        | Informasi pengajuan kedaluwarsa               |
| COMPLETED                      | Dashboard User                                |

### 4.7 Kedaluwarsa otomatis

Backend menjalankan proses pemeriksaan berkala menggunakan scheduler.

Proses ini bertugas:

* Mencari sesi dengan status `WAITING_EXIT_SCAN`.

* Memeriksa waktu pengajuan.

* Membandingkan waktu server dengan batas 2 jam.

* Mengubah status menjadi `EXPIRED` apabila sudah melewati batas.

* Menolak transaksi scan keluar untuk sesi yang kedaluwarsa.

* Mencatat perubahan status pada audit log.

Proses kedaluwarsa tetap berjalan meskipun pengguna tidak membuka website.

### 5\. Rancangan API Backend

API digunakan sebagai penghubung antara frontend Next.js, proses backend, dan PostgreSQL.

| Metode | Endpoint                    | Fungsi                                   |
| ------ | --------------------------- | ---------------------------------------- |
| POST   | /api/auth/login             | Login menggunakan NRP dan PIN.           |
| POST   | /api/auth/logout            | Mengakhiri sesi login.                   |
| GET    | /api/auth/me                | Mengambil identitas dan role akun aktif. |
| GET    | /api/dashboard              | Mengambil data dashboard sesuai role.    |
| GET    | /api/users                  | Mengambil daftar pengguna.               |
| POST   | /api/users                  | Membuat akun pengguna.                   |
| PUT    | /api/users/:id              | Memperbarui data pengguna.               |
| POST   | /api/exit-requests          | Membuat pengajuan keluar.                |
| GET    | /api/exit-requests          | Mengambil daftar pengajuan.              |
| GET    | /api/activities/current     | Mengambil sesi aktivitas aktif.          |
| POST   | /api/qr/scan                | Memproses scan QR keluar atau kembali.   |
| POST   | /api/gps/locations          | Mengirim koordinat GPS.                  |
| GET    | /api/gps/current            | Mengambil posisi terbaru pengguna.       |
| GET    | /api/gps/history/:sessionId | Mengambil riwayat perjalanan.            |
| GET    | /api/reports                | Mengambil data laporan.                  |
| GET    | /api/audit-logs             | Mengambil riwayat aktivitas sistem.      |
| GET    | /api/settings               | Mengambil konfigurasi sistem.            |
| PUT    | /api/settings               | Memperbarui konfigurasi sistem.          |

Endpoint merupakan rancangan awal dan dapat disesuaikan dengan implementasi aplikasi.

### 6\. Keamanan Backend

Backend harus menerapkan pengamanan untuk menjaga kerahasiaan data pengguna dan keakuratan transaksi operasional.

Ketentuan keamanan:

* Autentikasi: PIN disimpan dalam bentuk hash menggunakan Argon2id atau bcrypt.

* Otorisasi: Setiap permintaan diperiksa berdasarkan role dan kewenangan.

* Pembatasan data: User hanya dapat mengakses data miliknya sendiri.

* Validasi QR: QR Code harus terdaftar dan berstatus aktif.

* Validasi waktu: Seluruh waktu transaksi ditentukan oleh server.

* Perlindungan transaksi ganda: Scan QR yang dikirim berulang tidak boleh menghasilkan transaksi ganda.

* Keamanan API: Menerapkan validasi input, pembatasan percobaan login, dan perlindungan akses tidak sah.

* Audit trail: Aktivitas login, perubahan data, pengajuan, scan QR, dan pengaturan sistem dicatat.

* Backup database: Menyediakan pencadangan data secara berkala.

* HTTPS: Seluruh komunikasi website dan server menggunakan koneksi terenkripsi.

### 7\. Pengaturan Konfigurasi Sistem

Pengaturan operasional disimpan pada tabel `system_settings` agar dapat diubah oleh Super Admin tanpa perlu mengubah kode program.

| Pengaturan              | Nilai awal                            |
| ----------------------- | ------------------------------------- |
| Batas waktu scan keluar | 2 jam                                 |
| Persetujuan pengajuan   | Otomatis                              |
| Jenis QR Code           | Satu QR Code untuk keluar dan kembali |
| Penentuan transaksi QR  | Berdasarkan status sesi               |
| Interval GPS            | 60 detik                              |
| Awal GPS                | Setelah scan keluar berhasil          |
| Penghentian GPS         | Setelah scan kembali berhasil         |
| Waktu transaksi         | Waktu server                          |
| Zona waktu tampilan     | Asia/Jakarta                          |
| Format koordinat        | Latitude dan Longitude                |
| Kedaluwarsa otomatis    | Aktif                                 |

### 8\. Integritas dan Konsistensi Data

Backend wajib menjaga agar seluruh data aktivitas saling terhubung dan tidak menghasilkan transaksi yang bertentangan.

Ketentuan:

* Satu NRP maksimal memiliki satu akun Personel dan satu akun Pasangan/Persit.

* Setiap akun memiliki identitas dan riwayat aktivitas masing-masing.

* Satu akun tidak boleh memiliki lebih dari satu sesi keluar aktif secara bersamaan.

* Setiap sesi aktivitas terhubung dengan satu pengajuan.

* Setiap sesi aktivitas memiliki maksimal satu waktu keluar dan satu waktu kembali.

* Satu sesi dapat memiliki banyak catatan koordinat GPS.

* Sesi yang sudah selesai tidak dapat diaktifkan kembali melalui scan biasa.

* Transaksi keluar dan kembali harus mengikuti urutan status sesi.

* Data GPS tidak boleh diterima lagi setelah sesi selesai.

* Riwayat transaksi tetap tersimpan untuk kebutuhan laporan dan audit.

### 9\. Arsitektur Backend dan Database

Berikut gambaran hubungan komponen utama GATEOPS.

Frontend Next.js

Login · Dashboard · Pengajuan · QR Scanner · GPS · Laporan

Backend Next.js API

Authentication · Authorization · Business Logic · Validation

PostgreSQL Database

Users · Profiles · Requests · Sessions · QR · GPS · Reports · Audit Logs

Infrastruktur VPS

Dokploy · Docker · HTTPS · Domain · Scheduler · Backup

### 10\. Kesimpulan

Backend GATEOPS dirancang sebagai pusat pengelolaan seluruh proses bisnis platform dengan PostgreSQL sebagai database utama dan Next.js API sebagai pengolah transaksi.

Sistem mengintegrasikan:

* Pengelolaan akun Personel dan Pasangan/Persit berdasarkan NRP.

* Login terpusat dengan PIN masing-masing akun.

* Pengajuan keluar dengan persetujuan otomatis.

* Batas waktu scan keluar maksimal 2 jam.

* Satu QR Code untuk transaksi keluar dan kembali.

* GPS monitoring dengan interval pengiriman 60 detik.

* Pencatatan waktu keluar dan kembali secara otomatis.

* Penghitungan durasi aktivitas di luar.

* Riwayat perjalanan dan laporan operasional.

* Pengaturan hak akses dan audit trail.

Dengan rancangan ini, backend GATEOPS memiliki dasar yang jelas untuk dilanjutkan ke tahap perancangan ERD (Entity Relationship Diagram), struktur database PostgreSQL secara detail, dan implementasi API.
