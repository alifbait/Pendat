# UTS Pengantar Data Mining
## Klasifikasi Kesuburan Tanah Menggunakan K-Nearest Neighbor (KNN)

| | |
|---|---|
| **Nama** | Alif Baiatur Ridhwan El Habibie |
| **NIM** | 240411100156 |
| **Mata Kuliah** | Pengantar Data Mining |
| **Tools** | KNIME Analytics Platform v5.11.0 |
| **Tanggal** | 22 April 2026 |

---
## 1. Pendahuluan

Kesuburan tanah merupakan faktor krusial dalam produktivitas pertanian. Penentuan kesuburan tanah secara konvensional membutuhkan waktu dan biaya yang besar. Pendekatan *data mining* menggunakan algoritma **K-Nearest Neighbor (KNN)** dapat mengklasifikasikan kesuburan tanah secara otomatis berdasarkan karakteristik kimia dan fisik tanah.

### Tujuan
1. Melakukan preprocessing dataset kesuburan tanah (missing value, encoding, normalisasi)
2. Membangun model klasifikasi KNN menggunakan KNIME Analytics Platform
3. Mengevaluasi performa model menggunakan Confusion Matrix dan Accuracy Statistics

### Workflow KNIME

![Workflow KNIME](../img/01_workflow.png)

Alur analisis terdiri dari **7 node utama** yang saling terhubung:

| No | Node | Fungsi |
|---|---|---|
| 1 | Excel Reader | Membaca dataset dari file `.xlsx` |
| 2 | Missing Value | Menangani nilai kosong (imputasi mean) |
| 3 | One to Many | Encoding fitur kategorikal → dummy variables |
| 4 | Normalizer | Normalisasi Min-Max pada fitur numerik |
| 5 | Column Filter | Memilih kolom fitur yang relevan |
| 6 | Table Partitioner | Membagi data training (80%) dan testing (20%) |
| 7 | K Nearest Neighbor | Klasifikasi dengan algoritma KNN (k=5) |
| 8 | Scorer | Evaluasi performa model |

---
## 2. Dataset

### 2.1 Deskripsi Dataset

Dataset yang digunakan adalah **`dataset_kesuburan_tanah_missing.xlsx`** yang berisi data sampel tanah dari berbagai lokasi.

| Atribut | Nilai |
|---|---|
| **Jumlah Data** | 2.000 baris |
| **Jumlah Kolom** | 12 kolom |
| **Fitur Numerik** | 9 fitur |
| **Fitur Kategorikal** | 1 fitur (Tekstur Tanah) |
| **Label** | Subur / Tidak Subur |
| **Missing Value** | Ada (pada beberapa kolom numerik) |

### 2.2 Fitur Dataset

| No | Nama Kolom | Tipe Data | Keterangan |
|---|---|---|---|
| 1 | ID | Number | Nomor identifikasi data |
| 2 | pH Tanah | Number | Tingkat keasaman tanah |
| 3 | N Total (%) | Number | Kandungan Nitrogen total |
| 4 | P Tersedia | Number | Kandungan Fosfor tersedia |
| 5 | K Tersedia | Number | Kandungan Kalium tersedia |
| 6 | C Organik (%) | Number | Kandungan Karbon Organik |
| 7 | KTK (meq/100g) | Number | Kapasitas Tukar Kation |
| 8 | Kejenuhan Basa | Number | Persentase kejenuhan basa |
| 9 | Tekstur Tanah | String | Jenis tekstur tanah |
| 10 | Kadar Air (%) | Number | Kandungan air dalam tanah |
| 11 | Bulk Density | Number | Kerapatan massa tanah |
| 12 | **Label** | String | **Subur / Tidak Subur** |

---
## 3. Tahapan Analisis (Step by Step)

### Step 1 – Excel Reader

Node **Excel Reader** digunakan untuk membaca dataset dari file Excel ke dalam KNIME.

![Excel Reader](../img/02_excel_reader.png)

**Konfigurasi:**
- **Source:** `C:\Users\Alif\dataset_kesuburan_tanah_missing.xlsx`
- **Mode:** File
- **Select sheet:** First sheet with data

