# Data Understanding

Data Understanding merupakan fase eksplorasi awal dalam metodologi CRISP-DM yang bertujuan mengenali karakteristik, struktur, dan kondisi data sebelum diproses lebih lanjut. Proses ini bukan sekadar membaca data, melainkan membangun pemahaman mendalam tentang apa yang dimiliki, apa makna setiap variabel, sejauh mana kualitasnya, dan apakah data tersebut benar-benar siap dijadikan dasar pengambilan keputusan. Pemahaman yang baik terhadap data adalah kunci keberhasilan seluruh proses analisis berikutnya.

## Konsep Dasar

### Kualitas dan Kebersihan Data

Keberadaan data yang tidak bersih (_dirty data_) dapat menurunkan akurasi hasil analisis secara signifikan. Beberapa masalah umum yang ditemui pada data mentah antara lain:

1. **Redudansi data**: Data yang tercatat lebih dari sekali sehingga menyebabkan duplikasi dalam dataset.
2. **Data tidak konsisten**: Data yang merepresentasikan informasi yang sama namun memiliki format atau nilai yang berbeda-beda.
3. **Missing values**: Nilai yang tidak terisi pada satu atau lebih kolom, yang dapat memengaruhi hasil analisis apabila tidak ditangani dengan tepat.
4. **Outlier**: Nilai ekstrem yang menyimpang jauh dari distribusi umum data dan berpotensi mendistorsi model.

Penilaian kualitas data dilakukan dengan memeriksa adanya _missing values_, baris duplikat, dan nilai-nilai yang tidak wajar. Secara umum, _missing values_ masih dapat ditoleransi apabila jumlahnya di bawah 10 persen dari keseluruhan data.

### Komponen Utama Memahami Data

Proses pemahaman data mencakup empat komponen yang saling berkaitan:

1. **Pengumpulan data awal**: Menghimpun data mentah dari berbagai sumber yang relevan, termasuk basis data relasional seperti PostgreSQL.
2. **Deskripsi data**: Menjabarkan karakteristik umum dari data yang telah dikumpulkan, mencakup jumlah baris, kolom, tipe data, dan distribusi nilai.
3. **Eksplorasi data**: Mengidentifikasi pola dan keterkaitan awal antar variabel melalui statistik maupun visualisasi.
4. **Kualitas data**: Memverifikasi bahwa data telah memenuhi standar kelayakan untuk digunakan dalam pemodelan.

### Struktur Data dan Seleksi Fitur

Data tabular tersusun dalam bentuk baris (_record_) dan kolom (_fitur_). Tidak semua kolom yang tersedia diperlukan dalam analisis, sehingga dilakukan seleksi fitur untuk menyisakan hanya variabel yang relevan dan berkontribusi terhadap tujuan pemodelan. Pemilihan fitur yang tepat dapat meningkatkan efisiensi komputasi sekaligus mengurangi risiko _overfitting_.

### Sumber dan Tipe Data

Sumber data dapat bervariasi, mulai dari sistem basis data internal perusahaan seperti PostgreSQL, platform media sosial, log aktivitas sistem, hingga kuesioner survei. Adapun tipe data yang lazim dijumpai dalam analisis meliputi:

1. **Biner**: Data dengan dua kemungkinan nilai. Dibedakan menjadi biner simetris, yaitu ketika kedua nilai memiliki bobot yang setara (contoh: jenis kelamin), dan biner asimetris, yaitu ketika satu nilai lebih diutamakan (contoh: hasil tes penyakit).
2. **Kategorikal**: Data yang berisi lebih dari dua pilihan kategori tanpa nilai numerik yang melekat.
3. **Numerik**: Data berupa angka yang diperoleh dari hasil pengukuran atau pencacahan langsung.

### Pengukuran Jarak Antar Data

Dalam analisis data, jarak antar titik data digunakan untuk mengukur tingkat kemiripan atau perbedaan antara dua observasi. Konsep ini menjadi fondasi berbagai algoritma seperti _k-Nearest Neighbors_ (kNN) dan _k-Means Clustering_. Terdapat beberapa metrik jarak yang umum digunakan:

1. **Jarak Manhattan** (_City Block Distance_): Mengukur jarak sebagai selisih absolut antar dimensi. Cocok digunakan ketika perbedaan tiap fitur bersifat independen.

$$d_{\text{Manhattan}}(x, y) = \sum_{i=1}^{n} |x_i - y_i|$$

