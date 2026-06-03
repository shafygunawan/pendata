# Prediksi Kadar NO2 di Wilayah Probolinggo

## Gambaran Umum

Pertumbuhan aktivitas industri, mobilitas transportasi, dan laju pertambahan penduduk yang tinggi telah mendorong kenaikan pencemaran udara di banyak daerah. Salah satu polutan yang paling diperhatikan adalah Nitrogen Dioksida (NO2), yaitu gas berbahaya yang umumnya muncul dari proses pembakaran bahan bakar fosil, seperti kendaraan bermotor, pembangkit listrik, dan aktivitas industri. NO2 dapat menimbulkan dampak serius bagi kesehatan, mulai dari gangguan pernapasan, iritasi paru, sampai memperparah asma dan bronkitis. Di sisi lain, gas ini juga ikut berperan dalam pembentukan hujan asam serta menurunkan kualitas lingkungan secara umum.

## 1. Pengambilan Data

Langkah awal yang dilakukan adalah mengambil data deret waktu harian kadar NO2 di wilayah Probolinggo. Data diambil dari situs <https://dataspace.copernicus.eu/>. Sebelum melanjutkan, pastikan sudah membuat akun pada platform Copernicus tersebut.

Pada bagian ini, data NO2 Probolinggo akan diambil untuk rentang tanggal 1 Jun 2023 hingga 1 Jun 2026.

Sebelum mulai, pasang terlebih dahulu `openeo`:

```bash
pip install openeo
```

Setelah itu, jalankan kode berikut:

```python
import openeo

connection = openeo.connect("openeo.dataspace.copernicus.eu").authenticate_oidc()
```

Saat baris kode di atas (`connection`) dijalankan, sistem akan meminta
proses autentikasi dengan keluaran seperti berikut:

```
Visit {link} 📋 to authenticate.
✅ Authorized successfully

Authenticated using device code flow.
```

Klik tautan autentikasi yang muncul, lalu masuk menggunakan akun Copernicus yang dimiliki.

```python
aoi = {
    "type": "Polygon",
    "coordinates": [
        [
            [113.2264197, -7.714735],
            [113.3806245, -7.824401],
            [113.3028979, -7.9753391],
            [113.1189538, -7.9740395],
            [113.0568596, -7.8253705],
            [113.2264197, -7.714735],
        ]
    ]
}

s5post = connection.load_collection(
    "SENTINEL_5P_L2",
    temporal_extent=["2023-06-01", "2026-06-01"],
    spatial_extent={
        "west": 113.0568596,
        "south": -7.9740395,
        "east": 113.3806245,
        "north": -7.714735
    },
    bands=["NO2"],
)

# Now aggregate by day to avoid having multiple data per day
s5p_no2_daily = s5post.aggregate_temporal_period(reducer="mean", period="day")

# Now create a spatial aggregation to generate mean timeseries data
s5p_no2_aoi = s5p_no2_daily.aggregate_spatial(reducer="mean", geometries=aoi)
```

Kode tersebut membutuhkan koordinat area yang akan dijadikan sumber data NO2. Untuk menentukan koordinatnya, buka situs <https://geojson.io>. Di sana, pilih area yang diinginkan dengan menggambar bentuk pada wilayah yang akan diambil datanya.

![Teks alternatif](img/Screenshot%202025-10-23%20110952.png)

Pada panel sebelah kanan tersedia JSON berisi koordinat wilayah yang
dipilih. Salin data tersebut, lalu sesuaikan dengan kode sebelumnya pada
bagian variabel "aoi" dan spatial_extent.

Kemudian tambahkan baris kode berikut untuk memulai proses pengambilan data:

```python
job = s5post.execute_batch(title="NO2 in Probolinggo", outputfile="NO2Probolinggo.nc")
```

Tunggu proses pengambilan data, output proses seperti berikut:

```
0:00:00 Job 'j-260603032619403ab346cb67c816b0c0': send 'start'
0:00:12 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:00:17 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:00:23 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:00:31 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:00:41 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:00:54 Job 'j-260603032619403ab346cb67c816b0c0': queued (progress 0%)
0:01:09 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:01:29 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:01:53 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:02:23 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:03:00 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:03:47 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:04:45 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:05:46 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:06:46 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:07:46 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:08:46 Job 'j-260603032619403ab346cb67c816b0c0': running (progress N/A)
0:09:47 Job 'j-260603032619403ab346cb67c816b0c0': finished (progress 100%)
```

