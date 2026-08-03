# Setup Panel Waveshare 10.1" (MoodIn)

> Panel kiosk di balik two-way mirror. UI dibangun dengan PySide6 + QML (repo `Interface-Display`). Dokumen ini fokus ke koneksi fisik, resolusi/EDID, touch, dan rotasi.

## 1. Spesifikasi & Varian

| Item | Nilai |
| --- | --- |
| Ukuran | 10.1" IPS |
| Resolusi | 1024 x 600 (varian yang dipakai MoodIn) |
| Video in | HDMI (varian `10.1DP-CAPLCD` memakai DisplayPort) |
| Touch | Kapasitif, USB HID (plug-and-play) |
| Daya | 5V (via USB / barrel sesuai board) |

> Verifikasi varian panel: bila kode produk mengandung **DP**, input video = DisplayPort (relevan saat migrasi Jetson yang memang keluar DP). Untuk Jetson Orin Nano Super (HDMI), gunakan varian HDMI atau adapter yang sesuai.

## 2. Koneksi ke Jetson Orin Nano Super

1. **Video**: kabel micro-HDMI (HDMI0 Jetson Orin Nano Super) -> HDMI panel.
2. **Touch**: kabel USB panel -> port USB 2.0 Jetson Orin Nano Super (sisakan USB 3.0 untuk NVMe/aksesori).
3. **Daya**: sesuai board panel (umumnya 5V terpisah; jangan bebani rail GPIO Pi).

## 3. Konfigurasi Resolusi (Jetson Orin Nano Super OS / Wayland)

Bila EDID tidak otomatis benar, paksa 1024x600 di `/boot/firmware/config.txt`:

```ini
# paksa mode bila panel tidak ter-EDID dengan benar
hdmi_group=2
hdmi_mode=87
hdmi_cvt=1024 600 60 6 0 0 0
hdmi_drive=2
```

> Jetson Orin Nano Super default memakai stack KMS/Wayland; untuk sesi Wayland gunakan pengaturan tampilan desktop atau `wlr-randr`. Blok `hdmi_*` di atas tetap berguna sebagai fallback firmware.

## 4. Rotasi Layar (jika orientasi potret)

Banyak desain cermin memakai orientasi **potret**. Sesuaikan rotasi video **dan** touch.

- X11: `xrandr --output HDMI-1 --rotate left`
- Wayland (wlroots): `wlr-randr --output HDMI-A-1 --transform 90`
- Sesuaikan matriks transform touch (libinput) agar titik sentuh ikut terputar. Bila tidak, sumbu touch akan tertukar.

> Saat migrasi Jetson, panel via **DP->HDMI adapter aktif**; rotasi memakai `xrandr --output DP-0 --rotate left`.

## 5. Mode Kiosk

- Jalankan aplikasi UI fullscreen saat boot (autostart). Sembunyikan kursor, matikan screen blanking:

  ```bash
  xset s off; xset -dpms; xset s noblank   # sesi X11
  ```

- Detail integrasi aplikasi (PySide6/QML) ada di repo `Interface-Display`.

## 6. Troubleshooting singkat

| Gejala | Tindakan |
| --- | --- |
| Layar hitam | cek kabel HDMI0, paksa mode di config.txt |
| Resolusi salah/blur | set `hdmi_cvt` 1024x600 |
| Touch tak sesuai posisi | perbaiki transform libinput (rotasi) |
| Layar blank saat idle | matikan DPMS/screen blank |

## 7. TODO

- [ ] Konfirmasi varian panel (HDMI vs DP).
- [ ] Tetapkan orientasi (lanskap/potret) sesuai enclosure final.
- [ ] Simpan matriks transform touch bila potret.
