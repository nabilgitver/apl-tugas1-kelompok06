# A2. Stakeholder dan Peta Power/Interest

## Stakeholder register

| No | Stakeholder | Kelompok | Kepentingan terhadap layanan MSIB | Power | Interest | Strategi |
|---|---|---|---|---|---|---|
| 1 | Mahasiswa peserta MSIB | Pengguna | Mendaftar, mengisi logbook, mengikuti bimbingan, dan menerima hasil penilaian | Rendah | Tinggi | Keep informed |
| 2 | Dosen pembimbing | Pengguna | Memantau logbook, memberi bimbingan, dan memberi penilaian | Sedang | Tinggi | Keep informed |
| 3 | Staf unit akademik atau koordinator MSIB | Pemilik layanan | Mengelola pendaftaran, penempatan mitra, dan rekap nilai | Tinggi | Tinggi | Manage closely |
| 4 | Pimpinan fakultas atau program studi | Sponsor | Menyetujui kebijakan dan anggaran, serta menjaga mutu program | Tinggi | Sedang | Keep satisfied |
| 5 | Mitra industri (penyelenggara magang dan pembimbing lapangan) | Mitra eksternal | Menerima peserta, memantau kegiatan, dan memberi penilaian lapangan | Sedang | Sedang | Manage closely |
| 6 | Tim pengembang dan pengelola TI | Teknis | Membangun, mengoperasikan, dan merawat sistem | Sedang | Tinggi | Manage closely |
| 7 | Penyelenggara program MSIB tingkat nasional | Regulator | Menetapkan aturan program dan data yang harus dilaporkan | Tinggi | Rendah | Keep satisfied |
| 8 | Unit penjaminan mutu | Regulator | Memeriksa kesesuaian proses dan bukti penilaian | Sedang | Rendah | Monitor |

Kelima kelompok yang dipakai: pengguna, pemilik dan sponsor, mitra eksternal, teknis, dan regulator.

## Peta power/interest

```mermaid
%%{init: {"theme": "base", "themeVariables": {"quadrant1Fill": "#7C9CFF", "quadrant2Fill": "#F44336", "quadrant3Fill": "#C4C4C4", "quadrant4Fill": "#4CFF8A", "quadrant1TextFill": "#000000", "quadrant2TextFill": "#000000", "quadrant3TextFill": "#000000", "quadrant4TextFill": "#000000", "quadrantPointFill": "#000000", "quadrantPointTextFill": "#000000", "quadrantXAxisTextFill": "#000000", "quadrantYAxisTextFill": "#000000", "quadrantTitleFill": "#000000", "quadrantInternalBorderStrokeFill": "#000000", "quadrantExternalBorderStrokeFill": "#000000"}, "quadrantChart": {"chartWidth": 700, "chartHeight": 600, "pointRadius": 6, "pointLabelFontSize": 13}}}%%
quadrantChart
    title Peta Power dan Interest layanan MSIB
    x-axis Kepentingan rendah --> Kepentingan tinggi
    y-axis Pengaruh rendah --> Pengaruh tinggi
    quadrant-1 Manage Closely
    quadrant-2 Keep Satisfied
    quadrant-3 Monitor
    quadrant-4 Keep Informed
    Pimpinan fakultas: [0.17, 0.82]
    Penyelenggara MSIB nasional: [0.28, 0.68]
    Staf unit akademik: [0.75, 0.90]
    Tim pengembang: [0.85, 0.76]
    Mitra industri: [0.68, 0.62]
    Unit penjaminan mutu: [0.27, 0.30]
    Dosen pembimbing: [0.68, 0.34]
    Mahasiswa peserta: [0.85, 0.18]
```

Gambar PNG untuk laporan ada di `diagram/A2-peta-power-interest.png` (sumber: `diagram/A2-peta-power-interest.drawio`).
