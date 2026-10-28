# A5. Keputusan Arsitektur dan ADR

Penanggung jawab: Tedsua Alaidin KAmarullah

## Ringkasan

Subsistem Academic Management bertanggung jawab untuk mendukung proses operasional layanan MSIB, termasuk pendaftaran, penempatan mitra, pencatatan logbook, bimbingan, dan penilaian. Untuk memastikan layanan tersebut berjalan secara terpadu, konsisten, dan mudah dikelola, diputuskan untuk membangun subsistem ini dengan pendekatan arsitektur yang terstruktur dan modular.

Keputusan utama yang diambil dalam pengembangan subsistem ini meliputi:
- penggunaan arsitektur modular berbasis monolith;
- penggunaan basis data relasional PostgreSQL;
- penerapan kontrol akses berbasis peran (Role-Based Access Control / RBAC).

Keputusan tersebut diuraikan dalam dua Architecture Decision Record (ADR) berikut. ADR lengkap dapat diakses pada folder `adr/`.

---

## ADR-01: Penggunaan Arsitektur Modular Berbasis Monolith

- Status: Accepted

- Context:
  Subsistem Academic Management mencakup beberapa proses bisnis yang saling terkait, yaitu pendaftaran MSIB, penempatan mitra, pemantauan logbook, bimbingan, dan penilaian. Karena proses-proses tersebut memiliki keterkaitan data yang tinggi, namun tetap memerlukan pemisahan fungsi agar pengelolaan kode tetap teratur, maka perlu ada struktur arsitektur yang dapat menyeimbangkan fleksibilitas pengembangan dan kemudahan implementasi.

  Selain itu, pengembangan dilakukan dalam skala proyek akademik dengan tim yang relatif terbatas. Oleh karena itu, arsitektur yang sederhana, mudah diimplementasikan, dan tidak memerlukan kompleksitas operasional yang tinggi menjadi keputusan yang tepat pada tahap awal pengembangan.

- Decision:
  Diputuskan untuk menggunakan arsitektur modular berbasis monolith pada subsistem Academic Management. Aplikasi dibagi ke dalam modul-modul spesifik berdasarkan fungsi bisnis, yaitu:
  - modul pendaftaran MSIB;
  - modul penempatan mitra;
  - modul logbook;
  - modul bimbingan;
  - modul penilaian;
  - modul autentikasi dan otorisasi.

  Masing-masing modul dipisahkan berdasarkan concern-nya, tetapi tetap dikelola dalam satu aplikasi utama agar proses integrasi, deployment, dan pengujian dapat dilakukan dengan lebih efisien.

- Consequences:
  Keuntungan:
  - pengembangan dan pengujian lebih sederhana;
  - integrasi antar modul lebih cepat karena berada dalam satu aplikasi;
  - struktur kode lebih mudah dipahami dan dikelola;
  - deployment lebih efisien untuk skala proyek menengah;
  - biaya infrastruktur lebih rendah dibanding arsitektur yang terdistribusi.

  Kekurangan:
  - jika sistem berkembang sangat besar, monolith dapat menjadi kompleks dan sulit dikelola;
  - skalabilitas horizontal lebih terbatas dibanding sistem terdistribusi;
  - diperlukan disiplin pengelolaan modul agar tidak terjadi ketergantungan antar fungsi yang berlebihan.

- Compliance:
  Kepatuhan terhadap keputusan ini dijaga melalui:
  - pemisahan modul berdasarkan fungsi bisnis;
  - penggunaan struktur folder yang konsisten;
  - pembatasan dependency antar modul sesuai boundary yang telah ditentukan;
  - review kode berkala untuk menjaga integritas arsitektur;
  - dokumentasi modul agar tetap terjaga keterbacaan dan maintainability.

---

## ADR-02: Penggunaan Basis Data Relasional PostgreSQL

- Status: Accepted

- Context:
  Subsistem Academic Management menangani data yang bersifat terstruktur dan memiliki keterkaitan antar entitas, seperti data mahasiswa, mitra, pendaftaran, logbook, kegiatan bimbingan, dan hasil penilaian. Data tersebut memerlukan mekanisme penyimpanan yang aman, konsisten, dan dapat diquery untuk kebutuhan laporan, audit, serta evaluasi.

  Selain itu, kebutuhan validasi data, integritas relasional, dan pengelolaan transaksi menjadi pertimbangan penting dalam pemilihan teknologi basis data. Oleh karena itu, sistem memerlukan basis data yang mampu menangani relasi antar entitas dengan tingkat konsistensi yang tinggi.

- Decision:
  Diputuskan untuk menggunakan PostgreSQL sebagai basis data utama subsistem Academic Management. PostgreSQL dipilih karena mendukung model basis data relasional, memiliki kemampuan pengelolaan relasi antar tabel, serta menyediakan fitur keamanan, transaksi, dan integritas data yang sesuai dengan kebutuhan sistem akademik.

- Consequences:
  Keuntungan:
  - struktur data lebih jelas dan terorganisasi;
  - integritas data lebih terjaga melalui foreign key dan constraint;
  - query untuk laporan, evaluasi, dan analisis lebih mudah dilakukan;
  - proses pencatatan dan audit data menjadi lebih terstruktur;
  - kompatibel dengan kebutuhan sistem yang berbasis transaksi dan validasi data.

  Kekurangan:
  - perlu perancangan skema yang matang agar performa tetap optimal;
  - kurang fleksibel untuk data yang sangat tidak terstruktur;
  - kompleksitas meningkat jika volume data berkembang sangat besar dan beragam.

- Compliance:
  Kepatuhan terhadap keputusan ini dijaga melalui:
  - penyusunan skema tabel yang konsisten dan terdokumentasi;
  - pengaturan relasi antar entitas dengan foreign key;
  - penggunaan validasi data pada proses insert dan update;
  - penyimpanan data audit dan perubahan status secara terstruktur;
  - pemeliharaan keamanan akses basis data sesuai peran pengguna.

---

## Kesimpulan

Keputusan arsitektur ini dipilih untuk memastikan subsistem Academic Management dapat diimplementasikan secara efektif, mudah dikembangkan, dan sesuai kebutuhan operasional layanan MSIB. Arsitektur modular berbasis monolith memberikan keseimbangan antara kemudahan pengelolaan dan skalabilitas awal, sementara PostgreSQL memastikan data akademik dikelola secara konsisten, aman, dan dapat diandalkan.
