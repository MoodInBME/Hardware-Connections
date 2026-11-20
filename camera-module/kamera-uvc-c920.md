# Kamera UVC Logitech C920 (usulan)

> **Status: usulan, menunggu keputusan bimbingan** (ADR-15 di MoodIn-master). Kamera utama yang berlaku tetap Raspberry Pi HQ Camera (IMX477) lewat CSI sampai keputusan diambil.

## Setelan yang diusulkan

| Butir | Nilai |
| --- | --- |
| Antarmuka | USB, driver `uvcvideo` (V4L2) |
| Format | YUYV (tanpa kompresi) |
| Resolusi dan laju frame | 640 × 480, 30 fps |
| Pembukaan di perangkat lunak | OpenCV `CAP_V4L2` dengan FOURCC `YUYV` |
| Pilihan di Software-Architecture | `camera_backend: uvc` di `config/default.yaml` (bawaan `csi`) |

Periksa format yang benar-benar tersedia sebelum dipakai:

```bash
v4l2-ctl -d /dev/video0 --list-formats-ext
```

## Mengunci kontrol kamera

Jalankan `scripts/kunci_kontrol_uvc.sh` dari repo Software-Architecture sebelum akuisisi. Skrip membaca daftar kontrol perangkat, lalu hanya mengatur kontrol yang ada:

1. frekuensi jala-jala 50 Hz;
2. eksposur manual, dengan laju frame tidak boleh diturunkan otomatis oleh kamera;
3. _white balance_ manual;
4. fokus otomatis mati, fokus manual.

Nama kontrol berbeda antarversi kernel (misalnya `exposure_auto` dan `auto_exposure`); skrip menangani keduanya. Nilai eksposur, _white balance_, dan fokus dibaca dari `config/uvc_kontrol.conf` dan ditetapkan setelah kalibrasi di bawah lampu tetap penelitian. Skrip mencetak nilai yang terbaca kembali; simpan keluaran itu bersama catatan sesi.

## Uji banding CSI dan UVC

`scripts/uji_banding_kamera.py` membuka kamera dengan backend pilihan dan **hanya menulis angka ke CSV**, tanpa menyimpan frame:

- rerata dan simpangan baku selang antar-frame dari cap waktu;
- jumlah frame hilang;
- rerata kecerahan (luma) daerah wajah per jendela 10 detik, sebagai indikator relatif.

Simpan keluaran CSV di luar repo. Jalankan kedua backend pada lampu, jarak, dan durasi yang sama.

## Catatan

- Video dengan resolusi lebih tinggi dari C920 umumnya dikirim terkompresi; resolusi 640 × 480 dipilih agar tetap tanpa kompresi.
- Frame dan rekaman uji yang memuat wajah tidak di-commit (ADR-14).
