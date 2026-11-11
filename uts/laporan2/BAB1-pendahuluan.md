# BAB I. PENDAHULUAN (U1: Konteks dan ASR)

## 1.1 Ringkasan layanan

Layanan inti kelompok adalah **SIAKAD UMI** (siakad.umi.ac.id) pada subsistem *Academic Management* Smart Campus UMI. Fungsi intinya adalah pengelolaan kelas dan dosen pengampu, jadwal, KRS, dan nilai. Penggunanya adalah mahasiswa, dosen, dan admin akademik. Titik rawan yang kami angkat adalah **penggantian dosen pengampu di tengah semester**, karena perubahan itu menyentuh banyak data sekaligus (jadwal, presensi, nilai, hak akses) dan banyak pihak.

## 1.2 Masalah utama dan bukti

**Masalah utama:** dosen pengampu diganti secara tiba-tiba, tetapi **perubahan tidak tersebar ke seluruh data dan mahasiswa tidak diberi tahu**.

Dampaknya ada tiga:
1. **Terlambat tahu:** mahasiswa baru mengetahui pergantian saat hadir di kelas.
2. **Data tidak sinkron:** nama dosen di jadwal, presensi, dan nilai berbeda.
3. **Tanpa jejak:** tidak jelas siapa yang mengganti, kapan, dan atas alasan apa.

| Kode | Jenis bukti | Isi bukti | Dipakai pada |
|---|---|---|---|
| B-1 | Pengalaman langsung anggota kelompok (sebagai pengguna SIAKAD) | Dosen pengampu mata kuliah diganti di tengah semester; mahasiswa baru mengetahuinya saat hadir di kelas; nama dosen pada jadwal, presensi, dan nilai berbeda; tidak ada keterangan siapa dan kapan penggantian dilakukan. | 1.2, S1, S2 |
| B-2 | Tangkapan layar SIAKAD | **[ISI: mata kuliah, semester, dan tangkapan layar jadwal/presensi/nilai yang menunjukkan dosen berbeda. Tutup NIM, nama mahasiswa, dan data pribadi lain.]** | 1.2 |

*Tabel 1. Bukti masalah utama.*

Kami hanya memakai pengalaman sebagai pengguna dan tidak menguji keamanan sistem kampus maupun mengambil data pribadi orang lain.

## 1.3 Stakeholder

| No | Stakeholder | Kepentingan terhadap masalah utama |
|---|---|---|
| 1 | Mahasiswa | Segera tahu bila dosen pengampu berganti dan melihat data yang sama di semua menu. |
| 2 | Dosen pengampu lama | Hak akses ke kelas, presensi, dan nilai dicabut tepat waktu; penggantian tercatat jelas. |
| 3 | Dosen pengampu baru | Segera mendapat akses kelas dan daftar mahasiswa untuk mengajar dan menilai. |
| 4 | Admin akademik (prodi/fakultas) | Dapat mengganti dosen dengan mudah, sekali input, dan datanya konsisten. |
| 5 | Tim pengembang dan pengelola TI SIAKAD | Perubahan aturan dan perawatan sistem mudah; masalah mudah dilacak. |
| 6 | Pimpinan fakultas / penjaminan mutu | Riwayat penggantian dosen dapat diaudit. |

*Tabel 2. Stakeholder utama.*

## 1.4 ASR dan karakteristik pendorong

Tiga karakteristik pendorong:
- **K1** Konsistensi data dan ketertelusuran
- **K2** Ketepatan pemberitahuan
- **K3** Keamanan dan hak akses

| ASR | Kebutuhan signifikan | Karakteristik | ISO/IEC 25010 |
|---|---|---|---|
| ASR-1 | Satu kali penggantian dosen tercermin konsisten pada jadwal, presensi, nilai, dan KRS; tidak ada kondisi setengah berubah. Target ≤ 1 menit. | K1 | *Reliability*, *Functional suitability* |
| ASR-2 | Mahasiswa terdampak diberi tahu segera, dan pemberitahuan tidak hilang walau layanan pengirim gagal sementara. Target ≤ 5 menit. | K2 | *Reliability*, *Usability* |
| ASR-3 | Setiap penggantian tercatat (siapa, kapan, dosen lama dan baru, alasan) dan catatan tidak dapat diubah. | K1 | *Security* (*accountability*) |
| ASR-4 | Hanya admin akademik berwenang yang dapat mengganti dosen; hak dosen lama dicabut dan hak dosen baru diberikan. Target ≤ 1 menit. | K3 | *Security* |

*Tabel 3. ASR dan tiga karakteristik pendorong. Angka target adalah target rancangan dan dapat disesuaikan.*

## 1.5 Bagian Tugas 1 yang diperbarui

- Masalah utama dipertajam menjadi penggantian dosen pengampu yang tidak tersebar dan tidak diberitahukan.
- ASR dirumuskan ulang menjadi ASR-1 sampai ASR-4 dengan ukuran angka, dan tiga karakteristik pendorong (K1–K3) ditetapkan.
- ADR dari Tugas 1 dipertahankan; ADR baru ditambahkan pada U7.
