# Provisioning WiFi — MoodIn Edge

> Cara perangkat cermin terhubung ke jaringan (untuk sinkronisasi cloud/broker & update). Fokus Raspberry Pi 5; catatan Jetson di akhir.

## 1. Opsi Provisioning

| Metode | Kapan dipakai | Catatan |
| --- | --- | --- |
| Pra-konfigurasi via Raspberry Pi Imager | saat flashing OS | paling mudah untuk 1 jaringan tetap |
| NetworkManager (`nmcli`) | ganti/ tambah jaringan di lapangan | Pi OS Trixie memakai NetworkManager |
| Headless `firstrun`/`wpa_supplicant` | tanpa monitor | untuk deployment massal |
| Portal captive (softAP) | pengguna akhir memilih WiFi sendiri | perlu tooling tambahan (mis. Comitup) |

## 2. Raspberry Pi Imager (saat flashing)

Saat menulis OS, buka **advanced options**: set SSID + password, negara WiFi (**ID**), hostname, aktifkan SSH. Cocok untuk jaringan tim yang tetap.

## 3. NetworkManager (`nmcli`) — Pi OS Trixie

```bash
# daftar jaringan
nmcli dev wifi list

# sambungkan
sudo nmcli dev wifi connect "<SSID>" password "<password>"

# koneksi statis tersimpan (autoconnect)
sudo nmcli connection modify "<SSID>" connection.autoconnect yes
```

## 4. Headless (tanpa monitor)

Buat berkas `firstrun`/`wpa_supplicant.conf` di partisi boot saat flashing (untuk OS lama) atau pakai opsi Imager (untuk Trixie). Contoh `wpa_supplicant`:

```conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
country=ID
network={
    ssid="<SSID>"
    psk="<password>"
}
```

## 5. Pertimbangan Deployment

- **Band**: utamakan 2,4 GHz untuk jangkauan; 5 GHz bila broker/streaming butuh bandwidth & sinyal kuat.
- **Jaringan kampus (WPA2-Enterprise/captive portal)**: butuh konfigurasi EAP (identity + password/sertifikat); captive portal bisa menghambat perangkat headless — siapkan MAC registration bila perlu.
- **Fallback**: pertimbangkan Ethernet (Pi 5 punya GbE) untuk demo yang butuh kestabilan.
- **Keamanan**: jangan commit kredensial WiFi ke repo. Simpan sebagai secret/di luar VCS.

## 6. Verifikasi

```bash
nmcli -t -f STATE general      # connected?
ping -c 3 <broker-host>        # jangkau broker
ip addr show wlan0             # IP diperoleh?
```

## 7. Catatan Jetson (rencana)

Jetson (Ubuntu 22.04) juga memakai NetworkManager (`nmcli`) — langkah bagian 3 berlaku. Devkit umumnya butuh modul WiFi M.2 (bila belum terpasang) atau adapter USB WiFi.

## 8. TODO

- [ ] Tentukan jaringan target (rumah/lab/kampus) & metodenya.
- [ ] Uji reconnect otomatis setelah reboot & setelah kehilangan sinyal.
