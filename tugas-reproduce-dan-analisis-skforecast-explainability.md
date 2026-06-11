# Tugas: Reproduce & Analisis Skforecast Explainability (v0.15.1)

## Bagian 1: Reproduce Code

Berikut adalah langkah-langkah implementasi kode untuk mereproduksi (reproduce) analisis interpretabilitas (explainability) pada model peramalan sesuai dengan dokumentasi resmi Skforecast 0.15.1.

### 1. Persiapan Lingkungan (Install & Import Libraries)

Sebelum memulai, pastikan pustaka `skforecast` versi 0.15.1 beserta dependensi visualisasinya seperti `shap` telah terinstal.

```python
# Jalankan perintah ini di cell notebook jika library belum terinstal
!pip install skforecast==0.15.1 shap lightgbm pandas numpy matplotlib scikit-learn
```

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Model dan Forecasting
from lightgbm import LGBMRegressor
from skforecast.datasets import fetch_dataset
from skforecast.recursive import ForecasterRecursive

# Explainability & Metrics
import shap
from sklearn.inspection import permutation_importance
from sklearn.inspection import PartialDependenceDisplay
```

### 2. Memuat dan Menyiapkan Data

Dataset yang digunakan adalah data beban listrik `vic_electricity`. Data di-resample menjadi frekuensi harian ('D'), di mana nilai `Demand` dijumlahkan dan `Temperature` dirata-rata.

```python
# Memuat dataset contoh permintaan listrik Victoria
data = fetch_dataset(name="vic_electricity")

# Mengubah frekuensi data menjadi Harian (Daily)
data = data.resample('D').agg({'Demand': 'sum', 'Temperature': 'mean'})

# Memisahkan data training dan data testing
data_train = data.loc[: '2014-12-21']
data_test  = data.loc['2014-12-22':]

print(f"Ukuran Data Training : {data_train.shape}")
print(f"Ukuran Data Testing  : {data_test.shape}")
```

### 3. Inisialisasi dan Pelatihan Model Forecaster

Menggunakan `ForecasterRecursive` berbasis `LGBMRegressor` dengan konfigurasi parameter lags = 7 dan variabel eksogen `Temperature`.

```python
# Inisialisasi forecaster autoregresif rekursif
forecaster = ForecasterRecursive(
                 regressor = LGBMRegressor(random_state=123, verbose=-1),
                 lags      = 7
             )

# Melatih model dengan menyertakan variabel eksogen (Temperature)
forecaster.fit(
    y    = data_train['Demand'],
    exog = data_train['Temperature']
)
```

### 4. Analisis Feature Importance Global (Bawaan Model)

Mengekstrak nilai tingkat kepentingan fitur untuk melihat variabel apa yang memiliki pengaruh paling dominan secara struktural di dalam pohon keputusan LightGBM.

```python
# Mengambil nilai feature importance
importance = forecaster.get_feature_importances()
print(importance)
```

### 5. Analisis SHAP (SHapley Additive exPlanations)

Untuk membuat visualisasi SHAP, kita menggunakan matriks fitur internal (X) yang digenerate otomatis oleh `skforecast` dari data training melalui fungsi `create_train_X_y`.

```python
# Membuat matriks X dan y internal dari forecaster
X_train, y_train = forecaster.create_train_X_y(
                       y    = data_train['Demand'],
                       exog = data_train['Temperature']
                   )

# Inisialisasi SHAP JS
shap.initjs()

# Menghitung SHAP values menggunakan TreeExplainer
explainer = shap.TreeExplainer(forecaster.regressor)
shap_values = explainer.shap_values(X_train)

