# Dual ISP Internet Monitoring Template

Repository ini merapikan komponen monitoring internet Organization X yang berbasis:

- Huawei Firewall USG6555F via SNMP
- Prometheus untuk scraping metric dan rule evaluation
- Blackbox Exporter untuk pengecekan konektivitas internet per ISP
- Grafana untuk dashboard visualisasi

Repository ini juga sudah disiapkan agar bisa dijalankan dengan Docker Compose.

Fokus repository ini adalah monitoring dua link internet:

- ISP_A pada interface `INTERFACE_A`
- ISP_B pada interface `INTERFACE_B`

Kapasitas kedua link saat ini adalah `200 Mbps`.

## Catatan Sanitasi

Repository ini sudah disanitasi untuk kebutuhan dokumentasi dan GitHub:

- IP internal produksi diganti dengan hostname atau alamat contoh.
- community SNMP diganti placeholder.
- URL datasource Grafana diganti placeholder.
- file ini dimaksudkan sebagai template yang mudah dipahami, bukan salinan mentah dari server produksi.

## Menjalankan Dengan Docker

Stack ini bisa dijalankan dengan Docker Compose agar user lain lebih mudah mencoba atau mengadaptasinya.

Komponen yang akan dijalankan:

- Prometheus
- Grafana
- Alertmanager
- SNMP Exporter
- Blackbox Exporter
- Node Exporter

File yang dipakai:

- `docker-compose.yml`
- `.env.example`

Langkah cepat:

1. Copy `.env.example` menjadi `.env`
2. Ubah username dan password Grafana jika diperlukan
3. Review placeholder pada file:
   - `prometheus/prometheus.yml`
   - `prometheus/alert.rules.yml`
   - `prometheus/blackbox.yml`
   - `prometheus/snmp.yml`
   - `grafana/dashboards/dual-isp-internet-monitoring.json`
4. Jalankan:

```bash
docker compose up -d
```

Setelah stack berjalan:

- Grafana: `http://localhost:3000`
- Prometheus: `http://localhost:9090`
- Alertmanager: `http://localhost:9093`

Catatan penting:

- template ini akan langsung menjalankan servicenya, tetapi metric nyata baru akan masuk setelah placeholder target, interface, dan SNMP community Anda diganti.
- `blackbox-exporter` membutuhkan capability `NET_RAW` agar ICMP probe bisa berjalan.

## Alur Monitoring

1. Prometheus mengambil metric interface firewall Huawei melalui SNMP Exporter.
2. Blackbox Exporter melakukan probe ICMP untuk memverifikasi internet tiap ISP.
3. Prometheus menghitung traffic, status link, dan alert dari metric tersebut.
4. Grafana menampilkan hasilnya pada dashboard `DUAL ISP INTERNET MONITORING`.

## Cara Kerja Tiap Komponen

### 1. Firewall Huawei dan SNMP

Firewall menjadi sumber metric utama untuk status interface dan traffic bandwidth.

Metric yang dipakai dashboard terutama berasal dari:

- `ifOperStatus` untuk mengetahui apakah interface sedang UP atau DOWN
- `ifHCInOctets` untuk trafik masuk
- `ifHCOutOctets` untuk trafik keluar

Interface yang dimonitor di template ini:

- `INTERFACE_A` untuk `ISP_A`
- `INTERFACE_B` untuk `ISP_B`

### 2. SNMP Exporter

Prometheus tidak mengambil SNMP langsung dalam bentuk raw query per panel. Sebagai gantinya:

1. Prometheus memanggil `snmp-exporter`
2. `snmp-exporter` melakukan query SNMP ke perangkat target
3. hasil SNMP diubah menjadi metric Prometheus

File yang terkait:

- `prometheus/prometheus.yml`
  Menentukan job `snmp` dan `snmp_firewall`
- `prometheus/snmp.yml`
  Menentukan module SNMP seperti `fw_if_mib`

### 3. Blackbox Exporter

Blackbox Exporter dipakai untuk menjawab pertanyaan:

- link fisik UP, tetapi internetnya benar-benar bisa keluar atau tidak?

Caranya:

1. Prometheus memanggil endpoint `/probe`
2. Blackbox menjalankan ICMP probe ke target internet
3. hasilnya muncul sebagai metric seperti `probe_success`

Template ini memisahkan probe untuk dua ISP:

- `blackbox_isp_a`
- `blackbox_isp_b`

Tujuannya agar masing-masing jalur internet bisa diuji secara terpisah.

File yang terkait:

- `prometheus/prometheus.yml`
- `prometheus/blackbox.yml`

### 4. Prometheus

Prometheus adalah pusat pengumpulan dan evaluasi metric.

Peran Prometheus di project ini:

- scrape metric dari `snmp-exporter`
- scrape hasil probe dari `blackbox-exporter`
- menjalankan alert rules
- menyediakan datasource untuk Grafana

Query yang dipakai dashboard umumnya menghitung:

