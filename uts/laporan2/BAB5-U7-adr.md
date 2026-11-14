# U7 – Architecture Decision Record (ADR)

## 1. Pendahuluan

Architecture Decision Record (ADR) merupakan dokumentasi keputusan penting dalam perancangan arsitektur perangkat lunak. ADR digunakan untuk mencatat konteks permasalahan, keputusan yang dipilih, alternatif yang dipertimbangkan, serta konsekuensi dari keputusan tersebut.

Pada perancangan subsistem **Academic Management SIAKAD UMI**, terdapat dua keputusan arsitektur utama yang berfokus pada penyebaran perubahan dosen pengampu serta pengendalian dan pencatatan perubahan data akademik.

Kedua keputusan ini dirancang untuk mengatasi permasalahan pergantian dosen pengampu secara mendadak yang menyebabkan ketidaksinkronan data dan keterlambatan informasi kepada mahasiswa.

---

## 2. ADR-01: Penyebaran Perubahan Dosen melalui Event

| Atribut | Keterangan |
|---|---|
| ID | ADR-01 |
| Judul | Penyebaran Perubahan Dosen melalui Event |
| Status | Diterima (Accepted) |
| Kategori | Modifiabilitas dan Konsistensi Data |
| Pola terkait | Layered + Event-Driven |

### 2.1 Konteks

Data dosen pengampu digunakan oleh beberapa modul dalam subsistem Academic Management, seperti data kelas, jadwal, KRS, presensi, dan nilai.

Ketika terjadi pergantian dosen pengampu, perubahan tersebut berpotensi tidak langsung tersebar ke seluruh modul. Kondisi ini dapat menyebabkan perbedaan informasi dosen antara jadwal, presensi, dan nilai.

Oleh karena itu, diperlukan mekanisme penyebaran perubahan yang dapat memperbarui informasi pada seluruh modul terkait tanpa harus melakukan perubahan secara manual satu per satu.

### 2.2 Keputusan

Menggunakan mekanisme **Publish-Subscribe (Event-Driven)** melalui event bus internal.

Ketika admin akademik melakukan perubahan dosen pengampu, sistem akan:

1. Memvalidasi perubahan dan menyimpannya ke database utama.
2. Menerbitkan event bernama `DosenDiganti`.
3. Mengirimkan event kepada modul yang berlangganan.
4. Memperbarui data pada modul jadwal, KRS, presensi, dan nilai.
5. Menampilkan informasi dosen terbaru ketika mahasiswa membuka kembali SIAKAD.

Target sinkronisasi adalah maksimal 2 menit dengan 0 perbedaan data setelah proses sinkronisasi selesai.

### 2.3 Alternatif yang Dipertimbangkan

| Alternatif | Penjelasan | Alasan Tidak Dipilih |
|---|---|---|
| Panggilan langsung antarmodul | Modul saling memanggil untuk memperbarui data | Ketergantungan antarmodul lebih tinggi |
| Microservices | Setiap modul berjalan sebagai layanan terpisah | Kompleksitas dan biaya pengelolaan lebih tinggi |
| Publish-Subscribe | Perubahan disebarkan melalui event | Dipilih karena modul lebih longgar dan mudah dikembangkan |

### 2.4 Konsekuensi

**Kelebihan:**
- Ketergantungan antarmodul lebih rendah (loose coupling).
- Perubahan dosen dapat disebarkan secara otomatis.
- Memudahkan penambahan modul baru.
- Mendukung konsistensi data antarbagian sistem.

**Kekurangan:**
- Konsistensi data tidak selalu terjadi secara langsung (eventual consistency).
- Terdapat jeda sinkronisasi dengan target maksimal 2 menit.
- Proses debugging lebih sulit karena komunikasi melalui event.
- Sistem harus memastikan event tidak hilang atau gagal diproses.

### 2.5 Kriteria Keberhasilan

| Indikator | Target |
|---|---|
| Waktu sinkronisasi | ≤ 2 menit |
| Perbedaan data setelah sinkronisasi | 0 |
| Modul terkait menerima perubahan | Seluruh modul berlangganan |

---

## 3. ADR-02: Audit Trail dan Otorisasi Perubahan Dosen

| Atribut | Keterangan |
|---|---|
| ID | ADR-02 |
| Judul | Audit Trail dan Otorisasi Perubahan Dosen |
| Status | Diterima (Accepted) |
| Kategori | Keamanan dan Akuntabilitas |
| Taktik terkait | Audit Trail dan RBAC |

