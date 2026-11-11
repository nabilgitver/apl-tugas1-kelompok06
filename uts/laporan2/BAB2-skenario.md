# BAB 2: SKENARIO 
## [U2] Skenario Atribut Kualitas

Pada bab ini dijelaskan tiga skenario atribut kualitas yang dirancang untuk mengatasi permasalahan pada subsistem *Academic Management* SIAKAD UMI (`siakad.umi.ac.id`), khususnya terkait kasus penggantian dosen pengampu menjelang kuliah perdana. Perancangan skenario mengacu pada standar *General Scenario Pattern* oleh Bass et al. (2021).

---

### 2.1 Ringkasan Skenario Atribut Kualitas

Berikut adalah tabel ringkasan skenario atribut kualitas beserta ukuran angka (metrik) yang ditetapkan:

| ID | Atribut Kualitas | Kejadian / Stimulus | Ukuran Respons (Target Angka) |
|---|---|---|---|
| **S1** | **Modifiabilitas** (*Modifiability*) | Admin akademik mengganti dosen pengampu menjelang kuliah perdana. | • Jeda sinkron data $\le 2$ menit<br>• Data sinkron di semua modul<br>• **0** data berbeda setelah sinkronisasi |
| **S2** | **Usability** | Dosen pengampu diganti sebelum kuliah perdana tanpa notifikasi langsung. | • Tampilan dosen baru terlihat dalam $\le 1$ menit saat SIAKAD dibuka ulang<br>• **0** tampilan lama tersisa setelah proses sinkronisasi |
| **S3** | **Keamanan** (*Security*) | Pengguna tanpa hak mencoba mengubah dosen pengampu. | • **100%** perubahan sah tercatat di log audit<br>• **0** perubahan berhasil dilakukan tanpa izin (otorisasi RBAC + persetujuan ketua jurusan) |

---

### 2.2 Rincian Skenario Atribut Kualitas

#### 2.2.1 Skenario 1 (S1): Modifiabilitas (*Modifiability*)
* **Sumber Stimulus:** Admin Akademik.
* **Stimulus:** Mengganti dosen pengampu pada satu kelas kuliah menjelang kuliah perdana.
* **Lingkungan (*Environment*):** Waktu operasional normal sistem / *runtime*.
* **Artifak:** Modul Kelas-Dosen, Jadwal, KRS, Presensi, dan Nilai.
* **Respons:** Sistem memperbarui data dosen pengampu dan menyebarkan perubahan ke seluruh modul terkait secara otomatis.
* **Ukuran Respons (*Response Measure*):**
  1. Jeda waktu sinkronisasi data antar-modul $\le 2$ menit.
  2. Seluruh data terkait (Jadwal, KRS, Presensi, dan Hak Input Nilai) konsisten secara otomatis.
  3. **0** inkonsistensi data (data berbeda) setelah alur sinkronisasi selesai.

#### 2.2.2 Skenario 2 (S2): Usability
* **Sumber Stimulus:** Mahasiswa.
* **Stimulus:** Membuka ulang (*refresh*/akses kembali) aplikasi SIAKAD untuk memastikan jadwal dan dosen pengampu secara mandiri (karena tidak ada notifikasi langsung).
* **Lingkungan (*Environment*):** Aplikasi web SIAKAD pada perangkat pengguna (*user interface*).
* **Artifak:** Antarmuka Pengguna (UI) Jadwal dan Informasi Kelas Mahasiswa.
* **Respons:** Sistem menampilkan data dosen pengampu yang paling mutakhir pada antarmuka pengguna.
* **Ukuran Respons (*Response Measure*):**
  1. Informasi dosen baru berhasil ditampilkan dalam waktu $\le 1$ menit setelah aplikasi dibuka ulang oleh mahasiswa.
  2. **0** tampilan informasi dosen lama yang tersisa di layar setelah sinkronisasi selesai.

#### 2.2.3 Skenario 3 (S3): Keamanan (*Security*)
* **Sumber Stimulus:** Pengguna tanpa hak akses (*unauthorized user*).
* **Stimulus:** Mencoba melakukan pengubahan data dosen pengampu pada modul Kelas-Dosen.
* **Lingkungan (*Environment*):** Sistem operasional aktif (*online system*).
* **Artifak:** Modul Otorisasi dan *Audit Log* (Log Perubahan).
* **Respons:** Sistem menolak pengubahan tanpa otorisasi sah, dan hanya memproses pengubahan yang dilakukan oleh Admin Akademik dengan persetujuan Ketua Jurusan serta mencatat riwayat perubahan tersebut.
* **Ukuran Respons (*Response Measure*):**
  1. **100%** perubahan data dosen yang sah tercatat di dalam *Audit Trail* (mencatat aktor, timestamp, nilai lama, dan nilai baru).
  2. **0** pengubahan data berhasil dilakukan tanpa izin/otorisasi yang sah.

---

### 2.3 Taktik Arsitektur yang Dipilih

Untuk memenuhinya target ukuran angka pada ketiga skenario di atas, taktik arsitektur yang diterapkan adalah:

1. **Publish-Subscribe / Event Bus (S1 & S2):**
   * *Cara Kerja:* Perubahan dosen disimpan satu kali di modul utama (Kelas-Dosen), kemudian menerbitkan event `DosenDiganti` ke modul lainnya (Jadwal, KRS, Presensi, Nilai).
   * *Harga/Consequence:* Menghasilkan *eventual consistency* dengan jeda sinkronisasi $\le 2$ menit dan alur *debugging* event yang lebih kompleks.

2. **Audit Trail & Otorisasi RBAC + Persetujuan (S3):**
   * *Cara Kerja:* Membatasi hak akses pengubahan hanya untuk Admin Akademik yang telah disetujui Ketua Jurusan, serta mencatat setiap aktivitas pengubahan data ke dalam tabel `LogPerubahan`.
   * *Harga/Consequence:* Memerlukan alur proses pengubahan yang sedikit lebih lama serta peningkatan kebutuhan penyimpanan database untuk log.
