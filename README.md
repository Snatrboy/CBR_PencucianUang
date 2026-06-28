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
│   ├── raw/              # Taruh semua file PDF putusan di sini sebelum menjalankan notebook
│   ├── processed/        # Otomatis terisi: cases.csv, cases.json, embeddings, model
│   ├── eval/             # Otomatis terisi: metrics, confusion matrix, error analysis
│   └── results/          # Otomatis terisi: predictions.csv, predictions_bert.csv
│
├── logs/                 # Otomatis terisi: cleaning.log, failed files log
│
├── notebooks/
│   └── CBR_PencucianUang.ipynb   # Notebook utama (semua tahap CBR)
│
├── requirements.txt      # Daftar library Python
└── README.md             # Dokumentasi ini
```

> **Catatan:** Folder `data/processed/`, `data/eval/`, `data/results/`, dan `logs/`
> akan terisi otomatis saat notebook dijalankan. Satu-satunya folder yang perlu
> diisi manual adalah `data/raw/` — taruh semua file PDF putusan di sana.

---

## Persyaratan Sistem

- Python 3.9 atau lebih baru
- RAM minimal 8 GB (16 GB direkomendasikan untuk BERT)
- Tidak memerlukan GPU (CPU sudah cukup)

---

## Instalasi & Cara Menjalankan

### 1. Clone repository

```bash
git clone https://github.com/haitsam/CBR_PencucianUang.git
cd CBR_PencucianUang
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Taruh file PDF putusan

Salin semua file PDF putusan ke folder `data/raw/`:

```
CBR_PencucianUang/
└── data/
    └── raw/
        ├── putusan_408_pk_pid.sus_2019_xxx.pdf
        ├── putusan_2401_k_pid.sus_2018_xxx.pdf
        └── ... (116 file PDF)
```

### 4. Jalankan notebook

```bash
jupyter notebook notebooks/CBR_PencucianUang.ipynb
```

Jalankan semua cell dari atas ke bawah secara berurutan.
Semua output (CSV, model, visualisasi, log) akan otomatis tersimpan
ke folder `data/` dan `logs/` di dalam project.

---

## Pipeline End-to-End

### Tahap 1 — Membangun Case Base (Cell 1–5)

| Cell | Fungsi |
|------|--------|
| Cell 1 | Install library & import |
| Cell 2 | Setup path lokal otomatis, buat struktur folder |
| Cell 3 | Definisi fungsi ekstraksi & pembersihan teks |
| Cell 4 | Definisi fungsi ekstraksi metadata |
| Cell 5 | Scan PDF → ekstrak → bersihkan → simpan ke `data/raw/*.txt` + `logs/cleaning.log` |

**Output:** `data/raw/*.txt`, `logs/cleaning.log`

### Tahap 2 — Case Representation (Cell 6–7)

| Cell | Fungsi |
|------|--------|
| Cell 6 | Buat DataFrame, normalisasi amar putusan, simpan CSV & JSON |
| Cell 7 | Train/test split 80:20, buat `queries.json` & `ground_truth.json` |

**Output:** `data/processed/cases.csv`, `data/processed/cases.json`, `data/eval/queries.json`

### Tahap 3 — Case Retrieval (Cell 8, 8B, 9)

| Cell | Fungsi |
|------|--------|
| Cell 8  | Build TF-IDF retriever, fungsi `retrieve_tfidf()`, `predict_outcome()` |
| Cell 8B | SVM & Naive Bayes classifier di atas TF-IDF, cross-validation 5-fold |
| Cell 9  | BERT embedding, fungsi `retrieve_bert()` |

**Output:** `data/processed/train_embeddings.npy`, `data/processed/svm_model.pkl`

### Tahap 4 — Case Solution Reuse (Cell 10–11)

| Cell | Fungsi |
|------|--------|
| Cell 10 | Prediksi test set dengan TF-IDF + weighted/majority vote |
| Cell 11 | Prediksi test set dengan BERT + weighted/majority vote |

**Output:** `data/results/predictions.csv`, `data/results/predictions_bert.csv`

### Tahap 5 — Evaluasi Model (Cell 12–15)

| Cell | Fungsi |
|------|--------|
| Cell 12  | K-Fold Cross Validation 5-fold (TF-IDF) |
| Cell 13  | Accuracy, Precision, Recall, F1 — TF-IDF vs BERT |
| Cell 13B | Retain step — kasus prediksi benar masuk case base |
| Cell 14  | Confusion matrix + visualisasi perbandingan model |
| Cell 15  | Error analysis & analisis kegagalan |

**Output:** `data/eval/retrieval_metrics.csv`, `data/eval/classifier_metrics.csv`, file PNG visualisasi

### Tahap 6 — Visualisasi & Laporan (Cell 16–18)

| Cell | Fungsi |
|------|--------|
| Cell 16 | Distribusi amar putusan + Word Cloud |
| Cell 17 | Demo interaktif (ipywidgets) |
| Cell 18 | Generate laporan akhir Markdown |

**Output:** `data/eval/final_report.md`

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

## Hasil Evaluasi

| Model | Accuracy | Precision | Recall | F1-Score |
|-------|----------|-----------|--------|----------|
| TF-IDF + Cosine Similarity | — | — | — | — |
| Multilingual BERT (MiniLM) | — | — | — | — |
| SVM Linear (TF-IDF) | — | — | — | — |
| Complement Naive Bayes | — | — | — | — |

> Nilai diisi otomatis setelah notebook dijalankan.
> Lihat `data/eval/retrieval_metrics.csv` dan `data/eval/classifier_metrics.csv` untuk hasil aktual.

---

## Teknologi yang Digunakan

| Library | Versi | Fungsi |
|---------|-------|--------|
| PyPDF2 | 3.0.1 | Ekstraksi teks dari PDF |
| pandas | 2.2.2 | Manipulasi data |
| numpy | 1.26.4 | Komputasi numerik |
| scikit-learn | 1.5.0 | TF-IDF, SVM, NB, metrics |
| sentence-transformers | 3.0.1 | BERT embedding |
| torch | 2.3.0 | Backend PyTorch untuk BERT |
| matplotlib | 3.9.0 | Visualisasi |
| seaborn | 1.13.2 | Visualisasi statistik |
| wordcloud | 1.9.3 | Word cloud |
| ipywidgets | 8.1.3 | Demo interaktif |

---

## Anggota Tim

| Nama | NIM |
|------|-----|
| Haitsam Dzaki Dasiyanto | 202310370311232 |
| Farrel Juan Abimanyu | 202310370311368 |

**Program Studi:** Informatika  
**Fakultas:** Teknik  
**Universitas:** Universitas Muhammadiyah Malang  
**Tahun Akademik:** 2025/2026 (Genap)

---

## Lisensi

Repository ini dibuat untuk keperluan akademik mata kuliah Penalaran Komputer UMM.