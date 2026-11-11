# BAB 3: TAKTIK
## [U3] Taktik Arsitektur

Bab ini menjelaskan taktik arsitektur untuk mendukung skenario atribut kualitas pada subsistem *Academic Management* SIAKAD UMI, khususnya proses penggantian dosen pengampu menjelang kuliah perdana. Taktik dipilih untuk mencapai target modifiabilitas, usability, dan keamanan yang telah ditetapkan pada Bab 2.

---

### 3.1 Tujuan dan Pemetaan Taktik

| ID Skenario | Atribut Kualitas | Taktik yang Digunakan | Tujuan Utama |
|---|---|---|---|
| **S1** | Modifiabilitas | *Publish-Subscribe*, *Event Bus*, dan *Transactional Outbox* | Menyebarkan perubahan dosen secara terkontrol ke modul terkait, dengan waktu sinkronisasi maksimal 2 menit. |
| **S2** | Usability | *Cache Invalidation*, *Read-Your-Writes/Latest Data*, dan indikator waktu pembaruan | Memastikan mahasiswa melihat informasi dosen terbaru setelah membuka ulang aplikasi. |
| **S3** | Keamanan | *Role-Based Access Control* (RBAC), persetujuan Ketua Jurusan, dan *Audit Trail* | Mencegah perubahan tanpa wewenang dan menyediakan catatan lengkap untuk setiap perubahan sah. |

---

### 3.2 Taktik untuk S1: Modifiabilitas

#### 3.2.1 Publish-Subscribe melalui Event Bus

**Masalah yang diatasi:** Perubahan dosen pengampu perlu diperbarui pada beberapa modul, seperti Jadwal, KRS, Presensi, dan Nilai. Pembaruan langsung satu per satu berisiko membuat modul memiliki data yang berbeda.

**Cara kerja:**
1. Admin Akademik mengajukan perubahan dosen melalui modul Kelas-Dosen.
2. Sistem memvalidasi perubahan dan menyimpannya pada sumber data utama.
3. Setelah perubahan berhasil disimpan, sistem menerbitkan event `DosenDiganti` melalui *event bus*. Event membawa identitas kelas, dosen lama dan baru, waktu perubahan, serta identitas transaksi.
4. Modul Jadwal, KRS, Presensi, dan Nilai berlangganan event tersebut dan memperbarui data yang menjadi tanggung jawab masing-masing.
5. Setiap modul mengakui pemrosesan event. Jika pemrosesan gagal, event dicoba kembali dan kegagalan dicatat untuk ditindaklanjuti.

**Taktik pendukung:**
- **Transactional Outbox:** Perubahan pada data Kelas-Dosen dan pencatatan event ke tabel *outbox* dilakukan dalam satu transaksi database. Dengan demikian, perubahan tidak tersimpan tanpa event yang dapat disebarkan.
- **Pemrosesan idempoten:** Setiap event memiliki ID unik. Modul penerima memeriksa ID tersebut agar pengiriman ulang tidak menyebabkan perubahan ganda.
- **Retry dan penanganan kegagalan:** Event yang gagal diproses dicoba kembali dengan jeda bertahap. Event yang tetap gagal ditempatkan pada antrean kegagalan (*dead-letter queue*) dan menghasilkan notifikasi untuk administrator.
- **Pemantauan sinkronisasi:** Sistem mencatat waktu penerbitan dan waktu pemrosesan event. Peringatan diberikan jika event belum diproses atau sinkronisasi mendekati batas 2 menit.

**Ukuran keberhasilan:**
- Perubahan telah diproses oleh seluruh modul terkait dalam waktu **maksimal 2 menit**.
- Status pemrosesan dapat diperiksa untuk setiap modul.
- Setelah seluruh modul menyatakan berhasil, tidak terdapat perbedaan data dosen pengampu.

**Konsekuensi:**
- Data antar-modul dapat mengalami *eventual consistency* selama event masih diproses.
- Arsitektur memerlukan pengelolaan event, mekanisme retry, pemantauan, dan prosedur pemulihan.
- Tabel *outbox* dan log pemrosesan menambah kebutuhan penyimpanan.

---

### 3.3 Taktik untuk S2: Usability

#### 3.3.1 Menampilkan Data Terbaru dan Memperbarui Cache

**Masalah yang diatasi:** Mahasiswa tidak menerima notifikasi langsung dan dapat melihat informasi dosen lama apabila aplikasi menampilkan data yang tersimpan di cache atau belum menerima hasil sinkronisasi.

**Cara kerja:**
1. Ketika mahasiswa membuka ulang halaman Jadwal atau Informasi Kelas, aplikasi meminta data terbaru dari layanan yang menjadi sumber informasi resmi.
2. Cache untuk informasi kelas yang berubah diperbarui atau dibatalkan (*invalidate*) setelah event `DosenDiganti` berhasil diproses.
3. Respons layanan mencantumkan waktu pembaruan data sehingga aplikasi dapat memastikan informasi yang ditampilkan tidak lebih lama dari data pada sumber resmi.
4. Jika sinkronisasi masih berlangsung, aplikasi menampilkan status pembaruan yang jelas dan melakukan pemuatan ulang data secara berkala, bukan menyatakan bahwa perubahan telah selesai.

