# Tugas Decision Tree

## Analisis Data: Pohon Keputusan (Decision Tree)

### 1. Konsep Dasar Pohon Keputusan

Pohon Keputusan adalah salah satu metode klasifikasi yang paling populer karena mudah diinterpretasikan oleh manusia. Strukturnya menyerupai diagram alur atau pohon terbalik, yang terdiri dari tiga komponen utama:

- **Node Akar (Root Node):** Mewakili seluruh kumpulan data dan merupakan pertanyaan atau atribut pertama yang membagi data.
- **Node Internal (Internal Node):** Mewakili fitur atau atribut yang sedang diuji. Setiap cabang dari node ini menunjukkan hasil dari pengujian tersebut.
- **Node Daun (Leaf Node):** Merupakan hasil akhir atau label kelas (keputusan). Di titik ini, data tidak bisa dibagi lagi.

Secara sederhana, algoritma ini bekerja dengan cara **"divide and conquer"**, yakni memecah data menjadi kelompok yang lebih kecil berdasarkan aturan IF-THEN hingga mencapai sebuah kesimpulan.

---

### 2. Membangun Pohon dengan Gain Ratio

Dalam membangun pohon keputusan, kita harus memilih atribut mana yang paling efektif untuk dijadikan pemecah (splitter) di setiap percabangan. Salah satu ukuran yang digunakan adalah **Gain Ratio**.

#### Mengapa menggunakan Gain Ratio?

Gain Ratio merupakan pengembangan dari _Information Gain_. Masalah pada _Information Gain_ biasa adalah kecenderungannya untuk memilih atribut yang memiliki banyak variasi nilai (misalnya: ID Pelanggan atau Tanggal), padahal atribut tersebut belum tentu berguna untuk prediksi. Gain Ratio hadir untuk "menghukum" atribut-atribut yang memiliki terlalu banyak cabang tersebut agar hasil model lebih objektif.

#### Cara Kerja Gain Ratio

Gain Ratio dihitung dengan membagi dua komponen utama:

1. **Information Gain:** Mengukur seberapa besar penurunan kekacauan (entropi) dalam data setelah dibagi berdasarkan suatu atribut. Semakin tinggi nilai Gain, semakin baik atribut tersebut dalam memisahkan kelas.
2. **Split Information:** Mengukur seberapa lebar dan seragam data terbagi. Jika suatu atribut membagi data menjadi banyak kelompok kecil, nilai _Split Info_ akan tinggi.

**Rumus Sederhana:**

$$\text{Gain Ratio} = \frac{\text{Information Gain}}{\text{Split Information}}$$

**Kesimpulan:**
Atribut dengan nilai **Gain Ratio tertinggi** akan dipilih sebagai node pemecah. Dengan metode ini, Pohon Keputusan yang dihasilkan akan lebih efisien dan terhindar dari bias terhadap atribut yang memiliki terlalu banyak nilai unik.

## Implementasi
