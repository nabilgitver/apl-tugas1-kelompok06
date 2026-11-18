# BAB 5: EVALUASI 
## [U6] Evaluasi Arsitektur

Pada bab ini dievaluasi arsitektur subsistem *Academic Management* SIAKAD UMI (`siakad.umi.ac.id`) menggunakan pohon utilitas mini, matriks *trade-off* berbobot, analisis ala ATAM (*Architecture Tradeoff Analysis Method*), dan dua keputusan arsitektur utama (ADR). Evaluasi merujuk pada Kazman et al. (2000) dan Bass et al. (2021).

---

### 5.1 Pohon Utilitas Mini (*Utility Tree*)

Setiap skenario dinilai berdasarkan kepentingan (*importance*) dan kesulitan (*difficulty*): H = High, M = Medium.

| Skenario | Atribut Kualitas | (Kepentingan, Kesulitan) | Target |
|---|---|---|---|
| **S1** | Modifiabilitas | (H, H) | Data sinkron $\le 2$ menit |
| **S2** | Usability | (H, M) | Dosen baru terlihat $\le 1$ menit saat dibuka ulang |
| **S3** | Keamanan | (M, M) | 100% perubahan tercatat |

---

### 5.2 Analisis *Trade-off* Berbobot

Tiga alternatif dibandingkan: **A** (Layered polos), **B** (Layered + *Event Bus*, dipilih), dan **C** (*Microservices*). Skor 1–5 berdasarkan penilaian kelompok; untuk kriteria biaya, skor 5 berarti paling murah dan sederhana.

| Kriteria (Bobot) | A | B (Dipilih) | C |
|---|---|---|---|
| **Modifiabilitas** (35%) | 2 | 4 | 5 |
| **Usability** (25%) | 2 | 4 | 4 |
| **Keamanan** (20%) | 2 | 4 | 4 |
| **Biaya rendah + sederhana** (20%) | 5 | 4 | 1 |
| **Total** | **2,60** | **4,00** | **3,75** |

Alternatif B dipilih karena paling seimbang antara kemudahan perubahan dan kesederhanaan bagi tim kecil. Analisis sensitivitas: C baru mengungguli B jika bobot Modifiabilitas lebih dari 3 kali bobot Biaya (saat ini 35% berbanding 20%).

---

### 5.3 Analisis ala ATAM

* **Sensitivitas:** Keandalan dan urutan *event* menentukan konsistensi data antar-modul.
* **Trade-off:** *Event Bus* fleksibel (*loose coupling*), tetapi ada jeda sinkron ($\le 2$ menit).
* **Risiko:** *Event* gagal terkirim; mahasiswa harus mengecek sendiri karena tidak ada notifikasi langsung.
* **Non-risiko:** Satu sumber data dosen dan *audit log* membuat riwayat perubahan mudah ditelusuri.

---

### 5.4 Keputusan Arsitektur Utama (ADR)

1. **ADR-01: Sebar Perubahan Dosen via Event**
   * **Status:** Diterima.
   * **Konteks:** Data dosen dipakai banyak modul; perubahan manual rawan tidak sinkron.
   * **Keputusan:** Simpan sekali, lalu publish event `DosenDiganti`; modul lain berlangganan.
   * **Alternatif:** Panggilan langsung antarmodul; *microservices*.
   * **Konsekuensi:** (+) Modul longgar, perubahan menyebar otomatis; (−) konsistensi akhir (jeda sinkron $\le 2$ menit).

2. **ADR-02: *Audit Trail* dan Otorisasi**
   * **Status:** Diterima.
   * **Konteks:** Penggantian dosen tidak berjejak dan tidak jelas siapa yang berwenang.
   * **Keputusan:** Hanya Admin Akademik dengan persetujuan Ketua Jurusan; semua perubahan dicatat.
   * **Alternatif:** Dosen atau prodi mengubah langsung; log hanya di *file server*.
   * **Konsekuensi:** (+) 100% terlacak, 0 perubahan tanpa izin; (−) proses lebih lama.

---

### 5.5 Kesimpulan Evaluasi

Masalah utama **teratasi sebagian pada level desain**: dampak penggantian dosen tersebar otomatis, terlihat saat SIAKAD dibuka ulang, dan tercatat. Notifikasi langsung ke mahasiswa belum tercakup. Seluruh hasil masih target rancangan dan belum divalidasi dengan prototipe.

---

### 5.6 Saran

1. Buat prototipe dan ukur waktu sinkron data ($\le 2$ menit).
2. Uji apakah dosen baru langsung tampil saat SIAKAD dibuka ulang.
3. Tambahkan notifikasi langsung ke mahasiswa.
4. Pastikan *event* tidak hilang saat dikirim.
