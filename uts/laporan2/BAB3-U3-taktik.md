# BAB 3: TAKTIK
## [U3] Taktik Arsitektur

Bab ini menjelaskan taktik arsitektur yang dirancang untuk mendukung ASR-1 sampai ASR-3 pada subsistem *Academic Management* SIAKAD UMI. Fokus rancangan adalah penggantian dosen pengampu menjelang kuliah perdana agar perubahan tersebar secara konsisten ke seluruh modul, dosen baru dapat dilihat mahasiswa saat aplikasi dibuka ulang, dan setiap perubahan dibatasi serta memiliki jejak audit.

Rancangan ini dibatasi pada masalah sinkronisasi data, tampilan dosen mutakhir, dan pencatatan keputusan. Sistem tidak menambahkan notifikasi langsung kepada mahasiswa; pembaruan dilakukan saat mahasiswa membuka ulang atau mengakses kembali SIAKAD.

---

### 3.1 Tujuan dan Pemetaan Taktik

| ID Skenario | ASR | Atribut Kualitas | Taktik yang Digunakan | Tujuan Utama |
|---|---|---|---|---|
| **S1** | **ASR-1** | **Modifiabilitas** (*Modifiability*) | *Publish-Subscribe*, *Event Bus*, dan *Transactional Outbox* | Menyebarkan satu perubahan dosen secara konsisten ke Modul Kelas-Dosen, Jadwal, KRS, Presensi, dan hak input Nilai dalam **maksimal 2 menit**. |
| **S2** | **ASR-2** | **Usability** | *Publish-Subscribe* untuk pembaruan tampilan, *Cache Invalidation*, validasi sumber, dan indikator sinkronisasi | Memastikan mahasiswa melihat dosen baru dalam **maksimal 1 menit** setelah membuka ulang atau mengakses kembali SIAKAD, dengan **0 tampilan dosen lama** setelah sinkron selesai. |
| **S3** | **ASR-3** | **Keamanan** (*Security*) | RBAC, pemisahan kewenangan dan persetujuan Ketua Jurusan, serta *Audit Trail* | Memastikan **100%** perubahan sah tercatat lengkap dan **0** perubahan berhasil diterapkan tanpa otorisasi serta persetujuan yang sah. |

---

### 3.2 Taktik untuk S1: Modifiabilitas

#### 3.2.1 Publish-Subscribe melalui Event Bus

**Masalah yang diatasi:** Perubahan dosen pada satu kelas perlu memperbarui Modul Kelas-Dosen, Jadwal, KRS, Presensi, dan hak input Nilai. Pembaruan langsung satu per satu berisiko lambat, berbeda antar-modul, dan dapat meninggalkan dosen lama pada modul tertentu.

**Cara kerja:**
1. Admin Akademik mengajukan pergantian dosen melalui Modul Kelas-Dosen.
2. Sistem memvalidasi permintaan terhadap identitas, peran, dan persetujuan Ketua Jurusan.
3. Jika valid, perubahan dosen disimpan satu kali pada sumber data utama sebagai satu transaksi perubahan.
4. Sistem menerbitkan event `DosenDiganti` melalui *event bus*. Event membawa ID kelas, dosen lama, dosen baru, waktu perubahan, dan ID transaksi.
5. Modul Jadwal, KRS, Presensi, dan Nilai berlangganan event tersebut, lalu memperbarui data atau hak akses menjadi tanggung jawab masing-masing.
6. Setiap modul mengirimkan status berhasil atau gagal beserta waktu pemrosesan.

