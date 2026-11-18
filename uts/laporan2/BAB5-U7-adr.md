# U7 – Architecture Decision Record (ADR)

## 1. Pendahuluan

Architecture Decision Record (ADR) merupakan dokumentasi yang mencatat keputusan penting dalam perancangan arsitektur perangkat lunak, termasuk konteks permasalahan, keputusan yang dipilih, alternatif yang dipertimbangkan, serta konsekuensi dari keputusan tersebut.

Pada perancangan arsitektur subsistem **Academic Management SIAKAD UMI**, terdapat dua keputusan arsitektur utama yang berfokus pada penyebaran perubahan dosen pengampu serta pengendalian dan pencatatan perubahan data akademik.

Kedua keputusan tersebut dirancang untuk mengatasi permasalahan penggantian dosen pengampu menjelang perkuliahan perdana yang menyebabkan perubahan data lambat tersebar ke seluruh modul dan mahasiswa tidak mendapatkan notifikasi langsung.

---

## 2. ADR-01: Penyebaran Perubahan Dosen melalui Event

| Atribut | Keterangan |
|---|---|
| ID | ADR-01 |
| Judul | Penyebaran Perubahan Dosen melalui Event |
| Status | Diterima (Accepted) |
| Fokus | Modifiabilitas dan sinkronisasi data |
| Pola terkait | Layered + Event-Driven |
| Taktik | Publish-Subscribe (Event) |

### 2.1 Konteks

Data dosen pengampu digunakan oleh beberapa modul dalam subsistem Academic Management, seperti modul jadwal kuliah, KRS, presensi, dan nilai.

Ketika terjadi penggantian dosen pengampu menjelang perkuliahan perdana, perubahan data berpotensi lambat tersebar ke seluruh modul. Hal ini dapat menyebabkan ketidaksesuaian informasi dosen antara jadwal, presensi, dan nilai.

Diperlukan mekanisme yang dapat menyebarkan perubahan secara otomatis sehingga seluruh modul terkait memperoleh informasi dosen pengampu terbaru.

### 2.2 Keputusan

Memilih mekanisme **Publish-Subscribe (Event-Driven)** menggunakan event bus internal.

Ketika admin akademik melakukan penggantian dosen pengampu yang telah memperoleh persetujuan, sistem akan menjalankan proses berikut:

1. Menyimpan perubahan dosen pengampu ke database utama.
2. Mencatat perubahan ke dalam audit log.
3. Menerbitkan event bernama `DosenDiganti`.
4. Menyebarkan event kepada modul yang berlangganan.
5. Memperbarui informasi pada modul jadwal, KRS, presensi, dan nilai.
6. Menampilkan informasi dosen terbaru ketika mahasiswa membuka kembali SIAKAD.

Target waktu sinkronisasi adalah maksimal 2 menit dengan 0 perbedaan data setelah proses sinkronisasi selesai.

### 2.3 Alternatif yang Dipertimbangkan

| Alternatif | Penjelasan | Alasan Tidak Dipilih |
|---|---|---|
| Panggilan langsung antarmodul | Modul saling memanggil untuk memperbarui data | Ketergantungan antarmodul lebih tinggi |
| Microservices | Setiap modul berjalan sebagai layanan terpisah | Kompleksitas dan biaya pengelolaan lebih tinggi |
| Publish-Subscribe | Perubahan disebarkan melalui event bus internal | Dipilih karena modul lebih longgar dan mudah dikembangkan |

### 2.4 Konsekuensi

**Kelebihan:**

- Ketergantungan antarmodul lebih rendah (loose coupling).
- Perubahan dosen dapat disebarkan secara otomatis.
- Memudahkan penambahan modul baru.
- Mendukung sinkronisasi data antarbagian sistem.

**Kekurangan:**

- Konsistensi data tidak selalu terjadi secara langsung (eventual consistency).
- Terdapat jeda sinkronisasi dengan target maksimal 2 menit.
- Proses debugging lebih sulit karena komunikasi melalui event.
- Sistem harus memastikan event tidak hilang atau gagal diproses.
- Urutan pemrosesan event perlu diperhatikan agar data tetap konsisten.

### 2.5 Kriteria Keberhasilan

| Indikator | Target |
|---|---|
| Waktu sinkronisasi | ≤ 2 menit |
| Perbedaan data setelah sinkronisasi | 0 |
| Modul yang menerima perubahan | Seluruh modul terkait yang berlangganan |

---

## 3. ADR-02: Audit Trail dan Otorisasi Perubahan Dosen

| Atribut | Keterangan |
|---|---|
| ID | ADR-02 |
| Judul | Audit Trail dan Otorisasi Perubahan Dosen |
| Status | Diterima (Accepted) |
| Fokus | Keamanan dan akuntabilitas |
| Taktik | Audit Trail dan RBAC + Persetujuan |

