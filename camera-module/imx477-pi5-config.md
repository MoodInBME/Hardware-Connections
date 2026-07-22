# Konfigurasi Arducam IMX477 di Raspberry Pi 5 (MoodIn)

> Setup kamera **utama** untuk Cross-Eval 2. Target: akuisisi wajah stabil untuk rPPG (STC-rPPG) dan landmark (ST-GCN) di balik two-way mirror. Konvensi FoV & lensa: lihat `fov-calculation.md` + `lens-m12-selection.md`.

## 1. Prasyarat

- Raspberry Pi 5, Raspberry Pi OS (Debian **Trixie**, 64-bit).
- Board Arducam IMX477 + lensa M12 65° (rekomendasi).
- Kabel adapter CSI **15-pin -> 22-pin** (port kamera Pi 5 = 22-pin 0.5 mm).

## 2. Aktivasi Driver

IMX477 didukung native oleh stack `libcamera` Raspberry Pi (driver kernel `imx477`).

1. Pasang kabel CSI dalam kondisi **mati** (perhatikan orientasi kontak).
2. Pada Pi 5, konfigurasi overlay di `/boot/firmware/config.txt`:

   ```ini
   # nonaktifkan auto-detect agar overlay eksplisit
   camera_auto_detect=0
   # aktifkan IMX477 (tambahkan ,cam0 atau ,cam1 sesuai port)
   dtoverlay=imx477
   ```

3. Reboot, lalu verifikasi:

   ```bash
   rpicam-hello --list-cameras
   # harus muncul entri imx477 dengan mode-mode sensornya
   ```

> Bila board Arducam varian tertentu butuh driver khusus (mis. motorized focus), ikuti installer Arducam. Untuk varian fokus-manual M12 standar, driver native sudah cukup.

## 3. Uji Cepat Pratinjau & Rekam

```bash
# pratinjau
rpicam-hello -t 5000

# rekam 10 detik ke file (H.264)
rpicam-vid -t 10000 --width 1332 --height 990 --framerate 60 -o test.h264

# tangkap still resolusi penuh (cek fokus/kualitas optik)
rpicam-still -o still_full.jpg
```

## 4. Mode Sensor untuk rPPG

Utamakan **frame rate tinggi + piksel besar** (SNR lebih baik di balik mirror yang meredam cahaya). Ringkasan mode (lihat `fov-calculation.md` bagian 5):

| Mode | Resolusi | FPS | Pakai untuk |
| --- | --- | --- | --- |
| Binned 2x2 | 1332 x 990 | ~120 | **rPPG (rekomendasi)** |
| — | 2028 x 1080 | ~50 | alternatif |
| — | 2028 x 1520 | ~40 | alternatif |
| Full | 4056 x 3040 | ~10 | terlalu lambat untuk rPPG |

Sampling rPPG minimal ~30 fps; 1332x990 @ 60–120 fps memberi margin.

## 5. Kunci AE/AWB & Anti-Flicker (KRITIS untuk rPPG)

Sinyal rPPG sangat sensitif terhadap perubahan gain/eksposur otomatis dan kedip lampu AC.

- **Kunci** auto-exposure & auto-white-balance setelah warm-up agar intensitas kanal stabil.
- Set **exposure kelipatan 10 ms** (10.000 us) untuk menekan flicker jala-jala **50 Hz** (Indonesia).

Contoh dengan Picamera2 (dipakai di pipeline software):

```python
from picamera2 import Picamera2

picam2 = Picamera2()
config = picam2.create_video_configuration(
    main={"size": (1332, 990), "format": "RGB888"},
    controls={"FrameRate": 60},
)
picam2.configure(config)
picam2.start()

# warm-up singkat lalu kunci AE/AWB
import time; time.sleep(2)
meta = picam2.capture_metadata()
picam2.set_controls({
    "AeEnable": False,
    "AwbEnable": False,
    "ExposureTime": 10000,          # 10 ms -> anti-flicker 50 Hz
    "AnalogueGain": meta["AnalogueGain"],
    "ColourGains": meta["ColourGains"],
})
```

> Kompensasi redaman two-way mirror dilakukan lewat **iluminasi + eksposur**, bukan beralih ke sensor NoIR (rPPG butuh kanal RGB utuh). Lihat `mirror-attenuation-tests.md`.

## 6. Undistortion untuk ST-GCN

Landmark adalah input geometris; distorsi lensa membiaskan fitur. Kalibrasi intrinsik (papan catur, OpenCV), simpan `calib_imx477_65deg.yaml`, dan undistort frame sebelum ekstraksi landmark. Prosedur: `fov-calculation.md` bagian 7.

## 7. Troubleshooting singkat

| Gejala | Kemungkinan sebab | Tindakan |
| --- | --- | --- |
| Kamera tak terdeteksi | kabel CSI terbalik/longgar, overlay salah | reseat kabel, cek `dtoverlay`, reboot |
| Gambar gelap di balik mirror | redaman mirror + eksposur rendah | tambah iluminasi, naikkan exposure (jaga kelipatan 10 ms) |
| Sinyal rPPG berdenyut tak wajar | AE/AWB masih aktif atau flicker | kunci AE/AWB, set exposure 10 ms |
| Warna bergeser | AWB belum dikunci | kunci ColourGains |

Lihat juga `../troubleshooting.md`.

## 8. TODO

- [ ] Konfirmasi port CSI (cam0/cam1) yang dipakai + update `dtoverlay`.
- [ ] Kalibrasi intrinsik lensa 65°, simpan YAML.
- [ ] Catat setting exposure/gain final setelah uji mirror.
