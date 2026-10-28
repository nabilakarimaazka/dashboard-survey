# Project Brief: Survey Data Pipeline & Dashboard

Proyek: Project Bike, survei konsumen sepeda motor dari AHM
Penyusun: Nabila Karima Azka
Status: draft v1

## 1. Latar belakang dan masalah

Setiap bulan tim data analyst menerima file hasil survei konsumen dari AHM dan harus mengolahnya menjadi insight untuk tim marketing. Saat ini belum ada standar pengolahan data, sehingga setiap periode data dibersihkan secara manual.

Kesulitan utamanya ada pada struktur file:

- Setiap opsi jawaban menjadi kolom sendiri. Satu file berisi 1.026 kolom, dan sekitar 64% kolom selalu kosong karena routing pertanyaan.
- Format jawaban tidak konsisten. Contohnya angka ditulis dengan satuan ("23 tahun"), rentang rupiah memakai pemisah campuran titik dan koma, dan nama model ditulis dengan format berbeda antar pertanyaan.
- File datang hampir setiap bulan dengan jadwal yang tidak pasti.

Dampaknya, pengolahan memakan waktu [isi: berapa jam/hari per periode], dan hasilnya berisiko berbeda tergantung siapa yang mengolah.

## 2. Tujuan

1. Data survei siap dianalisis segera setelah file diupload, tanpa proses manual.
2. Cara pengolahan sama di setiap periode dan terdokumentasi.
3. Tidak ada kolom yang hilang: seluruh 1.026 kolom tetap tersedia dalam format yang lebih rapi.
4. Dashboard menjawab pertanyaan bisnis utama marketing.
5. User dapat mengunduh data bersih untuk analisis lanjutan secara mandiri.

## 3. Pengguna dan kebutuhan

| Pengguna | Kebutuhan utama |
|---|---|
| GM Marketing | Ringkasan satu halaman berisi angka kunci |
| Manager Marketing | Perbandingan antar segmen, brand, dan area |
| Sales Analyst | Filter sampai detail, upload file baru, dan download data bersih |

## 4. Pertanyaan bisnis

| Kode | Pertanyaan | Sumber kolom | Catatan |
|---|---|---|---|
| a | Siapa pembeli tiap model atau segmen: gender, usia, pekerjaan, pendapatan, pengeluaran? | Q6, Q7, Q112, Q114, Q119, Q121, Q124 | Rentang rupiah perlu dikategorikan ulang |
| b | Jika model yang dibeli tidak tersedia, konsumen akan pindah ke model apa? | Q9, Q57 | Data lengkap dan formatnya konsisten |
| c | Atribut apa yang paling penting dan paling tidak penting bagi konsumen? | Q99–Q111 (MaxDiff) | Skor = (paling penting − paling tidak penting) ÷ jumlah tampil |
| d | Media apa yang dipakai untuk aktif mencari info, dan info apa yang dicari sebelum membeli? | Q42, Q43 | Isian "Lainnya, sebutkan" tidak ter-export; Q43 isian bebas perlu dikategorikan |
| e | Seberapa besar kemauan konsumen merekomendasikan modelnya (NPS)? | Q69 | Skala 0–10 atau 1–10 perlu dipastikan |
| f | Untuk apa motor dipakai di hari kerja dan hari libur? | Q63, Q65 | Data lengkap |
| g | Konsumen pindah dari model apa ke model apa (brand switching)? | Q29, Q9 | 74% responden adalah pembeli motor pertama, sehingga sampel kecil |
| h | Topik apa yang diminati pembeli tiap segmen? | Q72 | Q70–Q71 dan Q73–Q78 hampir kosong, tidak dipakai |

Catatan: sampel per model kecil (contoh periode Q1-2026: 84 responden). Perbandingan disajikan di level segmen, brand, atau area, dan setiap grafik menampilkan jumlah responden (n).

## 5. Sumber data

| Item | Keterangan |
|---|---|
| Sumber | Hasil survei dari AHM (ekspor SurveyMonkey) |
| Format | Excel (.xlsx), data di sheet RAW |
| Frekuensi | Hampir setiap bulan, jadwal tidak tetap |
| Struktur | 1 baris = 1 responden, 1.026 kolom |
| Kolom tambahan | Validity, Area, MD, MC, Q-Data (diisi tim survei) |
| Periode | Q-Data = gelombang berdasarkan kuartal pembelian, bukan bulan survei |

## 6. Ruang lingkup

Termasuk:

- Upload file melalui aplikasi web
- Validasi file (format, kolom wajib, ID duplikat)
- Penyimpanan data mentah dan riwayat upload
- Transformasi seluruh kolom berdasarkan codebook
- Cleaning tambahan untuk kolom yang dipakai dashboard
- Dashboard untuk 8 pertanyaan bisnis
- Fitur extract data bersih beserta codebook

Tidak termasuk:

- Sheet target dan achievement (Ach A/B/C)
- Analisis statistik lanjutan atau model prediksi
- Pengumpulan data survei

## 7. Gambaran solusi

Alur data: upload → validasi → bronze (data mentah) → silver (data rapi berbasis codebook) → gold (tabel siap dashboard) → dashboard dan extract.

| Komponen | Tools |
|---|---|
| Bahasa | Python |
| Database | DuckDB |
| Transformasi | Python / SQL (dbt opsional) |
| Aplikasi dan dashboard | Streamlit |
| Version control | Git dan GitHub |

## 8. Privasi data

- Data mentah survei tidak disimpan di GitHub.
- Kolom data pribadi (IP address, email, nama responden) tetap tersimpan di bronze, tetapi disamarkan (hash) di silver, gold, dashboard, dan file extract. [usulan, menunggu keputusan]
- Aplikasi dijalankan di server internal, bukan hosting publik.
- Versi portofolio memakai data dummy dengan struktur yang sama.

## 9. Keputusan yang masih terbuka

1. Penanganan kolom data pribadi di file extract
2. Format file extract yang paling sesuai untuk Sales Analyst (usulan: Excel dengan sheet data_wide, data_long, codebook)
3. Pertanyaan g tetap dipakai dengan catatan sampel kecil, atau diganti persentase pembeli motor pertama per segmen
4. Isian "Lainnya, sebutkan" Q41–Q42 yang tidak ter-export (konfirmasi ke tim survei AHM)
5. Skala Q69 (0–10 atau 1–10)

## 10. Kriteria selesai

- File baru dapat diupload dan diproses tanpa langkah manual.
- File yang sama tidak terproses dua kali; file revisi menggantikan data lama.
- File yang tidak valid ditolak dengan pesan error yang jelas.
- Seluruh 1.026 kolom tersedia di hasil extract, termasuk kolom baru yang belum ada di codebook.
- Dashboard menjawab pertanyaan a–h dan menampilkan jumlah responden.
- Ada dokumentasi: README, codebook, dan panduan upload.