# Verifikasi Ijazah: OCR Nomor Ijazah + Deteksi Tanda Tangan

**Nama:** Muhammad Yani  **NIM:** F1G124069

Mini project Citra Digital. Sistem membaca **nomor ijazah** dengan OCR (Tesseract) dan menentukan **ada/tidaknya tanda tangan** pejabat (Rektor/Dekan) dengan OpenCV.

```
Input : data/ijazah_001.jpg
Output:
Nomor Ijazah : 571012022000056
Tanda Tangan : PRESENT
```

## Isi Repository

| File | Keterangan |
|---|---|
| `Tugas7_CitraDigital` | Seluruh kode (pipeline, OCR, deteksi tanda tangan, evaluasi CER) beserta outputnya |
| `Hasil.md` | Hasil pengujian, penjelasan metode, dan analisis metode enhancement berdasarkan CER |
| `data/` | `ijazah_001.jpg` (asli), `ijazah_hanya_dekan.jpg` dan `ijazah_tanpa_ttd.jpg` (citra uji) |
| `requirements.txt` | Daftar library Python |

## Pipeline

```
Citra Ijazah → Grayscale → Image Enhancement
      ├─ Area Nomor → Enhancement → OCR → Nomor Ijazah
      └─ Area Tanda Tangan → Thresholding → Morphology → Signature Detection
                         ↓
                 Hasil Verifikasi
```

## Cara Menjalankan (How to Run)

### Opsi A — Google Colab (paling mudah)
1. Buka <https://colab.research.google.com>, pilih **File → Upload notebook**, lalu unggah `F1G124069_miniproject_citra_digital.ipynb`.
2. Di panel kiri (ikon folder), buat folder `data` lalu unggah citra ijazah ke dalamnya. Jika folder `data` kosong, Sel 2 akan menampilkan dialog upload otomatis.
3. Jalankan semua sel: **Runtime → Run all**. Sel 1 memasang Tesseract dan library secara otomatis.
4. Hasil `Nomor Ijazah` dan `Tanda Tangan` tampil di Sel 7.

### Opsi B — Komputer lokal
1. Pasang **Tesseract OCR**:
   - Ubuntu/Debian: `sudo apt install tesseract-ocr`
   - macOS: `brew install tesseract`
   - Windows: unduh dari <https://github.com/UB-Mannheim/tesseract/wiki>, lalu tambahkan folder instalasinya ke PATH. Jika tetap tidak terdeteksi, tambahkan di sel import:
     `pytesseract.pytesseract.tesseract_cmd = r"C:\Program Files\Tesseract-OCR\tesseract.exe"`
2. Pasang library Python (Python 3.9+):
   ```bash
   git clone <URL-REPOSITORY-ANDA>
   cd <nama-repo>
   pip install -r requirements.txt
   pip install notebook
   ```
3. Jalankan notebook:
   ```bash
   jupyter notebook Tugas7_CitraDigital.ipynb
   ```
   Lalu pilih **Run → Run All Cells**. Citra dibaca dari folder `data/`.

### Memakai citra sendiri
Taruh file `.jpg`/`.png` di folder `data/`. Koordinat area nomor dan tanda tangan (Sel 4) dibuat untuk tata letak ijazah pada contoh. Untuk format ijazah lain, sesuaikan `ROI_NOMOR`, `ROI_REKTOR`, dan `ROI_DEKAN`.

Penjelasan metode dan analisis CER ada di [Hasil.md](Hasil.md).
