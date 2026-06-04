# 🥫 OCR Nutrition Label Dataset — Food & Beverage Packaging

> Dataset gambar crop teks informasi gizi dari kemasan makanan dan minuman, disiapkan untuk pelatihan model **CRNN (Convolutional Recurrent Neural Network)** dengan pendekatan **OCR berbasis sequence recognition**.

---

## 📋 Deskripsi Proyek

Proyek ini merupakan bagian dari pipeline persiapan data untuk sistem OCR yang mendeteksi dan membaca teks informasi gizi pada kemasan makanan dan minuman. Dataset ini mencakup lima kategori nutrisi utama: **Calories, Total Fat, Carbohydrates, Sugar, dan Sodium**.

Seluruh data telah melalui proses **crawling gambar → cropping region teks → pelabelan manual → quality cleaning → EDA**, dan kini siap diserahkan ke AI Engineer untuk proses pelatihan model CRNN.

---

## 📁 Struktur Dataset

```
ocr_crop_v3/
├── cleaned/
│   ├── test/                  # ✅ Gambar crop siap training (2391 gambar)
│   ├── valid/
│   │   └── labels.csv         # ✅ Label teks hasil pelabelan manual
│   └── eda_output/
│       ├── eda_dashboard.png       # Visualisasi EDA
│       └── rekomendasi_ai_engineer.txt
├── test/                      # Gambar asli sebelum cleaning
└── valid/                     # Label asli sebelum cleaning
```

---

## 📊 Statistik Dataset

| Atribut | Nilai |
|---|---|
| **Total gambar** | 2.391 |
| **Total label CSV** | 2.391 |
| **Match rate (CSV ↔ Gambar)** | 100.0% |
| **Kelas nutrisi** | 5 |
| **Resolusi rata-rata** | 596 × 64 px |
| **Rasio aspek median** | 8.88 |
| **Panjang teks median** | 22 karakter |
| **Label kosong** | 0 |
| **Duplikat file_name** | 0 |

---

## 🏷️ Kelas Nutrisi

| Kelas | Prefix File | Contoh Label |
|---|---|---|
| `calories` | `calories_XXXX.png` | `Calories: 250 kcal` |
| `fat` | `fat_XXXX.png` | `Total Fat: 8 g` |
| `carbs` | `carbs_XXXX.png` | `Carbohydrates: 30 g` |
| `sugar` | `sugar_XXXX.png` | `Sugar: 12 g` |
| `sodium` | `sodium_XXXX.png` | `Sodium: 480 mg` |

### Distribusi Kelas

```
sodium    : 491 gambar
carbs     : 481 gambar
calories  : 479 gambar
fat       : 478 gambar
sugar     : 462 gambar
```

Distribusi antar kelas sangat seimbang dengan **imbalance ratio = 1.1x** — tidak diperlukan teknik handling khusus seperti oversampling.

---

## 🔄 Alur Pembuatan Dataset

```
1. Pengumpulan gambar kemasan makanan & minuman
        ↓
2. Cropping region teks informasi gizi (per kategori nutrisi)
        ↓
3. Auto-labeling dengan EasyOCR → draft CSV
        ↓
4. Review & koreksi manual di Excel (2.490 gambar)
        ↓
5. Data Cleaning (hapus blur berat, gelap, overexposed → resize 64px)
        ↓
6. EDA & quality check final
        ↓
7. ✅ Dataset siap training (2.391 gambar + labels.csv)
```

---

## 🖼️ Visualisasi EDA

Dashboard EDA dihasilkan dari data **cleaned** (setelah proses cleaning).

