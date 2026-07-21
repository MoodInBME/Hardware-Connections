# Perhitungan FoV & Cakupan Kamera — MoodIn (Arducam IMX477 + M12)

> Dokumen **kanonik** untuk menghitung Field of View (FoV) dan cakupan bidang kamera pada berbagai jarak kerja. Menggantikan angka ilustratif di `README.md` repo ini.
>
> Konteks: cermin pintar MoodIn — akuisisi wajah untuk rPPG (STC-rPPG) dan geometri landmark (ST-GCN). Kamera dipasang di balik two-way mirror.

## 1. Spesifikasi sensor (IMX477)

| Parameter | Nilai |
| --- | --- |
| Optical format | 1/2.3" |
| Resolusi penuh | 4056 x 3040 (12,3 MP) |
| Ukuran piksel | 1,55 um x 1,55 um |
| Area aktif (H x V) | 6,287 mm x 4,712 mm |
| Diagonal sensor | 7,857 mm |
| Mount | M12 (S-mount), back-focus dapat disetel |

Turunan ukuran: 4056 x 1,55 um = 6,287 mm; 3040 x 1,55 um = 4,712 mm; diagonal = akar(6,287^2 + 4,712^2) = 7,857 mm. [High]

## 2. Model & asumsi

Model **pinhole rektilinear** (lensa low-distortion, belum termasuk koreksi distorsi):

- f = (dimensi_sensor / 2) / tan(FoV / 2)
- FoV_dim = 2 . atan((dimensi_sensor / 2) / f)
- Cakupan(jarak d) = 2 . d . tan(FoV_dim / 2)

**Asumsi penting [Medium]:** angka FoV pada spec lensa M12 (45/55/65/70/90 derajat) diperlakukan sebagai **FoV diagonal (DFoV)** — konvensi paling umum pada kit lensa M12 Arducam. **Verifikasi ke datasheet lensa.** Bila yang dikutip ternyata FoV horizontal, hitung ulang dengan mengganti acuan "diagonal" -> "horizontal" (skrip di bagian 6 menyediakan opsi tersebut); pada kasus itu cakupan akan lebih besar.

Lensa wide (>= 70 derajat) memiliki distorsi barel signifikan sehingga cakupan riil di tepi bidang sedikit lebih besar dari model rektilinear; tetap lakukan kalibrasi (bagian 7).

## 3. Konversi DFoV -> focal length, HFoV, VFoV

Dihitung dari diagonal 7,857 mm (H/2 = 3,1435 mm; V/2 = 2,356 mm):

| DFoV (spec) | f (mm) | HFoV | VFoV |
| --- | --- | --- | --- |
| 45 deg | 9,48 | 36,7 deg | 27,9 deg |
| 55 deg | 7,55 | 45,2 deg | 34,7 deg |
| 65 deg | 6,17 | 54,0 deg | 41,8 deg |
| 70 deg | 5,61 | 58,5 deg | 45,6 deg |
| 90 deg | 3,93 | 77,3 deg | 61,9 deg |

## 4. Tabel cakupan (lebar H x tinggi V bidang, cm)

Kepala penuh (dagu -> ubun-ubun) approx 23 cm. Target margin untuk variasi tinggi antar-pengguna + gerak alami: cakupan tinggi ideal approx 40-45 cm.

### Jarak kerja 40 cm

| DFoV | Tinggi V (cm) | Lebar H (cm) | Muat kepala 23 cm? |
| --- | --- | --- | --- |
| 45 deg | 19,9 | 26,5 | TIDAK (terpotong) |
| 55 deg | 25,0 | 33,3 | Pas, margin +/-1 cm |
| 65 deg | 30,6 | 40,8 | Ya, margin +/-3,8 cm |
| 70 deg | 33,6 | 44,8 | Ya, margin +/-5,3 cm |
| 90 deg | 48,0 | 64,0 | Longgar (wajah kecil) |

### Jarak kerja 45 cm (nominal desain)

| DFoV | Tinggi V (cm) | Lebar H (cm) | Muat kepala 23 cm? |
| --- | --- | --- | --- |
| 45 deg | 22,4 | 29,8 | Pas-pasan |
| 55 deg | 28,1 | 37,5 | Ya, margin +/-2,5 cm |
| 65 deg | 34,4 | 45,9 | Ya, margin +/-5,7 cm |
| 70 deg | 37,8 | 50,4 | Ya, margin +/-7,4 cm |
| 90 deg | 54,0 | 72,0 | Longgar (wajah kecil) |

### Jarak kerja 50 cm