2. **Jarak Euclidean**: Mengukur jarak lurus (garis lurus) antara dua titik di ruang multidimensi. Merupakan metrik jarak yang paling umum digunakan.

$$d_{\text{Euclidean}}(x, y) = \sqrt{\sum_{i=1}^{n} (x_i - y_i)^2}$$

## Pemahaman Data Iris

Dataset Iris adalah salah satu dataset klasik dalam _machine learning_ yang terdiri dari 150 observasi bunga iris dengan empat fitur pengukuran morfologi: panjang dan lebar sepal serta panjang dan lebar petal. Setiap observasi diklasifikasikan ke dalam salah satu dari tiga spesies iris, yaitu _Iris setosa_, _Iris versicolor_, dan _Iris virginica_.

Pemahaman data Iris dilakukan secara bertahap, dimulai dari pengambilan data dari sumber, penelusuran struktur dan kelas, pendeteksian anomali, hingga penarikan kesimpulan mengenai kualitas data secara keseluruhan.

### Pengambilan Data dari Basis Data PostgreSQL

Langkah pertama dalam proses pemahaman data adalah memastikan data dapat diakses dari sumbernya. Dalam konteks ini, data Iris disimpan dalam basis data PostgreSQL dan diambil menggunakan koneksi Python melalui pustaka `psycopg2`. Proses ini mencerminkan bagaimana data dalam skenario nyata umumnya berasal dari sistem manajemen basis data perusahaan, bukan sekadar file CSV statis.

Koneksi ke database mencakup penyediaan _host_, nama basis data, nama pengguna, kata sandi, dan port. Setelah koneksi berhasil dibuat, kueri SQL dijalankan untuk mengambil seluruh data dari tabel `iris`, kemudian hasilnya dimuat ke dalam sebuah _DataFrame_ pandas untuk keperluan analisis selanjutnya.

```python
import pandas as pd
import psycopg2

# Konfigurasi koneksi ke PostgreSQL
conn = psycopg2.connect(
    host="localhost",
    database="dataset_db",
    user="postgres",
    password="password",
    port=5432
)

# Ambil data dari tabel iris menggunakan kueri SQL
query = "SELECT * FROM iris;"
df = pd.read_sql(query, conn)

# Tutup koneksi setelah data berhasil diambil
conn.close()

print(f"Data berhasil diambil: {df.shape[0]} baris, {df.shape[1]} kolom")
print(df.head())
```

![Pengambilan Data dari Basis Data PostgreSQL](./img/Screenshot_2026-03-04_05-32-25.png)

### Informasi Umum dan Struktur Data

Setelah data berhasil diambil, langkah berikutnya adalah memahami strukturnya secara menyeluruh. Pemeriksaan awal ini mencakup jumlah baris dan kolom, nama serta tipe setiap fitur, dan ada tidaknya nilai yang hilang. Informasi ini memberikan gambaran pertama apakah data yang diterima lengkap dan sesuai dengan ekspektasi.

```python
import pandas as pd

df = pd.read_csv('iris.csv')

# Tentukan tipe data level dataset untuk setiap kolom
# (bukan tipe teknis Python, melainkan kategori analitik)
dataset_types = {
    'sepal_length': 'Numerik (kontinu)',
    'sepal_width' : 'Numerik (kontinu)',
    'petal_length': 'Numerik (kontinu)',
    'petal_width' : 'Numerik (kontinu)',
    'species'     : 'Kategorikal (nominal)'
}

# Informasi struktur dataset
print("=== Dimensi Dataset ===")
print(f"Jumlah baris   : {df.shape[0]}")
print(f"Jumlah kolom   : {df.shape[1]}")

print("\n=== Tipe Data dan Missing Values ===")
info = pd.DataFrame({
    'Kolom'        : df.columns,
    'Tipe Dataset' : [dataset_types[col] for col in df.columns],
    'Non-Null'     : df.notnull().sum().values,
    'Missing'      : df.isnull().sum().values,
    '% Missing'    : (df.isnull().sum().values / len(df) * 100).round(2)
})
print(info.to_string(index=False))

print("\n=== Lima Baris Pertama ===")
print(df.head())
```

![Informasi Umum dan Struktur Data](./img/Screenshot_2026-03-04_06-17-18.png)

