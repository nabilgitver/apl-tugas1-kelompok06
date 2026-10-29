# A3. Batasan Sistem

**Subsistem:** Academic Management  
**Layanan:** Magang dan Studi Independen (MSIB)  
**Kelompok:** III-6  
**Penanggung jawab:** Agim Algifahri

## 1. Batasan Teknologi

### T1. Integrasi dengan Sistem Akademik

Sistem MSIB perlu mempertimbangkan integrasi dengan sistem akademik kampus apabila tersedia akses resmi atau API yang dapat digunakan.

**Dampak terhadap arsitektur:** Arsitektur perlu menyediakan lapisan integrasi agar pertukaran data dengan sistem eksternal dapat dilakukan tanpa mengubah keseluruhan sistem. Jika integrasi belum tersedia, sistem harus tetap dapat beroperasi secara mandiri.

### T2. Akses Berbasis Web

Sistem harus dapat diakses melalui browser pada komputer maupun perangkat seluler dengan koneksi internet.

**Dampak terhadap arsitektur:** Sistem memerlukan antarmuka web yang responsif, komunikasi melalui HTTPS, serta penanganan kesalahan jaringan agar tetap mudah digunakan pada berbagai perangkat.

## 2. Batasan Bisnis

### B1. Mengikuti Ketentuan Program MSIB

Proses pendaftaran, penempatan mitra, bimbingan, logbook, dan penilaian harus mengikuti ketentuan program serta kebijakan kampus yang berlaku.

**Dampak terhadap arsitektur:** Alur kerja perlu dirancang agar tahapan dan aturan program dapat disesuaikan tanpa harus mengubah seluruh sistem.

### B2. Kewenangan Pengguna

Mahasiswa, dosen pembimbing, dan administrator memiliki kewenangan yang berbeda dalam mengakses serta mengelola data MSIB.

**Dampak terhadap arsitektur:** Sistem memerlukan kontrol akses berbasis peran (Role-Based Access Control/RBAC) agar setiap pengguna hanya dapat melakukan tindakan sesuai kewenangannya.

## 3. Batasan Operasional

### O1. Ketersediaan Pengelola Sistem

Pengelolaan akun, perubahan data program, dan penanganan kendala bergantung pada ketersediaan petugas yang berwenang.

**Dampak terhadap arsitektur:** Sistem perlu menyediakan pencatatan kesalahan, dokumentasi pengelolaan, dan mekanisme pemulihan agar kendala dapat ditangani dengan lebih mudah.

### O2. Perubahan Data oleh Banyak Pengguna

Mahasiswa, dosen pembimbing, dan administrator dapat memperbarui data pada tahapan yang berbeda.

**Dampak terhadap arsitektur:** Sistem perlu menjaga konsistensi data, menerapkan validasi saat perubahan dilakukan, serta mencatat aktivitas penting untuk memudahkan penelusuran perubahan.

## Catatan

Batasan di atas merupakan usulan awal untuk analisis arsitektur subsistem MSIB. Batasan tersebut perlu divalidasi dengan ketentuan resmi kampus dan hasil observasi atau wawancara sebelum ditetapkan sebagai fakta sistem.