![EDA Dashboard](https://github.com/ilmalyakinn/assets/blob/f43008efebe09a13fd3f086a34638cacc8724ec5/eda_dashboard.png)

> **Keterangan grafik (kiri ke kanan, atas ke bawah):**
> - **Distribusi Kelas** — jumlah gambar per kategori nutrisi, menunjukkan dataset seimbang
> - **Distribusi Panjang Teks** — sebaran panjang sequence label, median 22 karakter dengan dua puncak (~15 dan ~42 karakter) mencerminkan variasi format teks pendek dan panjang
> - **Sebaran Resolusi** — semua gambar berhasil dinormalisasi ke tinggi 64px, lebar bervariasi 200–1400px proporsional
> - **Distribusi Rasio Aspek** — median 8.88, distribusi right-skewed wajar untuk teks horizontal kemasan
> - **Blur Score** — mayoritas gambar memiliki skor sangat tinggi (>400), menandakan kualitas ketajaman sangat baik pasca cleaning
> - **Distribusi Brightness** — distribusi normal terpusat di 125–175, semua gambar dalam rentang optimal
> - **Anomali per Kelas** — kelas `calories` memiliki anomali terbanyak (62), namun tetap proporsional terhadap jumlah datanya
> - **Distribusi Lebar per Kelas** — `carbs` dan `calories` memiliki lebar lebih bervariasi, wajar karena teks labelnya lebih panjang

---

## ✅ Kualitas Dataset — Ringkasan untuk AI Engineer

```
REKOMENDASI UNTUK AI ENGINEER — CRNN TRAINING
============================================================
✅  Kualitas ketajaman gambar baik (0.5% blur).
✅  Class balance cukup baik (ratio=1.1x).
✅  Tinggi gambar cukup untuk CRNN (median=64px). Target resize: 32px atau 64px.
✅  Tidak ada label kosong dalam CSV.
✅  Konsistensi CSV-gambar sangat baik (100.0%).

📌 Saran preprocessing CRNN:
   - Resize semua gambar ke tinggi tetap (32px atau 64px), lebar proporsional
   - Normalisasi pixel ke [0,1] atau [-1,1]
   - Gunakan CTC Loss untuk training sequence labeling
   - Split dataset: 80% train / 10% val / 10% test (stratified per kelas)
```

---

## 🧹 Proses Data Cleaning

Sebelum masuk EDA final, dataset melewati proses cleaning otomatis dengan kriteria berikut:

| Filter | Threshold | Tindakan |
|---|---|---|
| Blur berat | Laplacian variance < 15 | Hapus |
| Blur ringan | Laplacian variance 15–50 | Sharpen lalu pertahankan |
| Terlalu gelap | Mean brightness < 30 | Hapus |
| Overexposed | Mean brightness > 225 | Hapus |
| Rasio aspek ekstrem | > 15.0 | **Pertahankan** (CRNN handle via CTC) |
| Resize | Target height = 64px | Lebar proporsional |

**Data asli tidak diubah.** Hasil cleaning disimpan di folder `cleaned/` yang terpisah.

Dari **2.490 gambar** input, sebanyak **2.391 gambar** lolos cleaning (**99 gambar dihapus / ~4%**).

---

## 📄 Format Label CSV

File `labels.csv` memiliki dua kolom:

```
file_name,text
calories_0001.png,Calories: 250 kcal
fat_0001.png,Total Fat 8 g
carbs_0001.png,Carbohydrates 30 g
sugar_0001.png,Sugar 12 g
sodium_0001.png,Sodium 480 mg
```

Label diisi **secara otomatis menggunakan EasyOCR berdasarkan tulisan asli pada gambar lalu dilakukan pengecekan manual** oleh data scientist setelah review gambar satu per satu, memastikan akurasi teks sesuai dengan konten gambar crop.

---

## 📦 Download Dataset

| Versi | Kondisi | Jumlah Gambar | Link |
|---|---|---|---|
| v1.0-raw | Mentah, sebelum cleaning | 2.490 | [Download](https://github.com/ilmalyakin-n/ocr-nutrition-label-dataset/releases/tag/untagged-3c6c4b733839c125b72f) |
| v2.0-cleaned | Bersih, siap training | 2.391 | [Download](https://github.com/ilmalyakin-n/ocr-nutrition-label-dataset/releases/tag/v2.0-cleaned) |

> Untuk akses kode program dan seluruh file proyek lengkap (notebook, EDA output, dokumentasi):
> 🔗 **[Google Drive — Proyek Lengkap](https://drive.google.com/drive/folders/16s37U7-6pC2BhObfmi-6iRmdQ5KVm4X5?usp=sharing)**
---

## 🛠️ Tools & Teknologi

| Tahap | Tools |
|---|---|
| Auto-labeling OCR | EasyOCR |
| Review & koreksi label | Microsoft Excel |
| Data cleaning | OpenCV, pandas, Python |
| EDA & visualisasi | matplotlib, seaborn |
| Environment | Google Colab (GPU T4) |

---

## 📌 Catatan untuk AI Engineer

1. **Gunakan `cleaned/test/`** sebagai folder gambar training — bukan folder `test/` di root
2. **Gunakan `cleaned/valid/labels.csv`** sebagai file label — sudah diverifikasi manual, match rate 100%
3. **CTC Loss wajib digunakan** karena lebar gambar bervariasi (200–1400px)
4. **Charset** mencakup huruf Latin (A-Z, a-z), angka (0-9), spasi, titik dua, titik, dan satuan (g, mg, kcal)
5. **Jangan resize ulang tinggi** — gambar sudah di-resize ke 64px, cukup normalisasi pixel saja
6. 
---

*Dataset ini disiapkan sebagai bagian dari proyek pengembangan sistem OCR deteksi informasi gizi pada kemasan makanan dan minuman.*