Dataset terdiri dari 150 baris dan 5 kolom. Keempat fitur pengukuran (`sepal_length`, `sepal_width`, `petal_length`, `petal_width`) bertipe **numerik kontinu** karena nilainya berupa hasil pengukuran morfologi yang dapat mengambil nilai pecahan dalam rentang tertentu. Kolom `species` bertipe **kategorikal nominal** karena berisi label spesies tanpa urutan atau hierarki yang melekat. Tidak ada satu pun nilai yang hilang, sehingga seluruh data dapat langsung digunakan tanpa perlu penanganan _missing values_.

### Informasi Kelas (Class Distribution)

Pemahaman terhadap distribusi kelas sangat penting, terutama dalam konteks klasifikasi. Distribusi kelas yang tidak seimbang (_imbalanced_) dapat memengaruhi performa model secara signifikan. Pada tahap ini, ditelusuri berapa banyak observasi yang mewakili setiap spesies iris.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('iris.csv')

# Distribusi kelas
class_dist = df['species'].value_counts().reset_index()
class_dist.columns = ['species', 'jumlah']
class_dist['persentase'] = (class_dist['jumlah'] / len(df) * 100).round(2)

print("=== Distribusi Kelas ===")
print(class_dist.to_string(index=False))

# Visualisasi distribusi kelas
plt.figure(figsize=(6, 4))
plt.bar(class_dist['species'], class_dist['jumlah'], color=['steelblue', 'darkorange', 'seagreen'])
plt.xlabel('Spesies')
plt.ylabel('Jumlah')
plt.title('Distribusi Kelas Dataset Iris')
plt.tight_layout()
plt.show()
```

![Informasi Kelas (Class Distribution)](./img/Screenshot_2026-03-04_05-37-02.png)

Ketiga kelas terwakili secara merata dengan masing-masing 50 observasi atau 33,33 persen dari total data. Kondisi ini disebut _perfectly balanced_, artinya tidak ada kelas yang mendominasi maupun kekurangan representasi. Dataset yang seimbang seperti ini sangat ideal untuk pelatihan model klasifikasi karena model tidak akan cenderung memihak kelas tertentu.

### Deteksi Outlier

Outlier adalah nilai yang menyimpang jauh dari pola umum data. Keberadaannya dapat dideteksi menggunakan metode _Interquartile Range_ (IQR), di mana nilai di bawah $Q1 - 1.5 \times IQR$ atau di atas $Q3 + 1.5 \times IQR$ dianggap sebagai outlier. Visualisasi menggunakan _boxplot_ mempermudah identifikasi outlier secara intuitif.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

# Deteksi outlier dengan metode IQR
print("=== Deteksi Outlier (Metode IQR) ===")
outlier_summary = []

for col in df_numeric.columns:
    Q1  = df_numeric[col].quantile(0.25)
    Q3  = df_numeric[col].quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    outliers = df_numeric[(df_numeric[col] < lower) | (df_numeric[col] > upper)]
    outlier_summary.append({
        'Fitur'   : col,
        'Q1'      : round(Q1, 4),
        'Q3'      : round(Q3, 4),
        'IQR'     : round(IQR, 4),
        'Batas Bawah': round(lower, 4),
        'Batas Atas' : round(upper, 4),
        'Jumlah Outlier': len(outliers)
    })

outlier_df = pd.DataFrame(outlier_summary)
print(outlier_df.to_string(index=False))

# Visualisasi boxplot
fig, axes = plt.subplots(1, 4, figsize=(14, 5))
for ax, col in zip(axes, df_numeric.columns):
    ax.boxplot(df_numeric[col], patch_artist=True,
               boxprops=dict(facecolor='lightblue'))
    ax.set_title(col, fontsize=10)
    ax.set_ylabel('Nilai')
plt.suptitle('Boxplot Deteksi Outlier — Dataset Iris', fontsize=12)
plt.tight_layout()
plt.show()
```

![Deteksi Outlier](./img/Screenshot_2026-03-04_05-39-24.png)

Hasil deteksi menunjukkan bahwa hanya fitur `sepal_width` yang memiliki 4 outlier. Ketiga fitur lainnya tidak mengandung outlier sama sekali. Jumlah outlier yang sangat sedikit (2,67 persen dari total data) mengindikasikan bahwa data secara keseluruhan bersih dari anomali ekstrem dan dapat digunakan tanpa perlu penghapusan data yang berarti.

### Statistik Deskriptif

