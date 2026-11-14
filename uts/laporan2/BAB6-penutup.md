# BAB VI – PENUTUP

## 6.1 Kesimpulan

Berdasarkan hasil analisis dan perancangan arsitektur subsistem **Academic Management pada SIAKAD UMI**, permasalahan utama yang ditemukan adalah pergantian dosen pengampu secara mendadak menjelang perkuliahan perdana yang menyebabkan perubahan informasi tidak tersebar ke seluruh modul dan mahasiswa terlambat mengetahui informasi tersebut.

Solusi yang dirancang menggunakan pola arsitektur **Layered + Event-Driven** dengan mekanisme Publish-Subscribe, Audit Trail, serta Role-Based Access Control (RBAC) yang dilengkapi persetujuan ketua jurusan.

Berdasarkan rancangan tersebut, masalah utama dinyatakan **teratasi sebagian pada level desain**, dengan hasil yang diharapkan sebagai berikut:

| Skenario | Target Respons | Hasil yang Diharapkan |
|---|---|---|
| S1 – Modifiabilitas | Sinkronisasi ≤ 2 menit | Data seluruh modul tersinkronisasi |
| S2 – Usability | Dosen baru terlihat ≤ 1 menit setelah SIAKAD dibuka ulang | Mahasiswa dapat melihat informasi terbaru |
| S3 – Keamanan | 100% perubahan tercatat | Seluruh perubahan dapat ditelusuri dan tidak ada perubahan tanpa izin |

Meskipun demikian, masih terdapat keterbatasan pada rancangan ini. Mahasiswa belum menerima notifikasi langsung ketika terjadi pergantian dosen pengampu sehingga harus membuka kembali SIAKAD untuk memeriksa informasi terbaru.

Seluruh target tersebut juga belum divalidasi melalui prototipe. Oleh karena itu, diperlukan pengujian lebih lanjut untuk memastikan rancangan dapat memenuhi kebutuhan kualitas yang telah ditentukan.

## 6.2 Langkah Pengembangan Selanjutnya

Untuk memastikan rancangan arsitektur dapat diterapkan dengan baik, beberapa langkah pengembangan yang perlu dilakukan adalah:

1. **Validasi kebutuhan:** Mengonfirmasi alur penggantian dosen pengampu bersama admin akademik.
2. **Pengembangan prototipe:** Menerapkan mekanisme event untuk menyebarkan perubahan data dosen ke seluruh modul terkait.
3. **Pengujian sinkronisasi:** Mengukur waktu penyebaran perubahan dengan target maksimal 2 menit.
4. **Pengujian tampilan:** Memastikan informasi dosen terbaru terlihat ketika mahasiswa membuka kembali SIAKAD.
5. **Pengujian keamanan:** Memastikan hanya pengguna berwenang yang dapat melakukan perubahan serta seluruh perubahan tercatat dalam audit log.

## 6.3 Pembagian Tugas Anggota Kelompok

Pembagian tugas dalam penyusunan dan presentasi Mid Test Kelompok III-6 adalah sebagai berikut:

| No. | Nama Anggota | Bagian yang Dikerjakan | Durasi Presentasi |
|---|---|---|---|
| 1 | Feri Ardiansyah | Sampul dan Konteks (Slide 1–2) | 2 menit |
| 2 | Muh Nabil Abyansam | Skenario Atribut Kualitas dan Taktik (Slide 3–4) | 2 menit |
| 3 | Tedsua Alaidin Kamarullah | Pola dan Diagram (Slide 5–6) | 2 menit |
| 4 | Marshandy Resky Saputra | Evaluasi dan ADR (Slide 7–8) | 2 menit |
| 5 | Agim Algifahri | Kesimpulan dan Daftar Pustaka (Slide 9–10) | 2 menit |

**Total durasi presentasi:** 10 menit.

**Sesi tanya jawab:** 5 menit.

Setiap anggota bertanggung jawab menjelaskan bagian yang telah dikerjakan serta memahami keseluruhan rancangan arsitektur yang disusun oleh kelompok.

## 6.4 Penutup

Demikian hasil analisis dan perancangan arsitektur subsistem Academic Management pada SIAKAD UMI. Diharapkan rancangan ini dapat menjadi solusi untuk meningkatkan konsistensi data akademik, mempercepat penyebaran informasi perubahan dosen pengampu, serta meningkatkan keamanan dan akuntabilitas pengelolaan data.

Perancangan ini masih memerlukan validasi dan pengujian lebih lanjut agar solusi yang diusulkan dapat diterapkan secara efektif sesuai kebutuhan pengguna.

Dengan adanya rancangan ini, diharapkan permasalahan pergantian dosen pengampu yang tidak terinformasikan dengan baik dapat diminimalkan.

**Wassalamu'alaikum warahmatullahi wabarakatuh.**