- persentase bandwidth:
  `rate(ifHCInOctets / ifHCOutOctets) -> bit per second -> dibandingkan kapasitas 200 Mbps`
- status internet:
  `probe_success` digabung dengan `ifOperStatus`

File yang terkait:

- `prometheus/prometheus.yml`
- `prometheus/alert.rules.yml`

### 5. Grafana

Grafana membaca data dari Prometheus lalu menampilkannya menjadi dashboard visual.

Dashboard utama menampilkan:

- persentase traffic `ISP_A`
- persentase traffic `ISP_B`
- status internet `ISP_A`
- status internet `ISP_B`
- bandwidth download dan upload masing-masing ISP

File yang terkait:

- `grafana/dashboards/dual-isp-internet-monitoring.json`
- `grafana/provisioning/datasources/prometheus.yaml`
- `grafana/provisioning/dashboards/dashboards.yaml`

## Urutan Setup yang Disarankan

Supaya pembaca tidak bingung, urutan setup stack ini idealnya seperti berikut:

1. Siapkan akses SNMP pada firewall Huawei
   Pastikan community SNMP tersedia dan interface yang ingin dimonitor memang expose metric yang dibutuhkan.

2. Siapkan `snmp-exporter`
   Gunakan `prometheus/snmp.yml` dan sesuaikan community serta module yang dipakai.

3. Siapkan `blackbox-exporter`
   Gunakan `prometheus/blackbox.yml` dan sesuaikan source IP atau jenis probe yang dibutuhkan.

4. Siapkan Prometheus
   Ubah target pada `prometheus/prometheus.yml`, pastikan job `snmp_firewall`, `blackbox_isp_a`, dan `blackbox_isp_b` mengarah ke endpoint yang benar.

5. Siapkan alert rules
   Review `prometheus/alert.rules.yml`, terutama nama interface, kapasitas bandwidth, dan kondisi alert.

6. Siapkan Grafana datasource
   Import atau provision datasource Prometheus dengan file `grafana/provisioning/datasources/prometheus.yaml`.

7. Siapkan dashboard Grafana
   Import JSON dashboard dari `grafana/dashboards/dual-isp-internet-monitoring.json` atau gunakan provisioning dashboard.

8. Verifikasi end-to-end
   Pastikan metric SNMP masuk, probe blackbox sukses, alert rules berjalan, dan semua panel di Grafana menampilkan data.

Jika menggunakan Docker Compose, langkah 2 sampai 7 tetap berlaku, hanya deployment servicenya disederhanakan lewat satu file compose.

## Ringkasan Data Flow

```text
Huawei Firewall
  -> SNMP
SNMP Exporter
  -> metric Prometheus
Prometheus
  -> query / rule / alert
Blackbox Exporter
  -> probe_success metric
Prometheus
  -> datasource
Grafana
  -> dashboard monitoring dual ISP
```

## Struktur Folder

```text
dual-isp-monitoring-template/
├── README.md
├── .gitignore
├── grafana/
│   ├── dashboards/
│   │   └── dual-isp-internet-monitoring.json
│   └── provisioning/
│       ├── dashboards/
│       │   └── dashboards.yaml
│       └── datasources/
│           └── prometheus.yaml
└── prometheus/
    ├── README.md
    ├── prometheus.yml
    ├── alert.rules.yml
    ├── blackbox.yml
    └── snmp.yml
```

## Isi Tiap Folder

- `grafana/dashboards/`
  Berisi export dashboard Grafana agar dashboard tidak hanya tersimpan di database internal Grafana.

- `grafana/provisioning/`
  Berisi contoh provisioning datasource dan dashboard agar Grafana bisa memuat dashboard secara otomatis.

- `prometheus/prometheus.yml`
  File utama Prometheus untuk mendefinisikan job scrape, blackbox probe, rule file, dan alertmanager.

- `prometheus/alert.rules.yml`
  Rule alert untuk kondisi link down, internet down, dan traffic rendah.

- `prometheus/blackbox.yml`
  Konfigurasi module Blackbox Exporter yang dipakai untuk menguji konektivitas masing-masing ISP.

- `prometheus/snmp.yml`
  Konfigurasi SNMP Exporter. File ini dihasilkan oleh generator dan tidak ideal untuk diubah manual.

## Catatan Penting Sebelum Push ke GitHub

- Jangan commit `grafana.db`.
- Jangan commit secret, credential, atau community SNMP jika nanti ditambahkan.
- Jika repository akan dibuat public, pertimbangkan menyamarkan IP internal dan nama host operasional.
- Jika ingin menjadikan repo ini sebagai source of truth, biasakan mengubah dashboard dari file JSON/provisioning, bukan hanya dari UI Grafana.

## Langkah Lanjut yang Disarankan

1. Review kembali IP target dan path provisioning agar sesuai dengan server tujuan.
2. Putuskan apakah repo ini akan bersifat `private` atau `public`.
3. Jika sudah final, gunakan file di repo ini sebagai basis deployment dan backup konfigurasi monitoring.
