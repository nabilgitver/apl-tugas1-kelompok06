# BAB 3: POLA ARSITEKTUR
## [U4] Pola Arsitektur

Pada bab ini dijelaskan pola arsitektur yang digunakan untuk mendukung ASR-1 sampai ASR-3 pada subsistem *Academic Management* SIAKAD UMI. Fokus rancangan adalah penggantian dosen pengampu menjelang kuliah perdana agar data tersebar secara konsisten, dosen baru terlihat saat mahasiswa membuka ulang SIAKAD, dan seluruh perubahan terlindungi serta tercatat.

Rancangan dibatasi pada sinkronisasi data, tampilan dosen mutakhir, serta otorisasi dan pencatatan perubahan. Sistem tidak menyediakan notifikasi langsung kepada mahasiswa; mahasiswa memperoleh pembaruan ketika membuka ulang atau mengakses kembali SIAKAD.

---

### 3.1 Pola Arsitektur yang Dipilih

Berdasarkan kebutuhan ASR-1 sampai ASR-3, dipilih tiga pola arsitektur utama yang saling melengkapi:

| Pola | Deskripsi | Kebutuhan yang Didukung | Dampak Utama |
|---|---|---|---|
| **Layered Architecture** | Memisahkan antarmuka, logika aplikasi, domain akademik, dan penyimpanan data ke dalam lapisan yang jelas. | Struktur pemeliharaan, konsistensi aturan, dan integrasi modul. | Keteraturan meningkat, tetapi komunikasi antar-lapisan dapat menambah overhead. |
| **Event-Driven Architecture** | Perubahan dosen diterbitkan sebagai event dan diterima oleh modul terkait melalui event bus. | Sinkronisasi **S1** dalam maksimal 2 menit serta pembaruan tampilan **S2** dalam maksimal 1 menit setelah sinkron selesai. | Terjadi *eventual consistency* sementara, sehingga memerlukan outbox, retry, idempotensi, dan pemantauan. |
| **RBAC + Audit Trail** | Hak akses dibatasi berdasarkan peran, persetujuan dipisahkan dari pengajuan, dan perubahan dicatat secara append-only. | Keamanan **S3**: 100% perubahan tercatat dan 0 perubahan berhasil tanpa otorisasi serta persetujuan sah. | Proses menjadi lebih formal dan kebutuhan penyimpanan serta beban administrasi bertambah. |

*Tabel 4. Pola arsitektur pada subsistem Academic Management SIAKAD UMI.*

---

### 3.2 Pola 1: Layered Architecture

#### 3.2.1 Struktur lapisan

Sistem dibagi menjadi empat lapisan utama:

1. **Lapisan presentasi (Presentation Layer)**
   - Menyediakan antarmuka web untuk Admin Akademik, Ketua Jurusan, dosen, dan mahasiswa.
   - Menampilkan pengajuan perubahan, persetujuan, jadwal, informasi kelas, presensi, nilai, serta status sinkronisasi.

2. **Lapisan aplikasi (Application Layer)**
   - Menangani pengajuan perubahan dosen, autentikasi, otorisasi, persetujuan, penerbitan event, dan pemantauan status.
   - Menghubungkan kebutuhan pengguna dengan layanan domain dan penyimpanan data.

3. **Lapisan domain atau bisnis (Domain Layer)**
   - Memuat aturan kelas, relasi dosen dan mata kuliah, kewenangan peran, persetujuan Ketua Jurusan, dan aturan perubahan dosen.
   - Menjadi pusat aturan yang menjamin satu alur perubahan digunakan secara konsisten oleh seluruh modul.

4. **Lapisan data (Data Layer)**
   - Menyimpan data kelas, jadwal, KRS, presensi, nilai, `LogPerubahan`, status persetujuan, dan tabel *outbox*.
   - Menyediakan akses data yang aman dan konsisten untuk lapisan aplikasi.

#### 3.2.2 Keunggulan pola

- Pemisahan tanggung jawab membuat perubahan pada satu lapisan tidak harus mengubah seluruh modul.
- Aturan penggantian dosen dapat dipelihara secara terpusat pada lapisan domain.
- Modul Jadwal, KRS, Presensi, dan Nilai dapat dikembangkan secara terpisah tanpa mengubah aturan inti.
- Tim pengelola dapat menelusuri kegagalan berdasarkan lapisan dan tanggung jawabnya.

#### 3.2.3 Keterkaitan dengan masalah utama

Dengan *Layered Architecture*, penggantian dosen diproses melalui lapisan aplikasi dan domain yang sama, bukan melalui pembaruan langsung pada setiap modul. Setelah perubahan disetujui dan disimpan, lapisan aplikasi menerbitkan event untuk memperbarui modul akademik yang terdampak.