Abaikan ketika ada N/A.

Selama proses pengambilan data, aktivitas akan tercatat di halaman <https://editor.openeo.org/?server=https%3A%2F%2Fopeneo.dataspace.copernicus.eu%2Fopeneo%2F1.2>. Di sana akan terlihat nama dataset dan status pengambilan data.

![Teks alternatif](img/Screenshot%202025-10-24%20121430.png)

## 2. Praproses Data

Setelah data berhasil diambil, file dapat diunduh melalui halaman <https://editor.openeo.org/?server=https%3A%2F%2Fopeneo.dataspace.copernicus.eu%2Fopeneo%2F1.2>. File yang diperoleh berbentuk .nc. Dari file tersebut, kita hanya memerlukan kolom date dan NO2 dengan bantuan kode berikut:

```python
import netCDF4

file_path = "NO2Probolinggo.nc"
ds = netCDF4.Dataset(file_path)

# Lihat seluruh variabel yang tersedia
print("📦 Variabel dalam file:")
print(ds.variables.keys())
# dict_keys(['t', 'x', 'y', 'crs', 'NO2'])

# Ambil NO2
no2 = ds.variables["NO2"][:]

# Ambil Time
time = ds.variables["t"][:]

# Konversi waktu ke format tanggal jika punya atribut 'units'
try:
    time_units = ds.variables["t"].units
    dates = netCDF4.num2date(time, units=time_units)
except Exception:
    dates = time  # fallback kalau tidak ada units

# Tampilkan struktur data NO2
print(type(no2))
# type <class 'numpy.ma.core.MaskedArray'>

print(len(no2))
# banyaknya data record NO2 725

print(len(no2[0]))
# panjang data perbaris 9

print(len(no2[0][0]))
# panjang perdata 8

print(no2[0][0][0])
# 3.7701793e-05
```

Melalui kode di atas, kita bisa melihat bentuk data pada kolom NO2.

Jadi, struktur data NO2 per baris dapat digambarkan sebagai berikut:

```python
[
    [[] * 6] * 8
]
```

Untuk melihat 10 data pertama, gunakan kode berikut:

```python
print("Contoh data pertama:")
for i in range(0, 10):
    print(no2[i])
```

Dalam satu hari, data NO2 jumlahnya cukup banyak, sehingga perlu dirata-ratakan agar setiap cell hanya memiliki satu nilai. Namun, data NO2 juga memiliki kendala berupa missing value. Contohnya dapat dilihat pada keluaran di bawah ini:

```
[1.5991034160833806e-05 3.192083022440784e-05 -- -- -- --]
```

### a. Mengisi Missing Value dengan Interpolasi Linear

Berikutnya, missing value pada data NO2 akan ditangani terlebih dahulu.

```python
import numpy as np
import pandas as pd

# Interpolasi Linear
no2_filled = np.zeros_like(no2)
# Untuk jaga-jaga jika terdapat '--' tidak berubah menjadi 0
no2_filled = no2_filled.filled(0)

# loop tiap grid (y,x)
for i in range(no2.shape[1]):     # 9 baris
    for j in range(no2.shape[2]): # 8 kolom
        series = pd.Series(no2[:, i, j])
        no2_filled[:, i, j] = series.interpolate(method='linear', limit_direction='both').to_numpy()
```

Kode di atas akan mengisi missing value pada data NO2 secara otomatis menggunakan metode interpolasi linear.

### b. Menghitung Rata-rata dan Mengubah Datetime

Setelah missing value diperbaiki, data NO2 kemudian dirata-ratakan agar satu record hanya berisi satu nilai. Sekaligus, tanggalnya diambil dan disimpan ke dalam array. Format datetime juga diubah dari (2023-06-04 00:00:00) menjadi (2023-06-04), karena data yang digunakan adalah deret waktu harian sehingga komponen jam, menit, dan detik tidak diperlukan.

```python
new_dates = []
new_no2 = []
for i in range(len(dates)):
    # ubah format datetime
    new_date = dates[i].strftime('%Y-%m-%d')
    new_dates.append(new_date)
    new_no2.append(np.mean(no2_filled[i]))
```

