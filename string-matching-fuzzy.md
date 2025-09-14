Analisis Fuzzy Matching untuk Menghubungkan Data Penjualan
================
Ferdian Bangkit Wijaya
14 September 2025

- [Latar Belakang](#latar-belakang)
  - [1. Memuat Data dari Excel](#1-memuat-data-dari-excel)
  - [2. Eksplorasi Data Awal](#2-eksplorasi-data-awal)
  - [3. Proses Fuzzy Matching](#3-proses-fuzzy-matching)
    - [Tahap 1: Menghubungkan Transaksi dengan Profil
      Pelanggan](#tahap-1-menghubungkan-transaksi-dengan-profil-pelanggan)
    - [Tahap 2: Menghubungkan dengan Master
      Produk](#tahap-2-menghubungkan-dengan-master-produk)
  - [4. Hasil Akhir](#4-hasil-akhir)

# Latar Belakang

Laporan ini mendemonstrasikan proses pembersihan dan penggabungan data
dari tiga sumber yang berbeda menggunakan teknik **Fuzzy Matching**.
Skenario yang diangkat adalah menghubungkan data transaksi penjualan
yang tidak standar dengan data master pelanggan dan produk yang sudah
terstruktur.

**Tujuan:** Menghasilkan satu tabel data (dataset) yang bersih, utuh,
dan diperkaya dengan ID pelanggan serta ID produk yang valid, sehingga
siap untuk dianalisis lebih lanjut.

## 1. Memuat Data dari Excel

Data mentah bersumber dari sebuah file Excel
(`Contoh Data Penjualan Toko.xlsx`) yang terdiri dari tiga sheet
terpisah.

``` r
# Mendefinisikan path ke file Excel
file_path <- "C:/Users/user/OneDrive - untirta.ac.id/UNTIRTA/Bahan Ajar/Pengantar Data Sains/25-26/Contoh Data Penjualan Toko.xlsx"

# Membaca setiap sheet menjadi data frame
df_profil <- read_excel(file_path, sheet = "profil_pelanggan")
df_transaksi <- read_excel(file_path, sheet = "transaksi_pembelian")
df_produk <- read_excel(file_path, sheet = "master_produk")
```

## 2. Eksplorasi Data Awal

Berikut adalah pratinjau singkat dari ketiga tabel yang telah dimuat.

``` r
# Menampilkan 5 baris pertama dari setiap tabel
kable(head(df_profil), caption = "Tabel Master Profil Pelanggan")
```

| id_profil | nama_lengkap | alamat_domisili |
|:---|:---|:---|
| P001 | Ferdian Bangkit Wijaya | Jl. Merpati Putih 5 Blok C-21, Taman Cilegon |
| P002 | Cynthia Dewi Lestari | Komp. Mawar Melati Kavling 8, Sukajadi, Bandung |
| P003 | Ahmad Zaelani | Apartemen Cempaka Tower B Lantai 11 No. 1108, Jakarta Pusat |
| P004 | Weksi Budiaji | Perumahan Griya Serang Indah Blok E4 Nomor 12 |

Tabel Master Profil Pelanggan

``` r
kable(head(df_transaksi), caption = "Tabel Transaksi Pembelian (Data Mentah)")
```

| id_transaksi | nama_pelanggan | alamat_pengiriman | nama_barang | jumlah |
|:---|:---|:---|:---|---:|
| TRX01 | Ferdian B. | Jl. Merpati Putih V Blok C21, Cilegon | Kopi Gula Aren 1 Liter | 1 |
| TRX02 | Weksi B. | Griya Serang Indah E4 No. 12 | Roti Sobek Coklat | 2 |
| TRX03 | Chintia Lestari | Komp. Mawar Melati Kav 8 Sukajadi | Donat Klasik 6pcs | 1 |
| TRX04 | Ahmad Zaelany | Apt. Cempaka Tower B/11/1108 Jakpus | Kopi Susu Aren 1L | 1 |
| TRX05 | Weksi Budiaji | Perum. GSI Blok E4 No. 12, Serang | Teh Melati 500ml | 3 |

Tabel Transaksi Pembelian (Data Mentah)

``` r
kable(head(df_produk), caption = "Tabel Master Produk")
```

| id_produk | nama_produk_resmi       | harga |
|:----------|:------------------------|------:|
| PROD-A    | Kopi Susu Gula Aren 1L  | 55000 |
| PROD-B    | Roti Sobek Cokelat Keju | 25000 |
| PROD-C    | Donat Gula Klasik Isi 6 | 45000 |
| PROD-D    | Teh Melati Segar 500ml  | 15000 |

Tabel Master Produk

## 3. Proses Fuzzy Matching

### Tahap 1: Menghubungkan Transaksi dengan Profil Pelanggan

Kita menggunakan validasi ganda: mencocokkan berdasarkan kemiripan
**nama** dan divalidasi dengan kemiripan **alamat**.

``` r
# Fuzzy join berdasarkan nama
df_intermediate <- stringdist_join(
  df_transaksi, df_profil,
  by = c("nama_pelanggan" = "nama_lengkap"),
  mode = "left", method = "jw", max_dist = 0.3, distance_col = "jarak_nama"
) %>%
  group_by(id_transaksi) %>%
  slice_min(order_by = jarak_nama, n = 1) %>%
  ungroup()

# Hitung jarak alamat
df_intermediate <- df_intermediate %>%
  rowwise() %>%
  mutate(
    jarak_alamat = ifelse(!is.na(alamat_domisili), 
                           stringdist(alamat_pengiriman, alamat_domisili, method = "jw"), 
                           NA)
  ) %>%
  ungroup()
```

#### Analisis Jarak untuk Kalibrasi Threshold

Tabel di bawah ini menunjukkan skor jarak untuk semua pasangan, yang
membantu kita menentukan nilai ambang batas (threshold) yang realistis.

``` r
tabel_analisis_jarak <- select(df_intermediate, id_transaksi, nama_pelanggan, nama_lengkap, jarak_nama, jarak_alamat)
kable(tabel_analisis_jarak, caption = "Analisis Jarak Nama & Alamat (Sebelum Filter)")
```

| id_transaksi | nama_pelanggan  | nama_lengkap           | jarak_nama | jarak_alamat |
|:-------------|:----------------|:-----------------------|-----------:|-------------:|
| TRX01        | Ferdian B.      | Ferdian Bangkit Wijaya |  0.2303030 |    0.1020225 |
| TRX02        | Weksi B.        | Weksi Budiaji          |  0.1955128 |    0.2625220 |
| TRX03        | Chintia Lestari | Cynthia Dewi Lestari   |  0.1833333 |    0.1548463 |
| TRX04        | Ahmad Zaelany   | Ahmad Zaelani          |  0.0512821 |    0.3145669 |
| TRX05        | Weksi Budiaji   | Weksi Budiaji          |  0.0000000 |    0.2681180 |

Analisis Jarak Nama & Alamat (Sebelum Filter)

Berdasarkan tabel analisis di atas, kita menetapkan threshold berikut.

``` r
JARAK_NAMA_MAX <- 0.25
JARAK_ALAMAT_MAX <- 0.4
df_linked_profil <- df_intermediate %>%
  filter(jarak_nama <= JARAK_NAMA_MAX & jarak_alamat <= JARAK_ALAMAT_MAX)
```

### Tahap 2: Menghubungkan dengan Master Produk

Kita lanjutkan dengan mencocokkan `nama_barang` dengan
`nama_produk_resmi`.

``` r
# Fuzzy join dengan data produk
df_final <- stringdist_join(
  df_linked_profil, df_produk,
  by = c("nama_barang" = "nama_produk_resmi"),
  mode = "left", method = "jw", max_dist = 0.3, distance_col = "jarak_produk"
) %>%
  group_by(id_transaksi) %>%
  slice_min(order_by = jarak_produk, n = 1) %>%
  ungroup()
```

## 4. Hasil Akhir

Berikut adalah tabel final setelah melalui seluruh proses pembersihan
dan penggabungan.

``` r
# Memilih dan menata ulang kolom untuk hasil akhir
final_columns <- c(
  'id_transaksi', 'id_profil', 'nama_pelanggan', 'nama_lengkap',
  'id_produk', 'nama_barang', 'nama_produk_resmi',
  'jumlah', 'harga', 'jarak_nama', 'jarak_alamat', 'jarak_produk'
)
final_columns_exist <- intersect(final_columns, names(df_final))

# Menampilkan tabel final dengan format yang rapi
kable(df_final[final_columns_exist], caption = "Tabel Final Hasil Fuzzy Matching")
```

| id_transaksi | id_profil | nama_pelanggan | nama_lengkap | id_produk | nama_barang | nama_produk_resmi | jumlah | harga | jarak_nama | jarak_alamat | jarak_produk |
|:---|:---|:---|:---|:---|:---|:---|---:|---:|---:|---:|---:|
| TRX01 | P001 | Ferdian B. | Ferdian Bangkit Wijaya | PROD-A | Kopi Gula Aren 1 Liter | Kopi Susu Gula Aren 1L | 1 | 55000 | 0.2303030 | 0.1020225 | 0.2323232 |
| TRX02 | P004 | Weksi B. | Weksi Budiaji | PROD-B | Roti Sobek Coklat | Roti Sobek Cokelat Keju | 2 | 25000 | 0.1955128 | 0.2625220 | 0.0869565 |
| TRX03 | P002 | Chintia Lestari | Cynthia Dewi Lestari | PROD-C | Donat Klasik 6pcs | Donat Gula Klasik Isi 6 | 1 | 45000 | 0.1833333 | 0.1548463 | 0.2551577 |
| TRX04 | P003 | Ahmad Zaelany | Ahmad Zaelani | PROD-A | Kopi Susu Aren 1L | Kopi Susu Gula Aren 1L | 1 | 55000 | 0.0512821 | 0.3145669 | 0.1247772 |
| TRX05 | P004 | Weksi Budiaji | Weksi Budiaji | PROD-D | Teh Melati 500ml | Teh Melati Segar 500ml | 3 | 15000 | 0.0000000 | 0.2681180 | 0.0909091 |

Tabel Final Hasil Fuzzy Matching
