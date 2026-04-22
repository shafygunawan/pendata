# UTS

```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.impute import SimpleImputer
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, classification_report

df = pd.read_csv('dataset_kesuburan_tanah_missing.xlsx - Dataset.csv')

columns_to_convert_to_numeric = [
    'pH Tanah', 'N Total (%)', 'P Tersedia (ppm)', 'K Tersedia (meq/100g)',
    'C Organik (%)', 'KTK (meq/100g)', 'Kejenuhan Basa (%)', 'Kadar Air (%)',
    'Bulk Density (g/cm³)'
]

for col in columns_to_convert_to_numeric:
    if col in df.columns:
        df[col] = df[col].astype(str).str.replace(',', '.', regex=False)
        df[col] = pd.to_numeric(df[col], errors='coerce')

df.fillna(df.median(numeric_only=True), inplace=True)

df['Tekstur Tanah'] = df['Tekstur Tanah'].fillna(df['Tekstur Tanah'].mode()[0])

df['Label'] = LabelEncoder().fit_transform(df['Label'])
df = pd.get_dummies(df, columns=['Tekstur Tanah'], drop_first=True)

X = df.drop('Label', axis=1)
y = df['Label']

X_scaled = StandardScaler().fit_transform(X)

X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.2, random_state=42, stratify=y
)

knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)
y_pred = knn.predict(X_test)

print("--- Hasil Evaluasi KNN ---")
print(f"Accuracy  : {accuracy_score(y_test, y_pred):.4f}")
print(f"Precision : {precision_score(y_test, y_pred):.4f}")
print(f"Recall    : {recall_score(y_test, y_pred):.4f}")
print(f"F1-Score  : {f1_score(y_test, y_pred):.4f}")

print("\nDetail Laporan Klasifikasi:")
print(classification_report(y_test, y_pred, target_names=['Tidak Subur', 'Subur']))
```