Statistik deskriptif merangkum karakteristik utama dari setiap fitur numerik dalam satu tabel yang komprehensif. Ukuran yang dihitung mencakup _mean_ (rata-rata), _mode_ (nilai terbanyak), _median_ (nilai tengah), nilai terkecil (_min_), nilai terbesar (_max_), deviasi standar (_std_), serta jumlah data yang hilang (_missing values_).

```python
import pandas as pd

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

# Tabel statistik deskriptif lengkap
descriptive_stats = pd.DataFrame({
    'Fitur'  : df_numeric.columns,
    'Mean'   : df_numeric.mean().round(4).values,
    'Mode'   : df_numeric.mode().iloc[0].values,
    'Median' : df_numeric.median().values,
    'Std'    : df_numeric.std().round(4).values,
    'Min'    : df_numeric.min().values,
    'Max'    : df_numeric.max().values,
    'Missing': df_numeric.isnull().sum().values
})

print(descriptive_stats.to_string(index=False))
```

![Statistik Deskriptif](./img/Screenshot_2026-03-04_05-40-48.png)

Distribusi tiap fitur divisualisasikan menggunakan histogram dengan lebar _bin_ 0,5 untuk mengamati bentuk sebaran secara langsung.

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

fig, axes = plt.subplots(2, 2, figsize=(12, 8))
axes = axes.flatten()

for i, column in enumerate(df_numeric.columns):
    values = df_numeric[column]
    bins = np.arange(values.min() - (values.min() % 0.5), values.max() + 0.5, 0.5)
    axes[i].hist(values, bins=bins, color='steelblue', edgecolor='white')
    axes[i].set_xlabel(column)
    axes[i].set_ylabel("Frekuensi")
    axes[i].set_title(f"Distribusi {column} (bin = 0.5)")

plt.suptitle('Histogram Distribusi Fitur — Dataset Iris', fontsize=13)
plt.tight_layout()
plt.show()
```

![Distribusi](./img/Screenshot_2026-03-04_05-42-04.png)

### Analisis Korelasi Antar Fitur

Korelasi mengukur tingkat keterkaitan linear antar fitur numerik. Koefisien bernilai antara -1 dan 1: nilai positif menunjukkan hubungan searah, nilai negatif menunjukkan hubungan berlawanan arah, dan nilai mendekati nol mengindikasikan tidak adanya hubungan linear yang berarti.

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

# Matriks korelasi
corr = df_numeric.corr().round(6)
print("=== Matriks Korelasi ===")
print(corr)

# Scatter plot antar spesies
fig, axes = plt.subplots(1, 1, figsize=(7, 5))
for species in df['species'].unique():
    subset = df[df['species'] == species]
    axes.scatter(subset['sepal_length'], subset['sepal_width'], label=species, alpha=0.7)

axes.set_xlabel('Sepal Length')
axes.set_ylabel('Sepal Width')
axes.set_title('Scatter Plot: Sepal Length vs Sepal Width (per Spesies)')
axes.legend()
plt.tight_layout()
plt.show()
```

![Analisis Korelasi Antar Fitur](./img/Screenshot_2026-03-04_05-43-26.png)

Korelasi tertinggi terdapat antara `petal_length` dan `petal_width` (0.963), diikuti oleh `sepal_length` dengan `petal_length` (0.872) dan `sepal_length` dengan `petal_width` (0.818). Sebaliknya, `sepal_width` memiliki korelasi sangat rendah—bahkan sedikit negatif—terhadap ketiga fitur lainnya.

### Kesimpulan Kualitas Data

Berdasarkan seluruh proses eksplorasi yang telah dilakukan, dataset Iris berada dalam kondisi yang sangat baik dan siap digunakan untuk pemodelan. Berikut adalah ringkasan temuan:

| Aspek            | Kondisi                                                                     |
| :--------------- | :-------------------------------------------------------------------------- |
| Kelengkapan data | ✅ Tidak ada _missing values_ pada semua fitur                              |
| Duplikasi data   | ✅ Tidak ditemukan baris duplikat                                           |
| Distribusi kelas | ✅ Seimbang sempurna — 50 observasi per kelas                               |
| Outlier          | ⚠️ 4 outlier pada `sepal_width` (2,67%) — dapat ditoleransi                 |
| Tipe data        | ✅ Konsisten — 4 fitur **numerik kontinu**, 1 fitur **kategorikal nominal** |
| Rentang nilai    | ✅ Wajar dan tidak ada nilai negatif yang tidak logis                       |
| Korelasi fitur   | ✅ Petal berhubungan kuat; sepal_width bersifat independen                  |