Pemisahan ini mendukung modifiabilitas karena satu perubahan dosen tidak perlu diimplementasikan berulang pada setiap modul. Lapisan presentasi juga dapat menampilkan status sinkronisasi atau data yang belum siap tanpa menyatakan bahwa dosen lama adalah data terbaru.

#### 3.2.4 Konsekuensi

- Dibutuhkan kontrak antarmuka yang jelas antara lapisan presentasi, aplikasi, domain, dan data.
- Komunikasi berlapis dapat menambah *overhead* bila tidak dirancang efisien.
- Perubahan struktur lapisan atau penambahan modul baru tetap harus mengikuti batas tanggung jawab agar tidak membentuk arsitektur yang sulit dipelihara.

---

### 3.3 Pola 2: Event-Driven Architecture

#### 3.3.1 Tujuan pola

Banyak modul bergantung pada informasi dosen pengampu, yaitu Modul Kelas-Dosen, Jadwal, KRS, Presensi, Nilai, dan tampilan Informasi Kelas mahasiswa. Ketika dosen diganti, perubahan harus disebarkan secara otomatis tanpa membuat satu modul mengendalikan pembaruan setiap modul lainnya.

Pola event-driven dipilih untuk mendukung:
- **S1:** semua modul selesai memproses dalam maksimal 2 menit dan 0 data berbeda setelah sinkronisasi.
- **S2:** mahasiswa melihat dosen baru dalam maksimal 1 menit setelah membuka ulang atau mengakses kembali SIAKAD, dengan asumsi sinkronisasi S1 telah selesai.

#### 3.3.2 Cara kerja pola

1. Admin Akademik mengajukan penggantian dosen melalui Modul Kelas-Dosen.
2. Sistem memeriksa RBAC dan persetujuan Ketua Jurusan.
3. Jika valid, perubahan disimpan pada sumber data utama dan catatan `LogPerubahan` serta event `DosenDiganti` ditulis dalam satu transaksi.
4. Event dicatat pada tabel *outbox* lalu diterbitkan melalui *event bus*.
5. Modul Jadwal, KRS, Presensi, dan Nilai berlangganan event dan memperbarui data atau hak akses masing-masing.
6. Modul Tampilan Mahasiswa membatalkan atau memperbarui cache yang terdampak.
7. Setiap modul mengirim status berhasil atau gagal beserta waktu pemrosesan.

#### 3.3.3 Taktik pendukung

- **Transactional Outbox:** perubahan dan pencatatan event memiliki keterikatan atomik sehingga perubahan tidak tersimpan tanpa event yang tercatat.
- **Idempotency:** event diproses berdasarkan ID unik agar retry tidak menghasilkan perubahan ganda.
- **Retry dengan jeda bertahap:** event gagal dicoba kembali dengan interval bertahap.
- **Dead-letter atau antrean kegagalan:** event yang terus gagal dipisahkan dan memicu peringatan operasional.
- **Monitoring status:** waktu penerbitan dan pemrosesan setiap modul dipantau terhadap target 2 menit.
- **Cache invalidation:** event digunakan untuk memperbarui atau membatalkan cache Jadwal dan Informasi Kelas.

#### 3.3.4 Hubungan dengan skenario kualitas

- **S1—Modifiabilitas:** satu perubahan diterbitkan sebagai satu event dan diterima oleh seluruh modul tanpa pembaruan silang yang kompleks.
- **S2—Usability:** cache atau tampilan dapat diperingan dan divalidasi terhadap sumber resmi sehingga mahasiswa melihat dosen baru setelah membuka ulang SIAKAD.
- **Target tampilan:** dosen baru maksimal 1 menit setelah sinkron S1 selesai; jika sinkronisasi belum selesai, indikator status ditampilkan dan dosen lama tidak dinyatakan sebagai data terbaru.

#### 3.3.5 Konsekuensi

- Sinkronisasi bersifat *eventual consistency* selama event diproses, sehingga seluruh modul belum tentu langsung sama pada waktu yang sama.
- Sistem memerlukan mekanisme outbox, retry, antrean kegagalan, monitoring, dan pemulihan.
- Debugging alur event dan operasional komponen asinkron lebih kompleks dibanding pembaruan sinkron langsung.

---

### 3.4 Pola 3: Role-Based Access Control (RBAC) dan Audit Trail

#### 3.4.1 Tujuan pola

Penggantian dosen memengaruhi jadwal, hak mengajar, KRS, presensi, nilai, dan reputasi akademik. Karena itu, perubahan hanya boleh diajukan oleh pihak berwenang, disetujui oleh pihak yang terpisah, diterapkan setelah pemeriksaan, dan dicatat lengkap.

Pola ini mendukung S3 dengan memastikan:
- **100%** perubahan sah tercatat pada *Audit Trail*.
- **0** perubahan berhasil diterapkan tanpa RBAC dan persetujuan Ketua Jurusan yang sah.
- Upaya tanpa kewenangan ditolak dan dapat ditelusuri.