### 3.1 Konteks

Penggantian dosen pengampu dapat menimbulkan permasalahan apabila tidak terdapat pencatatan mengenai siapa yang melakukan perubahan, kapan perubahan dilakukan, serta data apa yang diubah.

Selain itu, perubahan dosen pengampu harus dibatasi agar tidak dapat dilakukan oleh pengguna yang tidak memiliki kewenangan.

Oleh karena itu, diperlukan mekanisme otorisasi dan pencatatan riwayat perubahan agar setiap perubahan data akademik dapat dikendalikan dan dipertanggungjawabkan.

### 3.2 Keputusan

Memilih mekanisme **Role-Based Access Control (RBAC)** yang dilengkapi persetujuan ketua jurusan dan Audit Trail.

Ketentuan penerapannya adalah:

1. Hanya admin akademik yang berwenang melakukan penggantian dosen pengampu.
2. Perubahan harus memperoleh persetujuan ketua jurusan.
3. Setiap perubahan yang dilakukan dicatat ke dalam audit log.
4. Pencatatan mencakup identitas pengguna, waktu perubahan, kelas yang diubah, dosen lama, dan dosen baru.
5. Riwayat perubahan dapat ditelusuri kembali untuk kebutuhan pemeriksaan.

Target keamanan adalah 100% perubahan tercatat dan 0 perubahan tanpa izin.

### 3.3 Alternatif yang Dipertimbangkan

| Alternatif | Penjelasan | Alasan Tidak Dipilih |
|---|---|---|
| Dosen atau prodi mengubah langsung | Perubahan dapat dilakukan oleh pihak lain tanpa mekanisme persetujuan yang ditentukan | Risiko perubahan tanpa kewenangan lebih tinggi |
| Log hanya di file server | Riwayat perubahan disimpan di luar mekanisme audit sistem | Penelusuran perubahan kurang terintegrasi |
| RBAC + persetujuan + Audit Trail | Akses dibatasi dan seluruh perubahan dicatat dalam sistem | Dipilih karena lebih mudah dikendalikan dan diaudit |

### 3.4 Konsekuensi

**Kelebihan:**

- Setiap perubahan memiliki jejak audit.
- Identitas pengguna dan waktu perubahan dapat diketahui.
- Mengurangi risiko perubahan data tanpa izin.
- Meningkatkan akuntabilitas pengelolaan data akademik.
- Memudahkan penelusuran apabila terjadi kesalahan perubahan data.

**Kekurangan:**

- Proses perubahan membutuhkan waktu lebih lama karena harus melalui persetujuan ketua jurusan.
- Penyimpanan audit log menambah kebutuhan kapasitas database.
- Proses pencatatan menambah beban penulisan data.

### 3.5 Kriteria Keberhasilan

| Indikator | Target |
|---|---|
| Perubahan yang tercatat | 100% |
| Perubahan tanpa izin | 0 |
| Informasi audit | Pengguna, waktu, kelas, dosen lama, dosen baru |

---

## 4. Hubungan Kedua ADR dengan Arsitektur Sistem

Kedua keputusan arsitektur saling melengkapi dalam menangani permasalahan penggantian dosen pengampu pada subsistem Academic Management SIAKAD UMI.

| Keputusan | Fokus Utama | Atribut Kualitas |
|---|---|---|
| ADR-01 | Penyebaran perubahan dosen melalui event | Modifiabilitas |
| ADR-02 | Pembatasan akses dan pencatatan perubahan | Keamanan |

ADR-01 memastikan perubahan dosen pengampu dapat disebarkan ke modul jadwal, KRS, presensi, dan nilai melalui event bus internal.

Sementara itu, ADR-02 memastikan perubahan hanya dilakukan oleh pihak yang berwenang serta memiliki riwayat yang dapat ditelusuri.

Kedua keputusan tersebut diterapkan dalam rancangan pola **Layered + Event-Driven**, dengan database utama sebagai sumber data dan event bus internal sebagai mekanisme penyebaran perubahan.

## 5. Kesimpulan

Berdasarkan kedua keputusan arsitektur tersebut, rancangan subsistem Academic Management SIAKAD UMI diharapkan dapat mengurangi keterlambatan penyebaran perubahan dosen pengampu dan ketidaksinkronan data akademik.

Mekanisme Publish-Subscribe mendukung penyebaran perubahan secara otomatis, sedangkan RBAC dan Audit Trail memastikan perubahan dapat dikendalikan serta dipertanggungjawabkan.

Namun, rancangan ini belum mencakup notifikasi langsung kepada mahasiswa. Mahasiswa masih perlu membuka kembali SIAKAD untuk melihat informasi dosen terbaru.

Seluruh target yang ditetapkan masih berupa target rancangan dan belum divalidasi melalui pengujian prototipe.