Data Iris memenuhi semua kriteria kualitas data yang baik. Tidak diperlukan imputasi _missing values_ maupun penanganan ketidakseimbangan kelas. Satu-satunya perhatian kecil adalah empat outlier pada `sepal_width`, namun jumlahnya jauh di bawah ambang batas 10 persen sehingga tidak perlu dieliminasi.

## Perhitungan Jarak Antar Data Iris

Untuk memahami seberapa "dekat" atau "jauh" dua observasi dalam dataset Iris, dilakukan perhitungan jarak menggunakan dua metrik: **Manhattan** dan **Euclidean**. Perhitungan dilakukan pada empat fitur numerik dari dua sampel data yang dipilih sebagai contoh perbandingan, yaitu baris ke-0 (_Iris setosa_) dan baris ke-50 (_Iris versicolor_).

```python
import pandas as pd
import numpy as np

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

# Ambil dua sampel: baris ke-0 (setosa) dan baris ke-50 (versicolor)
x = df_numeric.iloc[0].values   # [5.1, 3.5, 1.4, 0.2]
y = df_numeric.iloc[50].values  # [7.0, 3.2, 4.7, 1.4]

print(f"Titik x (baris 0  — setosa)     : {x}")
print(f"Titik y (baris 50 — versicolor) : {y}")
print(f"Selisih |x - y|                 : {np.abs(x - y)}")
print()

# --- Jarak Manhattan ---
manhattan = np.sum(np.abs(x - y))
print(f"Jarak Manhattan  = {' + '.join([str(round(v, 1)) for v in np.abs(x - y)])}")
print(f"               = {manhattan:.4f}")

print()

# --- Jarak Euclidean ---
squared = (x - y) ** 2
euclidean = np.sqrt(np.sum(squared))
print(f"Jarak Euclidean  = sqrt({' + '.join([str(round(v, 2)) for v in squared])})")
print(f"               = {euclidean:.4f}")
```

![Perhitungan Jarak Antar Data Iris](./img/Screenshot_2026-03-04_06-18-06.png)

Dari hasil di atas diperoleh:

- **Jarak Manhattan** = $|1.9| + |0.3| + |3.3| + |1.2| = 6.7000$ — menjumlahkan seluruh selisih absolut per dimensi.
- **Jarak Euclidean** = $\sqrt{1.9^2 + 0.3^2 + 3.3^2 + 1.2^2} = 3.9293$ — mengukur jarak lurus di ruang empat dimensi.

Jarak Manhattan selalu lebih besar atau sama dengan jarak Euclidean karena tidak "memotong diagonal" antar dimensi, melainkan menjumlahkan setiap selisih secara terpisah.

### Perbandingan Jarak Seluruh Pasangan Kelas

Untuk memahami pemisahan antar kelas secara kuantitatif, dihitung rata-rata jarak Manhattan dan Euclidean antara semua pasangan observasi dari kelas yang berbeda:

```python
import pandas as pd
import numpy as np
from itertools import combinations

df = pd.read_csv('iris.csv')
df_numeric = df.select_dtypes(include=['number'])

species_list = df['species'].unique()
results = []

for sp1, sp2 in combinations(species_list, 2):
    data1 = df_numeric[df['species'] == sp1].values
    data2 = df_numeric[df['species'] == sp2].values

    # Hitung semua pasang jarak lintas kelas
    manhattan_dists = [np.sum(np.abs(a - b)) for a in data1 for b in data2]
    euclidean_dists = [np.sqrt(np.sum((a - b) ** 2)) for a in data1 for b in data2]

    results.append({
        'Pasangan Kelas'      : f"{sp1} vs {sp2}",
        'Rata-rata Manhattan' : round(np.mean(manhattan_dists), 4),
        'Min Manhattan'       : round(np.min(manhattan_dists), 4),
        'Rata-rata Euclidean' : round(np.mean(euclidean_dists), 4),
        'Min Euclidean'       : round(np.min(euclidean_dists), 4),
    })

result_df = pd.DataFrame(results)
print(result_df.to_string(index=False))
```

![Perhitungan Jarak Antar Data Iris](./img/Screenshot_2026-03-04_06-18-47.png)

