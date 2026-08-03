# Anggaran Daya — MoodIn (Persamaan 5)

> Estimasi konsumsi daya sistem untuk sizing catu daya & termal. Angka rating **bukan** konsumsi aktual; pembuktian ada di `measured-current-log.csv`.

## 1. Persamaan 5

```text
(Pers. 5)   P_total = sum(P_i)  untuk semua beban i
            P_PSU  >= P_total / eta * (1 + headroom)
```

dengan `eta` = efisiensi konversi catu (~0,85–0,9) dan `headroom` = margin keamanan (anjuran 0,30). Arus per rail: `I = P / V`.

## 2. Estimasi Konsumsi (baseline Jetson Orin Nano Super) [Medium — ganti dengan hasil ukur]

| Beban | Kondisi | Tegangan (V) | Arus tipikal (A) | Daya (W) |
| --- | --- | --- | --- | --- |
| Jetson Orin Nano Super (SoC+RAM) | inferensi aktif | 5,1 | 1,2–1,5 | ~6–8 |
| Kamera IMX477 (CSI) | streaming | 3,3 | ~0,2 | ~0,7 |
| NVMe SSD (opsional) | baca/tulis | 3,3 | 0,3–0,6 | ~1–2 |
| Panel Waveshare 10.1" | backlight nyala | 5 | 0,5–0,9 | ~2,5–4,5 |
| Touch panel (USB) | aktif | 5 | ~0,05 | ~0,25 |
| Strip LED iluminasi | kompensasi mirror | 5–12 | bergantung desain | ~2–5 |
| **Total sistem** | puncak | — | — | **~14–22 W** |

> Jetson Orin Nano Super sendiri sudah butuh PSU **19V/2.7A (43 W)** resmi; angka itu menyediakan headroom untuk SoC + kamera + NVMe. Display & LED **harus** dari catu terpisah (lihat `schematics/power-distribution.md`).

## 3. Sizing Catu

Contoh dengan `P_total` = 22 W, `eta` = 0,88, `headroom` = 0,30:

```text
P_PSU >= 22 / 0,88 * 1,30 = 32,5 W
```

- **Compute (Jetson Orin Nano Super)**: PSU PD resmi 43 W — cukup untuk Pi + kamera + NVMe (tanpa membebani display/LED).
- **Display + LED**: catu 5V terpisah dengan margin (mis. 5V/3–4A) atau 12V untuk LED bila dipilih 12V.

## 4. Termal

- Panas Jetson Orin Nano Super + LED dalam enclosure tertutup menaikkan suhu sensor -> **noise rPPG naik**. Sediakan ventilasi/aliran udara (lihat DfM di `../enclosure/print-settings.md`).
- Pantau throttling Pi: `vcgencmd get_throttled` (0x0 = sehat).

## 5. Catatan Migrasi Jetson (rencana)

Jetson Orin Nano Super mode **7–25 W** (lebih tinggi dari Jetson Orin Nano Super pada beban penuh Super Mode) + catu barrel DC. Saat migrasi, hitung ulang bagian 2–3 dengan angka Jetson dan perbarui rating catu.

## 6. TODO

- [ ] Ukur arus aktual tiap kondisi (idle/inferensi/display+LED) -> isi `measured-current-log.csv`.
- [ ] Bandingkan aktual vs estimasi; revisi tabel bagian 2.
- [ ] Finalkan rating catu display+LED.
