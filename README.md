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
│   ├── imx708-noir-config.md
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

- Compute: Raspberry Pi 5 (16 GB). Bila desain 3D (Gambar 3 proposal) memakai Compute Module 5, samakan penamaan di `pinout-raspberrypi5.md` dan proposal agar tidak ada inkonsistensi Pi 5 vs CM5.
- Kamera: IMX708 NoIR (66° H / 41° V / 75° D).
- Display: Waveshare 10.1" IPS 1024×600 HDMI.
- Komunikasi: MQTT over TLS (port 8883).

## Catatan Teknis Kritis

- **Atenuasi mirror + NoIR (uji di `camera-module/`)**: two-way mirror menurunkan cahaya yang sampai ke sensor dan kamera NoIR mengubah karakter kanal warna. Ini berdampak langsung pada SNR sinyal rPPG dan pada domain input model (yang umumnya dilatih pada citra RGB). Dokumentasikan hasil `mirror-attenuation-tests.md` sebagai data untuk mengoreksi/kalibrasi, bukan sekadar lampiran.
- **FoV vs jarak kerja (`fov-calculation.md`)**: pada IMX708 dengan FoV vertikal 41°, tinggi cakupan pada jarak 30 cm hanya sekitar 22 cm — berisiko memotong kepala pengguna. Rekomendasi: jarak kerja ≥ 40 cm atau gunakan lensa wide. Cantumkan angka hasil hitung di file ini.
- **Anggaran daya (`power/`)**: bedakan rating PSU (mis. 5V/5A) dari konsumsi aktual. Kolom `measured-current-log.csv` adalah pembuktian; jangan mengklaim konsumsi memakai angka rating catu daya.

## Kontrak MQTT

`network/mqtt-topic-schema.md` **tidak** mendefinisikan ulang skema — ia mereferensikan kontrak kanonik di `MoodIn-master/docs/ipc-contract.md` dan hanya menambahkan detail transport (broker, TLS, QoS, retensi). Ini mencegah dua sumber kebenaran yang bisa saling menyimpang saat integrasi.

## Quick Start (perakitan)

Lihat [assembly-guide.md](assembly-guide.md).
