# UTS

knime:

![Screenshot KNIME](./img/Screenshot_2026-04-22_11-24-36.png)

1. Import dataset ke KNIME dengan menggunakan CSV.

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-38-07.png)

2. Lakukan pembersihan data dengan mengubah tipe data string menjadi numerik untuk kolom-kolom yang relevan (pH Tanah, N Total (%), P Tersedia (ppm), K Tersedia (meq/100g), C Organik (%), KTK (meq/100g), Kejenuhan Basa (%), Tekstur Tanah, Kadar Air (%), Bulk Density (g/cm³)).

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-40-16.png)

3. Missing value pada kolom numerik diisi dengan median, sedangkan untuk kolom kategorikal diisi dengan modus.

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-40-55.png)

4. Melakukan partisi data menjadi data latih dan data uji dengan rasio 80:20.

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-41-29.png)

5. Melakukan normalisasi data menggunakan z-score.

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-42-00.png)

6. Membangun model KNN dengan k=3 dan melatihnya menggunakan data latih.

    ![Screenshot KNIME](./img/Screenshot_2026-04-22_11-42-24.png)

7. Menilai performa model menggunakan metrik evaluasi seperti accuracy, precision, recall, dan F1-score.

    ![Screenshot Performa Model](./img/Screenshot_2026-04-22_11-35-51.png)