**Taktik pendukung:**
- Gunakan masa berlaku cache yang terbatas untuk informasi dosen dan jadwal, serta lakukan *cache invalidation* berdasarkan event perubahan.
- Pada saat halaman dibuka ulang, lakukan validasi versi atau waktu perubahan data terhadap sumber resmi sebelum menampilkan hasil cache.
- Tampilkan informasi dosen yang diperbarui beserta waktu pembaruan agar mahasiswa dapat mengenali data terbaru.

**Ukuran keberhasilan:**
- Informasi dosen terbaru tampil dalam waktu **maksimal 1 menit** setelah mahasiswa membuka ulang aplikasi, dengan asumsi perubahan telah tersinkronisasi.
- Tidak ada tampilan dosen lama setelah sinkronisasi selesai dan halaman memuat data terbaru.
- Jika data belum tersinkronisasi, aplikasi menampilkan status yang informatif dan tidak menyamarkan data lama sebagai data terbaru.

**Konsekuensi:**
- Permintaan validasi data dapat menambah beban layanan dan waktu respons.
- Pengaturan cache perlu memperhatikan keseimbangan antara kecepatan akses dan kesegaran data.
- Antarmuka perlu menangani keadaan ketika sinkronisasi masih berjalan atau layanan sedang tidak tersedia.

---

### 3.4 Taktik untuk S3: Keamanan

#### 3.4.1 Otorisasi RBAC dan Persetujuan Ketua Jurusan

**Masalah yang diatasi:** Pengguna yang tidak berwenang tidak boleh mengubah dosen pengampu. Perubahan juga harus memperoleh persetujuan Ketua Jurusan sesuai aturan proses akademik.

**Cara kerja:**
1. Sistem mengautentikasi identitas pengguna sebelum menerima permintaan perubahan.
2. Sistem menerapkan RBAC untuk memastikan hanya Admin Akademik dengan hak yang sesuai yang dapat mengajukan perubahan.
3. Perubahan berstatus menunggu persetujuan sampai Ketua Jurusan yang berwenang menyetujui permintaan tersebut.
4. Sistem memeriksa identitas, peran, status persetujuan, dan validitas permintaan kembali saat perubahan akan diterapkan. Permintaan yang tidak memenuhi seluruh pemeriksaan ditolak.
5. Setelah disetujui dan diterapkan, sistem menyimpan perubahan dan catatan audit secara konsisten.

**Taktik pendukung:**
- Terapkan prinsip *least privilege*: setiap peran hanya memperoleh hak minimum yang dibutuhkan.
- Pisahkan hak mengajukan perubahan dan hak menyetujui perubahan agar persetujuan tidak dilakukan oleh pemohon yang sama.
- Tolak permintaan secara *default-deny* apabila identitas, hak akses, atau persetujuan tidak valid.
- Catat upaya perubahan yang ditolak untuk membantu pemantauan keamanan.

#### 3.4.2 Audit Trail

**Cara kerja:** Setiap perubahan sah dicatat pada *Audit Trail* dengan sekurang-kurangnya informasi berikut:
- identitas aktor yang mengajukan perubahan dan identitas pemberi persetujuan;
- waktu pengajuan, persetujuan, dan penerapan perubahan;
- identitas kelas atau mata kuliah;
- nilai dosen pengampu sebelum dan sesudah perubahan;
- hasil tindakan, termasuk perubahan yang ditolak, serta alasan penolakan bila tersedia.

Catatan audit dibuat secara *append-only* sehingga riwayat tidak dapat diubah melalui fungsi aplikasi biasa. Akses untuk membaca maupun mengelola catatan audit dibatasi kepada peran yang berwenang.

**Ukuran keberhasilan:**
- **100%** perubahan yang berhasil diterapkan memiliki catatan audit yang lengkap.
- **0** perubahan berhasil diterapkan tanpa otorisasi RBAC dan persetujuan yang sah.
- Upaya perubahan tanpa wewenang ditolak dan dapat ditelusuri melalui log keamanan.

**Konsekuensi:**
- Alur perubahan menjadi lebih panjang karena memerlukan persetujuan.
- Pencatatan audit menambah penggunaan penyimpanan dan memerlukan kebijakan retensi.
- Pengelolaan peran dan proses persetujuan harus diperbarui ketika struktur kewenangan berubah.

---

### 3.5 Alur Taktik Terpadu

1. Admin Akademik mengajukan perubahan dosen pengampu pada modul Kelas-Dosen.
2. Sistem memeriksa hak akses RBAC dan memastikan persetujuan Ketua Jurusan telah diberikan. Jika salah satu pemeriksaan gagal, permintaan ditolak dan upaya tersebut dicatat.
3. Jika valid, sistem menyimpan perubahan beserta catatan audit dan event pada *Transactional Outbox* dalam transaksi yang konsisten.
4. *Event bus* menyebarkan event `DosenDiganti` kepada modul Jadwal, KRS, Presensi, dan Nilai. Modul memproses event secara idempoten dan mengirimkan status pemrosesan.
5. Sistem memantau proses hingga seluruh modul selesai, dengan target sinkronisasi maksimal 2 menit. Kegagalan ditangani melalui retry dan peringatan operasional.
6. Setelah mahasiswa membuka ulang aplikasi, cache informasi kelas divalidasi atau dibatalkan dan aplikasi menampilkan data terbaru, dengan target maksimal 1 menit.

Dengan kombinasi taktik tersebut, perubahan dosen pengampu dapat disebarkan secara terukur, ditampilkan dengan jelas kepada mahasiswa, dan dibatasi serta dicatat sesuai kebutuhan keamanan.
