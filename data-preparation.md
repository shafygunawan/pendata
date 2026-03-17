# Data Preparation

## Tugas

1. Missing values : menyelesaikan dengan WKNN (manual) + code menghitung WKNN
2. Normalisasi data (materi)
    - macam macam normalisasi data dan beri contoh hasil normalisasi data
    - Gunakan library sklearn untuk melakukan normalisasi data atau membuat fungsi sendiri

Penyelesaian:

| Data | Nilai (x) | Jam Belajar (y) |
| ---- | --------- | --------------- |
| 1    | 80        | 2               |
| 2    | 90        | 3               |
| 3    | 70        | 1               |
| 4    | ?         | 2               |

### 1. Menyelesaikan Missing Values dengan WKNN

#### Manual

| Tetangga | Selisih (yi - yj) | Jarak (d) | Bobot (w) |
| -------- | ----------------- | --------- | --------- |
| 1        | (2 - 2) = 0       | 0         | 0         |
| 2        | (2 - 3) = -1      | 1         | 1         |
| 3        | (2 - 1) = 1       | 1         | 1         |

Pembilang:

| Data  | Bobot (w) \* Nilai (xj) |
| ----- | ----------------------- |
| 1     | 0 \* 80 = 0             |
| 2     | 1 \* 90 = 90            |
| 3     | 1 \* 70 = 70            |
| Total | 160                     |

Penyebut:

| Data  | Bobot (w) |
| ----- | --------- |
| 1     | 0         |
| 2     | 1         |
| 3     | 1         |
| Total | 2         |

```
x4 = Pembilang / Penyebut
x4 = 160 / 2
x4 = 80
```

#### Code Menghitung WKNN

```python
import numpy as np
# Data
data = np.array([[80, 2], [90, 3], [70, 1], [np.nan, 2]])
# Menghitung jarak
distances = np.abs(data[:, 1] - data[3, 1])
# Menghitung bobot
weights = 1 / (distances + 1e-5)  # Menambahkan epsilon untuk menghindari pembagian dengan nol
# Menghitung nilai yang hilang
missing_value = np.sum(weights[:-1] * data[:-1, 0]) / np.sum(weights[:-1])
print(missing_value)
```

### 2. Normalisasi Data

Normalisasi data adalah proses mengubah nilai-nilai dalam dataset ke dalam skala yang sama. Ada beberapa metode normalisasi data, di antaranya:

- Min-Max Normalization: Mengubah nilai ke dalam rentang [0, 1].
- Z-Score Normalization: Mengubah nilai berdasarkan rata-rata dan standar deviasi.

#### Min-Max Normalization

Manual:

| Data | Nilai (x) | Jam Belajar (y) | Normalisasi (x')            | Normalisasi (y')        |
| ---- | --------- | --------------- | --------------------------- | ----------------------- |
| 1    | 80        | 2               | (80 - 70) / (90 - 70) = 0.5 | (2 - 1) / (3 - 1) = 0.5 |
| 2    | 90        | 3               | (90 - 70) / (90 - 70) = 1.0 | (3 - 1) / (3 - 1) = 1.0 |
| 3    | 70        | 1               | (70 - 70) / (90 - 70) = 0.0 | (1 - 1) / (3 - 1) = 0.0 |
| 4    | ?         | 2               | ?                           | (2 - 1) / (3 - 1) = 0.5 |

Code menggunakan sklearn:

```python
from sklearn.preprocessing import MinMaxScaler
import numpy as np
# Data
data = np.array([[80, 2], [90, 3], [70, 1], [np.nan, 2]])
# Normalisasi
scaler = MinMaxScaler()
normalized_data = scaler.fit_transform(data)
print(normalized_data)
```

#### Z-Score Normalization

Manual:

```
Rata-rata (mean) untuk x = (80 + 90 + 70) / 3
                         = 80
Standar deviasi (std) untuk x = sqrt(((80-80)^2 + (90-80)^2 + (70-80)^2) / 3)
                              = sqrt((0 + 100 + 100) / 3)
                              = sqrt(66.67)
                              = 8.16
Rata-rata (mean) untuk y = (2 + 3 + 1 + 2) / 4
                         = 2
Standar deviasi (std) untuk y = sqrt(((2-2)^2 + (3-2)^2 + (1-2)^2 + (2-2)^2) / 4)
                              = sqrt((0 + 1 + 1 + 0) / 4)
                              = sqrt(0.5)
                              = 0.71
```

| Data | Nilai (x) | Jam Belajar (y) | Normalisasi (x')         | Normalisasi (y')       |
| ---- | --------- | --------------- | ------------------------ | ---------------------- |
| 1    | 80        | 2               | (80 - 80) / 8.16 = 0.0   | (2 - 2) / 0.71 = 0.0   |
| 2    | 90        | 3               | (90 - 80) / 8.16 = 1.22  | (3 - 2) / 0.71 = 1.41  |
| 3    | 70        | 1               | (70 - 80) / 8.16 = -1.22 | (1 - 2) / 0.71 = -1.41 |
| 4    | ?         | 2               | ?                        | (2 - 2) / 0.71 = 0.0   |

Code menggunakan sklearn:

```python
from sklearn.preprocessing import StandardScaler
import numpy as np
# Data
data = np.array([[80, 2], [90, 3], [70, 1], [np.nan, 2]])
# Normalisasi
scaler = StandardScaler()
normalized_data = scaler.fit_transform(data)
print(normalized_data)
```
