
# Inferensi Model YOLO untuk Ekstraksi Label Nutrisi (Nutrition Facts)

Repositori ini berisi *notebook* inferensi dasar untuk mendeteksi dan mengekstrak label pada tabel informasi nilai gizi (nutrition facts) menggunakan **YOLO (You Only Look Once)**, dilanjutkan dengan proses *Optical Character Recognition* (OCR) menggunakan **Tesseract**.

Tujuan utamanya adalah mendapatkan koordinat (Bounding Box) dari setiap komponen nutrisi (`gula`, `lemak`, `kalori`, dll.) dan kemudian melakukan OCR hanya pada segmen gambar yang di-*crop* tersebut.

## ⚙️ KESIMPULAN OPTIMAL (MODEL)

Berdasarkan pengujian, hasil deteksi optimal diperoleh dengan **preprocessing gambar minimal**, yaitu hanya **resize** gambar input menjadi **640x640** sebelum dimasukkan ke model YOLO.

* **Optimal:** Resize menjadi 640x640.
* **Belum Optimal:** Percobaan penambahan *upscaling* dan *deskewing* pada gambar input (sebelum deteksi YOLO) belum menunjukkan peningkatan hasil yang signifikan.

---

## 🚀 CARA PENGGUNAAN

Notes: Download dataset ada di `Capstone-Dicoding/Readme.md`
Asumsi: Anda telah menginstal Python, `pip`, dan mengaktifkan *virtual environment* Anda (`daisee`).

### 1. Instalasi Dependensi

Instal semua *library* yang diperlukan (disarankan menggunakan `requirements.txt` jika Anda membuatnya, atau instal satu per satu):

```bash
pip install ultralytics opencv-python numpy pytesseract pillow
````

> **Catatan Tesseract:** Pastikan perangkat Anda memiliki *engine* Tesseract OCR yang terinstal secara sistem. Jika menggunakan Windows/Linux, Anda mungkin perlu mengunduhnya secara terpisah.

### 2\. Struktur Direktori

Pastikan struktur direktori Anda sesuai dengan *path* yang digunakan dalam *notebook*:

```
.
├── YOLO Model/
│   └── my_model.pt             <-- Model YOLO Anda
├── image_dataset/
│   └── image1.jpeg             <-- Gambar yang akan diinferensi
├── infrence_base_yolo.ipynb    <-- Notebook ini
└── cropped_objects/            <-- Akan dibuat otomatis sebagai folder output crop
```

### 3\. Eksekusi Notebook

Buka file **`infrence_base_yolo.ipynb`** dan jalankan sel-sel secara berurutan:

#### 3.1. Sel 1 & 2: Instalasi & *Inferensi* YOLO

Sel ini akan memuat `my_model.pt` dan melakukan deteksi pada `image_dataset/image1.jpeg` (setelah di-*resize* ke 640x640).

  * **Input:** Gambar berukuran 640x640.
  * **Output Konsol:** Daftar semua objek yang terdeteksi (`garam`, `gula`, `lemak`, dll.), beserta koordinat *Bounding Box* (BBox) dan skor *confidence*-nya.

#### 3.2. Sel 3: *Cropping* (Pemotongan)

Sel ini mengambil BBox yang dihasilkan pada langkah 3.1 dan memotong bagian-bagian nutrisi tersebut dari gambar 640x640.

  * **Logika Penting:** Objek dengan label `"nutrition-fact"` **dilewati**, karena yang diperlukan adalah komponen di dalamnya (gula, garam, dll.).
  * **Output Folder:** Segmen gambar hasil *crop* disimpan di direktori **`cropped_objects/`**.

#### 3.3. Sel 4: OCR dengan *Upscaling*

Sel ini mengiterasi semua gambar yang baru saja di-*crop* dan menerapkan OCR.

  * **Preprocessing OCR:** Hanya menggunakan teknik **Upscaling 3x** (`UPSCALE_FACTOR = 3`) untuk memperbesar teks kecil, yang membantu meningkatkan akurasi Tesseract.
  * **Konfigurasi Tesseract:** Menggunakan `custom_config = r'--oem 3 --psm 7'`
      * `--oem 3`: Menggunakan *Engine Mode* terbaru.
      * `--psm 7`: Mengasumsikan teks adalah satu baris (`Single Line`), ideal untuk komponen nutrisi.
  * **Output Konsol:** Menampilkan teks yang diekstrak untuk setiap objek, dan ringkasan akhir.

-----

## 📝 HASIL OCR YANG DIOBSERVASI

Berikut adalah ringkasan hasil OCR yang diamati dari eksekusi *notebook*:

| Label | Teks Terekstrak (Contoh) | Keterangan |
| :---: | :---: | :--- |
| **Gula** | `Guia §=— 3g` | Berhasil menangkap nilai (3g) tetapi ada distorsi karakter. |
| **Garam** | `. Raram (natrium) 20mg 27` | Berhasil menangkap nilai (20mg), tetapi teks label awal tidak bersih. |
| **Protein** | `4 ete eer 2 g S, 3` | Distorsi parah, nilai sulit diekstrak. |
| **Takaran** | `Salian per kemasan: 1` | Hasil yang baik untuk label ini. |
| **Lemak** | `ae ere ec es 4 O5q 1%` | Distorsi parah, nilai (0.5g atau 4g) sulit diekstrak. |

**Kesimpulan OCR:** Meskipun *cropping* oleh model YOLO sudah akurat, hasil OCR masih memerlukan *preprocessing* tambahan (seperti *binarization*, *sharpening*, atau *denoising*) pada gambar yang di-*crop* tersebut untuk meningkatkan akurasi ekstraksi nilai numerik.

```
```