Kedua metrik menghasilkan kesimpulan yang konsisten: _Iris setosa_ paling mudah dipisahkan dari dua spesies lainnya karena memiliki nilai jarak rata-rata yang paling besar. Sebaliknya, _versicolor_ dan _virginica_ memiliki jarak rata-rata paling kecil (Manhattan ~2.71, Euclidean ~1.99), yang berarti kedua spesies ini paling mirip secara morfologis dan paling sulit dibedakan — sesuai dengan visualisasi scatter plot yang telah diamati sebelumnya.

## Pengukuran Jarak untuk Data Bertipe Campuran — Netflix Movies and TV Shows

Dataset Iris hanya memiliki fitur numerik, sehingga perhitungan Manhattan dan Euclidean dapat langsung diterapkan. Namun pada dataset nyata, lazimnya fitur-fitur yang ada memiliki **tipe data yang berbeda-beda** dalam satu tabel — ada yang numerik, ordinal, nominal, hingga biner asimetris. Menghitung jarak secara langsung pada tipe data campuran tidak dapat dilakukan begitu saja, karena setiap tipe memerlukan cara pengukuran selisih yang berbeda.

Dataset **Netflix Movies and TV Shows** yang diambil dari Kaggle menjadi contoh kasus yang tepat untuk mengilustrasikan hal ini. Dataset ini mencatat konten yang tersedia di platform Netflix beserta metadata seperti jenis konten, negara asal, tahun rilis, rating, durasi, dan genre.

### Identifikasi Tipe Data Setiap Fitur

Sebelum menghitung jarak, setiap fitur harus diidentifikasi tipe datanya secara analitik — bukan berdasarkan tipe teknis Python, melainkan berdasarkan **makna dan sifat nilai** yang dikandungnya.

| Fitur          | Tipe Dataset            | Alasan                                                                                   |
| :------------- | :---------------------- | :--------------------------------------------------------------------------------------- |
| `show_id`      | Identifier              | Kode unik per konten, tidak memiliki makna analitik; dikeluarkan dari perhitungan        |
| `type`         | Biner simetris          | Hanya dua nilai: "Movie" dan "TV Show"; kedua nilai setara, tidak ada yang lebih penting |
| `title`        | Nominal (teks bebas)    | Label unik per konten; terlalu granular untuk perhitungan jarak                          |
| `director`     | Nominal                 | Nama sutradara; banyak _missing values_; dikecualikan                                    |
| `cast`         | Nominal (multi-nilai)   | Daftar nama aktor; terlalu kompleks untuk satu perbandingan sederhana                    |
| `country`      | Nominal                 | Nama negara tanpa urutan atau bobot tertentu                                             |
| `date_added`   | Numerik (ordinal date)  | Direpresentasikan sebagai tahun ditambahkan; dapat dibandingkan secara urutan            |
| `release_year` | Numerik (kontinu)       | Tahun rilis dalam angka bulat; dapat dihitung selisihnya secara langsung                 |
| `rating`       | Ordinal                 | Tingkat pembatasan usia yang memiliki urutan dari ringan ke ketat                        |
| `duration`     | Numerik (setelah parse) | Film: menit; TV Show: jumlah musim; perlu diparsing sebelum digunakan                    |
| `listed_in`    | Biner asimetris         | Kehadiran genre tertentu (1 = ada, 0 = tidak); nilai "0-0" tidak informatif              |
| `description`  | Teks bebas              | Narasi panjang; dikecualikan dari perhitungan jarak numerik                              |

Fitur yang dipilih untuk perhitungan jarak: `type`, `country`, `release_year`, `rating`, dan `listed_in` (genre tertentu).

### Pendekatan: Jarak Manhattan Adaptif per Tipe Data (Gower)

Ketika fitur-fitur memiliki tipe berbeda, pendekatan yang digunakan adalah **Jarak Gower** — sebuah generalisasi Manhattan yang menormalisasi kontribusi setiap fitur ke dalam skala 0–1 sesuai tipe datanya, lalu merata-ratakannya.

$$d_{\text{Gower}}(x, y) = \frac{\sum_{i=1}^{p} w_i \cdot d_i(x, y)}{\sum_{i=1}^{p} w_i}$$

Cara menghitung $d_i$ untuk setiap tipe:

