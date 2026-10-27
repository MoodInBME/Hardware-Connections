# Hardware-Connections

Perangkat keras Tugas Akhir **MoodIn**: kamera di balik cermin dua arah, NVIDIA Jetson Orin Nano Super, layar, pencahayaan, casing, dan daya. Sejak fase TA, repo ini juga menampung **integrasi dan _deployment_ sistem di Jetson**, yaitu skrip pemasangan, layanan, dan diagnosis.

Bagian dari proyek [MoodIn](https://github.com/MoodInBME/MoodIn-master). Model ada di [Software-Architecture](https://github.com/MoodInBME/Software-Architecture), dan antarmuka di [Interface-App](https://github.com/MoodInBME/Interface-App).

![layer](https://img.shields.io/badge/layer-hardware-lightgrey)
![tahap](https://img.shields.io/badge/tahap-preliminary%20research-yellow)

> **Status (27 Okt 2026):** dokumen di repo ini masih versi era GEMASTIK, sama dengan tag [`gemastik-xix-2026`](https://github.com/MoodInBME/Hardware-Connections/tree/gemastik-xix-2026). Beberapa komponen akan berubah mengikuti bimbingan TA. Tabel di bawah menandai mana yang tetap dan mana yang menunggu keputusan.

## 1. Komponen prototipe

| Komponen               | Prototipe saat ini                                             | Status fase TA                                               |
| ---------------------- | -------------------------------------------------------------- | ------------------------------------------------------------ |
| Komputer edge          | NVIDIA Jetson Orin Nano Super 8 GB, Ubuntu 24.04, JetPack 7.2  | Tetap                                                        |
| Kamera                 | Arducam IMX477 (RGB, filter IR-cut tetap) dengan lensa M12 55° | Opsi modul dengan IR-cut bermotor menunggu bimbingan         |
| Layar                  | Waveshare 10,1" 1024×600                                       | Akan diganti dengan layar yang lebih besar                   |
| Cermin dan pencahayaan | Cermin dua arah di depan kamera dan layar                      | Susunan dan sumber cahaya menunggu bimbingan                 |
| Casing                 | Cetak 3D, desain era GEMASTIK                                  | Disesuaikan setelah layar dan kamera final                   |
| Jaringan               | MQTT di jaringan lokal                                         | Tanpa TLS; _username/password_ broker dan enkripsi isi pesan |

Angka daya, termal, dan FoV di dokumen era GEMASTIK perlu diukur atau dihitung ulang setelah komponen final.

## 2. Menyalakan kamera CSI (IMX477 di Jetson)

Kamera IMX477 di Jetson Orin Nano Super memakai kabel CSI dengan adapter **15-pin ke 22-pin**. Error **`-121`** biasanya menandakan masalah kabel: terbalik, tidak terkunci, atau terlalu panjang.

Urutan diagnosis:

1. `ls /dev/video*`
2. `dmesg | grep -i imx477`
3. `v4l2-ctl --list-devices`, lalu `v4l2-ctl --list-formats-ext`
4. Ambil satu _frame_ dengan `nvarguscamerasrc` (GStreamer) atau libargus. Jangan lanjut ke pemrosesan wajah sebelum tahap ini lulus.

## 3. Struktur repo

```text
Hardware-Connections/
├── bom/                  daftar komponen dan harga (era GEMASTIK)
├── camera-module/        lensa, FoV, konfigurasi IMX477, uji atenuasi cermin
├── display/              pemasangan layar Waveshare (era GEMASTIK)
├── enclosure/            CAD (STP), STL, berkas cetak, pengaturan cetak
├── network/              jaringan dan MQTT
├── power/                anggaran dan log pengukuran daya
├── schematics/           distribusi daya
├── assembly-guide.md     urutan perakitan
└── troubleshooting.md    diagnosis masalah umum
```

Folder `jetson/` untuk skrip integrasi dan _deployment_ ditambahkan setelah skrip diagnosis dipindah dari Software-Architecture.

## 4. Status dokumen

| Dokumen                                                          | Status                                                                        |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `camera-module/fov-calculation.md`, `lens-m12-selection.md`      | Berlaku untuk lensa M12 55°; hitung ulang bila lensa atau jarak kerja berubah |
| `camera-module/imx477-jetson-config.md`, `troubleshooting.md`    | Berlaku sebagian; dirapikan di M2                                             |
| `camera-module/mirror-attenuation-tests.md`                      | Log uji era GEMASTIK; diulang setelah susunan cermin final                    |
| `display/waveshare-10.1-setup.md`                                | Era GEMASTIK; layar akan diganti                                              |
| `bom/full-bom.csv`, `power/`, `schematics/`, `assembly-guide.md` | Era GEMASTIK; diperbarui setelah komponen final (M2)                          |
| `enclosure/`                                                     | Desain era GEMASTIK; direvisi mengikuti layar dan kamera baru                 |
| `network/mqtt-tls-config.md`                                     | **Arsip.** Fase TA tidak memakai TLS                                          |
| `network/mqtt-topic-schema.md`                                   | **Arsip.** Mengacu kontrak v1.0.0; kontrak versi TA menyusul di MoodIn-master |
| `network/wifi-provisioning.md`                                   | Ditinjau di M2                                                                |

## 5. Pekerjaan berikutnya (M2)

- [ ] Keputusan bimbingan: kamera, layar pengganti, susunan cermin, dan pencahayaan.
- [ ] Uji kamera di Jetson dengan susunan cermin yang baru.
- [ ] Skrip diagnosis Jetson dipindah dari Software-Architecture ke folder `jetson/` di repo ini.
- [ ] BOM, daya, dan termal diperbarui setelah komponen final.

## Lisensi

Belum ditetapkan; menunggu keputusan pembimbing. Tanpa berkas `LICENSE`, isi repo ini tidak dilisensikan untuk dipakai ulang.
