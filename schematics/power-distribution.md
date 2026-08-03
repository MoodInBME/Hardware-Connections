# Distribusi Daya — MoodIn

> Peta jalur daya dari sumber ke setiap beban. Angka konsumsi & anggaran ada di `power/power-budget-calc.md`; pengukuran aktual di `power/measured-current-log.csv`.

## 1. Topologi Daya

```text
PLN 220V AC
   |
   v
[Adaptor USB-C PD 43 W]  (Jetson Orin Nano Super)
   |-- 19V/2.7A --> Jetson Orin Nano Super (SoC, RAM, NVMe, kamera CSI)
   |               |-- 3,3V (regulator on-board) --> logika GPIO, kamera
   |
[Catu 5V terpisah / DC 12V]  (beban display & iluminasi)
   |-- 5V ---------> Panel Waveshare 10.1" (via board driver panel)
   |-- 5V/12V -----> Strip LED iluminasi (via driver + MOSFET)
```

**Prinsip kritis:** display dan LED iluminasi **tidak** ditarik dari rail 5V GPIO Jetson Orin Nano Super. Jetson Orin Nano Super sendiri sudah mendekati batas anggaran arus USB-C PD; menambah beban lewat GPIO 5V berisiko brownout (under-voltage) yang memicu throttling atau reboot.

## 2. Rail & Beban

| Rail | Sumber | Beban | Catatan proteksi |
| --- | --- | --- | --- |
| 19V/2.7A | PSU PD resmi | Jetson Orin Nano Super + kamera + NVMe | pakai PSU resmi; PSU underrated -> under-voltage warning |
| 5V (panel) | catu display / board panel | Waveshare 10.1" | ikuti spesifikasi board panel |
| 5V/12V (LED) | catu terpisah | strip LED | sekring/PTC + MOSFET; dimming via PWM GPIO18 |

## 3. Grounding

- **Common ground** antara Jetson Orin Nano Super, driver LED, dan panel bila ada sinyal kontrol lintas-domain (mis. PWM dari GPIO ke MOSFET LED). Tanpa common ground, sinyal PWM tidak punya referensi.
- Hindari ground loop: satu titik ground utama (star grounding) untuk domain daya berbeda.

## 4. Proteksi

- Sekring/PTC pada jalur LED (beban paling mungkin korslet saat perakitan).
- Kapasitor bulk (mis. 1000 uF) dekat strip LED untuk meredam lonjakan saat switching.
- Kabel USB-C berkualitas & pendek untuk Jetson Orin Nano Super (kabel tipis menyebabkan drop tegangan -> under-voltage).

## 5. Catatan Migrasi Jetson (rencana)

Jetson Orin Nano Super memakai barrel DC (mode 7–25 W) dan mengeluarkan DisplayPort. Saat migrasi: ganti domain daya compute ke catu DC Jetson, tinjau ulang anggaran di `power/power-budget-calc.md`, dan tambah adapter aktif DP->HDMI untuk panel.

## 6. TODO

- [ ] Tentukan apakah LED iluminasi 5V atau 12V (pengaruh ke pilihan catu).
- [ ] Finalkan rating catu display + LED setelah pengukuran (`measured-current-log.csv`).
- [ ] Gambar skema daya final di `wiring-diagram.svg`.