**Hasil Output:**
- **Rows:** 2.000 baris
- **Columns:** 12 kolom

Dataset berhasil dimuat dengan 10 fitur agronomis, 1 kolom ID, dan 1 kolom Label. Terdapat **missing value** yang ditandai dengan ikon lingkaran merah pada beberapa sel (terlihat pada kolom N Total dan C Organik di Row2).

### Step 2 – Missing Value

Node **Missing Value** digunakan untuk menangani data yang hilang (*missing value*) dalam dataset.

![Missing Value](../img/03_missing_value.png)

**Konfigurasi:**
- **Treatment by Data Type:**
  - `Number (Integer)` → **Mean** (diisi dengan nilai rata-rata kolom)
  - `Number (Double)` → **Mean**
  - `String` → **Most frequent value (Mode)**

**Alasan Pemilihan Metode:**
- **Mean** dipilih untuk data numerik karena tidak mengubah distribusi data secara signifikan
- **Mode** dipilih untuk data kategorikal agar konsisten dengan nilai yang paling umum

**Hasil Output:**
- **Rows:** 2.000 baris (tetap)
- **Columns:** 12 kolom (tetap)
- Missing value berhasil diisi → contoh: Row2 kolom N Total yang semula kosong menjadi **0.231** (nilai mean kolom)

### Step 3 – One to Many (Encoding)

Node **One to Many** digunakan untuk mengkonversi fitur kategorikal menjadi representasi numerik (*dummy variables* / *one-hot encoding*).

![One to Many](../img/04_one_to_many.png)

**Konfigurasi:**
- **Columns to transform:** `Tekstur Tanah`

**Penjelasan:**

Kolom **Tekstur Tanah** memiliki beberapa kategori (Debu, Liat, Lempung Berpa, Lempung Berlia, dll). Algoritma KNN tidak dapat memproses data bertipe String secara langsung, sehingga setiap kategori dikonversi menjadi kolom biner (0 atau 1).

Contoh:
| Tekstur Tanah | Tekstur=Debu | Tekstur=Liat | Tekstur=Lempung Berpa |
|---|---|---|---|
| Debu | 1 | 0 | 0 |
| Liat | 0 | 1 | 0 |
| Lempung Berpa | 0 | 0 | 1 |

**Hasil Output:**
- **Rows:** 2.000 baris (tetap)
- **Columns:** 18 kolom (bertambah 6 kolom dummy dari Tekstur Tanah)

### Step 4 – Normalizer

Node **Normalizer** digunakan untuk menyamakan skala nilai pada semua fitur numerik menggunakan metode **Min-Max Normalization**.

![Normalizer](../img/05_normalizer.png)

**Rumus Min-Max Normalization:**

$$x' = \frac{x - x_{min}}{x_{max} - x_{min}}$$

**Konfigurasi:**
- **Excludes (tidak dinormalisasi):** `ID`
- **Includes (dinormalisasi):**
  - pH Tanah, N Total (%), P Tersedia, K Tersedia
  - C Organik (%), KTK (meq/100g), Kejenuhan Basa
  - Kadar Air (%), Bulk Density

**Alasan Normalisasi:**
KNN menggunakan perhitungan **jarak Euclidean** sehingga fitur dengan nilai besar (misal: KTK = 33-82) akan mendominasi fitur dengan nilai kecil (misal: N Total = 0.1-0.9). Normalisasi memastikan setiap fitur berkontribusi secara seimbang.

**Hasil Output:**
- **Rows:** 2.000 baris (tetap)
- **Columns:** 18 kolom (tetap)
- Semua nilai fitur numerik berada pada rentang **[0, 1]**

Contoh nilai sebelum dan sesudah normalisasi:

| Kolom | Sebelum | Sesudah |
|---|---|---|
| pH Tanah (Row0) | 8.93 | 0.987 |
| N Total (Row0) | 0.183 | 0.353 |
| KTK (Row0) | 31.86 | 0.243 |

### Step 5 – Column Filter

Node **Column Filter** digunakan untuk memilih kolom yang akan digunakan dalam proses klasifikasi dan membuang kolom yang tidak diperlukan.

