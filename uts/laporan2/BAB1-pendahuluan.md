# BAB I. PENDAHULUAN (U1: Konteks dan ASR)

## 1.1 Ringkasan layanan

Layanan inti kelompok adalah **SIAKAD UMI** (siakad.umi.ac.id) [1] pada subsistem *Academic Management* Smart Campus UMI. Fungsi intinya adalah pengelolaan kelas, dosen pengampu, jadwal kuliah, KRS, dan nilai. Penggunanya adalah mahasiswa, dosen, admin akademik, ketua jurusan, dan staf. Titik rawan yang kami angkat adalah **penggantian dosen pengampu di detik-detik kuliah perdana**, karena perubahan itu menyentuh banyak data sekaligus (jadwal, KRS, presensi, nilai, hak input nilai) dan melibatkan banyak pihak.

## 1.2 Masalah utama dan bukti

**Masalah utama:** di detik-detik kuliah perdana, dosen pengampu diganti dengan dosen lain, tetapi **perubahan lambat tersebar ke seluruh data dan mahasiswa tidak mendapatkan notifikasi**.

Dampaknya ada tiga:
1. **Terlambat tahu:** mahasiswa baru mengetahui pergantian saat tiba di kelas kuliah perdana.
2. **Data tidak sinkron:** nama dosen di jadwal, presensi, dan nilai berbeda.
3. **Tanpa jejak:** tidak jelas siapa yang mengganti dan kapan penggantian dilakukan.

| Kode | Jenis bukti | Isi bukti | Dipakai pada |
|---|---|---|---|
| B-1 | Pengalaman langsung anggota kelompok sebagai pengguna SIAKAD | Pada mata kuliah Pemodelan dan Simulasi (mata kuliah wajib, semester 7), dosen pengampu diganti di detik-detik kuliah perdana. Mahasiswa baru tahu saat tiba di kelas, nama dosen di jadwal, presensi, dan nilai berbeda, dan tidak ada keterangan siapa serta kapan penggantian dilakukan. | 1.2, S1, S2 |
| B-2 | Dokumentasi wawancara (foto) | Foto kegiatan wawancara anggota kelompok yang ditampilkan pada slide presentasi "Konteks dan Masalah Utama". Berkas: `[isi path foto di repo, mis. uts/lampiran/foto-wawancara.jpg]`. | 1.2 |

*Tabel 1. Bukti masalah utama.*

**Batas cakupan.** Rancangan pada laporan ini menyelesaikan masalah data yang tidak sinkron dan tidak berjejak, serta memastikan dosen baru terlihat saat mahasiswa membuka ulang SIAKAD. Pemberitahuan langsung (notifikasi) ke mahasiswa belum tercakup dan menjadi pekerjaan lanjutan. Kami hanya memakai pengalaman sebagai pengguna dan dokumentasi wawancara. Kami tidak menguji keamanan sistem kampus dan tidak mengambil data pribadi orang lain.

## 1.3 Stakeholder

| No | Stakeholder | Kepentingan terhadap masalah utama |
|---|---|---|
| 1 | Mahasiswa | Melihat dosen yang benar dan data yang sama di jadwal, presensi, dan nilai. |
| 2 | Dosen pengampu lama | Hak akses kelas dan input nilai dicabut tepat waktu; penggantian tercatat jelas. |
| 3 | Dosen pengampu baru | Segera mendapat akses kelas dan daftar mahasiswa untuk mengajar dan menilai. |
| 4 | Admin akademik | Mengganti dosen cukup sekali input dan datanya konsisten di semua modul. |
| 5 | Ketua jurusan | Menyetujui penggantian dosen dan dapat menelusuri riwayat keputusan. |
| 6 | Tim pengelola TI SIAKAD | Perubahan dan perawatan sistem mudah; masalah mudah dilacak. |

*Tabel 2. Stakeholder utama.*

## 1.4 Karakteristik pendorong dan ASR

Tiga karakteristik pendorong, dipilih dari kebutuhan pada masalah utama [2], [3]:
- **K1 Modifiabilitas:** perubahan dosen cukup dilakukan sekali dan menyebar ke semua modul.
- **K2 Usability:** mahasiswa dapat melihat dosen baru dengan cepat saat membuka ulang SIAKAD.
- **K3 Keamanan:** hanya pihak berwenang yang dapat mengganti dosen dan setiap perubahan tercatat.

| ASR | Kebutuhan signifikan | Ukuran respons | Karakteristik | Skenario |
|---|---|---|---|---|
| ASR-1 | Menjelang kuliah perdana, admin akademik mengganti dosen pengampu; jadwal, KRS, presensi, dan hak input nilai ikut berubah di semua modul. | Sinkron ≤ 2 menit; 0 data berbeda setelah sinkron | K1 | S1 |
| ASR-2 | Setelah dosen diganti, mahasiswa yang membuka ulang SIAKAD melihat dosen baru, bukan tampilan lama. | Dosen baru terlihat ≤ 1 menit setelah sinkron; 0 tampilan lama | K2 | S2 |
| ASR-3 | Pengguna tanpa hak tidak dapat mengubah dosen pengampu; admin berwenang dengan persetujuan ketua jurusan melakukan perubahan yang sah, dan setiap perubahan tercatat (siapa, kapan, dosen lama, dosen baru). | 100% perubahan tercatat; 0 perubahan tanpa izin | K3 | S3 |

*Tabel 3. ASR dan tiga karakteristik pendorong. Angka target adalah target rancangan, belum diuji dengan prototipe.*

ASR-1 menjawab "tidak tersebar ke seluruh data", ASR-2 menjawab "terlambat tahu" tanpa notifikasi langsung, dan ASR-3 menjawab "tanpa jejak". Ketiganya dikembangkan menjadi skenario atribut kualitas pada BAB II.

## 1.5 Bagian Tugas 1 yang diperbarui

- Masalah utama dipertajam menjadi penggantian dosen pengampu di detik-detik kuliah perdana yang tidak tersebar dan tidak berjejak.
- Daftar stakeholder diperluas dan dikaitkan langsung dengan masalah utama.
- ASR dirumuskan ulang menjadi ASR-1 sampai ASR-3 dengan ukuran angka, dan tiga karakteristik pendorong (K1–K3) ditetapkan.
- ADR dari Tugas 1 dipertahankan; ADR baru ditambahkan pada U7.
