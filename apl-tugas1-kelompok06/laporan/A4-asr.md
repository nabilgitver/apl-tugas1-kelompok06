# A4. Kebutuhan Signifikan (ASR)

**Subsistem:** Academic Management  
**Layanan:** Magang dan Studi Independen (MSIB)  
**Kelompok:** III-6  
**Penanggung jawab:** Agim Algifahri

Architecturally Significant Requirements (ASR) adalah kebutuhan yang berpengaruh besar terhadap keputusan dan struktur arsitektur perangkat lunak.

## ASR-1. Keamanan dan Privasi Data

**Jenis:** Atribut kualitas — Security

**Kebutuhan:** Sistem harus membatasi akses terhadap data pendaftaran, dokumen, logbook, bimbingan, dan penilaian sesuai peran serta kewenangan pengguna.

**Alasan signifikan secara arsitektural:** Data MSIB bersifat spesifik bagi setiap mahasiswa dan melibatkan beberapa pihak dengan hak akses berbeda. Karena itu, arsitektur harus mempertimbangkan autentikasi, otorisasi, kebijakan akses database, dan perlindungan data sejak awal.

## ASR-2. Konsistensi dan Ketertelusuran Data

**Jenis:** Atribut kualitas — Reliability dan Auditability

**Kebutuhan:** Perubahan data penting, seperti status pendaftaran, penempatan mitra, logbook, dan penilaian, harus divalidasi dan dapat ditelusuri berdasarkan pengguna serta waktu perubahan.

**Alasan signifikan secara arsitektural:** Kesalahan atau ketidaksesuaian data dapat memengaruhi proses magang dan hasil penilaian mahasiswa. Oleh sebab itu, arsitektur perlu mendukung validasi data, transaksi yang konsisten, dan pencatatan aktivitas penting.

## ASR-3. Kemudahan Akses dan Umpan Balik Sistem

**Jenis:** Atribut kualitas — Usability dan Reliability

**Kebutuhan:** Sistem harus mudah digunakan melalui komputer maupun perangkat seluler serta memberikan informasi yang jelas ketika pengguna menyimpan data, mengunggah dokumen, atau mengalami kegagalan koneksi.

**Alasan signifikan secara arsitektural:** Mahasiswa dan dosen dapat mengakses sistem dari perangkat serta kondisi jaringan yang berbeda. Hal ini memengaruhi rancangan antarmuka responsif, mekanisme penanganan kesalahan, validasi formulir, dan pemberian umpan balik kepada pengguna.

## Kesimpulan

Ketiga ASR tersebut menjadi pertimbangan utama dalam perancangan arsitektur layanan MSIB. Keamanan melindungi data pengguna, konsistensi menjaga keandalan proses akademik, sedangkan kemudahan akses membantu pengguna menyelesaikan aktivitas MSIB secara efektif.
