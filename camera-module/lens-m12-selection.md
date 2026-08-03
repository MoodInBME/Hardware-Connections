# Pemilihan Lensa M12 — MoodIn (Arducam IMX477)

> Panduan memilih lensa M12 dari 5 opsi FoV (45/55/65/70/90 derajat). Angka cakupan diturunkan di `fov-calculation.md` (dokumen kanonik).

## Rekomendasi (ringkas)

- **Utama: lensa 55 derajat (FoV diagonal), jarak kerja nominal ~45 cm.** [High untuk 45-50 cm; Medium untuk 40 cm]
- **Cadangan: 70 derajat** bila enclosure memaksa pengguna lebih dekat (35-40 cm) atau tim ingin margin framing lebih besar saat demo langsung.
- **Hindari 45 & 55 derajat** (kepala rawan terpotong / margin terlalu tipis untuk variasi tinggi pengguna) dan **90 derajat** (distorsi besar, wajah kecil, menangkap latar berlebih -> risiko privasi).

**Alasan inti:** pada jarak 45 cm, lensa 55 derajat memuat kepala 23 cm dengan margin +/-5-6 cm (toleran terhadap perbedaan tinggi & gerak), distorsi moderat (penting untuk geometri landmark ST-GCN), dan bidang tidak terlalu lebar sehingga mengurangi tangkapan orang/latar lain (relevan dengan UU PDP No. 27/2022). Kepadatan piksel wajah (~660 px pada mode 1332x990) jauh di atas kebutuhan rPPG.

## Kriteria keputusan

1. **Framing robust** — cermin dipakai banyak orang dengan tinggi berbeda; butuh margin vertikal, bukan sekadar "muat".
2. **Kepadatan piksel wajah (rPPG)** — makin banyak piksel skin ROI makin baik SNR, tetapi di atas ~300-400 px sudah memadai.
3. **Distorsi (ST-GCN)** — landmark adalah input geometris; lensa wide menambah distorsi barel yang membiaskan fitur ekspresi.
4. **Cahaya** — two-way mirror meredam cahaya; utamakan lensa aperture besar (f-number kecil) + mode sensor binned.
5. **Privasi** — FoV lebih sempit menangkap lebih sedikit latar/bystander.

## Perbandingan (jarak kerja 45 cm, mode 1332 x 990)

| Lensa (DFoV) | Cakupan H x V (cm) | Margin kepala | Piksel wajah | Distorsi | Latar tertangkap | Verdikt |
| --- | --- | --- | --- | --- | --- | --- |
| 45 deg | 29,8 x 22,4 | pas-pasan | ~740 | rendah | sempit | Ditolak (margin tipis) |
| 55 deg | 37,5 x 28,1 | +/-2,5 cm | ~810 | rendah | sempit | Marginal |
| 55 deg | 45,9 x 34,4 | +/-5,7 cm | ~660 | moderat | sedang | **PILIHAN** |
| 70 deg | 50,4 x 37,8 | +/-7,4 cm | ~600 | moderat-tinggi | agak lebar | Cadangan |
| 90 deg | 72,0 x 54,0 | sangat longgar | ~420 | tinggi | lebar | Ditolak (distorsi & privasi) |

## Catatan pemasangan

- Pasang kamera setinggi ~mata pengguna rata-rata di balik area cermin; hindari sudut tilt ekstrem (distorsi perspektif pada landmark).
- Tetapkan "sweet spot" berdiri (tanda lantai atau panduan di UI) pada ~45 cm agar wajah konsisten terframe dan jarak fokus stabil.
- Setel back-focus M12 satu kali pada jarak kerja nominal lalu kunci (thread-lock) — fokus manual, tidak ada autofokus pada varian ini.

## Mode capture yang disarankan

- 1332 x 990 @ ~60-90 fps (binned) untuk rPPG: fps tinggi + SNR lebih baik. Kunci AE/AWB dan frame rate; exposure kelipatan 10 ms (anti-flicker 50 Hz). Lihat `imx477-pi5-config.md`.

## Verifikasi sebelum final [wajib]

1. Konfirmasi konvensi FoV lensa (diagonal vs horizontal) dari datasheet. Bila horizontal, cakupan lebih besar dan 55 derajat bisa kembali jadi kandidat. Lihat `fov-calculation.md` bagian 2.
2. Ukur jarak kerja aktual pada enclosure; hitung ulang memakai skrip di `fov-calculation.md` bagian 6.
3. Uji SNR rPPG di balik two-way mirror untuk lensa 55 derajat (dan 70 derajat sebagai pembanding).
4. Kalibrasi intrinsik + undistortion untuk lensa terpilih (`fov-calculation.md` bagian 7).

## Keputusan tim (isi saat final)

- Lensa dipilih: __________ (target: 55 derajat)
- Jarak kerja final: __________ cm
- Tanggal kalibrasi: __________ / file: `calib_imx477_<lens>.yaml`