#### 3.4.2 Komponen utama

1. **RBAC (Role-Based Access Control)**
   - Admin Akademik berhak mengajukan perubahan dosen.
   - Ketua Jurusan berhak menyetujui penggantian.
   - Dosen lama dan dosen baru tidak dapat mengubah data secara sepihak.
   - Hak akses diterapkan dengan prinsip *least privilege* dan *default-deny*.

2. **Pemisahan kewenangan dan persetujuan**
   - Setiap perubahan dimulai sebagai permintaan berstatus menunggu persetujuan.
   - Pemohon tidak dapat menyetujui permintaan miliknya sendiri.
   - Sistem memeriksa identitas, peran, status persetujuan, dan validitas permintaan sebelum menerapkan perubahan.

3. **Audit trail dan log perubahan**
   - `LogPerubahan` mencatat identitas aktor, Ketua Jurusan yang menyetujui, waktu pengajuan, persetujuan dan penerapan, kelas atau mata kuliah, dosen lama, dosen baru, hasil tindakan, serta alasan penolakan.
   - Catatan bersifat *append-only* dan aksesnya dibatasi pada peran berwenang.

#### 3.4.3 Alur keamanan

1. Pengguna melakukan autentikasi identitas.
2. Sistem memeriksa peran pengguna dengan kebijakan *default-deny*.
3. Admin Akademik mengajukan perubahan; pengguna tanpa kewenangan ditolak dan dicatat.
4. Ketua Jurusan memberikan persetujuan terpisah dari pengajuan.
5. Sistem memeriksa kembali identitas, peran, validitas permintaan, dan persetujuan sebelum menyimpan perubahan.
6. Permintaan sah dicatat pada `LogPerubahan` dan penerbitan event baru dilakukan dalam transaksi yang sama dengan perubahan.

#### 3.4.4 Hubungan dengan skenario kualitas

- **S3—Keamanan:** perubahan ilegal dicegah dan setiap keputusan dapat diaudit.
- **Akuntabilitas:** pengguna dapat mengetahui siapa yang mengajukan, siapa yang menyetujui, kapan perubahan diterapkan, serta nilai lama dan baru.
- **Kepatuhan:** konfigurasi peran harus diperbarui sesuai struktur jabatan agar kewenangan tetap valid.

#### 3.4.5 Konsekuensi

- Alur perubahan menjadi lebih panjang karena memerlukan persetujuan.
- Penyimpanan, beban tulis, retensi, dan pengelolaan akses log audit bertambah.
- Kebijakan peran dan pemisahan kewenangan harus dijaga agar tidak tumpang tindih atau kedaluwarsa.

---

### 3.5 Integrasi Pola Arsitektur

Ketiga pola saling melengkapi dalam satu alur perubahan dosen:

1. Lapisan presentasi menerima pengajuan dari Admin Akademik.
2. Lapisan aplikasi dan domain memeriksa otorisasi serta persetujuan Ketua Jurusan.
3. Permintaan yang ditolak tidak diterapkan dan dicatat dalam log keamanan.
4. Jika valid, lapisan data menyimpan perubahan, mencatat `LogPerubahan`, dan menulis event `DosenDiganti` ke tabel *outbox* dalam satu transaksi.
5. *Event bus* mengirim event kepada Jadwal, KRS, Presensi, dan Nilai.
6. Modul penerima memproses event secara idempotent dan mengirim status proses; target sinkronisasi adalah maksimal 2 menit.
7. Cache atau tampilan informasi kelas diperbarui atau divalidasi sehingga dosen baru dapat terlihat maksimal 1 menit setelah mahasiswa membuka ulang SIAKAD dan sinkronisasi selesai.

Dengan integrasi tersebut, *Layered Architecture* membentuk struktur pemeliharaan, *Event-Driven Architecture* mendukung penyebaran perubahan dan tampilan mutakhir, sedangkan RBAC dan *Audit Trail* melindungi otoritas serta ketertelusuran keputusan.

---

### 3.6 Kesimpulan

Pola arsitektur yang dipilih untuk subsistem *Academic Management* SIAKAD UMI adalah kombinasi:
- **Layered Architecture** untuk pemisahan tanggung jawab dan kemudahan pemeliharaan,
- **Event-Driven Architecture** untuk sinkronisasi data antar-modul dalam maksimal 2 menit,
- **RBAC + Audit Trail** untuk memastikan 0 perubahan ilegal dan 100% perubahan tercatat.

Kombinasi pola ini menjawab masalah utama, yaitu data dosen yang tidak sinkron, tampilan dosen lama ketika mahasiswa membuka ulang SIAKAD, dan tidak adanya jejak keputusan pada penggantian dosen menjelang kuliah perdana.

---

*Bab ini menutup pembahasan pola arsitektur yang menjadi fondasi implementasi skenario atribut kualitas pada subsistem Academic Management SIAKAD UMI.*