### 3.1 Konteks

Pergantian dosen pengampu dapat menimbulkan permasalahan apabila tidak terdapat pencatatan mengenai siapa yang melakukan perubahan, kapan perubahan dilakukan, serta data apa yang diubah.

Selain itu, perubahan data dosen pengampu harus dibatasi agar tidak dapat dilakukan oleh pengguna yang tidak memiliki kewenangan.

Diperlukan mekanisme pengendalian akses dan pencatatan riwayat perubahan untuk memastikan setiap perubahan dapat dipertanggungjawabkan.

### 3.2 Keputusan

Menggunakan mekanisme **Role-Based Access Control (RBAC)** yang dilengkapi persetujuan ketua jurusan dan Audit Trail.

Ketentuan penerapannya:

1. Hanya admin akademik yang berwenang mengajukan dan melakukan perubahan dosen pengampu.
2. Perubahan harus memperoleh persetujuan ketua jurusan.
3. Setiap perubahan dicatat dalam audit log.
4. Catatan perubahan mencakup identitas pengguna, waktu perubahan, kelas yang diubah, dosen lama, dan dosen baru.
5. Riwayat perubahan dapat ditelusuri kembali melalui sistem.

Target keamanan adalah 100% perubahan tercatat dan 0 perubahan tanpa izin.

### 3.3 Alternatif yang Dipertimbangkan

| Alternatif | Penjelasan | Alasan Tidak Dipilih |
|---|---|---|
| Dosen atau prodi mengubah langsung | Perubahan dapat dilakukan oleh lebih banyak pihak | Risiko perubahan tanpa kewenangan lebih tinggi |
| Log hanya di file server | Riwayat disimpan di luar mekanisme audit sistem | Penelusuran perubahan kurang terintegrasi |
| RBAC + persetujuan + Audit Trail | Akses dibatasi dan perubahan dicatat dalam sistem | Dipilih karena lebih mudah dikendalikan dan diaudit |

### 3.4 Konsekuensi

**Kelebihan:**
- Setiap perubahan memiliki jejak audit.
- Identitas pengguna dan waktu perubahan dapat diketahui.
- Mengurangi risiko perubahan data tanpa izin.
- Meningkatkan akuntabilitas pengelolaan data akademik.

**Kekurangan:**
- Proses perubahan membutuhkan waktu lebih lama karena persetujuan.
- Penyimpanan audit log menambah kebutuhan kapasitas database.
- Proses pencatatan menambah beban penulisan data.

### 3.5 Kriteria Keberhasilan

| Indikator | Target |
|---|---|
| Perubahan yang tercatat | 100% |
| Perubahan tanpa izin | 0 |
| Informasi audit | Pengguna, waktu, kelas, dosen lama, dosen baru |

---

## 4. Hubungan ADR dengan Perancangan Arsitektur

Kedua ADR saling melengkapi dalam menyelesaikan permasalahan pergantian dosen pengampu pada SIAKAD UMI.

| Keputusan | Fokus | Atribut Kualitas |
|---|---|---|
| ADR-01 | Penyebaran perubahan melalui event | Modifiabilitas |
| ADR-02 | Pembatasan akses dan pencatatan perubahan | Keamanan |

ADR-01 memastikan perubahan dosen dapat disebarkan ke modul akademik terkait, sedangkan ADR-02 memastikan perubahan tersebut dilakukan oleh pihak yang berwenang dan memiliki riwayat yang dapat ditelusuri.

Kedua keputusan diterapkan dalam rancangan pola Layered + Event-Driven, dengan database utama sebagai sumber data dan event bus internal sebagai mekanisme penyebaran perubahan.

## 5. Kesimpulan

Berdasarkan kedua keputusan arsitektur tersebut, rancangan subsistem Academic Management SIAKAD UMI diharapkan mampu mengurangi ketidaksinkronan data akibat pergantian dosen pengampu.

Mekanisme Publish-Subscribe mendukung penyebaran perubahan secara otomatis, sedangkan RBAC dan Audit Trail memastikan perubahan dapat dikendalikan dan dipertanggungjawabkan.

Namun, seluruh target masih berupa kriteria desain dan belum dinyatakan tercapai melalui pengujian prototipe. Validasi lebih lanjut diperlukan untuk memastikan mekanisme sinkronisasi, otorisasi, dan pencatatan berjalan sesuai target.
