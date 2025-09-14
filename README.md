# ⚙️ Fuzzy Matching untuk Penggabungan Data dengan Python & R

![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)
![R](https://img.shields.io/badge/R-4.0+-purple.svg)
![License](https://img.shields.io/badge/License-MIT-red.svg)
![Affiliation](https://img.shields.io/badge/Affiliation-UNTIRTA-orange.svg)

Repositori ini menyajikan alur kerja komprehensif untuk proses pembersihan dan penggabungan data (*data cleaning & record linkage*) menggunakan teknik **Fuzzy Matching**. Ini adalah pendekatan yang sangat kuat untuk mengatasi masalah data yang tidak konsisten, di mana join standar atau VLOOKUP gagal karena adanya kesalahan ketik, singkatan, atau format yang berbeda. Proyek ini mendemonstrasikan implementasi paralel di **Python** dan **R** untuk menghubungkan tiga sumber data penjualan yang tidak sempurna.

## ✨ Fitur Utama

- **🎯 Penggabungan Multi-Tabel**: Menghubungkan data transaksi utama dengan dua tabel master yang terpisah (profil pelanggan dan master produk).
- **⚙️ Validasi Multi-Faktor**: Menerapkan proses verifikasi dua tahap yang canggih untuk pencocokan pelanggan, yaitu memvalidasi berdasarkan kemiripan **nama** DAN **alamat** untuk akurasi yang lebih tinggi.
- **🔄 Kalibrasi Threshold Interaktif**: Menunjukkan cara menganalisis skor kemiripan (Python) atau jarak (R) dari data perantara untuk menentukan nilai ambang batas (*threshold*) yang paling sesuai.
- **🛠️ Implementasi Berdampingan**:
    - **Python**: Menggunakan pustaka `thefuzz` untuk menghitung skor kemiripan berbasis *token*, ideal untuk menangani singkatan dan urutan kata yang berbeda.
    - **R**: Menggunakan pustaka `fuzzyjoin` dan `stringdist` dengan metode **Jaro-Winkler**, yang sangat efektif untuk string pendek seperti nama.
- **📊 Laporan Reproducibel**: Menyediakan file R Markdown (`.Rmd`) yang dapat menghasilkan laporan analisis lengkap yang siap untuk dikonsumsi di GitHub (`.md`).
- **💡 Analisis Komparatif**: Memungkinkan perbandingan metodologi, penanganan threshold, dan hasil antara pendekatan berbasis skor di Python dan pendekatan berbasis jarak di R.

## 🔧 Tumpukan Teknologi (Tech Stack)

- **Bahasa**: `Python 3.7+`, `R 4.0+`
- **Pustaka Python**: `pandas`, `thefuzz`, `python-Levenshtein`, `openpyxl`
- **Pustaka R**: `fuzzyjoin`, `stringdist`, `dplyr`, `readxl`, `knitr`, `rmarkdown`
- **Lingkungan**: `Jupyter Notebook / Lab`, `RStudio`

## 🚀 Cara Memulai

### Prasyarat
- **Perangkat Lunak**: Python 3.7+, R 4.0+, Jupyter Notebook/Lab, dan RStudio.
- **File Data**: Pastikan file `Contoh Data Penjualan Toko.xlsx` berada di dalam direktori proyek Anda.

### Instalasi & Penggunaan

1.  **Unduh Repositori**: *Clone* atau unduh proyek ini ke komputer Anda.

2.  **Untuk Pengguna Python**:
    - Buka terminal atau command prompt, lalu instal paket yang diperlukan:
      ```bash
      pip install pandas openpyxl thefuzz python-Levenshtein jupyterlab
      ```
    - Jalankan Jupyter Lab/Notebook dan buka file Python (`.ipynb`).
    - **Penting**: Ubah `file_path` di dalam skrip agar sesuai dengan lokasi file `Contoh Data Penjualan Toko.xlsx` di komputer Anda.

3.  **Untuk Pengguna R**:
    - Buka RStudio, lalu instal paket yang diperlukan dengan menjalankan perintah berikut di *console*:
      ```r
      install.packages(c("fuzzyjoin", "stringdist", "dplyr", "readxl", "knitr", "rmarkdown"))
      ```
    - Buka file `.Rmd` dan tekan "Knit" untuk menghasilkan laporan Markdown (`.md`).
    - **Penting**: Ubah `file_path` di dalam skrip agar sesuai dengan lokasi file `Contoh Data Penjualan Toko.xlsx` di komputer Anda.

4.  **Jalankan Analisis**: Eksekusi semua sel (Python) atau *chunks* (R) untuk mereplikasi seluruh alur kerja.

## 📊 Metrik Hasil & Ringkasan Output Fuzzy Matching

Fokus evaluasi adalah pada skor/jarak yang dihasilkan dan bagaimana kita menggunakan threshold untuk memfilter kecocokan yang valid.

| Metrik                 | Deskripsi                                                                                                 | Contoh Nilai (Python) | Contoh Nilai (R) |
| ---------------------- | --------------------------------------------------------------------------------------------------------- | --------------------- | ---------------- |
| `Skor Kemiripan`       | Dihasilkan oleh `thefuzz` (0-100). Semakin tinggi nilainya, semakin mirip. Kita mencari nilai **di atas** threshold. | `88` (untuk nama)     | -                |
| `Jarak (Distance)`     | Dihasilkan oleh `stringdist` (0-1). Semakin rendah nilainya, semakin mirip. Kita mencari nilai **di bawah** threshold. | -                     | `0.119` (untuk nama) |
| `Threshold (Ambang)`   | Batas nilai yang kita tentukan untuk memutuskan apakah dua entri dianggap cocok. Kalibrasi adalah kunci.      | `70` (untuk nama)     | `0.25` (untuk nama) |
| `Jumlah Record Cocok`  | Jumlah baris akhir yang berhasil dihubungkan setelah melalui semua tahap filter.                            | `5 dari 5`            | `5 dari 5`         |


## ⚖️ Lisensi & Ketentuan Penggunaan

- **Hak Cipta**: © 2025 Ferdian Bangkit Wijaya - Seluruh Hak Dilindungi.
- **Penggunaan Akademik**: Diizinkan secara bebas untuk keperluan penelitian, tesis, dan pendidikan non-komersial dengan atribusi.
- **Penggunaan Komersial**: Memerlukan izin tertulis dari pengembang.

## 👨‍💻 Author

- **Ferdian Bangkit Wijaya**
- Program Studi Statististika, Fakultas Teknik
- **Universitas Sultan Ageng Tirtayasa (UNTIRTA)**
- Banten, Indonesia 🇮🇩

---

> Proyek ini dirancang sebagai panduan praktis untuk salah satu tugas paling fundamental dalam data sains: mengubah data yang berantakan dan terisolasi menjadi sebuah dataset yang bersih, terhubung, dan kaya akan informasi.
