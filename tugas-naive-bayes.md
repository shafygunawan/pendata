# Tugas Naive Bayes

## Data Understanding

### Deskripsi Dataset

Dataset Car Evaluation berasal dari Kaggle. Dataset ini digunakan untuk memprediksi keputusan penerimaan mobil (decision) berdasarkan enam atribut fitur.

### Struktur & Tipe Data Kolom

| Kolom             | Tipe Data            | Deskripsi                  | Nilai Unik              |
| ----------------- | -------------------- | -------------------------- | ----------------------- |
| buying price      | Kategorikal (String) | Harga beli mobil           | vhigh, high, med, low   |
| maintenance cost  | Kategorikal (String) | Biaya perawatan            | vhigh, high, med, low   |
| number of doors   | Kategorikal (String) | Jumlah pintu               | 2, 3, 4, 5more          |
| number of persons | Kategorikal (String) | Kapasitas penumpang        | 2, 4, more              |
| lug_boot          | Kategorikal (String) | lug_boot                   | small, med, big         |
| safety            | Kategorikal (String) | Tingkat keselamatan        | low, med, high          |
| decision (Target) | Kategorikal (String) | Label kelas hasil evaluasi | unacc, acc, good, vgood |

### Target Variable: decision

| Nilai | Arti                                  |
| ----- | ------------------------------------- |
| unacc | Unacceptable (Tidak direkomendasikan) |
| acc   | Acceptable (Dapat diterima)           |
| good  | Good (Baik)                           |
| vgood | Very Good (Sangat baik)               |

### Statistik Singkat

![Statistik Singkat](./img/Screenshot_2026-05-06_07-38-23.png)

## Exploratory Data Analysis (EDA)

### Distribusi Target Variable

Dataset Car Evaluation terdiri dari **1.728 instance** dengan distribusi kelas sebagai berikut:

| Kelas | Jumlah Instance | Persentase |
| ----- | --------------- | ---------- |
| unacc | 1.210           | 70.02%     |
| acc   | 384             | 22.22%     |
| good  | 69              | 3.99%      |
| vgood | 65              | 3.76%      |

**Observasi:**

- Dataset mengalami **ketidakseimbangan kelas (class imbalance)** yang signifikan, di mana kelas `unacc` mendominasi lebih dari 2/3 dari seluruh data
- Kelas `good` dan `vgood` merupakan kelas minoritas dengan total hanya ~7.75% dari dataset
- Ketidakseimbangan ini wajar dalam konteks evaluasi mobil: standar kualitas yang ketat membuat hanya sedikit mobil yang memenuhi kriteria "baik" atau "sangat baik"
- Implikasi untuk modeling: model mungkin akan bias memprediksi kelas mayoritas, sehingga metrik seperti **precision dan recall per kelas** lebih informatif daripada akurasi global saja

## Workflow KNIME

![Workflow KNIME](./img/Screenshot_2026-05-06_07-39-56.png)

## Detail Node & Konfigurasi

### CSV Reader Node

Membaca file CSV dan mengubahnya menjadi tabel KNIME untuk diproses node berikutnya.

### Missing Value Node

Menangani data yang tidak lengkap (missing values) agar tidak menyebabkan error saat training model. Saya menggunakan strategi "Most Frequent Value" untuk mengisi nilai yang hilang dengan nilai yang paling sering muncul di kolom tersebut.

![Missing Value Node Configuration](./img/Screenshot_2026-05-06_07-47-31.png)

### Python Script Node

Library yang digunakan

```python
import knime.scripting.io as knio
from sklearn.naive_bayes import CategoricalNB
from sklearn.preprocessing import LabelEncoder
```

Script Python

