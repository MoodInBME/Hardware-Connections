# Panduan Perakitan — MoodIn Smart Mirror

> Urutan perakitan dari komponen ke unit cermin pintar yang berfungsi. Rujuk `schematics/` (wiring), `enclosure/` (casing), dan dokumen modul terkait di tiap langkah.

## 0. Persiapan

- Cetak enclosure (`enclosure/print-settings.md`): `Top_Frame`, `base_enclosure1`, `base_enclosure2`.
- Siapkan komponen sesuai `bom/full-bom.csv`.
- Flash OS Jetson Orin Nano Super (Ubuntu 24.04) + provisioning WiFi (`network/wifi-provisioning.md`).

## 1. Rakit Rangka & Casing

1. Gabungkan `base_enclosure1` + `base_enclosure2` (casing bawah). Cek fit; beri clearance bila seret (lihat DfM).
2. Pasang two-way mirror pada `Top_Frame`. Sisi reflektif menghadap keluar (ke pengguna); sisi transmisi menghadap kamera.
3. Jangan kunci penuh Top_Frame dulu — elektronik dipasang lebih dahulu.

## 2. Pasang Compute (Jetson Orin Nano Super)

1. Pasang active cooler pada Jetson Orin Nano Super.
2. Tempatkan Jetson Orin Nano Super pada dudukan casing bawah (standoff), sisakan akses port HDMI/USB/USB-C.
3. (Opsional) pasang M.2 HAT + SSD NVMe.

## 3. Kamera

1. Pasang lensa M12 55° ke board IMX477; setel back-focus nanti.
2. Sambungkan kamera ke Jetson Orin Nano Super lewat adapter CSI **15->22 pin** (Pi mati). Perhatikan orientasi kontak.
3. Posisikan kamera di belakang mirror, setinggi ~mata pengguna, tegak lurus bidang (hindari tilt ekstrem). Jarak kerja nominal **~45 cm** ke posisi berdiri.
4. Aktivasi & uji: `camera-module/imx477-pi5-config.md`.

## 4. Display + Touch

1. Micro-HDMI (HDMI0) -> panel; kabel USB touch -> USB 2.0 Pi.
2. Pasang panel pada frame; sejajarkan area tampil dengan bagian mirror yang akan "menyala".
3. Setup resolusi/rotasi: `display/waveshare-10.1-setup.md`.

## 5. Iluminasi

1. Pasang strip LED di sekeliling bidang wajah (hindari pantulan langsung ke lensa).
2. Beri catu terpisah + MOSFET (kontrol PWM opsional dari GPIO18). Lihat `schematics/power-distribution.md`.

## 6. Daya

1. Jetson Orin Nano Super dari PSU PD 43 W resmi.
2. Display & LED dari catu terpisah (jangan dari 5V GPIO Pi).
3. Nyalakan; cek under-voltage: `vcgencmd get_throttled` -> harus `0x0`.

## 7. Uji Fungsi

1. Kamera: `nvargus-hello --list-cameras` + pratinjau.
2. Display + touch berfungsi & orientasi benar.
3. Uji atenuasi mirror + SNR rPPG (`camera-module/mirror-attenuation-tests.md`).
4. Konektivitas broker MQTT (`network/mqtt-tls-config.md`).
5. Kunci back-focus M12 (thread-lock) setelah fokus tajam pada jarak kerja.

## 8. Penutupan

1. Rapikan kabel ke kanal (CSI/HDMI/USB/daya) tanpa menekan konektor.
2. Kunci `Top_Frame` ke casing bawah.
3. Verifikasi ventilasi tidak tertutup.

## 9. Checklist Akhir

- [ ] Kamera terdeteksi, fokus terkunci, undistortion siap
- [ ] Display + touch OK (orientasi benar)
- [ ] AE/AWB terkunci, exposure kelipatan 10 ms
- [ ] SNR rPPG lolos kriteria
- [ ] MQTT tersambung (TLS di produksi)
- [ ] Tidak ada under-voltage/throttling
- [ ] Kabel rapi & enclosure terkunci

> Masalah saat perakitan/uji: lihat `troubleshooting.md`.
