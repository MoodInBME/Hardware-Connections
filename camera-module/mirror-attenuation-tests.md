# Uji Atenuasi Two-Way Mirror — MoodIn (Persamaan 7)

> Log pengujian penurunan cahaya oleh two-way mirror di depan kamera Arducam IMX477. Ini **data kalibrasi**, bukan lampiran — hasilnya menentukan kebutuhan iluminasi & setting eksposur, dan memvalidasi keputusan memakai sensor RGB (bukan NoIR).

## 1. Latar & Persamaan 7

Two-way mirror meneruskan hanya sebagian cahaya ke sensor. Modelkan transmitansi efektif:

```text
(Pers. 7)   T_eff = I_sensor / I_scene
            Atenuasi (dB) = -10 * log10(T_eff)
            Loss stop (EV) = log2(1 / T_eff)
```

dengan `I_scene` = intensitas terukur tanpa mirror, `I_sensor` = intensitas terukur melalui mirror pada kondisi optik & eksposur identik. Kompensasi yang dibutuhkan (naik EV) = `log2(1/T_eff)`.

> Konsekuensi desain: karena rPPG memerlukan kanal **RGB utuh** dan model dilatih pada citra RGB, kompensasi dilakukan lewat **iluminasi + eksposur/gain**, bukan beralih ke NoIR. Target: menjaga SNR rPPG memadai tanpa menaikkan gain berlebih (gain tinggi -> noise naik).

## 2. Protokol Uji

1. **Kondisi kontrol**: target kalibrasi (kartu abu-abu 18% / color checker) pada jarak kerja nominal 45 cm, iluminasi tetap, AE/AWB **dikunci** (exposure & gain tetap, kelipatan 10 ms).
2. Rekam baseline **tanpa mirror** -> ukur rata-rata luminansi ROI (dan per-kanal R/G/B).
3. Pasang two-way mirror pada dudukan final -> rekam **dengan mirror**, setting optik/eksposur identik.
4. Hitung `T_eff`, atenuasi (dB), dan loss (EV) dengan Persamaan 7.
5. Ulangi untuk beberapa tingkat iluminasi & (opsional) beberapa lensa (55°, 70°).
6. Uji **SNR rPPG**: rekam wajah 30–60 dtk, ekstraksi sinyal PPG dari ROI dahi/pipi, hitung SNR (rasio daya puncak HR terhadap noise pita sekitar).

## 3. Metrik yang Dicatat

- Luminansi rata-rata ROI (0–255) kontrol vs mirror, per kanal.
- `T_eff`, atenuasi dB, loss EV.
- Setting kamera: exposure (us), analogue gain, ColourGains.
- Iluminasi (lux) di bidang wajah.
- SNR rPPG (dB) & deteksi HR (bpm) vs referensi (oximeter/kontak).

## 4. Tabel Hasil (isi saat pengujian)

| Tanggal | Lensa | Iluminasi (lux) | Exposure (us) | Gain | Lum. kontrol | Lum. mirror | T_eff | Atenuasi (dB) | Loss (EV) | SNR rPPG (dB) | Catatan |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| | 55° | | 10000 | | | | | | | | |
| | 55° | | 10000 | | | | | | | | |
| | 70° | | 10000 | | | | | | | | |

## 5. Kriteria Lolos (usulan)

- Loss two-way mirror terkompensasi tanpa gain ekstrem (target analogue gain <= ~8x agar noise terkendali). [Medium — finalkan setelah data]
- SNR rPPG cukup untuk deteksi HR stabil (kesalahan HR <= ~3 bpm vs referensi pada sesi >= 30 dtk).
- Tidak ada clipping kanal (histogram tidak menumpuk di 255) maupun crush (di 0).

## 6. Skrip Bantu (contoh)

```python
import numpy as np, math

def attenuation(lum_control, lum_mirror):
    t_eff = lum_mirror / lum_control
    att_db = -10 * math.log10(t_eff)
    loss_ev = math.log2(1 / t_eff)
    return t_eff, att_db, loss_ev

# contoh: kontrol 180, mirror 72 (0-255)
print(attenuation(180, 72))  # -> (0.40, 3.98 dB, 1.32 EV)
```

## 7. TODO

- [ ] Ukur baseline tanpa mirror (per kanal).
- [ ] Ukur melalui mirror final; hitung T_eff & loss.
- [ ] Tentukan iluminasi tambahan (LED) agar loss terkompensasi tanpa gain ekstrem.
- [ ] Validasi SNR rPPG untuk lensa 55° (dan 70° sebagai pembanding).