| DFoV | Tinggi V (cm) | Lebar H (cm) | Muat kepala 23 cm? |
| --- | --- | --- | --- |
| 45 deg | 24,9 | 33,2 | Ya, margin +/-1 cm |
| 55 deg | 31,2 | 41,7 | Ya |
| 65 deg | 38,2 | 51,0 | Ya |
| 70 deg | 42,0 | 56,0 | Ya |
| 90 deg | 60,0 | 80,0 | Longgar |

Cakupan berskala linear terhadap jarak: Cakupan(d) = d x faktor.

| DFoV | Faktor V | Faktor H |
| --- | --- | --- |
| 45 deg | 0,497 | 0,663 |
| 55 deg | 0,625 | 0,833 |
| 65 deg | 0,764 | 1,020 |
| 70 deg | 0,840 | 1,120 |
| 90 deg | 1,200 | 1,601 |

## 5. Piksel wajah & mode capture

Mode IMX477 yang relevan (>= 30 fps untuk rPPG):

- **1332 x 990 (binning 2x2), hingga ~120 fps** — direkomendasikan: fps tinggi + piksel lebih besar (SNR lebih baik di balik mirror yang meredam cahaya).
- 2028 x 1080 @ ~50 fps; 2028 x 1520 @ ~40 fps.
- 4056 x 3040 @ ~10 fps — terlalu lambat untuk rPPG.

Estimasi tinggi wajah dalam piksel (mode 1332 x 990, tinggi bingkai 990 px, jarak 45 cm, wajah 23 cm):

| DFoV | Tinggi bidang (cm) | Porsi wajah | Piksel wajah |
| --- | --- | --- | --- |
| 55 deg | 28,1 | 81,8% | ~810 px |
| 65 deg | 34,4 | 66,9% | ~660 px |
| 70 deg | 37,8 | 60,8% | ~600 px |
| 90 deg | 54,0 | 42,6% | ~420 px |

Semua > 400 px -> cukup untuk rPPG dan MediaPipe. Artinya **kepadatan piksel bukan faktor pembatas**; keputusan didorong oleh margin framing, distorsi, dan privasi (lihat `lens-m12-selection.md`).

## 6. Skrip hitung ulang (reproducible)

Jalankan untuk memverifikasi/menyesuaikan angka bila jarak kerja atau konvensi FoV berubah:

```python
import math

# Sensor IMX477 (mm)
H, V = 6.287, 4.712
D = math.hypot(H, V)  # 7.857

def fov_from_spec(spec_deg, ref="diagonal"):
    ref_dim = {"diagonal": D, "horizontal": H, "vertical": V}[ref]
    f = (ref_dim / 2) / math.tan(math.radians(spec_deg) / 2)
    hfov = 2 * math.degrees(math.atan((H / 2) / f))
    vfov = 2 * math.degrees(math.atan((V / 2) / f))
    return f, hfov, vfov

def coverage(fov_deg, distance_cm):
    return 2 * distance_cm * math.tan(math.radians(fov_deg) / 2)

WORKING_DISTANCE_CM = 45
for spec in (45, 55, 65, 70, 90):
    f, hfov, vfov = fov_from_spec(spec, ref="diagonal")
    h = coverage(hfov, WORKING_DISTANCE_CM)
    v = coverage(vfov, WORKING_DISTANCE_CM)
    print(f"{spec:>3} deg DFoV | f={f:5.2f} mm | HFoV={hfov:4.1f} VFoV={vfov:4.1f} "
          f"| @{WORKING_DISTANCE_CM}cm: {h:4.1f} x {v:4.1f} cm")
```

## 7. Kalibrasi & undistortion (wajib untuk ST-GCN)

- ST-GCN memakai koordinat landmark; distorsi lensa menggeser geometri -> membiaskan fitur ekspresi (valence-arousal). Lakukan kalibrasi intrinsik OpenCV (papan catur), simpan `camera_matrix` + `dist_coeffs`, dan **undistort** frame sebelum ekstraksi landmark.
- Simpan hasil kalibrasi per lensa: `camera-module/calib_imx477_<lens>.yaml`.

## 8. TODO

- [ ] Verifikasi konvensi FoV (diagonal vs horizontal) dari datasheet lensa.
- [ ] Ukur jarak kerja aktual pengguna pada enclosure final, lalu hitung ulang (bagian 6).
- [ ] Kalibrasi intrinsik untuk lensa terpilih; simpan file kalibrasi.
- [ ] Uji SNR rPPG di balik two-way mirror untuk lensa terpilih (lihat `mirror-attenuation-tests.md`).

## Referensi

- Sony IMX477 datasheet (dimensi sensor & mode).
- Arducam M12 lens kit untuk IMX477 (opsi FoV 45/55/65/70/90 derajat).
- OpenCV Camera Calibration (kalibrasi intrinsik & undistortion).
