# Konfigurasi Arducam IMX477 di Jetson Orin Nano Super (MoodIn)

> **Status: rencana peningkatan (pengisian berjalan).** Board edge yang **fix** untuk Cross-Eval 2 tetap Jetson Orin Nano Super (lihat `imx477-pi5-config.md`). Dokumen ini menyiapkan langkah migrasi ke Jetson setelah pendampingan teknis tersedia. Unit Jetson **sudah di tangan**.

## 1. Target Lingkungan

- Jetson Orin Nano Super Developer Kit (8 GB), boot dari **NVMe SSD** (KIOXIA BG6 256 GB).
- **JetPack 7.2** (L4T R39.2.0, Ubuntu 24.04, CUDA 13.2.1, kernel 6.8) — versi yang ditargetkan untuk dukungan ISP kamera penuh.

> Catatan [High]: unit saat ini ter-flash JetPack 7.2 (L4T R39.2.0, Ubuntu 24.04). Dukungan Argus/ISP penuh untuk IMX477 tersedia pada **6.2.1**, sehingga downgrade direncanakan lebih dulu (di luar cakupan dokumen ini). Pada JetPack 7.2, IMX477 hanya berjalan di level **V4L2 mentah** (RG10 Bayer), sedangkan `nvarguscamerasrc` belum tersedia.

## 2. Perbedaan Kunci vs Jetson Orin Nano Super

| Aspek | Jetson Orin Nano Super | Jetson Orin Nano Super |
| --- | --- | --- |
| Konektor CSI | 22-pin 0.5 mm | 2x MIPI CSI-2 **22-pin** (butuh adapter 15->22) |
| Stack kamera | `libargus` / `nvargus` / nvarguscamerasrc | **`nvarguscamerasrc` (GStreamer)** / libargus |
| Output display | micro-HDMI | **DisplayPort** (butuh adapter aktif DP->HDMI) |
| Runtime DL | ONNX Runtime / CPU-GPU | **TensorRT (INT8)**, CUDA 13.2.1 |
| Daya | USB-C PD 43 W | barrel DC, mode 7–25 W |

## 3. Aktivasi Driver IMX477 (JetPack 7.2)

Dua jalur:

1. **Overlay native via `jetson-io.py`** — pilih Camera IMX477 pada menu, simpan, reboot.

   ```bash
   sudo /opt/nvidia/jetson-io/jetson-io.py
   ```

2. **Driver Arducam** (bila butuh mode HQ penuh / fitur khusus):

   ```bash
   wget https://github.com/ArduCAM/... /install_full.sh   # sesuai rilis Arducam untuk L4T 36.4.x
   sudo ./install_full.sh -m imx477
   ```

Verifikasi:

```bash
ls /dev/video*                 # perangkat kamera
v4l2-ctl --list-formats-ext    # daftar mode & format sensor
gst-inspect-1.0 nvarguscamerasrc  # pastikan elemen tersedia
```

## 4. Pipeline Capture (GStreamer / Argus)

Pratinjau (JetPack 6+ memakai `nv3dsink`; `nvoverlaysink` sudah dihapus):

```bash
gst-launch-1.0 nvarguscamerasrc ! \
  'video/x-raw(memory:NVMM),width=1920,height=1080,framerate=60/1' ! \
  nvvidconv ! nv3dsink
```

Untuk pipeline software (OpenCV via GStreamer appsink):

```python
pipeline = (
    "nvarguscamerasrc ! "
    "video/x-raw(memory:NVMM),width=1332,height=990,framerate=60/1 ! "
    "nvvidconv ! video/x-raw,format=BGRx ! "
    "videoconvert ! video/x-raw,format=BGR ! appsink drop=true max-buffers=2"
)
# cap = cv2.VideoCapture(pipeline, cv2.CAP_GSTREAMER)
```

> Skrip nvarguscamerasrc dari Pi **tidak** kompatibel; harus diporting ke pipeline GStreamer/Argus. Prinsip rPPG tetap: kunci AE/AWB, exposure kelipatan 10 ms (anti-flicker 50 Hz) — pada Argus via properti `exposuretimerange`/`gainrange`/`aelock`.

## 5. Runtime Inferensi

- Konversi model ONNX -> **TensorRT engine (INT8)** untuk memenuhi target real-time >= 30 fps. Simpan langkah konversi di dokumen model (`Software-Architecture`).
- Aktifkan Super Mode: `sudo nvpmodel -m 2 && sudo jetson_clocks`.

## 6. TODO (menunggu pendampingan teknis)

- [ ] Downgrade JetPack 7.2 -> 6.2.1 (prosedur terpisah).
- [ ] Pasang adapter CSI 15->22 pin + adapter aktif DP->HDMI.
- [ ] Aktifkan IMX477 (jetson-io atau driver Arducam), verifikasi mode sensor.
- [ ] Port pipeline capture nvarguscamerasrc -> `nvarguscamerasrc`.
- [ ] Kalibrasi ulang exposure/gain untuk anti-flicker via Argus.
- [ ] Ukur konsumsi daya Jetson, perbarui `power/power-budget-calc.md`.
