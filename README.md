# Deteksi Dini Penyakit Jantung

Proyek klasifikasi biner untuk memprediksi apakah seorang pasien terindikasi penyakit jantung (`HeartDisease` = 1) atau tidak (0), berdasarkan data klinis dasar dan hasil pemeriksaan saat olahraga. Seluruh alur kerja, dari eksplorasi sampai pemodelan, ada di satu notebook: `deteksi-jantung.ipynb`.

Hasil terbaik di data uji: **Decision Tree (max_depth=3)** dengan akurasi 84,1% dan recall 87,5%.

> Proyek ini untuk latihan dan portofolio. Model tidak divalidasi secara klinis dan tidak boleh dipakai untuk diagnosis.

## Latar belakang

Menurut WHO, penyakit jantung iskemik adalah penyebab kematian nomor satu di dunia (8,6 juta kematian pada 2021). Makin awal penderita terdeteksi, makin besar peluang penanganannya berhasil. Pertanyaan yang ingin dijawab di sini:

- **Bisnis:** bisakah penyakit jantung dideteksi cukup dini supaya penderita bisa segera ditangani?
- **Teknis:** dari 11 fitur pada `heart.csv`, seberapa akurat kita bisa memprediksi `HeartDisease`?

## Dataset

Sumber: [arubhasy/dataset](https://github.com/arubhasy/dataset/blob/main/heart.csv), disimpan di repo ini sebagai `heart.csv`.

918 baris, 11 fitur, 1 target.

| Fitur | Keterangan |
|---|---|
| `Age` | Usia (tahun) |
| `Sex` | Jenis kelamin (M/F) |
| `ChestPainType` | Jenis nyeri dada: TA, ATA, NAP, ASY |
| `RestingBP` | Tekanan darah saat istirahat (mmHg) |
| `Cholesterol` | Kolesterol serum (mg/dL) |
| `FastingBS` | Gula darah puasa > 120 mg/dL (1 = ya, 0 = tidak) |
| `RestingECG` | Hasil EKG istirahat: Normal, ST, LVH |
| `MaxHR` | Detak jantung maksimum |
| `ExerciseAngina` | Angina akibat olahraga (Y/N) |
| `Oldpeak` | Depresi segmen ST akibat olahraga relatif terhadap istirahat |
| `ST_Slope` | Kemiringan segmen ST saat puncak olahraga: Up, Flat, Down |
| `HeartDisease` | Target: 1 = terkena, 0 = tidak |

Kelasnya cukup seimbang (sekitar 55% positif), jadi akurasi masih layak dipakai sebagai metrik, dilengkapi precision, recall, F1, dan AUC-ROC.

## Alur pengerjaan

**1. Eksplorasi.** Distribusi fitur, korelasi Pearson (numerik), ANOVA (kategorikal vs numerik), dan Cramér's V (kategorikal vs kategorikal).

Beberapa temuan awal: data didominasi laki-laki (sekitar 4:1), mayoritas pasien punya nyeri dada tipe ASY (tanpa gejala khas), dan ada nilai yang jelas tidak masuk akal, seperti `Age` 0 dan 177, serta `RestingBP` dan `Cholesterol` bernilai 0.

<img src="images/1.Distribusi%20variabel%20kategorikal.png" alt="Distribusi variabel kategorikal" width="720">

**2. Validasi dan pembersihan.**
- `Age`: 7 nilai kosong diisi median.
- `Sex`: 10 baris kosong dibuang (tersisa 908 baris).
- Outlier pada `Age`, `RestingBP`, `Cholesterol`, `MaxHR` ditangani dengan IQR; nilai di atas batas atas diganti Q3, di bawah batas bawah diganti Q1.

<table>
  <tr>
    <td align="center"><img src="images/4.Box%20plot%20variabel%20numerik%28sebelum%20cleaning%29.png" alt="Box plot sebelum cleaning" width="380"><br><sub>Sebelum</sub></td>
    <td align="center"><img src="images/5.Box%20plots%20variabel%20numerik%20%28after%20cleaning%29.png" alt="Box plot sesudah cleaning" width="380"><br><sub>Sesudah</sub></td>
  </tr>
</table>

**3. Encoding dan seleksi fitur.** Kategorikal di-encode dengan `OrdinalEncoder`. Fitur dipilih lewat korelasi terhadap target, Feature Importance Random Forest, dan nilai SHAP dari XGBoost.

<table>
  <tr>
    <td><img src="images/6.Heatmap%20matriks%20korelasi%20antar%20variabel.png" alt="Heatmap korelasi" width="420"></td>
    <td><img src="images/7.Skor%20SHAP.png" alt="Skor SHAP" width="420"></td>
  </tr>
</table>

`ST_Slope` konsisten jadi fitur paling berpengaruh di ketiga metode, disusul fitur seperti `ChestPainType`, `Oldpeak`, `MaxHR`, dan `ExerciseAngina` (urutannya sedikit berbeda antar metode). Secara medis ini masuk akal: segmen ST yang datar atau turun saat olahraga adalah salah satu tanda iskemia.

Lima fitur yang akhirnya dipakai untuk model: `ST_Slope`, `ChestPainType`, `Oldpeak`, `Cholesterol`, `Sex`.

**4. Konstruksi data.** Min-max scaling, standarisasi, binning `Age`, dan PCA 2 komponen dicoba di notebook (PCA hanya menjelaskan 37% varians). Hasilnya tidak dipakai di tahap pemodelan.

**5. Pemodelan.** Split hold-out 80/20 (`random_state=42`): 726 data latih, 182 data uji. Model yang dibandingkan: Decision Tree, Random Forest, Logistic Regression, dan AutoML ([MLJAR](https://github.com/mljar/mljar-supervised), mode Explain, batas waktu 300 detik).

## Hasil

Evaluasi di 182 data uji:

| Model | Accuracy | Precision | Recall | F1 | AUC-ROC |
|---|---|---|---|---|---|
| Decision Tree (depth 3) | **0,8407** | **0,8317** | 0,8750 | **0,8528** | **0,8985** |
| Decision Tree (depth 4) | 0,8242 | 0,8019 | 0,8854 | 0,8416 | 0,8741 |
| Decision Tree (depth 5) | 0,8242 | 0,7963 | **0,8958** | 0,8431 | 0,8640 |
| Random Forest (20 pohon) | 0,8022 | 0,8000 | 0,8333 | 0,8163 | 0,8633 |
| Random Forest (50 pohon) | 0,8077 | 0,8081 | 0,8333 | 0,8205 | 0,8649 |
| Logistic Regression | 0,8077 | 0,8081 | 0,8333 | 0,8205 | 0,8550 |
| AutoML (Ensemble) | 0,8242 | 0,8137 | 0,8646 | 0,8384 | 0,8698 |

Model paling sederhana justru yang terbaik di data uji. Karena ini deteksi dini, recall (kemampuan menangkap pasien yang benar-benar sakit) sebenarnya lebih penting dari precision. Decision Tree depth 5 punya recall tertinggi, tapi dengan harga lebih banyak alarm palsu.

Model AutoML punya akurasi 0,90 di data latih dan 0,82 di data uji, selisih sekitar 0,08 di semua metrik. Ini tanda overfitting, jadi keunggulannya di leaderboard internal (sekitar 0,90 di validasi) tidak terbawa ke data uji.

## Struktur repo

```
.
├── deteksi-jantung.ipynb    # notebook utama
├── heart.csv                # dataset
├── automl_object.joblib     # objek AutoML hasil training
├── AutoML_1/                # laporan dan artefak MLJAR (leaderboard, tiap model)
└── images/                  # gambar yang dipakai di README
```

## Menjalankan ulang

```bash
git clone https://github.com/MAndrataZ/deteksi-dini-penyakit-jantung.git
cd deteksi-dini-penyakit-jantung

pip install numpy pandas matplotlib seaborn scipy statsmodels scikit-learn xgboost shap mljar-supervised graphviz joblib jupyter
jupyter notebook deteksi-jantung.ipynb
```

Notebook membaca `heart.csv` dari direktori yang sama, jadi jalankan dari root repo. Untuk visualisasi pohon keputusan, program `graphviz` perlu terpasang di sistem, bukan hanya paketnya di Python. Notebook awalnya dijalankan di Google Colab (sel instalasi memakai `!pip`).

Untuk memuat model AutoML yang sudah tersimpan:

```python
import joblib
automl = joblib.load("automl_object.joblib")
```

Gunakan versi `mljar-supervised` yang sama dengan saat training, karena objek hasil `joblib` bisa gagal dimuat di versi yang berbeda.

## Catatan dan keterbatasan

- **Preprocessing dilakukan sebelum split.** Median, batas IQR, dan seleksi fitur dihitung dari seluruh data, termasuk data uji. Ini bisa membuat skor sedikit lebih optimis dari kenyataan. Perbaikannya: split dulu, lalu fit semua langkah preprocessing hanya di data latih (misalnya lewat `Pipeline`).
- **Evaluasi hanya satu kali split** dengan 182 data uji. Selisih satu-dua poin persentase antar model belum tentu bermakna. Cross-validation akan memberi gambaran yang lebih stabil.
- **Outlier diganti dengan Q1/Q3**, bukan dibuang. Termasuk di dalamnya nilai `Cholesterol` = 0 yang kemungkinan besar data hilang, bukan outlier sungguhan.
- **`OrdinalEncoder` dipakai pada fitur nominal** (misalnya `ChestPainType`), sehingga model pohon melihat urutan yang sebenarnya tidak ada. One-hot encoding lebih tepat.
- **Data tidak seimbang menurut jenis kelamin** (sekitar 4:1), jadi performa pada pasien perempuan perlu dicek terpisah sebelum kesimpulan apa pun diambil.
- Output evaluasi untuk Decision Tree depth 4 dan 5 di notebook masih berlabel "max_depth=3". Angkanya benar, hanya labelnya yang belum diperbarui.

## Rencana lanjutan

- Pindahkan preprocessing ke `Pipeline` scikit-learn dan tambahkan cross-validation.
- Tuning hyperparameter (grid atau Optuna) dan bandingkan lewat metrik yang menekankan recall.
- Cek performa per kelompok jenis kelamin dan usia.