**Taktik pendukung:**
- **Transactional Outbox:** Penyimpanan perubahan dan pencatatan event ke tabel *outbox* dilakukan dalam satu transaksi database. Dengan demikian, perubahan tidak tersimpan tanpa event yang dicatat untuk diterbitkan.
- **Pemrosesan idempoten:** Setiap event memiliki ID unik dan setiap modul memeriksa ID tersebut agar pengiriman ulang tidak mengubah data atau hak akses lebih dari sekali.
- **Retry dengan jeda bertahap:** Modul atau penerbit mencoba kembali event yang gagal. Event yang terus gagal dipindahkan ke antrean kegagalan dan menghasilkan peringatan operasional.
- **Pemantauan sinkronisasi:** Sistem mencatat waktu penerbitan dan waktu selesai pada setiap modul. Peringatan diberikan jika sinkronisasi mendekati atau melewati target 2 menit.

**Ukuran keberhasilan:**
- Semua modul terkait memproses perubahan dalam waktu **maksimal 2 menit**.
- Setelah sinkronisasi selesai, data dosen pada Modul Kelas-Dosen, Jadwal, KRS, Presensi, dan hak input Nilai tidak berbeda.
- Status pemrosesan dapat diperiksa untuk setiap modul berdasarkan ID transaksi.

**Konsekuensi:**
- Data dapat mengalami *eventual consistency* sementara event sedang diproses.
- Arsitektur memerlukan pengelolaan event, mekanisme retry, pemantauan, dan prosedur pemulihan.
- Tabel *outbox* dan log pemrosesan menambah kebutuhan penyimpanan.

---

### 3.3 Taktik untuk S2: Usability

#### 3.3.1 Penyajian Data Terbaru dan Pembaruan Cache

**Masalah yang diatasi:** Mahasiswa tidak menerima notifikasi langsung dan dapat melihat dosen lama apabila tampilan masih menggunakan cache lama atau belum menerima hasil sinkronisasi.

**Cara kerja:**
1. Setelah dosen diganti, event `DosenDiganti` digunakan untuk memperbarui atau membatalkan cache informasi pada tampilan Jadwal dan Informasi Kelas.
2. Saat mahasiswa membuka ulang atau mengakses kembali halaman Jadwal atau Informasi Kelas, aplikasi meminta data terbaru dari sumber resmi.
3. Aplikasi membandingkan versi atau waktu pembaruan data cache dengan sumber resmi sebelum menampilkan hasil.
4. Respons menampilkan dosen terbaru beserta waktu pembaruan terakhir.
5. Jika sinkronisasi masih berjalan, aplikasi menampilkan status pembaruan dan tidak menyatakan bahwa data lama telah diperbarui.

**Taktik pendukung:**
- Gunakan masa berlaku cache terbatas untuk informasi dosen dan jadwal.
- Lakukan *cache invalidation* berdasarkan event perubahan atau versi data.
- Gunakan validasi sumber resmi saat halaman dibuka ulang atau dimuat kembali.
- Tampilkan indikator waktu pembaruan agar mahasiswa dapat mengenali kesegaran data.

**Ukuran keberhasilan:**
- Dosen baru terlihat dalam waktu **maksimal 1 menit** setelah mahasiswa membuka ulang atau mengakses kembali SIAKAD, dengan asumsi sinkronisasi S1 telah selesai.
- Setelah sinkronisasi selesai, **0 tampilan dosen lama** pada Jadwal dan Informasi Kelas.
- Jika sinkronisasi belum selesai, aplikasi menampilkan status yang informatif dan bukan dosen lama seolah-olah sebagai data terbaru.

**Konsekuensi:**
- Validasi cache terhadap sumber resmi dapat menambah beban layanan dan waktu respons.
- Masa berlaku cache harus menyeimbangkan kecepatan akses dan kesegaran data.
- Antarmuka perlu menangani sinkronisasi yang masih berlangsung atau layanan yang tidak tersedia.

---

### 3.4 Taktik untuk S3: Keamanan

#### 3.4.1 Otorisasi RBAC dan Persetujuan Ketua Jurusan

**Masalah yang diatasi:** Pengguna tanpa hak tidak boleh mengubah dosen pengampu. Perubahan juga harus memperoleh persetujuan Ketua Jurusan sesuai proses akademik.

