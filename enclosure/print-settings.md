# Pengaturan Cetak 3D — Enclosure MoodIn (Bambu Lab P1S)

> Enclosure didesain di **Fusion 360** (oleh anggota tim), dicetak dengan **Bambu Lab P1S** melalui **Bambu Studio**. Dokumen ini mencatat berkas cetak, alur konversi, dan parameter slicing.

## 1. Berkas Model 3D & Penempatannya

Berkas mesh siap-cetak (STL) diletakkan di `enclosure/stl/`. Berkas CAD sumber (editable) di `enclosure/cad/`.

| Berkas STL (dari tim) | Fungsi | Padanan di struktur README |
| --- | --- | --- |
| `Top_Frame.stl` | Bingkai depan penahan cermin two-way + panel | ~ `mirror-frame.step` |
| `base_enclosure1.stl` | Casing bawah (bagian 1) | ~ `bottom-case.step` (bagian 1) |
| `base_enclosure2.stl` | Casing bawah (bagian 2) | ~ `bottom-case.step` (bagian 2) |

> **Catatan konsistensi penamaan** [Medium]: nama berkas aktual (`Top_Frame`, `base_enclosure1/2`) berbeda dari nama placeholder di README lama (`mirror-frame`, `bottom-case`). Struktur README sudah diperbarui agar memakai nama sebenarnya + folder `stl/`. Simpan juga sumber Fusion `.f3d` (`full-assembly.f3d`) di `enclosure/cad/` agar desain tetap dapat diedit.

## 2. Alur Konversi (Fusion 360 -> Bambu Studio -> P1S)

1. **Ekspor dari Fusion 360**: `Top_Frame`, `base_enclosure1`, `base_enclosure2` sebagai **STL** (satuan mm, refinement High) -> simpan ke `enclosure/stl/`. Simpan juga arsip **STEP** untuk pertukaran & `.f3d` untuk sumber.
2. **Impor ke Bambu Studio**: pilih printer **Bambu Lab P1S**, pelat baja PEI (default). Impor ketiga STL.
3. **Orientasi & pelat**: orientasikan tiap part agar permukaan tampak (menghadap pengguna) minim garis layer & minim support. Bila muat, tata di satu pelat; bila tidak, pisah ke beberapa job.
4. **Slice** dengan profil di bagian 3, pratinjau jalur & estimasi waktu/filament.
5. **Ekspor 3MF project** (menyimpan orientasi + setting) ke `enclosure/print/` (buat folder ini saat file 3MF sudah ada) dan kirim job ke P1S (LAN/SD/cloud).

> Bambu Studio membaca STL langsung; tidak perlu konversi manual ke format lain. "Konversi" yang dimaksud = **slicing STL -> G-code/3MF** di dalam Bambu Studio.

## 3. Profil Slicing yang Direkomendasikan

Disetel untuk keseimbangan kekuatan, dimensi presisi (dudukan kamera & panel), dan estetika permukaan.

| Parameter | Nilai anjuran | Alasan |
| --- | --- | --- |
| Material | **PETG** (utama) atau **PLA** (prototipe) | PETG lebih tahan panas (dekat LED/elektronik) & lebih ulet; PLA lebih mudah untuk uji bentuk |
| Nozzle | 0,4 mm (bawaan P1S) | standar; presisi cukup |
| Layer height | **0,20 mm** (0,16 mm untuk permukaan tampak) | 0,20 cepat & kuat; 0,16 lebih halus |
| Wall loops | 3–4 | kekakuan bingkai & ulir sekrup |
| Top/bottom layers | 4–5 | permukaan solid, tidak tembus cahaya |
| Infill | **20–25% gyroid** | rasio kekuatan/berat baik untuk casing |
| Support | otomatis, hanya overhang > 50° | minimalkan bekas support di permukaan tampak |
| Adhesion | brim 5 mm bila part tinggi/sempit | cegah warping (khusus PETG) |
| Suhu (PETG) | nozzle ~240–250°C, bed ~70–80°C | ikuti profil filament Bambu |
| Suhu (PLA) | nozzle ~210°C, bed ~55–60°C | |

> P1S adalah printer **enclosed** -> cocok untuk PETG/ABS (mengurangi warping). Untuk ABS/ASA aktifkan ventilasi ruangan (asap). Untuk MoodIn, **PETG** cukup dan lebih aman dikerjakan.

## 4. Pertimbangan Desain-untuk-Cetak (DfM)

- **Toleransi rakit**: beri clearance ~0,2–0,3 mm pada lubang kamera, slot panel, dan sambungan `base_enclosure1`<->`base_enclosure2`.
- **Ventilasi**: sisakan celah/lubang angin untuk panas Pi 5 + LED (hindari heat-soak sensor yang menaikkan noise rPPG).
- **Dudukan kamera**: pastikan sudut & jarak sesuai jarak kerja nominal ~45 cm dan lensa 65° (lihat `camera-module/lens-m12-selection.md`).
- **Manajemen kabel**: kanal untuk CSI, HDMI, USB, dan daya agar tidak menekan konektor.

## 5. Log Cetak (isi saat mencetak)

| Part | Material | Layer | Infill | Waktu | Filament (g) | Hasil | Catatan |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Top_Frame | | | | | | | |
| base_enclosure1 | | | | | | | |
| base_enclosure2 | | | | | | | |

## 6. TODO

- [ ] Letakkan `Top_Frame.stl`, `base_enclosure1.stl`, `base_enclosure2.stl` ke `enclosure/stl/`.
- [ ] Simpan sumber `.f3d` + STEP ke `enclosure/cad/`.
- [ ] Simpan 3MF project hasil slice ke `enclosure/print/`.
- [ ] Uji-cetak prototipe PLA untuk cek fit sebelum cetak final PETG.