```python
# 1. BACA DATA INPUT
df = knio.input_tables[0].to_pandas()
# Mengambil tabel dari KNIME dan konversi ke DataFrame pandas

# 2. DEFINISI KOLOM
target_col = 'Column6'  # Nama kolom target Anda
feature_cols = [col for col in df.columns if col != target_col]
# Memisahkan kolom fitur dan target secara otomatis

# 3. ENCODE FITUR (Label Encoding)
X = df[feature_cols].copy()
for col in feature_cols:
    le = LabelEncoder()
    X[col] = le.fit_transform(X[col].astype(str))
# Mengubah nilai kategorikal (vhigh, low, dll) menjadi angka (0,1,2,...)
# .astype(str) memastikan semua nilai bisa di-encode dengan aman

# 4. ENCODE TARGET
target_le = LabelEncoder()
y = target_le.fit_transform(df[target_col].astype(str))
# Target juga di-encode agar bisa diproses model

# 5. TRAIN MODEL CATEGORICAL NAIVE BAYES
model = CategoricalNB()
model.fit(X.values, y)
# CategoricalNB cocok untuk data kategorikal diskrit
# Model belajar pola hubungan fitur -> target

# 6. PREDIKSI
predictions = model.predict(X.values)
df['prediction'] = target_le.inverse_transform(predictions)
# Prediksi dikembalikan ke label asli (unacc/acc/good/vgood)

# 7. OUTPUT KE KNIME
output_cols = feature_cols + [target_col, 'prediction']
knio.output_tables[0] = knio.Table.from_pandas(df[output_cols])
# Mengirim hasil kembali ke KNIME untuk evaluasi di Scorer node
```

- Melakukan encoding data kategorikal -> numerik
- Melatih model Categorical Naive Bayes
- Melakukan prediksi pada data yang sama (atau bisa dimodifikasi untuk data test)
- Mengembalikan hasil prediksi ke KNIME dalam format tabel

### Scorer Node

Mengevaluasi performa model dengan membandingkan prediksi vs label asli.

![Scorer Node Configuration](./img/Screenshot_2026-05-06_07-48-26.png)

## Evaluasi Model

Setelah workflow dijalankan, Scorer node akan menghasilkan:

![Evaluasi Model](./img/Screenshot_2026-05-06_07-49-25.png)

### Metrik Evaluasi Detail

Berikut adalah metrik evaluasi untuk setiap kelas berdasarkan hasil Scorer node:

| Kelas | Precision | Recall | F-measure |
| ----- | --------- | ------ | --------- |
| unacc | 0.9304    | 0.9612 | 0.9455    |
| acc   | 0.7034    | 0.7474 | 0.7247    |
| good  | 0.3043    | 0.6364 | 0.4118    |
| vgood | 0.9459    | 0.5385 | 0.6863    |

**Keterangan Metrik:**

- **Precision**: Dari semua prediksi kelas X, berapa persen yang benar-benar kelas X _(mengukur keakuratan prediksi positif)_
- **Recall**: Dari semua instance kelas X yang sebenarnya, berapa persen yang berhasil diprediksi dengan benar _(mengukur kemampuan menemukan semua instance positif)_
- **F-measure**: Rata-rata harmonik dari precision dan recall, memberikan keseimbangan antara keduanya

**Interpretasi Hasil:**

**Kelas `unacc` (Unacceptable)**

- Precision 93.04% dan Recall 96.12% -> **Performa sangat baik**
- Model sangat handal dalam mengidentifikasi mobil yang tidak direkomendasikan
- Hanya ~4% mobil `unacc` yang terlewat (false negative)

**Kelas `acc` (Acceptable)**

- Precision 70.34% dan Recall 74.74% -> **Performa cukup baik**
- Masih ada ruang perbaikan: ~25% mobil `acc` terprediksi sebagai kelas lain
- Beberapa mobil `acc` mungkin "terkontaminasi" oleh fitur yang mirip dengan `unacc`

**Kelas `good`**

