
# BAB 5: EVALUASI 
## [U6] Evaluasi Arsitektur

Pada bab ini dijabarkan hasil evaluasi arsitektur subsistem *Academic Management* pada SIAKAD UMI (`siakad.umi.ac.id`) berdasarkan pohon utilitas mini, matriks trade-off berbobot, analisis metode ATAM (*Architecture Tradeoff Analysis Method*), serta ringkasan keputusan arsitektur utama. Evaluasi ini merujuk pada kerangka kerja Kazman et al. (2000) dan Bass et al. (2021) untuk memastikan konsistensi penuh dengan materi presentasi kelompok.

---

### 5.1 Pohon Utilitas Mini (*Utility Tree*)

Pohon utilitas memetakan prioritas kualitas sistem terhadap skenario yang telah diuji. Setiap skenario dinilai berdasarkan tingkat kepentingan (*importance*) dan tingkat kesulitan implementasinya (*difficulty*).

| Skenario | Atribut Kualitas | Penilaian (Kepentingan, Kesulitan) | Deskripsi Target & Hasil |
|---|---|---|---|
| **S1** | Modifiabilitas | (H, H) | Sinkronisasi data antar-modul $\le 2$ menit tercapai melalui pola *publish-subscribe*. |
| **S2** | Usability | (H, M) | Tampilan dosen baru terlihat dalam $\le 1$ menit saat aplikasi dibuka ulang oleh mahasiswa. |
| **S3** | Keamanan | (M, M) | 100% perubahan tercatat di *audit log* dengan otorisasi RBAC dan persetujuan ketua jurusan. |

---

### 5.2 Analisis *Trade-off* Berbobot

Penilaian komparatif dilakukan untuk menentukan pilihan arsitektur terbaik antara alternatif **A** (Layered polos), **B** (Layered + *Event Bus* / *Publish-Subscribe* yang dipilih), dan **C** (*Microservices*). Penilaian menggunakan skala 1–5 berdasarkan bobot kriteria kelompok:

| Kriteria Evaluasi | Bobot | Alternatif A (Layered Polos) | Alternatif B (Dipilih: Layered + Event Bus) | Alternatif C (Microservices) |
|---|---|---|---|---|
| **Modifiabilitas** | 35% | 2 | 4 | 5 |
| **Usability** | 25% | 2 | 4 | 4 |
| **Keamanan** | 20% | 2 | 4 | 4 |
| **Biaya Rendah + Sederhana** | 20% | 5 | 4 | 1 |
| **Total Skor Berbobot** | **100%** | **2,60** | **4,00** | **3,75** |

* **Kesimpulan Trade-off:** Alternatif B terpilih karena memberikan keseimbangan optimal antara fleksibilitas modifikasi subsistem dan tingkat kerumitan implementasi bagi tim pengembang, meskipun mengorbankan konsistensi seketika (*eventual consistency* dengan jeda $\le 2$ menit).

---

### 5.3 Analisis Metode ATAM (*Architecture Tradeoff Analysis Method*)

Berdasarkan pendekatan evaluasi arsitektur ala ATAM (Kazman et al., 2000):

* **Sensitivitas (*Sensitivity*):** Keandalan jaringan dan urutan pengiriman *event* pada *Event Bus* sangat menentukan konsistensi data di seluruh modul (Jadwal, KRS, Presensi, dan Nilai).
* **Trade-off:** Penggunaan pola *Event Bus* memberikan fleksibilitas tinggi (*loose coupling*), namun menimbulkan *trade-off* berupa jeda waktu sinkronisasi (*eventual consistency* $\le 2$ menit).
* **Risiko (*Risks*):** Potensi kegagalan pengiriman *event* di mana mahasiswa mungkin harus mengecek ulang secara mandiri apabila notifikasi otomatis belum terintegrasi secara penuh.
* **Non-Risiko (*Non-Risks*):** Keberadaan satu sumber data master dosen dan audit log yang terpusat membuat penelusuran riwayat perubahan menjadi sangat mudah dan transparan.

---

### 5.4 Keputusan Arsitektur Utama (ADR)

Ringkasan keputusan arsitektur (*Architecture Decision Records* / ADR) yang diterapkan dalam perancangan ini:

1. **ADR-01: Penyebaran Perubahan Dosen via Event (*Publish-Subscribe*)**
   * **Status:** Diterima
   * **Keluaran:** Data dosen dipakai banyak modul; perubahan manual rawan tidak sinkron. Keputusannya adalah menyimpan data sekali, lalu mempublikasikan event `DosenDiganti`.
   * **Konsekuensi:** (+) Modul menjadi longgar (*loose coupling*) dan perubahan menyebar otomatis; (−) Memiliki jeda sinkronisasi $\le 2$ menit (*eventual consistency*).

2. **ADR-02: *Audit Trail* dan Otorisasi Ketat**
   * **Status:** Diterima
   * **Keluaran:** Penggantian dosen sebelumnya tidak berjejak dan tidak jelas siapa yang berwenang. Keputusannya adalah membatasi hanya Admin Akademik dengan persetujuan Ketua Jurusan, serta mencatat seluruh aktivitas ke dalam log perubahan.
   * **Konsekuensi:** (+) 100% terlacak dan tidak ada perubahan tanpa izin; (−) Proses birokrasi pengubahan data menjadi sedikit lebih lama.

---

### 5.5 Kesimpulan Evaluasi

Masalah utama terkait keterlambatan informasi penggantian dosen pengampu di detik-detik kuliah perdana **teratasi sebagian pada level desain**:
* Dampak penggantian dosen tersebar otomatis ke modul terkait.
* Informasi terbaru dapat dilihat mahasiswa saat SIAKAD dibuka ulang.
* Seluruh tindakan tercatat secara transparan di dalam sistem.
* *Catatan:* Notifikasi langsung secara *real-time* ke perangkat mahasiswa belum tercakup dalam fase perancangan ini dan menjadi poin rekomendasi untuk pengembangan prototipe berikutnya.
