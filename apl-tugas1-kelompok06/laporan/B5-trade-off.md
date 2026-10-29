## B5. Analisis *Trade-off*

Dalam merancang sistem, memperkuat satu karakteristik sering membuat karakteristik lain melemah [2]. Tabel B5.1 membahas tiga pasang karakteristik pendorong layanan MSIB, dampaknya, dan mana yang didahulukan.

**Tabel B5.1 Analisis *trade-off* antarkarakteristik pendorong**

| No | Pasangan karakteristik | Dampak saling memengaruhi | Yang didahulukan | Alasan |
|----|------------------------|---------------------------|------------------|--------|
| 1 | Keamanan dan privasi data vs interoperabilitas | Kalau data dipertukarkan dengan SIAKAD dan mitra, ada jalur baru menuju data mahasiswa, jadi risiko bocor bertambah. Sebaliknya, kontrol keamanan yang ketat (otentikasi tiap sistem, pembatasan data yang dikirim, pemeriksaan format) membuat integrasi lebih rumit dan butuh waktu lebih lama. | Keamanan | Data yang dipertukarkan berisi identitas, berkas pendaftaran, dan nilai yang bersifat pribadi dan sulit ditarik kembali kalau sudah bocor. Karena itu hanya data yang perlu saja yang dibagikan, dan setiap pertukaran harus melalui otentikasi. Interoperabilitas tetap dipenuhi dalam batas itu. |
| 2 | Interoperabilitas vs ketersediaan | Kalau terhubung langsung, layanan bergantung pada SIAKAD dan sistem mitra. Saat salah satunya bermasalah, pendaftaran atau pengisian *logbook* ikut gagal. Ketergantungan ini bisa dikurangi dengan antrean, salinan data lokal, dan sinkronisasi tertunda, tetapi data antarsistem jadi tidak selalu langsung terbaru. | Ketersediaan | Mahasiswa terikat tenggat, sedangkan sinkronisasi ke SIAKAD atau mitra boleh tertunda sebentar tanpa merugikan siapa pun. Yang penting, *logbook* dan berkas tidak hilang atau gagal terkirim. |
| 3 | Keamanan dan privasi data vs ketersediaan | Verifikasi dua langkah, pembatasan percobaan masuk, dan pencatatan *log* menambah langkah dan beban sistem. Saat pendaftaran sedang ramai, ini bisa memperlambat atau bahkan mengunci pengguna yang sah, sehingga layanan terasa tidak tersedia. | Keamanan | Nilai dan penempatan berpengaruh pada konversi SKS dan kelulusan mahasiswa, dan sulit dikoreksi. Karena itu verifikasi dua langkah hanya diwajibkan untuk dosen pembimbing dan admin yang mengubah nilai atau penempatan. Mahasiswa cukup memakai *login* biasa supaya tidak terhambat saat tenggat. Ketersediaan tetap dijaga lewat cadangan data dan pemantauan. |

Dari ketiga pasangan itu, urutan prioritasnya adalah keamanan dan privasi data, lalu ketersediaan, lalu interoperabilitas.