- Precision sangat rendah (30.43%) -> **Masalah utama**
- Artinya: ketika model memprediksi `good`, hanya 3 dari 10 yang benar
- Namun recall 63.64% menunjukkan model cukup baik "menemukan" instance `good`, tapi banyak false positive
- Penyebab kemungkinan: kelas minoritas + fitur yang tumpang tindih dengan `acc`

**Kelas `vgood`**

- Precision tinggi (94.59%) -> prediksi `vgood` sangat terpercaya
- Namun recall rendah (53.85%) -> hampir setengah instance `vgood` terlewat
- Model konservatif: hanya memprediksi `vgood` ketika sangat yakin

**Kesimpulan Evaluasi**: Model Naive Bayes bekerja sangat baik untuk kelas mayoritas (`unacc`), namun mengalami kesulitan pada kelas minoritas (`good`, `vgood`). pola umum pada dataset imbalanced.

## Insight & Rekomendasi

### Insight Utama dari Analisis

1. **Model Sangat Andal untuk Deteksi "Unacceptable"**  
   Dengan precision 93% dan recall 96%, model hampir tidak pernah salah saat memprediksi mobil `unacc`. Ini sangat berharga dalam konteks screening awal: mobil yang berpotensi bermasalah dapat diidentifikasi dengan kepercayaan tinggi.

2. **Kelas `good` Adalah Tantangan Terbesar**  
   Precision hanya 30.43% menunjukkan bahwa model sering "terkecoh" memprediksi `good` untuk mobil yang sebenarnya `acc` atau `unacc`. Hal ini kemungkinan disebabkan oleh:
    - Jumlah instance `good` yang sangat sedikit (69 dari 1728)
    - Fitur-fitur `good` yang tumpang tindih dengan kelas lain
    - Asumsi independensi Naive Bayes yang kurang cocok untuk pola kompleks kelas ini

3. **Trade-off Precision-Recall pada `vgood`**  
   Model sangat konservatif untuk kelas `vgood`: hanya memprediksi ketika sangat yakin (precision 94.59%), namun mengorbankan recall (53.85%). Dalam konteks bisnis, ini bisa menjadi strategi yang disengaja: lebih baik melewatkan beberapa `vgood` daripada salah merekomendasikan mobil biasa sebagai "sangat baik".

4. **Akurasi Global Mungkin Menipu**  
   Jika hanya melihat akurasi keseluruhan (~94%), model terlihat sangat baik. Namun, metrik per-kelas mengungkap bahwa performa pada 7.75% data (kelas `good` + `vgood`) jauh lebih lemah. Ini menekankan pentingnya **evaluasi granular** pada dataset imbalanced.

5. **Fitur `safety` dan `lug_boot` Diduga Dominan**  
   Berdasarkan pola prediksi, kombinasi `safety=high` + `lug_boot=big` tampaknya menjadi penentu utama untuk kelas `good`/`vgood`. Eksplorasi lebih lanjut dengan feature importance dapat mengonfirmasi hal ini.

### Rekomendasi Bisnis & Teknis

1. **Untuk Stakeholder Bisnis**:
    - Gunakan model sebagai **filter awal** untuk menyaring mobil `unacc` (efektivitas tinggi)
    - Untuk keputusan `good`/`vgood`, kombinasikan prediksi model dengan review manual atau aturan bisnis tambahan

2. **Untuk Pengembangan Model Selanjutnya**:
    - Terapkan **stratified train-test split** atau **cross-validation** untuk estimasi performa yang lebih valid
    - Eksperimen dengan **class weights** atau **SMOTE** untuk menangani imbalance pada kelas `good`/`vgood`
    - Pertimbangkan model alternatif seperti **Random Forest** atau **Gradient Boosting** yang lebih robust terhadap ketidakseimbangan kelas

3. **Untuk Interpretasi Hasil**:
    - Selalu laporkan metrik per-kelas, bukan hanya akurasi global
    - Gunakan confusion matrix untuk mengidentifikasi pola kesalahan spesifik (misal: `good` sering tertukar dengan `acc`)
