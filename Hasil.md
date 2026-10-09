# Hasil dan Analisis

Muhammad Yani — F1G124069

## 1. Hasil Pengujian

Output notebook (Sel 7) pada tiga citra di folder `data/`:

```
[ijazah_001.jpg]  (ijazah asli)
Nomor Ijazah : 571012022000056
Tanda Tangan : PRESENT

[ijazah_hanya_dekan.jpg]  (tanda tangan Rektor dihapus sintetis)
Nomor Ijazah : 571012022000056
Tanda Tangan : PRESENT      (Rektor: tidak ada, Dekan: ada)

[ijazah_tanpa_ttd.jpg]  (tanda tangan Rektor dan Dekan dihapus sintetis)
Nomor Ijazah : 571012022000056
Tanda Tangan : ABSENT
```

- Nomor ijazah terbaca benar pada ketiga citra (CER = 0).
- Sistem mampu membedakan PRESENT dan ABSENT. Dua citra uji dibuat dengan menghapus tinta tanda tangan dari `ijazah_001.jpg` memakai `cv2.inpaint`, jadi merupakan kontrol negatif sintetis, bukan ijazah asli tanpa tanda tangan.

## 2. Metode yang Digunakan

### 2.1 Pra-pemrosesan
1. **Orientasi:** citra portrait diputar 90° searah jarum jam agar teks horizontal.
2. **Grayscale:** `cv2.cvtColor(BGR2GRAY)`. Informasi yang dibutuhkan (tinta gelap pada kertas) ada pada intensitas, sehingga warna tidak diperlukan.
3. **Image Enhancement (global):** citra dibagi estimasi background (*morphological closing* + Gaussian blur) untuk meratakan warna kuning kertas, lalu **CLAHE** menaikkan kontras lokal.
4. **Pemotongan area (ROI):** koordinat relatif terhadap ukuran citra, sehingga tahan terhadap perbedaan resolusi scan.

### 2.2 Pembacaan nomor ijazah
1. **ROI nomor:** baris "Nomor ijazah: ..." di kiri bawah.
2. **Enhancement:** *Adaptive Gaussian Thresholding* (ambang dihitung lokal per jendela 31×31 piksel, sehingga tahan terhadap pencahayaan tidak merata).
3. **OCR:** Tesseract dengan `--psm 7` (satu baris teks), cadangan `--psm 6`.
4. **Post-processing:** label "Nomor ijazah" dibuang, huruf yang sering tertukar dengan angka dikoreksi (O→0, I/l→1, S→5, B→8, Z→2), lalu angka diambil dengan regex `\d{10,}`.

### 2.3 Deteksi tanda tangan
1. **Thresholding:** operasi *black-hat* menonjolkan goresan gelap tipis di atas kertas. Ambang = max(Otsu, 35). Batas 35 mencegah Otsu memecah tekstur kertas ketika area sebenarnya kosong.
2. **Morphology:** *opening* (3×3) membuang bintik noise, *closing* (11×11) menyambung goresan yang putus.
3. **Signature detection:** *connected components*. Area dinyatakan **PRESENT** jika komponen terbesar memiliki bounding box ≥ 28% lebar dan ≥ 20% tinggi ROI, dan kepadatan tinta ≥ 1%. Tanda tangan berupa goresan bersambung yang besar, sedangkan teks cetak terpecah menjadi komponen kecil.
4. **Keputusan akhir:** PRESENT jika minimal satu dari Rektor/Dekan terdeteksi.

## 3. Metode Enhancement Paling Efektif Berdasarkan CER

**CER = (S + D + I) / N** dengan S substitusi, D penghapusan, I penyisipan, dan N panjang ground truth (15 digit: `571012022000056`). Semakin kecil semakin baik. Nilai 1,00 berarti OCR tidak menemukan deret angka yang valid.

Karena hanya ada satu citra bersih, area nomor diberi 9 kondisi degradasi sintetis, dan noise acak dirata-ratakan pada 5 seed.

| Peringkat | Metode | clean | blur | noise | low_contrast | uneven_light | low_res | jpeg_q6 | salt_pepper | kombinasi | **Rata-rata CER** |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | `adaptive_gaussian` | 0.00 | 0.00 | 0.25 | 0.00 | 0.00 | 0.00 | 0.00 | 0.43 | 0.12 | **0.089** |
| 2 | `gray_saja` | 0.00 | 0.00 | 0.01 | 0.00 | 0.27 | 0.00 | 0.00 | 0.45 | 0.83 | **0.173** |
| 3 | `gaussian_otsu` | 0.00 | 0.00 | 0.00 | 0.00 | 1.00 | 0.13 | 0.00 | 0.04 | 0.87 | **0.227** |
| 4 | `clahe` | 0.00 | 0.00 | 0.48 | 0.00 | 1.00 | 0.00 | 0.00 | 0.04 | 0.65 | **0.241** |
| 5 | `upscale_gauss_otsu` | 0.00 | 0.00 | 0.01 | 0.00 | 1.00 | 0.00 | 0.00 | 1.00 | 1.00 | **0.335** |
| 6 | `hist_equalization` | 0.00 | 1.00 | 0.59 | 0.33 | 1.00 | 1.00 | 1.00 | 1.00 | 1.00 | **0.769** |

### Kesimpulan
- **Metode paling efektif: Adaptive Gaussian Thresholding** dengan rata-rata CER terendah **0,089**. Keunggulannya jelas pada **iluminasi tidak merata** (CER 0,00, sedangkan Gaussian+Otsu, CLAHE, dan upscale+Otsu semuanya 1,00) dan pada kondisi **kombinasi** (0,12 vs 0,65–1,00 pada metode lain). Sebabnya, ambang dihitung lokal per jendela, sedangkan Otsu memakai satu ambang global untuk seluruh area.
- **Grayscale saja** (0,173) di peringkat 2. Tesseract sudah punya binarisasi internal, sehingga pada teks cetak yang bersih enhancement tambahan tidak selalu membantu.
- **Gaussian + Otsu** (kombinasi yang dipakai pada contoh referensi) bagus pada noise (CER 0,00) tetapi gagal total saat pencahayaan tidak merata (1,00).
- **Histogram Equalization** paling buruk (0,769): equalization global menguatkan tekstur latar sehingga karakter rusak, bahkan pada blur, low-res, dan JPEG.
- **CLAHE** menguatkan noise (CER 0,48 pada noise Gaussian).
- **Kelemahan adaptive threshold:** pada noise Gaussian kuat (0,25) dan salt-and-pepper (0,43), ia lebih buruk daripada Gaussian+Otsu (0,00 dan 0,04). Jadi tidak ada metode yang terbaik di semua kondisi. Adaptive threshold dipilih karena **paling stabil secara rata-rata**.

### Keterbatasan
- Hanya satu ijazah dan degradasi sintetis, sehingga peringkat bersifat indikatif. Peringkat juga bergantung pada campuran kondisi yang diuji: pada kondisi ringan (tanpa iluminasi tidak merata dan tanpa kombinasi), grayscale saja bisa setara atau sedikit lebih baik.
- Orientasi hanya menangani citra portrait (putar 90°). Citra terbalik 180° tidak ditangani.
- Koordinat ROI disesuaikan dengan satu tata letak ijazah. Format ijazah lain perlu menyesuaikan ROI di Sel 4.