![Column Filter](../img/06_column_filter.png)

**Konfigurasi:**
- **Excludes (dibuang):**
  - `ID` — hanya nomor identifikasi, tidak relevan untuk klasifikasi
  - `Tekstur Tanah` — sudah dikonversi ke dummy variables di step 3
- **Includes (disimpan):**
  - pH Tanah, N Total (%), P Tersedia, K Tersedia, C Organik (%)
  - KTK (meq/100g), Kejenuhan Basa, Kadar Air (%), Bulk Density
  - Kolom-kolom dummy Tekstur Tanah
  - **Label** (target klasifikasi)

**Hasil Output:**
- **Rows:** 2.000 baris (tetap)
- **Columns:** 16 kolom (berkurang 2 dari sebelumnya)
- Kolom Label terlihat dengan nilai **Subur** dan **Tidak Subur**

### Step 6 – Table Partitioner

Node **Table Partitioner** digunakan untuk membagi dataset menjadi dua bagian: **data training** dan **data testing**.

![Table Partitioner](../img/07_table_partitioner.png)

**Konfigurasi:**
- **First partition type:** Relative (%)
- **Relative size:** **80%** → untuk data training
- **Remaining:** **20%** → untuk data testing
- **Sampling strategy:** Random
- **Fixed random seed:** 1678807467440 (untuk reproduktibilitas hasil)

**Hasil Pembagian Data:**

| Partisi | Jumlah Data | Persentase |
|---|---|---|
| **First Partition (Training)** | **1.600 baris** | 80% |
| **Second Partition (Testing)** | **400 baris** | 20% |
| **Total** | **2.000 baris** | 100% |

**Alasan Pembagian 80:20:**
- Data training yang lebih banyak memungkinkan model belajar pola yang lebih baik
- Data testing yang representatif untuk evaluasi performa yang objektif

### Step 7 – K Nearest Neighbor

Node **K Nearest Neighbor** adalah inti dari proses klasifikasi. Algoritma ini mengklasifikasikan data baru berdasarkan mayoritas kelas dari **k tetangga terdekatnya**.

![KNN Config](../img/08_knn_config.png)

**Konfigurasi:**
- **Column with class labels:** `Label`
- **Number of neighbors to consider (k):** **5**
- **Weight neighbors by distance:** ✅ Aktif
- **Output class probabilities:** ✅ Aktif

**Konsep Algoritma KNN:**

**Rumus Jarak Euclidean:**

$$d(x, y) = \sqrt{\sum_{i=1}^{n}(x_i - y_i)^2}$$

**Langkah Kerja KNN:**
1. Hitung jarak Euclidean antara data uji dengan **semua** data training
2. Urutkan data training berdasarkan jarak terkecil
3. Pilih **k=5** data training terdekat
4. Tentukan kelas berdasarkan **mayoritas** kelas dari 5 tetangga terdekat
5. Karena *weight by distance* aktif → tetangga lebih dekat mendapat bobot lebih besar

**Alasan Pemilihan k=5:**
- Nilai k=5 adalah nilai default yang sering memberikan hasil baik
- Menghindari overfitting (k terlalu kecil) dan underfitting (k terlalu besar)
- Jumlah tetangga ganjil menghindari hasil seri pada klasifikasi biner

**Hasil Output:**
- **Rows:** 400 baris (data testing)
- **Columns:** 19 kolom (ditambah kolom `Class [kNN]` dan probabilitas)

### Step 8 – Scorer (Evaluasi Model)

Node **Scorer** digunakan untuk mengevaluasi performa model dengan membandingkan hasil prediksi KNN dengan label aktual.

**Konfigurasi:**
- **First column:** `Class [kNN]` (hasil prediksi)
- **Second column:** `Label` (nilai aktual)
- **Sorting strategy:** Insertion order

![Scorer Confusion Matrix](../img/09_scorer_confusion.png)

#### Confusion Matrix

| | **Prediksi: Tidak Subur** | **Prediksi: Subur** |
|---|---|---|
| **Aktual: Tidak Subur** | **200** (True Negative) | 0 (False Positive) |
| **Aktual: Subur** | 0 (False Negative) | **200** (True Positive) |