### c. Menyimpan Data ke CSV

Setelah itu, data akan dibentuk menjadi DataFrame Pandas untuk disimpan
ke dalam format CSV.

```python
df = pd.DataFrame({
    "date": dates,
    "NO2": no2_values
})

# Simpan ke CSV
df.to_csv("NO2_Probolinggo_timeseries.csv", index=False)
```

Proses pengisian missing value dan penyimpanan ke CSV telah berhasil dilakukan.

### d. Memeriksa Missing Value Harian pada CSV

Setelah data tersimpan dalam bentuk CSV, langkah berikutnya adalah memeriksa apakah deret waktu hariannya sudah lengkap. Untuk itu, gunakan kode berikut:

```python
import pandas as pd
import numpy as np

df = pd.read_csv("NO2_Probolinggo_timeseries.csv")

# Pastikan kolom 'date' bertipe datetime
df['date'] = pd.to_datetime(df['date'])

# Buat rentang tanggal lengkap
start_date = "2023-10-01"
end_date = "2025-09-30"
full_range = pd.date_range(start=start_date, end=end_date, freq='D')

# Cek tanggal yang hilang
missing_dates = full_range.difference(df['date'])

print(f"Jumlah hari missing: {len(missing_dates)}")
print("Daftar tanggal missing:")
print(missing_dates)
```

```
Jumlah hari missing: 8
Daftar tanggal missing:
DatetimeIndex(['2023-11-11', '2024-01-01', '2024-03-11', '2024-03-23',
               '2024-08-12', '2025-01-30', '2025-01-31', '2026-06-01'],
              dtype='datetime64[ns]', freq=None)
```

Pada kasus ini, masih ditemukan 8 hari yang hilang. Missing value
tersebut akan diperbaiki kembali dengan interpolasi linear. Kode yang
digunakan adalah sebagai berikut:

```python
import pandas as pd

# Pastikan datetime dan sorting
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date')

# Buat rentang tanggal lengkap
full_range = pd.date_range(start="2023-10-01", end="2025-09-30", freq='D')

# Reindex agar tanggal yang hilang muncul sebagai NaN
df = df.set_index('date').reindex(full_range)
df.index.name = 'date'

# Interpolasi linear berdasarkan indeks waktu
df['NO2'] = df['NO2'].interpolate(method='time')

# (Opsional) jika masih ada NaN di bagian awal/akhir bisa gunakan forward/backward fill
df['NO2'] = df['NO2'].fillna(method='bfill').fillna(method='ffill')

# Simpan kembali ke CSV
df.to_csv("no2_timeseries_interpolated.csv")
```

Setelah pengecekan dilakukan, tidak ditemukan lagi missing value pada data harian.

```
Jumlah hari missing: 0
Daftar tanggal missing:
DatetimeIndex([], dtype='datetime64[ns]', freq='D')
```

Dengan hasil tersebut, data akhirnya memiliki dua kolom: kolom pertama berisi tanggal, sedangkan kolom kedua berisi kadar NO2 yang sudah dirata-ratakan.

### e. Deteksi Outlier dengan IQR

Setelah missing value diperbaiki dengan interpolasi linear, tahap selanjutnya adalah mendeteksi outlier menggunakan metode IQR.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("no2_timeseries_interpolated.csv")

df['date'] = pd.to_datetime(df['date'])

# Hitung IQR
Q1 = df['NO2'].quantile(0.25)
Q3 = df['NO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['NO2'] < lower_bound) | (df['NO2'] > upper_bound)]

print("Jumlah Outlier (IQR):", len(outliers_iqr))
print(outliers_iqr[['date', 'NO2']].head())
```

```
Jumlah Outlier (IQR): 32
         date       NO2
0  2023-06-01  0.000051
11 2023-06-12  0.000050
12 2023-06-13  0.000049
13 2023-06-14  0.000058
14 2023-06-15  0.000067
```

Untuk memvisualisasikan outlier, gunakan kode berikut:

```python
# === Visualisasi ===
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['NO2'], label="NO2", linewidth=1)

# Titik Outlier
plt.scatter(outliers_iqr['date'], outliers_iqr['NO2'],
            color='red', marker='o', label="Outliers")

