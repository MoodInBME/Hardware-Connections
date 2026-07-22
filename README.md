# Hardware-Connections

Root konfigurasi hardware, protokol fisik, dan komunikasi untuk MoodIn.
Bagian dari proyek [MoodIn](https://github.com/MoodInBME/MoodIn-master.git).

![Repo](https://img.shields.io/badge/github-repo-blue?logo=github)
![Layer](https://img.shields.io/badge/layer-hardware-lightgrey)

## Isi

- `schematics/` — wiring & pinout
- `enclosure/` — desain 3D casing
- `camera-module/` — konfigurasi kamera & hasil uji sinyal
- `display/` — setup panel Waveshare 10.1"
- `network/` — MQTT, TLS, WiFi
- `power/` — perhitungan & pengukuran daya
- `bom/` — bill of materials

## Struktur Repository Ini

```text
Hardware-Connections/
├── README.md
├── schematics/
│   ├── wiring-diagram.fzz / .svg
│   ├── pinout-raspberrypi5.md
│   └── power-distribution.md
├── enclosure/
│   ├── cad/
│   │   ├── mirror-frame.step
│   │   ├── bottom-case.step
│   │   └── full-assembly.f3d
│   ├── renders/
│   │   └── exploded-view.png
│   └── print-settings.md            # material, infill, layer height
├── camera-module/
│   ├── imx477-pi5-config.md          # setup rpicam/Picamera2 di Pi 5
│   ├── imx477-jetson-config.md       # Jetson: nvarguscamerasrc/Argus (pengisian berjalan)
│   ├── lens-m12-selection.md         # pemilihan lensa M12 (FoV 45/55/65/70/90°)
│   ├── mirror-attenuation-tests.md  # log uji Persamaan 7 (atenuasi two-way mirror)
│   └── fov-calculation.md           # acuan Persamaan 9 (jarak kerja & FoV)
├── display/
│   └── waveshare-10.1-setup.md
├── network/
│   ├── mqtt-tls-config.md
│   ├── wifi-provisioning.md
│   └── mqtt-topic-schema.md         # referensi ke kontrak kanonik (lihat catatan)
├── power/
│   ├── power-budget-calc.md         # acuan Persamaan 5
│   └── measured-current-log.csv     # hasil ukur aktual vs estimasi
├── bom/
│   └── full-bom.csv
├── assembly-guide.md
└── troubleshooting.md
```

## Target Perangkat

- Compute (utama): **Raspberry Pi 5** (16 GB) — board edge yang dipakai untuk CE2 (kamera IMX477 native, plug-and-play). Bila desain 3D (Gambar 3 proposal) memakai Compute Module 5, samakan penamaan di `pinout-raspberrypi5.md` dan proposal agar tidak ada inkonsistensi Pi 5 vs CM5.
- Compute (rencana peningkatan): **Jetson Orin Nano Super** — unit **sudah di tangan**; migrasi (JetPack 6.2.1, L4T 36.4.4, Ubuntu 22.04, CUDA 12.6, **boot dari NVMe SSD**) direncanakan setelah pendampingan teknis tersedia. Lihat bagian Integrasi Jetson Orin Nano Super di bawah.
- Kamera: **Arducam IMX477 (RGB, 12.3 MP)** dengan dudukan lensa **M12**; tersedia 5 opsi lensa FoV — **45°, 55°, 65°, 70°, 90°** (pilih sesuai jarak kerja cermin; lihat `camera-module/lens-m12-selection.md`).
- Display: Waveshare 10.1" IPS 1024×600 HDMI.
- Komunikasi: MQTT over TLS (port 8883).

## Catatan Teknis Kritis

- **Atenuasi mirror + kebutuhan RGB (uji di `camera-module/`)**: two-way mirror menurunkan cahaya yang sampai ke sensor. IMX477 dipilih justru karena **sensor RGB** (bukan NoIR) — rPPG membutuhkan kanal warna utuh dan model umumnya dilatih pada citra RGB, sehingga NoIR akan merusak sinyal & domain input. Konsekuensinya, kompensasi cahaya harus di sisi **pencahayaan/eksposur** (tambah iluminasi, kunci AE/AWB, exposure kelipatan 10 ms untuk anti-flicker 50 Hz), bukan beralih ke NoIR. Dokumentasikan hasil `mirror-attenuation-tests.md` sebagai data kalibrasi, bukan sekadar lampiran.
- **FoV vs jarak kerja (`fov-calculation.md`)**: karena IMX477 memakai mount M12 dengan lensa yang bisa diganti, FoV kini **variabel desain**, bukan tetap. Angka spec lensa (45/55/65/70/90°) diperlakukan sebagai **FoV diagonal**; setelah dikonversi ke FoV vertikal pada sensor 1/2.3", cakupan tinggi pada jarak kerja **45 cm** kira-kira: 55° ≈ 28 cm, 65° ≈ 34 cm, 70° ≈ 38 cm, 90° ≈ 54 cm. **Rekomendasi: lensa 65° (diagonal) pada jarak kerja ~45 cm** — memuat kepala + margin tinggi antar-pengguna, distorsi moderat (penting untuk geometri landmark ST-GCN), dan tidak menangkap terlalu banyak latar (privasi). Perhitungan & pemilihan lengkap: `camera-module/fov-calculation.md` (kanonik) dan `camera-module/lens-m12-selection.md`. Trade-off: makin lebar FoV → wajah makin sedikit piksel + distorsi makin besar.
- **Anggaran daya (`power/`)**: bedakan rating PSU (mis. 5V/5A) dari konsumsi aktual. Kolom `measured-current-log.csv` adalah pembuktian; jangan mengklaim konsumsi memakai angka rating catu daya.

## Kontrak MQTT

`network/mqtt-topic-schema.md` **tidak** mendefinisikan ulang skema — ia mereferensikan kontrak kanonik di `MoodIn-master/docs/ipc-contract.md` dan hanya menambahkan detail transport (broker, TLS, QoS, retensi). Ini mencegah dua sumber kebenaran yang bisa saling menyimpang saat integrasi.

## Integrasi Jetson Orin Nano Super

Status: **rencana peningkatan** — unit **sudah di tangan** (JetPack 6.2.1, boot dari NVMe SSD KIOXIA 256 GB); migrasi dari Raspberry Pi 5 direncanakan setelah pendampingan teknis tersedia. Untuk CE2, **Raspberry Pi 5** tetap board utama; Jetson disiapkan sebagai peningkatan inferensi (ST-GCN + STC-rPPG paralel, target real-time ≥ 30 fps).

Perbedaan integrasi yang perlu disiapkan:

- **Kamera**: port kamera Jetson Orin Nano Super = 2× MIPI CSI-2 **22-pin**, sedangkan board IMX477 = **15-pin** → butuh kabel/adapter **15→22 pin**. IMX477 didukung di **JetPack 6.2.1**: aktifkan overlay lewat `jetson-io.py` (driver native), atau pasang driver Arducam (`install_full.sh -m imx477`) bila mode HQ penuh diperlukan.
- **Software kamera**: **tidak ada `rpicam`/`libcamera` khas Pi**. Gunakan **`nvarguscamerasrc`** (GStreamer) atau **libargus**; cek mode sensor dengan `v4l2-ctl --list-formats-ext`. Skrip capture Picamera2 dari Pi perlu diporting ke pipeline GStreamer.
- **Display**: Jetson mengeluarkan **DisplayPort**, panel Waveshare 10.1" masuk lewat HDMI → butuh adapter aktif **DP→HDMI**.
- **Penyimpanan**: slot **M.2 Key-M** untuk SSD NVMe (OS + dataset & checkpoint). Terpasang: **KIOXIA BG6 256 GB**; OS 6.2.1 di-boot langsung dari NVMe.
- **Daya**: mode 7–25 W (lebih tinggi dari Pi 5) → tinjau ulang `power/power-budget-calc.md` dan rating PSU.
- **Runtime DL**: manfaatkan **TensorRT (INT8)**; simpan langkah konversi ONNX→TensorRT di dokumen model terkait.

Rincian langkah-demi-langkah sedang diisi di `camera-module/imx477-jetson-config.md` (unit sudah tersedia).

## Quick Start (perakitan)

Lihat [assembly-guide.md](assembly-guide.md).