| Tipe Data           | Formula $d_i$                                                                                | Rentang   |
| :------------------ | :------------------------------------------------------------------------------------------- | :-------- |
| **Numerik**         | $d_i = \dfrac{\|x_i - y_i\|}{R_i}$ di mana $R_i$ = maks − min keseluruhan data               | $[0, 1]$  |
| **Nominal**         | $d_i = \begin{cases} 0 & \text{jika } x_i = y_i \\ 1 & \text{jika berbeda} \end{cases}$      | $\{0,1\}$ |
| **Ordinal**         | $d_i = \dfrac{\|r_{x_i} - r_{y_i}\|}{n_i - 1}$ di mana $r$ = peringkat, $n_i$ = jumlah level | $[0, 1]$  |
| **Biner simetris**  | $d_i = \begin{cases} 0 & x_i = y_i \\ 1 & x_i \neq y_i \end{cases}$                          | $\{0,1\}$ |
| **Biner asimetris** | $d_i = 0$ jika $(x_i,y_i)=(1,1)$; $d_i = 1$ jika berbeda; diabaikan ($w_i=0$) jika $(0,0)$   | $\{0,1\}$ |

Bobot $w_i = 0$ apabila salah satu nilai hilang, atau untuk biner asimetris ketika keduanya bernilai 0 (tidak menginformasikan apa-apa).

### Pemetaan Ordinal untuk Fitur `rating`

Fitur `rating` dalam dataset Netflix memuat peringkat dari dua sistem berbeda — film dan acara TV. Untuk memungkinkan perbandingan antar keduanya, dibuat **skala ordinal terpadu** dari tingkat kebebasan konten paling rendah ke paling tinggi:

| Peringkat | Level Ordinal | Keterangan                     |
| :-------- | :-----------: | :----------------------------- |
| G         |       1       | Semua usia (film)              |
| TV-Y      |       1       | Anak-anak sangat muda          |
| TV-Y7     |       2       | Anak 7 tahun ke atas           |
| TV-G      |       3       | Umum, semua usia (TV)          |
| PG        |       3       | Bimbingan orang tua disarankan |
| PG-13     |       4       | Orang tua sangat berhati-hati  |
| TV-PG     |       4       | Bimbingan orang tua (TV)       |
| TV-14     |       5       | Remaja 14 tahun ke atas        |
| R         |       6       | Terbatas — bawah 17 perlu wali |
| NC-17     |       7       | 17 tahun ke atas               |
| TV-MA     |       7       | Dewasa — konten eksplisit      |

### Implementasi Python

