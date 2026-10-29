# Konfigurasi MQTT over TLS — MoodIn

> **ARSIP (tahap Pra-TA):** dokumen ini berlaku untuk rancangan era GEMASTIK, yaitu MQTT dengan TLS di port 8883. Tahap Pra-TA memakai MQTT di jaringan lokal **tanpa TLS**, dengan _username/password_ broker dan enkripsi isi pesan (ADR-10 di [MoodIn-master/docs/adr.md](https://github.com/MoodInBME/MoodIn-master/blob/main/docs/adr.md)). Konfigurasi broker yang baru akan ditulis di dokumen baru.

> Transport aman untuk pesan MoodIn di produksi (port **8883**). Skema topik/payload mengacu kontrak kanonik (`MoodIn-master/docs/ipc-contract.md`); di sini fokus keamanan transport.

## 1. Kenapa TLS

Meski payload MoodIn hanya berisi fitur/hasil teragregasi (bukan citra wajah mentah — lihat prinsip privasi & UU PDP No. 27/2022), metadata mood tetap sensitif. TLS mencegah penyadapan & spoofing broker di jaringan.

## 2. Broker: Mosquitto (contoh)

`/etc/mosquitto/conf.d/moodin.conf`:

```conf
listener 8883
protocol mqtt

cafile   /etc/mosquitto/certs/ca.crt
certfile /etc/mosquitto/certs/server.crt
keyfile  /etc/mosquitto/certs/server.key

require_certificate false      # true bila memakai mTLS (sertifikat klien)
tls_version tlsv1.2

allow_anonymous false
password_file /etc/mosquitto/passwd
```

Buat kredensial:

```bash
sudo mosquitto_passwd -c /etc/mosquitto/passwd moodin_edge
```

## 3. Sertifikat

Untuk internal/demo cukup **self-signed CA**; untuk publik pakai CA tepercaya (mis. Let’s Encrypt).

```bash
# CA
openssl req -new -x509 -days 3650 -extensions v3_ca -keyout ca.key -out ca.crt
# server key + CSR + tandatangani dengan CA
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr   # CN = hostname broker
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 825
```

> CN/SAN server **harus** cocok dengan hostname yang dipakai klien, atau verifikasi TLS gagal.

## 4. Klien (Python, paho-mqtt 2.x)

```python
import paho.mqtt.client as mqtt

c = mqtt.Client(client_id="moodin-hw-edge-mirror01", protocol=mqtt.MQTTv5)
c.username_pw_set("moodin_edge", "<password>")
c.tls_set(ca_certs="ca.crt")            # + certfile/keyfile bila mTLS
# LWT di topik presence (lihat kontrak kanonik)
c.will_set("moodin/presence",
           '{"schema_version":"1.0.0","session_id":"-","present":false,"source":"mirror01","ts":0}',
           qos=1, retain=True)
c.connect("<broker-host>", 8883, keepalive=45)
```

## 5. Uji

```bash
mosquitto_sub -h <broker> -p 8883 --cafile ca.crt -u moodin_edge -P '<pw>' -t 'moodin/#' -v
```

## 6. Keamanan Operasional

- Simpan kunci privat & password **di luar** repo (jangan commit). Pakai variabel lingkungan / secret store.
- Rotasi kredensial berkala; cabut sertifikat yang bocor.
- Batasi ACL broker: perangkat edge hanya boleh publish/subscribe topik `moodin/#` yang relevan.

## 7. TODO

- [ ] Tentukan CA (self-signed vs publik).
- [ ] Buat akun broker per perangkat + ACL.
- [ ] Uji handshake TLS end-to-end dari edge ke broker.
