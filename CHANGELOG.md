# Changelog — Hardware-Connections

Perubahan penting pada repo ini dicatat di sini. Perubahan yang berdampak ke repo lain ditandai **[lintas repo]**.

Format mengikuti [Keep a Changelog](https://keepachangelog.com/id/1.1.0/).

## [Unreleased] — fase Tugas Akhir

### Diubah

- `README.md` ditulis ulang: peran repo pada fase TA (termasuk integrasi dan _deployment_ di Jetson), status tiap komponen, dan status tiap dokumen. Rujukan ke `schematics/wiring-diagram.svg` yang tidak pernah ada dihapus.
- `network/mqtt-tls-config.md` dan `network/mqtt-topic-schema.md` ditandai arsip.
  - **[lintas repo]** Fase TA memakai MQTT di jaringan lokal tanpa TLS (ADR-10 di MoodIn-master).

### Ditambahkan

- `CHANGELOG.md` dan `.gitignore`.

## [gemastik-xix-2026] — 2026-10-27

Arsip akhir era GEMASTIK XIX 2026 (tim VisionaryX). Tag menunjuk commit `b02fb41` (3 Agustus 2026).

Isi utama: BOM, perhitungan FoV dan termal, konfigurasi IMX477, pemasangan layar Waveshare, CAD dan STL casing, serta dokumen jaringan MQTT dengan TLS.

[Unreleased]: https://github.com/MoodInBME/Hardware-Connections/compare/gemastik-xix-2026...HEAD
[gemastik-xix-2026]: https://github.com/MoodInBME/Hardware-Connections/tree/gemastik-xix-2026