# Garis batas atas & bawah
plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue', linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data NO2 (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar NO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

![Teks alternatif](img/Figure_1.png)

Setelah outlier teridentifikasi, data tersebut akan dihapus terlebih dahulu. Karena data yang digunakan merupakan deret waktu, nilai outlier yang dihapus akan diisi ulang menggunakan interpolasi linear.

```python
# Tandai outlier menjadi NaN
df['NO2_cleaned'] = df['NO2'].mask((df['NO2'] < lower_bound) | (df['NO2'] > upper_bound))

print("Jumlah nilai yang dinyatakan sebagai outlier:", df['NO2_cleaned'].isna().sum())

# Interpolasi linear untuk mengisi kembali nilai outlier
df['NO2_filled'] = df['NO2_cleaned'].interpolate(method='linear')

# Jika masih tersisa NaN di ujung data, isi dengan forward/backward fill
df['NO2_filled'] = df['NO2_filled'].bfill().ffill()
# df['NO2_filled'] = df['NO2_filled'].fillna(method='bfill').fillna(method='ffill')

print("Jumlah missing setelah interpolasi:", df['NO2_filled'].isna().sum())
```

Visualisasi data setelah outlier dihapus dan diisi kembali dengan interpolasi linear:

```python
plt.figure(figsize=(15,5))
# Plot data hasil interpolasi
plt.plot(df['date'], df['NO2_filled'], label="NO2 (Interpolated)", linewidth=1)
# Tampilkan hanya tanggal awal dan akhir di sumbu X
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.title("Plot Data NO2 Setelah Outlier Removal & Interpolasi")
plt.xlabel("Tanggal")
plt.ylabel("Kadar NO2")
plt.legend()
plt.tight_layout()
plt.show()
```

![Teks alternatif](img/Figure_2.png)

## 3. Pemodelan dengan KNN Regression

Dengan data deret waktu harian kadar NO2 di Probolinggo, tujuan selanjutnya adalah memprediksi kadar NO2 satu hari ke depan. Pada tahap ini, data akan diubah untuk mencari hubungan antara satu hari dengan beberapa hari sebelumnya. Selain itu, akan dibandingkan juga apakah semakin banyak hari sebelumnya, performa model ikut meningkat.

### a. Uji Korelasi Data

Sebelum masuk ke tahap pemodelan, data masih berada dalam bentuk
unsupervised sehingga belum memiliki label. Karena itu, data akan diubah
menjadi supervised terlebih dahulu, lalu korelasinya terhadap label (t)
akan diuji. Fiturnya terdiri dari 30 hari sebelumnya (t-30, t-29, \...
t-1), sedangkan labelnya adalah (t).

```python
import pandas as pd

def create_supervised(data, n_lag=4):
    df_supervised = pd.DataFrame()

    # Membuat fitur t-4 sampai t-1
    for i in range(n_lag, 0, -1):
        df_supervised[f'NO2(t-{i})'] = data.shift(i)

    # Label hari H
    df_supervised['NO2(t)'] = data

    # Hapus baris yang masih mengandung NaN akibat shift
    df_supervised.dropna(inplace=True)

    return df_supervised

# contoh penggunaan
supervised_df30 = create_supervised(df['NO2_scaled'], n_lag=30)

# Ambil semua lag dan kolom target
lag_cols = supervised_df30.drop(columns="NO2(t)").columns
correlations = supervised_df30[lag_cols].corrwith(supervised_df30['NO2(t)'])

# Tampilkan nilai korelasi
print(correlations)
```

```
NO2(t-30)    0.442365
NO2(t-29)    0.454480
NO2(t-28)    0.475354
NO2(t-27)    0.411464
NO2(t-26)    0.381559
NO2(t-25)    0.368824
NO2(t-24)    0.353114
NO2(t-23)    0.364938
NO2(t-22)    0.372437
NO2(t-21)    0.380476
NO2(t-20)    0.350856
NO2(t-19)    0.342492
NO2(t-18)    0.312603
NO2(t-17)    0.283336
NO2(t-16)    0.288346
NO2(t-15)    0.292171
NO2(t-14)    0.311974
NO2(t-13)    0.327142
NO2(t-12)    0.341764
NO2(t-11)    0.374090
NO2(t-10)    0.397377
NO2(t-9)     0.419258
NO2(t-8)     0.455909
NO2(t-7)     0.462456
NO2(t-6)     0.460161
NO2(t-5)     0.491515
NO2(t-4)     0.523820
NO2(t-3)     0.593839
NO2(t-2)     0.675955
NO2(t-1)     0.796441
```

