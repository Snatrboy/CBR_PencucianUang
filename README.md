# Sistem Case-Based Reasoning (CBR) — Analisis Putusan Pencucian Uang

**Mata Kuliah:** Penalaran Komputer  
**SubCPMK-3:** Siklus Case-Based Reasoning  
**Domain:** Pidana Khusus Pencucian Uang  
**Sumber Data:** Direktori Putusan Mahkamah Agung Republik Indonesia  
**Jumlah Dokumen:** 116 putusan PDF  

---

## Deskripsi Proyek

Sistem ini mengimplementasikan siklus **Case-Based Reasoning (CBR)** lengkap untuk menganalisis dan memprediksi amar putusan pengadilan pidana pencucian uang. Sistem memanfaatkan dua pendekatan retrieval:

1. **TF-IDF + Cosine Similarity** — pendekatan statistik
2. **Multilingual BERT (MiniLM)** — pendekatan text embedding
3. **SVM & Naive Bayes** — classifier machine learning di atas TF-IDF

Siklus CBR yang diimplementasikan:

```
[Case Base] → [Case Representation] → [Case Retrieval] → [Solution Reuse] → [Retain]
                                             ↑
                                       [Evaluation]
```

---

## Struktur Repository

```
CBR_PencucianUang/
│
├── data/
│   ├── raw/                        # Teks putusan hasil ekstraksi PDF (.txt)
│   ├── processed/
│   │   ├── cases.csv               # Dataset terstruktur semua kasus
│   │   ├── cases.json              # Dataset dalam format JSON
│   │   ├── cases_retained.csv      # Case base setelah retain step
│   │   ├── train_embeddings.npy    # BERT embeddings data train
│   │   └── svm_model.pkl           # Model SVM & NB tersimpan
│   ├── eval/
│   │   ├── queries.json            # Query uji + ground truth
│   │   ├── ground_truth.json       # Ground truth test set
│   │   ├── retrieval_metrics.csv   # Metrics TF-IDF vs BERT
│   │   ├── classifier_metrics.csv  # Metrics SVM vs NB
│   │   ├── classification_report.csv
│   │   ├── prediction_metrics.csv
│   │   ├── error_analysis.csv      # Analisis kegagalan prediksi
│   │   ├── confusion_matrix.png
│   │   ├── model_comparison.png
│   │   ├── amar_distribution.png
│   │   ├── wordcloud.png
│   │   ├── error_analysis_chart.png
│   │   ├── roc_curve.png
│   │   └── final_report.md         # Laporan akhir otomatis
│   └── results/
│       ├── predictions.csv         # Prediksi TF-IDF
│       └── predictions_bert.csv    # Prediksi BERT
│
├── logs/
│   ├── cleaning.log                # Log preprocessing per dokumen
│   └── failed_*.txt                # Log file yang gagal diproses
│
├── notebooks/
│   └── CBR_PencucianUang.ipynb     # Notebook utama (semua tahap)
│
├── requirements.txt                # Daftar library Python
└── README.md                       # Dokumentasi ini
```

---

## Persyaratan Sistem

- Python 3.9 atau lebih baru
- Google Colab (direkomendasikan) **atau** lingkungan lokal dengan GPU opsional
- Google Drive (untuk menyimpan data dan output)
- RAM minimal 8 GB (16 GB direkomendasikan untuk BERT)

---

## Instalasi

### Opsi A — Google Colab (Direkomendasikan)

1. Upload notebook `CBR_PencucianUang.ipynb` ke Google Colab
2. Jalankan **Cell 1** untuk instalasi otomatis:
   ```python
   !pip install pypdf2 pandas scikit-learn transformers torch sentence-transformers wordcloud
   ```
3. Pastikan file PDF putusan sudah ada di Google Drive pada path:
   ```
   /content/drive/MyDrive/CBR_PencucianUang/MA_Raw_Putusan/PencucianUang/
   ```

### Opsi B — Lingkungan Lokal

```bash
# 1. Clone repository
git clone https://github.com/<username>/CBR_PencucianUang.git
cd CBR_PencucianUang

# 2. Buat virtual environment
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Jalankan Jupyter Notebook
jupyter notebook notebooks/CBR_PencucianUang.ipynb
```

---

## Cara Menjalankan Pipeline End-to-End

Jalankan sel-sel notebook secara berurutan dari atas ke bawah. Berikut urutan dan fungsi setiap tahap:

### Tahap 1 — Membangun Case Base (Cell 1–5)

| Cell | Fungsi |
|------|--------|
| Cell 1 | Install library & import |
| Cell 2 | Mount Google Drive, buat struktur folder |
| Cell 3 | Definisi fungsi ekstraksi & pembersihan teks |
| Cell 4 | Definisi fungsi ekstraksi metadata |
| Cell 5 | Scan PDF → ekstrak → bersihkan → simpan ke `/data/raw/` + `cleaning.log` |

**Output:** `/data/raw/*.txt`, `/logs/cleaning.log`

### Tahap 2 — Case Representation (Cell 6–7)

| Cell | Fungsi |
|------|--------|
| Cell 6 | Buat DataFrame, normalisasi amar putusan, simpan CSV & JSON |
| Cell 7 | Train/test split 80:20, buat `queries.json` & `ground_truth.json` |

