# Spaceship Titanic: prediksi penumpang yang berpindah dimensi

Proyek ini menjawab kompetisi [Spaceship Titanic](https://www.kaggle.com/competitions/spaceship-titanic) di Kaggle: dari data manifes kapal, prediksi penumpang mana yang `Transported` (berpindah ke dimensi lain). Seluruh alur ada dalam satu notebook yang mengikuti enam fase CRISP-DM, dari pemahaman data sampai berkas submisi.

## Hasil

| Model | Accuracy (nested CV) | ROC-AUC |
|---|---|---|
| Baseline (selalu memprediksi kelas mayoritas) | 0,498 ± 0,011 | 0,500 |
| Logistic Regression | 0,776 ± 0,012 | 0,854 |
| CatBoost, hyperparameter dituning | 0,816 ± 0,017 | 0,907 ± 0,012 |

Angka di atas adalah rata-rata dan simpangan baku antar fold. Dua baris pertama dari 15 fold (5-fold dengan 3 seed), baris terakhir dari 5 fold luar pada nested CV. Skor publik di leaderboard Kaggle untuk satu submisi adalah 0,8043. Selisih sekitar 0,01 dengan skor validasi masih dalam rentang ketidakpastian pengambilan sampel data uji dan estimasi validasi; perhitungannya ada di notebook.

## Pendekatan

- Aturan imputasi struktural dibuktikan dulu pada data (misalnya penumpang `CryoSleep` tidak berbelanja, satu grup berasal dari planet yang sama), baru dipakai mengisi nilai kosong tanpa memakai target.
- Penumpang satu grup cenderung bernasib sama, jadi validasi memakai StratifiedGroupKFold berdasarkan `Group` agar satu grup tidak terpecah antara data latih dan validasi.
- Imputasi statistik, encoding, dan scaling berada di dalam Pipeline per fold, sehingga tidak bocor ke data validasi.
- Hyperparameter dipilih dengan nested CV, yaitu pencarian acak di loop dalam dan penilaian di loop luar. Empat model dituning (Random Forest, HistGradientBoosting, XGBoost, CatBoost) dan dibandingkan dengan baseline, Logistic Regression, serta ensemble.
- Audit kebocoran: pelatihan dengan label diacak menghasilkan skor di tingkat kebetulan, dan adversarial validation tidak menemukan pergeseran antara data latih dan uji.
- Analisis kesalahan, kurva kalibrasi, dan interval kepercayaan bootstrap per grup melengkapi evaluasi.

## Temuan utama

- Sinyal terkuat adalah `CryoSleep` dan ada tidaknya belanja; keduanya saling terkait.
- Fitur yang paling berpengaruh menurut permutation importance adalah belanja di `Spa`, `VRDeck`, `FoodCourt`, dan `RoomService`, serta lokasi kabin (`Deck`).
- Tuning hyperparameter, ensemble, dan fitur tambahan (ukuran keluarga, agregat grup, indikator nilai kosong) tidak memberi perbaikan yang terbukti dibandingkan pengaturan default.
- Accuracy paling rendah ada pada penumpang dari Earth (Deck G) dan anak-anak.
- Hubungan yang ditemukan bersifat asosiatif, bukan sebab-akibat.

## Struktur repositori

| Berkas | Isi |
|---|---|
| `Notebook_Final.ipynb` | Seluruh alur: EDA, preprocessing, modeling, evaluasi, dan prediksi, lengkap dengan output |
| `requirements.txt` | Versi library yang dipakai |
| `LICENSE` | Lisensi MIT untuk kode |

## Data

Data kompetisi tidak disertakan di repositori ini karena aturan kompetisi melarang menyebarkannya ke pihak yang bukan peserta. Unduh dari [halaman data kompetisi](https://www.kaggle.com/competitions/spaceship-titanic/data) (butuh akun Kaggle dan persetujuan aturan), lalu taruh di folder berikut:

```
data/raw/train.csv
data/raw/test.csv
data/raw/sample_submission.csv
```

Notebook membuat `data/clean/` dan `data/submission.csv` sendiri saat dijalankan.

## Cara menjalankan

Buat lingkungan dengan Python 3.10, pasang dependensi, lalu jalankan notebook dari folder utama repositori (path data bersifat relatif).

```powershell
conda create -n spaceship python=3.10 -y
```

```powershell
conda activate spaceship
```

```powershell
pip install -r requirements.txt
```

```powershell
jupyter nbconvert --execute --to notebook --inplace Notebook_Final.ipynb
```

Satu kali eksekusi penuh memakan waktu sekitar 25 menit di laptop tanpa GPU, terutama karena nested CV. Semua proses memakai satu seed global, sehingga hasilnya bisa diulang.

## Batasan

- Skor berasal dari satu dataset latih, dan leaderboard baru dari satu submisi.
- Belum dicoba: LightGBM, imputasi berbasis model, dan ensemble dari model yang sudah dituning.
- Pemenang dipilih dari empat kandidat pada fold luar yang sama, jadi skornya sedikit optimis.

## Penggunaan AI

Proyek ini dikerjakan dengan bantuan asisten AI (Claude) untuk menyusun kode, notebook, dan dokumentasi.

## Lisensi

Kode berlisensi MIT (lihat `LICENSE`). Data kompetisi tunduk pada [aturan kompetisi](https://www.kaggle.com/competitions/spaceship-titanic/rules).

## Penulis

Arthur Pendragon (GitHub: Kingarthuria)