Nilai korelasi berada pada rentang -1 sampai 1. Dalam kasus ini, fitur
yang paling baik adalah yang memiliki nilai korelasi di atas 0.5, yaitu
fitur t-1 sampai t-4.

### c. Transformasi Data

Selanjutnya, data akan diubah dari bentuk sebelumnya menjadi data
dengan 4 hari ke belakang yang menghasilkan 5 kolom, yaitu
t-4, t-3, t-2, t-1, dan t sebagai label. Hal ini dilakukan karena hasil
uji korelasi menunjukkan bahwa empat fitur tersebut memiliki hubungan
terbaik, yaitu lebih dari 0.5. Selain itu, dibuat juga data dengan 10
hari sebelumnya untuk membandingkan apakah penambahan jumlah lag benar-
benar memperbaiki model.

```python
supervised_df = create_supervised(df['NO2_scaled'], n_lag=4)

print(supervised_df)
print(supervised_df.shape)
```

```
output/terminal
     NO2(t-4)  NO2(t-3)  NO2(t-2)  NO2(t-1)    NO2(t)
4    0.238203  0.192840  0.196854  0.149560  0.154247
5    0.192840  0.196854  0.149560  0.154247  0.185625
6    0.196854  0.149560  0.154247  0.185625  0.152010
7    0.149560  0.154247  0.185625  0.152010  0.149143
8    0.154247  0.185625  0.152010  0.149143  0.159907
..        ...       ...       ...       ...       ...
726  0.123092  0.325742  0.372653  0.145997  0.094458
727  0.325742  0.372653  0.145997  0.094458  0.089599
728  0.372653  0.145997  0.094458  0.089599  0.000000
729  0.145997  0.094458  0.089599  0.000000  0.014405
730  0.094458  0.089599  0.000000  0.014405  0.014405
[727 rows x 5 columns]
(727, 5)
```

Untuk membuat data dengan 10 hari sebelumnya, cukup ubah parameter
`n_lag` pada kode berikut.

```python
supervised_df10 = create_supervised(df['NO2_scaled'], n_lag=10)

print(supervised_df10)
print(supervised_df10.shape)
```

```
     NO2(t-10)  NO2(t-9)  NO2(t-8)  NO2(t-7)  NO2(t-6)  NO2(t-5)  NO2(t-4)  NO2(t-3)  NO2(t-2)  NO2(t-1)    NO2(t)
10    0.238203  0.192840  0.196854  0.149560  0.154247  0.185625  0.152010  0.149143  0.159907  0.242292  0.214105
11    0.192840  0.196854  0.149560  0.154247  0.185625  0.152010  0.149143  0.159907  0.242292  0.214105  0.166780
12    0.196854  0.149560  0.154247  0.185625  0.152010  0.149143  0.159907  0.242292  0.214105  0.166780  0.127252
13    0.149560  0.154247  0.185625  0.152010  0.149143  0.159907  0.242292  0.214105  0.166780  0.127252  0.083753
14    0.154247  0.185625  0.152010  0.149143  0.159907  0.242292  0.214105  0.166780  0.127252  0.083753  0.091532
..         ...       ...       ...       ...       ...       ...       ...       ...       ...       ...       ...
726   0.161874  0.128849  0.095824  0.062799  0.038033  0.059606  0.123092  0.325742  0.372653  0.145997  0.094458
727   0.128849  0.095824  0.062799  0.038033  0.059606  0.123092  0.325742  0.372653  0.145997  0.094458  0.089599
728   0.095824  0.062799  0.038033  0.059606  0.123092  0.325742  0.372653  0.145997  0.094458  0.089599  0.000000
729   0.062799  0.038033  0.059606  0.123092  0.325742  0.372653  0.145997  0.094458  0.089599  0.000000  0.014405
730   0.038033  0.059606  0.123092  0.325742  0.372653  0.145997  0.094458  0.089599  0.000000  0.014405  0.014405
[721 rows x 11 columns]
(721, 11)
```

