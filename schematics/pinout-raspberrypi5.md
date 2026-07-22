# Pinout & Koneksi — Raspberry Pi 5 (MoodIn)

> Board edge **utama** untuk Cross-Eval 2. Dokumen ini memetakan seluruh koneksi fisik peripheral MoodIn ke Raspberry Pi 5. Bila desain 3D memakai **Compute Module 5 (CM5)**, samakan penamaan dengan proposal (lihat catatan konsistensi di README).

## 1. Ikhtisar Antarmuka yang Dipakai

| Peripheral | Antarmuka Pi 5 | Konektor fisik |
| --- | --- | --- |
| Kamera Arducam IMX477 | MIPI CSI-2 | 1 dari 2 port CSI/DSI 4-lane (kabel 22-pin 0.5 mm di Pi 5) |
| Display Waveshare 10.1" | HDMI | micro-HDMI (port HDMI0) via kabel micro-HDMI -> HDMI |
| Touch panel Waveshare | USB | USB 2.0 (HID, plug-and-play) |
| Iluminasi LED (kompensasi mirror) | GPIO/5V eksternal | jangan tarik dari 5V GPIO bila arus > 500 mA (pakai catu terpisah) |
| Daya sistem | USB-C PD | PSU resmi 27 W (5,1 V / 5 A) |
| Penyimpanan | microSD / NVMe | microSD (baseline) atau SSD NVMe via M.2 HAT (PCIe) |

> Catatan CSI Pi 5: konektor kamera Pi 5 adalah **22-pin 0.5 mm** (berbeda dari 15-pin pada Pi 4/CM). Board IMX477 umumnya 15-pin -> gunakan **kabel 15-pin ke 22-pin** yang sesuai untuk Pi 5.

## 2. Header GPIO 40-pin (yang relevan)

Hanya pin yang dipakai MoodIn didaftarkan; sisanya bebas/cadangan.

| Pin fisik | BCM | Fungsi di MoodIn | Keterangan |
| --- | --- | --- | --- |
| 1 | 3V3 | Referensi 3,3 V | untuk sensor logika 3,3 V (mis. sensor presence PIR bila dipakai) |
| 2 | 5V | Suplai 5 V | hati-hati anggaran arus; jangan untuk LED besar |
| 4 | 5V | Suplai 5 V | idem |
| 6 | GND | Ground | ground bersama |
| 11 | GPIO17 | Trigger sensor presence (opsional) | input; jika modul PIR/ToF dipakai |
| 12 | GPIO18 | PWM iluminasi (opsional) | kontrol dimming LED via MOSFET, BUKAN drive langsung |
| 9 | GND | Ground | |
| 39 | GND | Ground | |

> Prinsip: GPIO Pi hanya untuk **sinyal kontrol arus kecil**. Beban daya (LED, kipas eksternal) harus lewat MOSFET/driver dengan catu terpisah. Lihat `power-distribution.md`.

## 3. Kamera (MIPI CSI-2)

- Port: CAM/DISP 0 atau 1 (keduanya bisa kamera di Pi 5).
- Orientasi kabel: sisi kontak biru menghadap port Ethernet (verifikasi ke silkscreen board).
- Aktivasi & pengujian: lihat `camera-module/imx477-pi5-config.md`.

## 4. Display + Touch

- Video: micro-HDMI (HDMI0) -> HDMI panel. Panel Waveshare 10.1" 1024x600.
- Touch: kabel USB ke port USB 2.0 (bukan USB 3.0 biru, agar USB 3.0 disisakan untuk NVMe/aksesori bila perlu).
- Detail EDID/resolusi & rotasi: lihat `display/waveshare-10.1-setup.md`.

## 5. Diagram Wiring

Diagram lengkap ada di `schematics/wiring-diagram.svg` (sumber editable `.fzz` Fritzing). Diagram harus mencerminkan:

1. Jalur daya USB-C PD -> Pi 5.
2. CSI -> kamera; HDMI + USB -> display.
3. Jalur iluminasi LED terpisah (catu sendiri) + kontrol MOSFET dari GPIO18 (bila dipakai).

## 6. TODO

- [ ] Finalkan apakah memakai Pi 5 board penuh atau CM5 (samakan dengan Gambar 3 proposal).
- [ ] Konfirmasi part number kabel adapter CSI 15->22 pin.
- [ ] Ekspor `wiring-diagram.svg` dari Fritzing setelah wiring final.
