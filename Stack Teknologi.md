### Stack Teknologi GATEOPS

| No. | Komponen         | Teknologi                                    | Kegunaan                                                                                    |
| --- | ---------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 1   | Deployment       | Dokploy                                      | Deployment dan pengelolaan aplikasi pada VPS.                                               |
| 2   | Container        | Docker                                       | Menjalankan dan mengisolasi aplikasi serta layanan sistem.                                  |
| 3   | Frontend         | Next.js                                      | Membangun tampilan website responsif untuk desktop, tablet, dan smartphone.                 |
| 4   | Backend          | Next.js API                                  | Menangani autentikasi, pengajuan keluar, transaksi QR Code, GPS, dan laporan.               |
| 5   | Database         | PostgreSQL                                   | Menyimpan data pengguna, akun, pengajuan, transaksi, koordinat GPS, dan riwayat aktivitas.  |
| 6   | Peta GPS         | Leaflet + OpenStreetMap                      | Menampilkan posisi pengguna dan visualisasi riwayat perjalanan pada peta.                   |
| 7   | QR Code          | Library QR Code                              | Membaca, memproses, dan memvalidasi QR Code untuk pencatatan keluar dan kembali.            |
| 8   | HTTPS dan Domain | Domain dan konfigurasi HTTPS melalui Dokploy | Menyediakan alamat website dan mengamankan komunikasi data menggunakan koneksi terenkripsi. |

### Catatan Penerapan

* Next.js digunakan untuk frontend dan backend agar pengembangan lebih terintegrasi.

* PostgreSQL menjadi database utama seluruh platform.

* Leaflet digunakan sebagai pustaka peta, sedangkan OpenStreetMap menyediakan data peta.

* QR Code menggunakan mekanisme validasi transaksi yang terhubung dengan status sesi pengguna.

* HTTPS wajib diterapkan karena GATEOPS memerlukan akses GPS dan kamera melalui browser.

* Backup database dan pengamanan akses server disiapkan sebagai bagian dari deployment.

### Kesimpulan

Stack teknologi tersebut menjadi fondasi pengembangan GATEOPS sebagai platform website terpusat, dengan deployment melalui Dokploy pada VPS dan seluruh data operasional dikelola melalui satu database PostgreSQL.