### d. Pemodelan dan Evaluasi {#d-modeling-dan-evaluation}

Setelah dua bentuk data tersebut disiapkan, model akan dilatih menggunakan KNN Regression.

```python
from sklearn.neighbors import KNeighborsRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

def MAPE(y_true, y_pred):
    y_true, y_pred = np.array(y_true), np.array(y_pred)
    # Hindari pembagian dengan nol
    nonzero = y_true != 0
    return np.mean(np.abs((y_true[nonzero] - y_pred[nonzero]) / y_true[nonzero])) * 100

def train_knn(df_supervised, model_name=""):
    # Pisahkan fitur & label
    X = df_supervised.drop(columns=['NO2(t)']).values
    y = df_supervised['NO2(t)'].values

    # Split data 80/20
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, shuffle=False
    )

    # Model KNN
    knn = KNeighborsRegressor(n_neighbors=5)
    knn.fit(X_train, y_train)

    # Prediksi
    y_pred = knn.predict(X_test)

    # Evaluasi
    mse = mean_squared_error(y_test, y_pred)
    rmse = np.sqrt(mse)
    r2 = r2_score(y_test, y_pred)
    mape = MAPE(y_test, y_pred)

    print(f"\n=== {model_name} ===")
    print(f"Train Size: {len(X_train)} — Test Size: {len(X_test)}")
    print(f"RMSE: {rmse:.6f}")
    print(f"R² Score: {r2:.4f}")
    print(f"MAPE: {mape:.4f}%")

    return knn, y_test, y_pred


# Train model untuk 4 hari sebelumnya
knn_4, y_test_4, y_pred_4 = train_knn(supervised_df, "KNN - 4 Hari Sebelumnya")

# Train model untuk 10 hari sebelumnya
knn_10, y_test_10, y_pred_10 = train_knn(supervised_df10, "KNN - 10 Hari Sebelumnya")
```

```
output/terminal
=== KNN - 4 Hari Sebelumnya ===
Train Size: 581 — Test Size: 146
RMSE: 0.065436
R² Score: 0.1395
MAPE: 61.0780%

=== KNN - 10 Hari Sebelumnya ===
Train Size: 576 — Test Size: 145
RMSE: 0.067567
R² Score: 0.0886
MAPE: 64.6611%
```

Berdasarkan hasil evaluasi di atas, terlihat bahwa penambahan hari
sebelumnya tidak selalu membuat model menjadi lebih baik. Untuk
membuktikannya, data 30 hari sebelumnya juga akan diuji.

```python
knn_30, y_test_30, y_pred_30 = train_knn(supervised_df30, "KNN - 30 Hari Sebelumnya")
```

```
=== KNN - 30 Hari Sebelumnya ===
Train Size: 560 — Test Size: 141
RMSE: 0.074803
R² Score: -0.0875
MAPE: 72.2295%
```

![Teks alternatif](img/Screenshot%202025-10-27%20115826.png)

Hasil evaluasi model KNN Regression menunjukkan bahwa peningkatan jumlah
fitur historis (lag) tidak serta merta meningkatkan performa prediksi.
Pada model dengan 4 hari sebelumnya, nilai RMSE paling kecil dan R²
masih positif sehingga model mampu menjelaskan sebagian kecil
variabilitas data target. Namun, ketika jumlah lag ditambah menjadi 10
dan 30 hari sebelumnya, performa model justru menurun yang ditunjukkan
oleh meningkatnya nilai RMSE dan MAPE, serta penurunan nilai R² hingga
bernilai negatif pada lag 30. Nilai MAPE yang cukup tinggi pada seluruh
model (lebih dari 60%) juga mengindikasikan bahwa akurasi prediksi masih
rendah dan terdapat deviasi besar antara nilai prediksi dan nilai
aktual. Secara keseluruhan, model KNN tidak memberikan performa yang
baik pada data ini, dan penambahan fitur historis justru menyebabkan
overfitting serta menurunkan kemampuan generalisasi model. Oleh karena
itu, diperlukan pemilihan model lain atau peningkatan strategi
preprocessing untuk memperoleh hasil prediksi yang lebih baik.
