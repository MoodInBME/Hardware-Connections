# Hardware-Connections

Perangkat keras MoodIn: pengkabelan, kamera CSI, termal, daya, dan casing. Repo ini tidak memuat model dan tidak memuat kode UI.
Bagian dari proyek [MoodIn](https://github.com/MoodInBME/MoodIn-master.git).

![Repo](https://img.shields.io/badge/github-repo-blue?logo=github)
![Layer](https://img.shields.io/badge/layer-hardware-lightgrey)

---

## 1. Bill of material

| Komponen | Spesifikasi | Catatan Penting |
|---|---|---|
| Compute | **NVIDIA Jetson Orin Nano Super 8GB** (Ubuntu 24.04, JetPack 7.2) | 67 TOPS INT8. Menggantikan Raspberry Pi 5 yang tidak sanggup menjalankan dua model neural simultan pada 30 fps. |
| Kamera | Arducam **IMX477** RGB 12,3 MP + lensa M12 55° | RGB, bukan NoIR. Cabang fisiologis (rPPG) sangat bergantung pada kanal hijau yang bersih. |
| Layar | Waveshare 10,1 inci 1024x600 HDMI IPS | Mode *landscape*. Gunakan adapter DP→HDMI aktif untuk Jetson. |
| Optik | Figura + kaca + film stiker satu arah | Ada masalah transmisi cahaya. Lihat Bagian 4. |
| Daya | 19 V, tipikal ~43 W, puncak ~52 W | Catu daya disesuaikan dengan spek Jetson. |
| Pendinginan | Kipas 60 mm, 15-20 CFM | Target suhu *junction* < 80°C. |

## 2. Geometri Optik (Field of View)

Pada jarak kerja 55 cm dengan lensa M12 55° diagonal:
```
tan(27,5°) = 0,5206
HFoV = 2 * arctan(0,5206 * 0,8) = 45,2°
VFoV = 2 * arctan(0,5206 * 0,6) = 34,7°
```
> **Koreksi Proposal**: HFoV yang benar adalah **45,2°**, bukan 42,2°.

## 3. Termal

Laju aliran udara minimum:
```
CFM = 1,76 * P / dT
```
Pada beban puncak (52 W) dengan delta temperatur 8°C, dibutuhkan sekitar 11,4 CFM. Kipas 15-20 CFM memberikan margin yang cukup, **dengan syarat** casing memiliki jalur udara masuk dan keluar (ventilasi memadai).

## 4. Peringatan: Transmisi Stiker Cermin Satu Arah

Desain fisik memakai bingkai figura berlapis film cermin (stiker rubben). Film ini hanya mentransmisikan **10-20% cahaya** dan *tidak rata secara spektral* (atenuasi berbeda untuk R, G, B).

Kehilangan SNR ~5 dB pada wajah (ROI) sangat fatal bagi modul STC-rPPG. Selain itu, distorsi rasio antar-kanal warna tidak bisa dikalibrasi.
**Mitigasi:** Buat potongan lubang kecil berdiameter **12 mm** pada film stiker, tepat di depan lensa kamera, dan sembunyikan di balik pinggiran bingkai figura.

## 5. Bring-up Kamera CSI (IMX477)

Kamera IMX477 di Jetson Orin Nano Super butuh adapter kabel **15-pin ke 22-pin**.
Error **`-121`** sering berarti masalah kabel (terbalik, tidak terkunci, terlalu panjang).

Urutan diagnosis:
1. `ls /dev/video*`
2. `dmesg | grep -i imx477`
3. `v4l2-ctl --list-devices` lalu `v4l2-ctl --list-formats-ext`
4. Uji satu *frame* pertama dengan `nvarguscamerasrc` (GStreamer) atau libargus. Jangan berpindah ke MediaPipe jika tahap ini belum lulus.

## 6. Struktur Repo

```
Hardware-Connections/
├── README.md
├── schematics/
│   ├── wiring-diagram.svg
│   └── power-distribution.md
├── enclosure/
│   ├── cad/                          # sumber CAD (Fusion 360)
│   ├── stl/                          # mesh siap-cetak
│   └── print-settings.md            # parameter cetak Bambu Lab P1S
├── camera-module/
│   ├── lens-m12-selection.md
│   └── fov-calculation.md
├── display/
├── network/
└── power/
```

## 7. Status & Tugas Tersisa

| # | Tugas | Status |
|---|---|---|
| 1 | Perakitan fisik figura, kaca, stiker | Pemesanan |
| 2 | Pembuatan lubang 12 mm di film stiker (opsional/menunggu ACC) | Belum |
| 3 | Uji nyala IMX477 (CSI) di JetPack 7.2 | Menunggu perangkat |
| 4 | Pengukuran beban *thermal* puncak | Menunggu (3) |
| 5 | Laporan `tegrastats` (TensorRT INT8) | Menunggu (4) |
| 6 | Evaluasi SNR ROI dengan vs tanpa stiker | Sangat kritikal untuk rPPG |