```python
import pandas as pd
import numpy as np

df = pd.read_csv('netflix_titles.csv')

# === Pemetaan ordinal rating ===
rating_order = {
    'G': 1, 'TV-Y': 1,
    'TV-Y7': 2,
    'TV-G': 3, 'PG': 3,
    'PG-13': 4, 'TV-PG': 4,
    'TV-14': 5,
    'R': 6,
    'NC-17': 7, 'TV-MA': 7
}
df['rating_rank'] = df['rating'].map(rating_order)

# === Parsing durasi menjadi angka ===
def parse_duration(row):
    if pd.isna(row['duration']):
        return np.nan
    if 'min' in str(row['duration']):
        return int(row['duration'].replace(' min', ''))
    elif 'Season' in str(row['duration']):
        return int(row['duration'].split()[0])
    return np.nan

df['duration_val'] = df.apply(parse_duration, axis=1)

# === Encoding biner asimetris: genre "Documentaries" ===
df['is_documentary'] = df['listed_in'].apply(
    lambda x: 1 if isinstance(x, str) and 'Documentar' in x else 0
)

# === Encoding biner simetris: type ===
df['type_bin'] = df['type'].map({'Movie': 0, 'TV Show': 1})

# === Range untuk normalisasi numerik ===
release_year_range = df['release_year'].max() - df['release_year'].min()
rating_n_levels = max(rating_order.values()) - min(rating_order.values())  # 7-1 = 6

# -------------------------------------------------------
# Ambil dua sampel: s1 (Movie) dan s2 (TV Show)
# -------------------------------------------------------
s1 = df[df['show_id'] == 's1'].iloc[0]  # Dick Johnson Is Dead
s2 = df[df['show_id'] == 's2'].iloc[0]  # Blood & Water

print("=== Perbandingan Dua Observasi ===")
print(f"{'Fitur':<20} {'s1 (Movie)':<25} {'s2 (TV Show)':<25}")
print("-" * 70)
print(f"{'type':<20} {s1['type']:<25} {s2['type']:<25}")
print(f"{'country':<20} {str(s1['country']):<25} {str(s2['country']):<25}")
print(f"{'release_year':<20} {s1['release_year']:<25} {s2['release_year']:<25}")
print(f"{'rating':<20} {str(s1['rating']):<25} {str(s2['rating']):<25}")
print(f"{'is_documentary':<20} {s1['is_documentary']:<25} {s2['is_documentary']:<25}")
print()

# -------------------------------------------------------
# Hitung jarak per komponen
# -------------------------------------------------------
# 1. type (biner simetris)
d_type = 0 if s1['type_bin'] == s2['type_bin'] else 1

# 2. country (nominal)
d_country = 0 if s1['country'] == s2['country'] else 1

# 3. release_year (numerik — dinormalisasi)
d_year = abs(s1['release_year'] - s2['release_year']) / release_year_range

# 4. rating (ordinal — dinormalisasi)
r1 = s1['rating_rank']
r2 = s2['rating_rank']
if pd.isna(r1) or pd.isna(r2):
    d_rating, w_rating = 0, 0
else:
    d_rating = abs(r1 - r2) / rating_n_levels
    w_rating = 1

# 5. is_documentary (biner asimetris)
a1, a2 = s1['is_documentary'], s2['is_documentary']
if a1 == 0 and a2 == 0:
    d_documentary, w_documentary = 0, 0   # keduanya 0 → diabaikan
else:
    d_documentary = 0 if a1 == a2 else 1
    w_documentary = 1

# -------------------------------------------------------
# Gower distance
# -------------------------------------------------------
components = [
    ('type (biner simetris)',   d_type,        1          ),
    ('country (nominal)',       d_country,     1          ),
    ('release_year (numerik)', d_year,        1          ),
    ('rating (ordinal)',        d_rating,      w_rating   ),
    ('documentary (bin.asim)', d_documentary, w_documentary),
]

print("=== Kontribusi Jarak per Fitur ===")
print(f"{'Fitur':<30} {'d_i':>8} {'w_i':>5}")
print("-" * 45)
for name, d, w in components:
    print(f"{name:<30} {d:>8.4f} {w:>5}")

total_d = sum(d * w for _, d, w in components)
total_w = sum(w for _, _, w in components)
gower = total_d / total_w if total_w > 0 else np.nan

print("-" * 45)
print(f"{'Jumlah':<30} {total_d:>8.4f} {total_w:>5}")
print(f"\nJarak Gower (s1 vs s2) = {total_d:.4f} / {total_w} = {gower:.4f}")
```

![Implementasi Python](./img/Screenshot_2026-03-04_06-29-35.png)

### Interpretasi Hasil

Jarak Gower antara **Dick Johnson Is Dead** (s1) dan **Blood & Water** (s2) adalah **0.7021** dari maksimum 1.0. Nilai ini menunjukkan bahwa kedua konten cukup berbeda satu sama lain. Berikut rincian kontribusi tiap fitur:

| Fitur                         | Nilai s1      | Nilai s2      | $d_i$  | Interpretasi                           |
| :---------------------------- | :------------ | :------------ | :----: | :------------------------------------- |
| `type` (biner simetris)       | Movie         | TV Show       | 1.0000 | Berbeda jenis konten sepenuhnya        |
| `country` (nominal)           | United States | South Africa  | 1.0000 | Negara asal berbeda                    |
| `release_year` (numerik)      | 2020          | 2021          | 0.0104 | Selisih 1 tahun dari rentang ~96 tahun |
| `rating` (ordinal)            | PG-13 (lvl 4) | TV-MA (lvl 7) | 0.5000 | Selisih 3 level dari total 6 level     |
| `listed_in` (biner asimetris) | Documentar→ 1 | Tidak ada → 0 | 1.0000 | s1 bertipe dokumenter, s2 tidak        |

Fitur `release_year` memberikan kontribusi paling kecil karena perbedaan 1 tahun sangat kecil relatif terhadap rentang dataset. Sebaliknya, perbedaan `type`, `country`, dan genre menyumbang penuh (1.0) karena keduanya benar-benar berbeda.

Nilai `(0, 0)` pada biner asimetris — misalkan kedua konten sama-sama bukan dokumenter — **tidak dihitung** karena tidak ada informasi yang dapat dibandingkan: "sama-sama tidak punya" tidak bermakna kedekatan.
