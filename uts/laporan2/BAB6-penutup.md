# BAB VI – PENUTUP

## 6.1 Kesimpulan

Berdasarkan hasil analisis dan perancangan arsitektur subsistem **Academic Management pada SIAKAD UMI**, ditemukan permasalahan utama berupa penggantian dosen pengampu menjelang perkuliahan perdana yang menyebabkan perubahan data lambat tersebar ke seluruh modul dan mahasiswa tidak mendapatkan notifikasi.

Untuk mengatasi permasalahan tersebut, kelompok mengusulkan rancangan arsitektur menggunakan pola **Layered + Event-Driven** dengan mekanisme Publish-Subscribe, Audit Trail, serta Role-Based Access Control (RBAC) yang dilengkapi persetujuan ketua jurusan.

Berdasarkan rancangan yang diusulkan, permasalahan utama dinyatakan **teratasi sebagian pada level desain**. Ketika dosen pengampu diganti, data pada modul jadwal, KRS, presensi, dan nilai diharapkan ikut diperbarui secara otomatis. Selain itu, setiap perubahan dirancang agar tercatat dan dapat ditelusuri kembali.

Pencapaian terhadap skenario atribut kualitas yang telah ditentukan dapat dilihat pada tabel berikut.

| Skenario | Target | Hasil Rancangan |
|---|---|---|
| S1 – Modifiabilitas | Data sinkron maksimal 2 menit | Diharapkan tercapai |
| S2 – Usability | Dosen baru terlihat maksimal 1 menit | Diharapkan tercapai |
| S3 – Keamanan | 100% perubahan tercatat | Diharapkan tercapai |

Meskipun demikian, rancangan ini masih memiliki keterbatasan, yaitu belum tersedianya mekanisme notifikasi langsung kepada mahasiswa. Mahasiswa masih harus membuka kembali SIAKAD untuk mengetahui perubahan dosen pengampu.

Selain itu, seluruh target yang telah ditentukan masih berupa target rancangan dan belum diuji menggunakan prototipe. Oleh karena itu, diperlukan pengembangan dan pengujian lebih lanjut untuk memastikan solusi yang diusulkan dapat berjalan sesuai kebutuhan.

## 6.2 Langkah Pengembangan Selanjutnya

Untuk memastikan rancangan arsitektur dapat diterapkan dengan baik, terdapat beberapa langkah pengembangan yang perlu dilakukan, yaitu:

1. **Konfirmasi alur penggantian dosen:** Melakukan konfirmasi kepada admin akademik mengenai proses dan prosedur penggantian dosen pengampu yang berlaku.
2. **Pembuatan prototipe:** Mengembangkan prototipe mekanisme Publish-Subscribe untuk menyebarkan perubahan data dosen ke seluruh modul terkait.
3. **Pengujian sinkronisasi data:** Mengukur waktu penyebaran perubahan dengan target maksimal 2 menit dan memastikan tidak terdapat perbedaan data setelah sinkronisasi.
4. **Pengujian tampilan mahasiswa:** Memastikan informasi dosen pengampu terbaru dapat ditampilkan maksimal 1 menit setelah mahasiswa membuka kembali SIAKAD.
5. **Pengembangan notifikasi:** Mempertimbangkan penambahan mekanisme notifikasi langsung agar mahasiswa memperoleh informasi ketika terjadi penggantian dosen pengampu.

## 6.3 Pembagian Tugas Anggota Kelompok

Pembagian tugas dalam penyusunan materi dan presentasi Mid Test Kelompok III-6 adalah sebagai berikut:

| No. | Nama Anggota | Bagian yang Dikerjakan | Waktu Presentasi |
|---|---|---|---|
| 1 | Feri Ardiansyah | Slide 1: Sampul dan Slide 2: Konteks | 2 menit |
| 2 | Muh Nabil Abyansam | Slide 3: Skenario Atribut Kualitas dan Slide 4: Taktik | 2 menit |
| 3 | Tedsua Alaidin Kamarullah | Slide 5: Pola dan Slide 6: Diagram | 2 menit |
| 4 | Marshandy Resky Saputra | Slide 7: Evaluasi dan Slide 8: ADR | 2 menit |
| 5 | Agim Algifahri | Slide 9: Kesimpulan dan Slide 10: Daftar Pustaka | 2 menit |

Total durasi presentasi adalah **10 menit**, dengan tambahan sesi tanya jawab selama **5 menit**.

Setiap anggota bertanggung jawab menjelaskan bagian yang telah dikerjakan serta memahami keseluruhan rancangan arsitektur yang disusun oleh kelompok.

## 6.4 Penutup

Demikian hasil analisis dan usulan rancangan arsitektur penggantian dosen pengampu pada subsistem Academic Management SIAKAD UMI.

Melalui rancangan ini, diharapkan proses penggantian dosen pengampu dapat dilakukan dengan penyebaran perubahan data yang lebih terkoordinasi, pencatatan perubahan yang jelas, serta pengendalian akses yang lebih baik.

Walaupun rancangan ini belum sepenuhnya menyelesaikan permasalahan, khususnya dalam hal pemberian notifikasi langsung kepada mahasiswa, solusi yang diusulkan diharapkan dapat menjadi dasar pengembangan sistem akademik yang lebih konsisten, terstruktur, dan dapat dipertanggungjawabkan.

**Wassalamu'alaikum warahmatullahi wabarakatuh.**
