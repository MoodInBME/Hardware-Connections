# Hardware-Connections

Root konfigurasi hardware, protokol fisik, dan komunikasi untuk MoodIn.
Bagian dari proyek [MoodIn](../../MoodIn-master).

![https://img.shields.io/badge/github-repo-blue?logo=github](https://img.shields.io/badge/github-repo-blue?logo=github)

## Isi

- `schematics/` — wiring & pinout
- `enclosure/` — desain 3D casing
- `camera-module/` — konfigurasi kamera & hasil uji sinyal
- `network/` — MQTT, TLS, WiFi
- `power/` — perhitungan & pengukuran daya
- `bom/` — bill of materials

## Struktur Repository Ini

```bash
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
│   ├── mirror-attenuation-tests.md  # log uji Persamaan 7
│   └── fov-calculation.md           # acuan Persamaan 9
├── display/
│   └── waveshare-10.1-setup.md
├── network/
│   ├── mqtt-tls-config.md
│   ├── wifi-provisioning.md
│   └── mqtt-topic-schema.md         # penting: skema topik MQTT
├── power/
│   ├── power-budget-calc.md         # acuan Persamaan 5
│   └── measured-current-log.csv     # hasil ukur aktual vs estimasi
├── bom/
│   └── full-bom.csv
├── assembly-guide.md
└── troubleshooting.md
```

## Quick Start (perakitan)

Lihat [assembly-guide.md](assembly-guide.md)