**Interpretasi:**
- **200** data Tidak Subur → diprediksi benar sebagai Tidak Subur ✅
- **200** data Subur → diprediksi benar sebagai Subur ✅
- **0** kesalahan klasifikasi ← model sempurna!

---
## 4. Hasil dan Pembahasan

### 4.1 Perhitungan Metrik Evaluasi

Berdasarkan Confusion Matrix di atas:
- **TP (True Positive)** = 200 → Subur diprediksi Subur
- **TN (True Negative)** = 200 → Tidak Subur diprediksi Tidak Subur  
- **FP (False Positive)** = 0 → Tidak Subur diprediksi Subur
- **FN (False Negative)** = 0 → Subur diprediksi Tidak Subur

### 4.2 Rumus dan Hasil Metrik

**Accuracy:**
$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN} = \frac{200 + 200}{200 + 200 + 0 + 0} = \frac{400}{400} = 100\%$$

**Precision:**
$$\text{Precision} = \frac{TP}{TP + FP} = \frac{200}{200 + 0} = 100\%$$

**Recall (Sensitivity):**
$$\text{Recall} = \frac{TP}{TP + FN} = \frac{200}{200 + 0} = 100\%$$

**F1-Score:**
$$\text{F1-Score} = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}} = 2 \times \frac{1.0 \times 1.0}{1.0 + 1.0} = 100\%$$

### 4.3 Ringkasan Performa Model

| Metrik | Kelas: Tidak Subur | Kelas: Subur | Overall |
|---|---|---|---|
| **Accuracy** | — | — | **100%** |
| **Precision** | 100% | 100% | 100% |
| **Recall** | 100% | 100% | 100% |
| **F1-Score** | 100% | 100% | 100% |
| **Data Benar** | 200/200 | 200/200 | 400/400 |

### 4.4 Analisis Hasil

Model KNN dengan k=5 mencapai **akurasi sempurna 100%**. Hal ini dapat dijelaskan oleh beberapa faktor:

1. **Fitur yang informatif** – Atribut agronomis seperti pH, N Total, P Tersedia, K Tersedia, dan C Organik memiliki korelasi tinggi dengan tingkat kesuburan tanah, sehingga batas keputusan antar kelas sangat jelas.

2. **Preprocessing yang tepat** – Normalisasi Min-Max memastikan semua fitur berkontribusi setara dalam perhitungan jarak Euclidean, sehingga tidak ada dominasi fitur dengan skala besar.

3. **Encoding yang benar** – Konversi Tekstur Tanah menjadi dummy variables memungkinkan KNN memproses informasi kategorikal secara efektif.

4. **Pembagian data yang stratified** – 1600 data training cukup untuk KNN mempelajari pola distribusi kedua kelas dengan baik.

---
## 5. Kesimpulan

Berdasarkan analisis yang telah dilakukan menggunakan KNIME Analytics Platform, dapat disimpulkan:

1. **Preprocessing** berperan penting dalam keberhasilan model:
   - *Missing value* berhasil ditangani dengan imputasi **mean** (numerik) dan **mode** (kategorikal)
   - *One-hot encoding* pada Tekstur Tanah menghasilkan 6 kolom dummy tambahan
   - Normalisasi Min-Max menyamakan skala semua fitur ke rentang [0, 1]
   - Column Filter membuang kolom ID dan Tekstur Tanah asli yang tidak diperlukan

2. **Pembagian data** menggunakan rasio **80:20** (1.600 training : 400 testing) dengan strategi random sampling.

3. **Model KNN** dengan parameter **k=5** dan *weight by distance* berhasil mengklasifikasikan kesuburan tanah dengan **akurasi 100%** pada data testing:
   - 200 data "Tidak Subur" → semua diprediksi benar
   - 200 data "Subur" → semua diprediksi benar

4. Hasil ini menunjukkan bahwa fitur-fitur agronomis yang digunakan memiliki **daya diskriminasi yang tinggi** untuk membedakan tanah subur dan tidak subur.
