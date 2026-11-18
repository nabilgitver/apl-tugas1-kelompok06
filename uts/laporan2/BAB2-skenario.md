# BAB 2: SKENARIO 
## [U2] Skenario Atribut Kualitas

Pada bab ini dijelaskan tiga skenario atribut kualitas yang dirancang untuk mengatasi permasalahan pada subsistem *Academic Management* SIAKAD UMI (`siakad.umi.ac.id`), khususnya terkait kasus penggantian dosen pengampu di detik-detik kuliah perdana, ketika perubahan lambat tersebar ke seluruh data dan mahasiswa tidak mendapatkan notifikasi. Perancangan skenario mengacu pada standar *General Scenario Pattern* oleh Bass et al. (2021). Lingkungan (*environment*) untuk ketiga skenario adalah menjelang kuliah perdana.

---

### 2.1 Ringkasan Skenario Atribut Kualitas

Berikut adalah tabel ringkasan skenario atribut kualitas beserta ukuran angka (metrik) yang ditetapkan:

| ID | Atribut Kualitas | Kejadian / Stimulus | Ukuran Respons (Target Angka) |
|---|---|---|---|
| **S1** | **Modifiabilitas** (*Modifiability*) | Menjelang kuliah perdana, admin akademik mengganti dosen pengampu. Sistem memperbarui jadwal, KRS, presensi, dan hak input nilai. | • Data sinkron di semua modul $\le 2$ menit<br>• **0** data berbeda setelah sinkron |
| **S2** | **Usability** | Dosen pengampu diganti sebelum kuliah perdana. Mahasiswa tidak mendapat notifikasi langsung; mereka membuka ulang SIAKAD, mengecek sendiri, lalu melihat dosen baru. | • Dosen baru terlihat $\le 1$ menit saat SIAKAD dibuka ulang<br>• **0** tampilan lama; diukur setelah sinkron |
| **S3** | **Keamanan** (*Security*) | Pengguna tanpa hak mencoba mengubah dosen pengampu; admin berwenang, dengan persetujuan ketua jurusan, melakukan perubahan yang sah. | • **100%** perubahan tercatat<br>• **0** perubahan tanpa izin |

---

### 2.2 Rincian Skenario Atribut Kualitas

#### 2.2.1 Skenario 1 (S1): Modifiabilitas (*Modifiability*)
* **Sumber Stimulus:** Admin Akademik.
* **Stimulus:** Mengganti dosen pengampu pada satu kelas kuliah.
* **Lingkungan (*Environment*):** Menjelang kuliah perdana.
* **Artifak:** Modul Kelas-Dosen, Jadwal, KRS, Presensi, dan Nilai.
* **Respons:** Sistem menyimpan perubahan satu kali, lalu memperbarui jadwal, KRS, presensi, dan hak input nilai secara otomatis.
* **Ukuran Respons (*Response Measure*):**
  1. Data sinkron di semua modul dalam waktu $\le 2$ menit.
  2. **0** data berbeda setelah sinkron.

#### 2.2.2 Skenario 2 (S2): Usability
* **Sumber Stimulus:** Mahasiswa.
* **Stimulus:** Dosen pengampu diganti sebelum kuliah perdana dan mahasiswa tidak mendapat notifikasi langsung, sehingga mahasiswa membuka ulang (*refresh*/akses kembali) SIAKAD dan mengecek sendiri.
* **Lingkungan (*Environment*):** Menjelang kuliah perdana, melalui aplikasi web SIAKAD pada perangkat mahasiswa.
* **Artifak:** Tampilan Mahasiswa (Jadwal dan Informasi Kelas).
* **Respons:** Sistem menampilkan data dosen pengampu yang paling mutakhir (dosen baru).
* **Ukuran Respons (*Response Measure*):**
  1. Dosen baru terlihat dalam waktu $\le 1$ menit saat SIAKAD dibuka ulang.
  2. **0** tampilan dosen lama; diukur setelah sinkron selesai.

#### 2.2.3 Skenario 3 (S3): Keamanan (*Security*)
* **Sumber Stimulus:** Pengguna tanpa hak akses (*unauthorized user*).
* **Stimulus:** Mencoba mengubah dosen pengampu.
* **Lingkungan (*Environment*):** Menjelang kuliah perdana, saat sistem aktif (*online system*).
* **Artifak:** Modul Otorisasi dan *Audit Log* (`LogPerubahan`).
* **Respons:** Sistem menolak perubahan tanpa izin. Perubahan hanya sah jika dilakukan Admin Akademik berwenang dengan persetujuan Ketua Jurusan, dan setiap perubahan dicatat.
* **Ukuran Respons (*Response Measure*):**
  1. **100%** perubahan tercatat di dalam *Audit Trail* (siapa, kapan, nilai lama, dan nilai baru).
  2. **0** perubahan tanpa izin.

---

### 2.3 Taktik Arsitektur yang Dipilih

Untuk memenuhi target ukuran angka pada ketiga skenario di atas, taktik arsitektur yang diterapkan adalah:

1. **Publish-Subscribe / Event (S1 & S2):**
   * *Cara Kerja:* Perubahan dosen disimpan satu kali, kemudian event `DosenDiganti` disebar ke modul lainnya (Jadwal, KRS, Presensi, Nilai, dan Tampilan Mahasiswa).
   * *Harga/Consequence:* Jeda sinkron $\le 2$ menit (*eventual consistency*) dan *debugging* alur event lebih sulit.

2. **Audit Trail (S3):**
   * *Cara Kerja:* Setiap perubahan mencatat siapa, kapan, nilai lama dan nilai baru ke dalam tabel `LogPerubahan`.
   * *Harga/Consequence:* Kebutuhan penyimpanan dan beban tulis bertambah.

3. **Otorisasi RBAC + Persetujuan (S3):**
   * *Cara Kerja:* Hanya Admin Akademik, dengan persetujuan Ketua Jurusan, yang dapat mengubah dosen pengampu.
   * *Harga/Consequence:* Proses perubahan lebih lama.

*Alur penggantian dosen pengampu:* Admin ubah dosen → Otorisasi + persetujuan → Simpan DB + catat audit → Publish event `DosenDiganti` → Jadwal dan KRS, Presensi dan nilai, serta Tampilan mahasiswa.
