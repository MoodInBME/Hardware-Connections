# Troubleshooting — Hardware MoodIn

> Masalah umum saat perakitan/pengujian dan langkah diagnosis. Untuk detail modul, rujuk dokumen terkait.

## Kamera (IMX477)

| Gejala | Kemungkinan sebab | Tindakan |
| --- | --- | --- |
| Tidak terdeteksi (`--list-cameras` kosong) | kabel CSI terbalik/longgar; overlay salah; adapter 15->22 bermasalah | matikan, reseat kabel (orientasi kontak), cek `dtoverlay=imx477`, reboot |
| Error I/O / `-121` (EREMOTEIO) | masalah fisik jalur CSI (kabel/adapter) | reseat konektor kedua ujung; ganti kabel adapter |
| Gambar gelap di balik mirror | redaman two-way mirror + eksposur rendah | tambah iluminasi; naikkan exposure (jaga kelipatan 10 ms); lihat `mirror-attenuation-tests.md` |
| rPPG berdenyut tak wajar / drift | AE/AWB masih aktif; flicker AC 50 Hz | kunci AE/AWB; set exposure 10 ms |
| Warna bergeser | AWB belum dikunci | kunci ColourGains |
| Landmark ST-GCN meleset di tepi | distorsi lensa belum dikoreksi | kalibrasi intrinsik + undistort (`fov-calculation.md` bag. 7) |

## Daya

| Gejala | Sebab | Tindakan |
| --- | --- | --- |
| Ikon petir / under-voltage | PSU underrated; kabel USB-C tipis | pakai PSU PD 27 W resmi + kabel pendek berkualitas |
| Reboot acak saat beban puncak | brownout (display/LED membebani rail Pi) | pisahkan catu display & LED (`power-distribution.md`) |
| Throttling / panas | ventilasi kurang | cek `vcgencmd get_throttled`; tambah aliran udara |
| LED tidak menyala/redup | catu/MOSFET/ground | cek common ground; ukur tegangan rail LED |

## Display Waveshare 10.1"

| Gejala | Tindakan |
| --- | --- |
| Layar hitam | cek HDMI0; paksa mode `hdmi_cvt 1024 600 60` |
| Resolusi/blur salah | set `hdmi_group/hdmi_mode/hdmi_cvt` |
| Touch tak sesuai posisi | perbaiki matriks transform libinput (terutama saat rotasi) |
| Blank saat idle | matikan DPMS/screen blank |

## Jaringan / MQTT

| Gejala | Tindakan |
| --- | --- |
| Tak dapat IP | cek `nmcli`, band 2,4/5 GHz, captive portal kampus |
| TLS handshake gagal | CN/SAN sertifikat harus cocok hostname broker; cek `cafile` |
| Pesan tak sampai | cek topik sesuai kontrak kanonik; QoS/retained; ACL broker |
| Perangkat "hilang" tanpa notifikasi | pastikan LWT terdaftar di `moodin/presence` |

## Enclosure / Cetak (Bambu Lab P1S)

| Gejala | Tindakan |
| --- | --- |
| Part warping (PETG) | pakai brim; suhu bed 70–80°C; printer enclosed (P1S) tutup rapat |
| Part tidak fit | tambah clearance 0,2–0,3 mm; uji-cetak prototipe PLA dulu |
| Permukaan tampak kasar | layer 0,16 mm untuk face; orientasi minim garis di permukaan tampak |
| Support susah dilepas | batasi support ke overhang > 50°; hindari di permukaan tampak |

## Jetson (rencana peningkatan)

| Gejala | Tindakan |
| --- | --- |
| `nvarguscamerasrc` tidak ada | pastikan JetPack 6.2.1 (bukan 7.2); Argus/ISP penuh di 6.2.1 |
| Kamera hanya V4L2 mentah (RG10) | driver ISP belum aktif; aktifkan overlay `jetson-io` / driver Arducam |
| Display tak keluar | Jetson = DisplayPort; pakai adapter aktif DP->HDMI |
| Inferensi lambat | konversi ONNX->TensorRT INT8; `nvpmodel -m 2` + `jetson_clocks` |

> Bila masalah menyangkut model/inferensi (bukan hardware), rujuk repo `Software-Architecture`.
