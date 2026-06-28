
# Laporan Akhir Proyek CBR — Pencucian Uang

**Tanggal**        : 2026-06-28 13:54:08
**Jumlah Dokumen** : 116
**Domain Perkara** : Pidana Khusus Pencucian Uang
**Train/Test Split**: 80:20 (stratified)

## Ringkasan Eksekutif

Sistem **Case-Based Reasoning (CBR)** dibangun untuk analisis putusan pidana pencucian uang Mahkamah Agung RI. Data leakage dihindari dengan split train/test eksplisit; TF-IDF difit hanya pada data train.

## Hasil Evaluasi

| Metrik       | TF-IDF        | BERT (MiniLM)  |
|--------------|---------------|----------------|
| Accuracy     | 0.4545       | 0.4091         |
| Precision    | 0.4432       | 0.4091         |
| Recall       | 0.4545       | 0.4091         |
| F1-Score     | 0.4207       | 0.3999         |
| Precision@3  | 0.7273       | —              |
| Precision@5  | 0.7727       | —              |
| CV 5-Fold Acc| 0.4723 ±0.0705| —              |

## Distribusi Amar Putusan (Top 10)

amar_normalized
Berat (9+ tahun)      45
Ringan (1-4 tahun)    31
Sedang (5-8 tahun)    23
Tidak diketahui       10
Bebas                  7

## Siklus CBR — Status

- ✅ Case Base     : 116 dokumen (≥30)
- ✅ Case Representation: Metadata lengkap + CSV/JSON
- ✅ Case Retrieval : TF-IDF (max_features=3000) + BERT (MiniLM-384d)
- ✅ Solution Reuse : Majority Vote + Weighted Vote
- ✅ Evaluation    : Accuracy, Precision, Recall, F1, K-Fold CV, Error Analysis

## Analisis Kegagalan

- Kasus dengan similarity rendah cenderung gagal diprediksi (rejection)
- Pencucian uang bervariasi dalam modus → teks antar dokumen kurang seragam
- Kelas dengan sampel sedikit (long-tail) sulit diprediksi dengan benar

## Rekomendasi

1. Fine-tune IndoBERT (indobenchmark/indobert-base-p1) dengan data hukum
2. Tambah fitur: jumlah kerugian, modus operandi, pasal yang diputuskan
3. Implementasi threshold confidence untuk rejection
4. Ensemble TF-IDF + BERT dengan score fusion

---
*Laporan digenerate otomatis oleh sistem CBR.*
