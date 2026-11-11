# BAB 3: POLA ARSITEKTUR
## [U4] Pola Arsitektur

Pada bab ini dijelaskan pola arsitektur yang digunakan untuk mendukung skenario atribut kualitas pada subsistem *Academic Management* SIAKAD UMI. Fokus utama pola ini adalah menangani masalah penggantian dosen pengampu yang tidak konsisten, tidak terdokumentasi dengan baik, dan tidak disampaikan ke mahasiswa secara tepat waktu. Dengan menerapkan pola yang sesuai, proses perubahan dapat dilakukan secara terkontrol, terdokumentasi, dan mudah dikembangkan di masa mendatang.

---

### 3.1 Pola Arsitektur yang Dipilih

Berdasarkan kebutuhan sistem, dipilih tiga pola arsitektur utama yang saling melengkapi:

| Pola | Deskripsi | Kebutuhan yang Didukung | Dampak utama |
|---|---|---|---|
| **Layered Architecture** | Memisahkan antarmuka, logika aplikasi, dan penyimpanan data ke dalam lapisan yang jelas. | Keteraturan pengelolaan data dan kemudahan pemeliharaan. | Modul menjadi lebih mudah dikembangkan dan diuji. |
| **Event-Driven Architecture** | Perubahan data dikirimkan melalui event agar modul terkait menerima update secara otomatis. | Sinkronisasi data antar modul dan kebutuhan *modifiability*. | Terjadi *eventual consistency* sementara proses sinkronisasi berlangsung. |
| **RBAC + Audit Trail** | Hak akses dibatasi per peran dan semua perubahan dicatat dalam log audit yang aman. | Keamanan, akuntabilitas, dan audit perubahan. | Proses perubahan menjadi lebih formal dan sedikit lebih lama. |

*Table 4. Pola arsitektur yang digunakan pada subsistem Academic Management.*

---

### 3.2 Pola 1: Layered Architecture

#### 3.2.1 Struktur lapisan

Sistem dibagi menjadi beberapa lapisan utama, yaitu:

1. **Lapisan presentasi (Presentation Layer)**
   - Menyediakan antarmuka web untuk admin akademik, dosen, dan mahasiswa.
   - Menampilkan jadwal, informasi kelas, presensi, nilai, dan status perubahan dosen.

2. **Lapisan aplikasi (Application Layer)**
   - Menangani logika bisnis seperti pengajuan perubahan dosen, validasi data, persetujuan, dan pengiriman event.
   - Menghubungkan kebutuhan pengguna dengan layanan backend.

3. **Lapisan domain / bisnis (Domain Layer)**
   - Memuat aturan inti perusahaan akademik, seperti validasi kelas, relasi dosen dengan mata kuliah, serta kebijakan perubahan dosen.
   - Menjadi tempat aturan yang paling penting bagi konsistensi data akademik.

4. **Lapisan data (Data Layer)**
   - Menyimpan data kelas, jadwal, KRS, presensi, nilai, log perubahan, dan catatan outbox event.
   - Menyediakan akses data yang aman dan konsisten untuk seluruh modul.

#### 3.2.2 Keunggulan pola ini

- **Pemisahan tanggung jawab:** setiap lapisan punya tugas yang jelas dan tidak bercampur.
- **Kemudahan pemeliharaan:** perubahan pada satu lapisan dapat dilakukan dengan risiko yang lebih kecil terhadap lapisan lain.
- **Skalabilitas pengembangan:** modul seperti Jadwal, Presensi, KRS, dan Nilai dapat dikembangkan secara terpisah tanpa mengganggu fungsi lain.
- **Konsistensi kepatuhan aturan bisnis:** semua perubahan dosen melewati satu alur logika yang sama.

#### 3.2.3 Keterkaitan dengan masalah utama

Masalah utama dalam kasus ini adalah perubahan dosen pengampu tidak tersebar dengan benar dan mahasiswa tidak diberi tahu. Dengan layered architecture, proses penggantian dosen diproses melalui satu titik logika utama, sehingga koreksi tidak perlu dilakukan di setiap modul secara manual. Hal ini menurunkan risiko inkonsistensi data dan memudahkan pengawasan alur perubahan.

#### 3.2.4 Konsekuensi

- Arsitektur ini membutuhkan desain antarmuka antar-lapisan yang jelas.
- Komunikasi antar lapisan dapat menambah overhead jika tidak dirancang dengan baik.
- Jika modul baru ditambahkan, struktur lapisan tetap harus dipelihara agar tidak menimbulkan “spaghetti architecture”.

---

### 3.3 Pola 2: Event-Driven Architecture

#### 3.3.1 Tujuan pola

Sistem akademik memiliki banyak modul yang bergantung pada informasi dosen pengampu, seperti:
- Jadwal
- KRS
- Presensi
- Nilai
- Informasi kelas mahasiswa

Ketika dosen pengampu berubah, perubahan harus disebarkan ke seluruh modul terkait secara otomatis. Karena modul-modul ini berbeda dan tidak selalu terikat dengan kode yang sama, pola event-driven sangat sesuai.

#### 3.3.2 Cara kerja pola

1. Admin Akademik melakukan perubahan dosen pada modul *Kelas-Dosen*.
2. Sistem menyimpan perubahan pada sumber data utama dan menuliskan event `DosenDiganti` ke tabel *outbox* dalam satu transaksi.
3. Event kemudian dikirim melalui *event bus* atau broker event.
4. Modul seperti Jadwal, KRS, Presensi, dan Nilai berlangganan event tersebut.
5. Setiap modul memproses event dan memperbarui data yang relevan.
6. Jika terjadi kegagalan pada salah satu modul, sistem melakukan retry dan mencatat status proses.

