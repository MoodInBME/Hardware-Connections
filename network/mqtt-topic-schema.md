# Skema Topik MQTT — Referensi Transport (MoodIn)

> **PENTING:** berkas ini **tidak** mendefinisikan ulang skema pesan. Sumber kebenaran tunggal untuk nama topik & payload adalah kontrak kanonik di **`MoodIn-master/docs/ipc-contract.md`**. Di sini hanya ditambahkan detail **transport** khas hardware (broker, TLS, QoS, retensi, LWT, identitas perangkat).

## 1. Kenapa Tidak Didefinisikan di Sini

Menyimpan definisi topik di banyak repo menimbulkan dua sumber kebenaran yang bisa saling menyimpang saat integrasi. Repo teknis (Hardware/Software/Interface) **mengimpor** kontrak dari `MoodIn-master`, tidak menyalinnya.

> Catatan migrasi [High]: draf lama berkas ini memakai pola `moodin/{device_id}/mood/classification` dengan field `label/v/a`. Pola itu **usang** dan diganti oleh kontrak kanonik v1.0.0 (topik `moodin/inference/result`, dst.). Jangan pakai pola lama.

## 2. Ringkasan Topik Kanonik (rujukan, bukan definisi)

| Topik | Publisher | Subscriber | QoS | Retained |
| --- | --- | --- | --- | --- |
| `moodin/presence` | Hardware / Software | Interface, Software | 1 | Ya |
| `moodin/signal/quality` | Software | Interface | 0 | Tidak |
| `moodin/inference/result` | Software | Interface | 1 | Tidak |
| `moodin/ui/command` | Software / sistem | Interface | 1 | Tidak |

Field payload, tipe, dan pemetaan `mood_id` -> label ada di kontrak kanonik. Semua payload JSON UTF-8 dengan field wajib `schema_version`, `session_id`, `ts`.

## 3. Detail Transport (khusus repo ini)

| Aspek | Dev | Produksi |
| --- | --- | --- |
| Host broker | broker lokal/LAN | broker MoodIn |
| Port | 1883 | **8883 (TLS)** |
| TLS | tidak | ya (lihat `mqtt-tls-config.md`) |
| Auth | opsional | username/password + CA |

- **`client_id`**: `moodin-<repo>-<peran>-<device>` (mis. `moodin-hw-edge-mirror01`). Identitas perangkat masuk ke `client_id`, **bukan** ke nama topik (topik tetap kanonik).
- **Last Will & Testament (LWT)**: daftarkan LWT pada `moodin/presence` (`{present:false, source:"<client>"}`) agar terputusnya perangkat terdeteksi.
- **Keepalive**: 30–60 dtk. **Clean session**: false untuk subscriber yang butuh kontinuitas.

## 4. Uji Cepat

```bash
# dev (tanpa TLS)
mosquitto_sub -h localhost -p 1883 -t 'moodin/#' -v

# produksi (TLS) — lihat mqtt-tls-config.md untuk sertifikat
mosquitto_sub -h <broker> -p 8883 --cafile ca.crt -t 'moodin/#' -v
```

## 5. TODO

- [ ] Tetapkan skema `client_id` per perangkat.
- [ ] Sinkronkan versi kontrak yang dipakai firmware dengan `ipc-contract.md` (`schema_version`).