# 5a. Global Interpretability (Summary Plot)
shap.summary_plot(shap_values, X_train, plot_type="bar")
```

### 6. Partial Dependence Plot (PDP)

Menampilkan grafik dependensi parsial menggunakan `PartialDependenceDisplay` dari scikit-learn.

```python
fig, ax = plt.subplots(figsize=(9, 4))
ax.set_title("Decision Tree")
display = PartialDependenceDisplay.from_estimator(
    estimator = forecaster.regressor,
    X         = X_train,
    features  = ["Temperature", "lag_1"],
    kind      = 'both',
    ax        = ax,
)
ax.set_title("Partial Dependence Plot")
fig.tight_layout();
```

---

## Bagian 2: Jawaban Pertanyaan Analisis

Berdasarkan hasil eksekusi program dan pemahaman terhadap dokumen [https://skforecast.org/0.15.1/user_guides/explainability.html](https://skforecast.org/0.15.1/user_guides/explainability.html), berikut adalah jawaban atas pertanyaan tugas:

### 1. Analisa prediksi tentang apa?

Analisis prediksi yang dilakukan pada dokumen ini adalah mengenai peramalan (forecasting) total permintaan energi listrik harian (Electricity Demand) di wilayah Victoria, Australia.

Fokus utama dari dokumen ini bukan hanya melatih model untuk menebak angka masa depan, melainkan menerapkan metode Interpretabilitas Model (Model Explainability). Tujuannya adalah untuk membongkar model Machine Learning yang kompleks agar kita dapat mengetahui secara transparan faktor apa saja (seperti suhu atau tren hari sebelumnya) yang paling memengaruhi naik turunnya hasil prediksi permintaan listrik tersebut.

### 2. Bagaimana bentuk data trainingnya (apa saja inputnya dan apa outputnya)?

Sebelum dilatih, data deret waktu harian ditransformasikan oleh `skforecast` menjadi format matriks tabular (X dan y). Komponen pembentuknya adalah:

- Input (Features / Predictors X):
    - Lags (lag_1 sampai lag_7): Nilai data historis permintaan listrik dari 1 hari lalu (t-1) hingga 7 hari lalu (t-7).
    - Exogenous Variable (Variabel Eksternal): Nilai rata-rata suhu udara harian (`Temperature`) pada hari prediksi dilakukan.
- Output (Target y):
    - Jumlah total nilai permintaan listrik pada hari tersebut (`Demand` pada waktu t).

### 3. Apa itu lag?

Dalam analisis data deret waktu (time series), Lag (keterlambatan) adalah nilai historis atau rekaman data masa lalu dari variabel target itu sendiri pada interval atau urutan waktu sebelumnya.

Pada kasus data harian ini:

- Lag 1: Menunjukkan nilai total permintaan listrik 1 hari sebelum hari yang diprediksi.
- Lag 7: Menunjukkan nilai total permintaan listrik 7 hari sebelum hari yang diprediksi (hari yang sama di minggu lalu).

Lag digunakan sebagai input esensial karena karakteristik data runtun waktu umumnya memiliki ketergantungan yang kuat pada pola tren atau kebiasaan yang terjadi pada masa-masa sebelumnya.

### 4. Jelaskan proses analysis yang dilakukan dari kasus diatas

Alur proses analisis yang dijalankan di dalam dokumen notebook tersebut meliputi tahapan berikut:

1. Preprocessing & Resampling Data: Mengambil dataset beban listrik, menyaringnya, dan mengubah skalanya dari data per jam menjadi data harian (`resample('D')`) dengan menjumlahkan nilai permintaan dan merata-rata suhu. Selanjutnya data dibagi menjadi subset training dan testing.
2. Model Training: Mengonfigurasi objek `ForecasterRecursive` berbasis `LGBMRegressor` dengan parameter 7 lags dan 1 variabel eksogen, lalu melakukan proses `.fit()` pada data training.
3. Mengekstrak Feature Importance Global: Menggunakan fungsi bawaan `.get_feature_importances()` untuk mengukur kontribusi dasar tiap fitur. Hasilnya menunjukkan variabel eksogen `Temperature` memiliki nilai kepentingan tertinggi, disusul oleh fitur `lag_1`.
4. Menghitung SHAP Values: Mengubah data runtun waktu menjadi matriks tabular reguler lewat `create_train_X_y()`, lalu menghitung kontribusi nilai SHAP menggunakan `TreeExplainer`. Di tahap ini dilakukan dua visualisasi:
    - Global Interpretability (Summary Plot): Melihat seberapa besar dampak positif/negatif dari nilai tinggi/rendahnya suatu fitur terhadap hasil akhir prediksi.
5. Partial Dependence Plot (PDP): Menampilkan grafik interaksi independen antara fitur (`Temperature` dan `lag_1`) terhadap nilai prediksi target guna memetakan hubungan non-linear yang dipelajari oleh model.

Kombinasi antara Feature Importance, SHAP, dan PDP ini memungkinkan kita untuk tidak hanya mengetahui variabel mana yang paling berpengaruh (seperti Temperature dan lag_1), tetapi juga memahami pola hubungan non-linear secara transparan. Contohnya, PDP dapat menunjukkan secara visual bahwa permintaan listrik akan melonjak drastis jika suhu berada di titik yang sangat ekstrem (sangat panas atau sangat dingin)."