**Cara kerja:**
1. Sistem mengautentikasi identitas pengguna sebelum menerima permintaan perubahan.
2. Sistem menerapkan RBAC untuk memastikan hanya Admin Akademik yang berwenang dapat mengajukan perubahan.
3. Permintaan berstatus menunggu persetujuan sampai Ketua Jurusan berwenang menyetujuinya.
4. Sistem memisahkan hak mengajukan dan hak menyetujui sehingga pemohon tidak dapat menyetujui permintaan miliknya sendiri.
5. Saat perubahan akan diterapkan, sistem memeriksa identitas, peran, status persetujuan, dan validitas permintaan. Permintaan yang tidak memenuhi pemeriksaan ditolak.

**Taktik pendukung:**
- Terapkan prinsip *least privilege* untuk setiap peran.
- Gunakan kebijakan *default-deny* apabila identitas, hak akses, atau persetujuan tidak valid.
- Catat upaya perubahan yang ditolak beserta identitas aktor, waktu, dan alasan penolakan apabila tersedia.
- Perbarui konfigurasi peran dan kewenangan jika struktur jabatan berubah.

#### 3.4.2 Audit Trail

**Cara kerja:** Setiap perubahan sah dicatat pada tabel `LogPerubahan` sebagai *audit trail*. Catatan berisi sekurang-kurangnya:
- identitas aktor yang mengajukan perubahan;
- identitas Ketua Jurusan yang memberikan persetujuan;
- waktu pengajuan, persetujuan, dan penerapan;
- identitas kelas atau mata kuliah;
- dosen lama dan dosen baru;
- hasil tindakan serta alasan penolakan bagi upaya yang ditolak.

Catatan dibuat secara *append-only* sehingga tidak dapat diubah melalui fungsi aplikasi biasa. Akses membaca dan mengelola audit dibatasi pada peran yang berwenang.

**Ukuran keberhasilan:**
- **100%** perubahan yang berhasil diterapkan tercatat lengkap pada *Audit Trail*.
- **0** perubahan berhasil diterapkan tanpa otorisasi RBAC dan persetujuan yang sah.
- Upaya perubahan tanpa wewenang ditolak dan dapat ditelusuri melalui log keamanan.

**Konsekuensi:**
- Persetujuan membuat alur perubahan menjadi lebih panjang.
- Pencatatan audit menambah beban tulis dan kebutuhan penyimpanan.
- Diperlukan kebijakan retensi dan pengelolaan akses terhadap catatan audit.

---

### 3.5 Alur Taktik Terpadu

1. Admin Akademik mengajukan perubahan dosen pada Modul Kelas-Dosen.
2. Sistem memeriksa RBAC dan memastikan persetujuan Ketua Jurusan telah diberikan. Jika gagal, permintaan ditolak, dicatat, dan event tidak diterbitkan.
3. Jika valid, sistem menyimpan perubahan dosen, mencatat `LogPerubahan`, dan mencatat event `DosenDiganti` dalam tabel *outbox* dalam satu transaksi.
4. *Event bus* menyebarkan event kepada Modul Jadwal, KRS, Presensi, dan Nilai. Modul penerima memproses event secara idempoten serta mengirimkan status pemrosesan.
5. Sistem memantau sinkronisasi hingga seluruh modul selesai dengan target **maksimal 2 menit**.
6. Mahasiswa yang membuka ulang atau mengakses kembali SIAKAD mendapatkan dosen baru melalui cache yang telah diperbarui atau divalidasi dari sumber resmi, dengan target tampilan **maksimal 1 menit** setelah sinkronisasi selesai.

Dengan kombinasi taktik tersebut, satu perubahan dosen dapat disebarkan secara terkontrol, ditampilkan secara mutakhir kepada mahasiswa, dan dibatasi serta dicatat sesuai kebutuhan keamanan. Rancangan ini menjawab masalah sinkronisasi data, tampilan dosen lama, dan tidak adanya jejak keputusan pada pergantian dosen menjelang kuliah perdana.