#### 3.3.3 Taktik pendukung

Pola ini didukung oleh beberapa mekanisme tambahan:
- **Transactional Outbox**: mencegah event dan data mutasi kehilangan keterkaitan.
- **Idempotency**: event yang diterima berkali-kali tidak menghasilkan perubahan ganda.
- **Retry mechanism**: event gagal diproses akan dicoba lagi secara berkala.
- **Monitoring status**: seluruh modul dipantau agar sinkronisasi selesai dalam target waktu.

#### 3.3.4 Hubungan dengan skenario kualitas

Pola event-driven berkontribusi pada:
- **S1 (Modifiabilitas)**: perubahan dapat disebarkan ke banyak modul tanpa menulis ulang logika di semua bagian.
- **S2 (Usability)**: mahasiswa bisa melihat data terbaru ketika membuka ulang aplikasi setelah informasi kelas diperbarui.

#### 3.3.5 Konsekuensi

- Terdapat keterlambatan sementara (*eventual consistency*) sebelum semua modul selesai diperbarui.
- Perlu pengelolaan log, retry, dan monitoring untuk memastikan tidak ada event yang tertinggal.
- Struktur arsitektur menjadi lebih kompleks dibandingkan pendekatan sinkron langsung.

---

### 3.4 Pola 3: Role-Based Access Control (RBAC) dan Audit Trail

#### 3.4.1 Tujuan pola

Perubahan dosen pengampu merupakan operasi kritis yang memengaruhi proses akademik, hak akses, serta reputasi pengajaran. Oleh karena itu, perubahan hanya boleh dilakukan oleh pihak yang berwenang dan harus dicatat secara permanen.

#### 3.4.2 Komponen utama

Pola keamanan ini terdiri dari beberapa bagian:

1. **RBAC (Role-Based Access Control)**
   - Admin Akademik berhak mengajukan perubahan dosen.
   - Ketua Jurusan berhak menyetujui perubahan.
   - Dosen lama dan dosen baru tidak dapat mengubah data secara sepihak.

2. **Persetujuan ganda / workflow**
   - Setiap perubahan dimulai dari pengajuan.
   - Setelah validasi, perubahan harus disetujui sebelum diterapkan.

3. **Audit trail / log perubahan**
   - Mencatat siapa yang mengubah, kapan, apa yang berubah, dan alasan perubahan.
   - Log dibuat secara *append-only* agar tidak mudah diubah oleh pengguna biasa.

#### 3.4.3 Alur keamanan

1. Pengguna masuk ke sistem dan autentikasi dilakukan.
2. Sistem memeriksa peran pengguna.
3. Hanya pengguna dengan role yang sah yang dapat mengajukan perubahan.
4. Permintaan dikirim ke proses persetujuan.
5. Setelah disetujui, data yang berubah dicatat pada audit trail.
6. Semua status pengajuan dan hasil keputusan disimpan dalam log keamanan.

#### 3.4.4 Keterkaitan dengan skenario kualitas

Pola ini berkontribusi pada:
- **S3 (Keamanan)**: mencegah perubahan ilegal dan menyediakan jejak audit yang jelas.
- **Kepatuhan terhadap aturan akademik**: perubahan dosen tidak dilakukan secara sewenang-wenang.

#### 3.4.5 Konsekuensi

- Proses perubahan menjadi lebih formal dan membutuhkan waktu lebih lama.
- Perlunya penyimpanan tambahan untuk log audit.
- Admin sistem harus menjaga kebijakan peran agar tetap sesuai dengan struktur organisasi.

---

### 3.5 Integrasi Pola Arsitektur

Ketiga pola di atas tidak berdiri sendiri, tetapi saling melengkapi dalam satu ekosistem arsitektur sistem. Integrasi utamanya adalah sebagai berikut:

1. **Lapisan arsitektur** membagi tanggung jawab sistem agar logika bisnis dan data tidak bercampur.
2. **Event-driven architecture** memastikan bahwa perubahan dosen diteruskan ke seluruh modul yang terdampak sesuai kebutuhan.
3. **RBAC dan audit trail** melindungi integritas proses perubahan agar tidak terjadi perubahan ilegal atau tidak betanggung jawab.

Secara ringkas, alur perubahan dosen pada sistem menjadi:

1. Admin akademik mengajukan perubahan.
2. Sistem mengecek otorisasi dan persetujuan.
3. Jika valid, perubahan disimpan di sumber data utama.
4. Event `DosenDiganti` diterbitkan ke event bus.
5. Modul terkait memperbarui data masing-masing.
6. Log perubahan dibuat dan dipertahankan untuk audit.
7. Mahasiswa melihat data terbaru setelah aplikasi dibuka ulang atau data berhasil disinkronkan.

---

### 3.6 Kesimpulan

Pola arsitektur yang dipilih untuk subsistem *Academic Management* SIAKAD UMI adalah kombinasi dari:
- **Layered Architecture** untuk struktur dan pemeliharaan,
- **Event-Driven Architecture** untuk sinkronisasi data antar modul,
- **RBAC + Audit Trail** untuk keamanan dan akuntabilitas.

Kombinasi ini sesuai dengan kebutuhan utama sistem, yaitu menjaga konsistensi data dosen pengampu, mempercepat penyebaran perubahan, memberi informasi kepada mahasiswa, dan menjaga bahwa setiap perubahan dapat dipertanggungjawabkan secara administratif. Dengan pola seperti ini, sistem dapat berkembang lebih terstruktur, aman, dan mudah diadaptasi terhadap perubahan di masa depan.

---

*Bab ini menutup pembahasan pola arsitektur yang menjadi fondasi implementasi skenario atribut kualitas pada subsistem Academic Management SIAKAD UMI.*