**Output:** `/data/processed/cases.csv`, `/data/processed/cases.json`, `/data/eval/queries.json`

### Tahap 3 — Case Retrieval (Cell 8–8B–9)

| Cell | Fungsi |
|------|--------|
| Cell 8  | Build TF-IDF retriever, fungsi `retrieve_tfidf()`, `predict_outcome()` |
| Cell 8B | SVM & Naive Bayes classifier di atas TF-IDF, cross-validation 5-fold |
| Cell 9  | BERT embedding, fungsi `retrieve_bert()` |

**Output:** `/data/processed/train_embeddings.npy`, `/data/processed/svm_model.pkl`

### Tahap 4 — Case Solution Reuse (Cell 10–11)

| Cell | Fungsi |
|------|--------|
| Cell 10 | Prediksi test set dengan TF-IDF + weighted/majority vote |
| Cell 11 | Prediksi test set dengan BERT + weighted/majority vote |

**Output:** `/data/results/predictions.csv`, `/data/results/predictions_bert.csv`

### Tahap 5 — Evaluasi Model (Cell 12–15)

| Cell | Fungsi |
|------|--------|
| Cell 12  | K-Fold Cross Validation (5-fold, TF-IDF) |
| Cell 13  | Accuracy, Precision, Recall, F1 — TF-IDF vs BERT |
| Cell 13B | Retain step — kasus benar masuk case base |
| Cell 14  | Confusion matrix + visualisasi perbandingan model |
| Cell 15  | Error analysis & analisis kegagalan |

**Output:** `/data/eval/retrieval_metrics.csv`, `/data/eval/classifier_metrics.csv`, visualisasi PNG

### Tahap 6 — Visualisasi & Laporan (Cell 16–18)

| Cell | Fungsi |
|------|--------|
| Cell 16 | Distribusi amar putusan + Word Cloud |
| Cell 17 | Demo interaktif (ipywidgets) |
| Cell 18 | Generate laporan akhir Markdown |

---

## Contoh Penggunaan Fungsi Utama

### Retrieve kasus mirip (TF-IDF)
```python
query = "Terdakwa terbukti melakukan pencucian uang dengan menyembunyikan asal usul uang melalui transaksi fiktif"

hasil = retrieve_tfidf(query, train_data, vectorizer, tfidf_matrix, top_k=5)
print(hasil[['case_id', 'amar_normalized', 'similarity']])
```

### Prediksi amar putusan
```python
prediksi, top_k_cases = predict_outcome(query, train_data, vectorizer, tfidf_matrix, k=5)
print(f"Prediksi vonis: {prediksi}")
```

### Retrieve kasus mirip (BERT)
```python
hasil_bert = retrieve_bert(query, train_data, train_embeddings, top_k=5)
print(hasil_bert[['case_id', 'amar_normalized', 'similarity']])
```

### Prediksi menggunakan SVM
```python
results, svm_label, confidence = retrieve_with_svm(
    query, train_data, tfidf_clf, svm_clf, top_k=5
)
print(f"Prediksi SVM: {svm_label} (confidence: {confidence:.2%})")
```

---

## Konfigurasi Path (Wajib Disesuaikan)

Sebelum menjalankan notebook, ubah dua variabel ini di **Cell 2** sesuai lokasi file di Google Drive Anda:

```python
# Path folder PDF putusan
SOURCE_PDF_FOLDER = "/content/drive/MyDrive/CBR_PencucianUang/MA_Raw_Putusan/PencucianUang"

# Path base project
BASE_PATH = "/content/drive/MyDrive/CBR_PencucianUang"
```

---

## Hasil Evaluasi

| Model | Accuracy | Precision | Recall | F1-Score |
|-------|----------|-----------|--------|----------|
| TF-IDF + Cosine Similarity | — | — | — | — |
| Multilingual BERT (MiniLM) | — | — | — | — |
| SVM Linear (TF-IDF) | — | — | — | — |
| Complement Naive Bayes | — | — | — | — |

> Nilai diisi otomatis setelah notebook dijalankan. Lihat `/data/eval/retrieval_metrics.csv` dan `/data/eval/classifier_metrics.csv` untuk hasil aktual.

---

## Teknologi yang Digunakan

| Library | Versi | Fungsi |
|---------|-------|--------|
| PyPDF2 | 3.0.1 | Ekstraksi teks dari PDF |
| pandas | 2.2.2 | Manipulasi data |
| scikit-learn | 1.5.0 | TF-IDF, SVM, NB, metrics |
| sentence-transformers | 3.0.1 | BERT embedding |
| torch | 2.3.0 | Backend PyTorch untuk BERT |
| matplotlib / seaborn | 3.9.0 | Visualisasi |
| wordcloud | 1.9.3 | Word cloud |
| ipywidgets | 8.1.3 | Demo interaktif |

---

## Anggota Tim

| Nama | NIM |
|------|-----|
| [Haitsam Dzaki Dasiyanto] | [202310370311232] |
| [Farrel Juan Abimanyu] | [202310370311368] |

**Program Studi:** Informatika  
**Fakultas:** Teknik  
**Universitas:** Universitas Muhammadiyah Malang  
**Tahun Akademik:** 2025/2026 (Genap)

---

## Lisensi

Repository ini dibuat untuk keperluan akademik mata kuliah Penalaran Komputer UMM.
