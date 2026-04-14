# Grafana Folder Notes

Folder ini menyimpan dua jenis file:

- `dashboards/`
  Berisi export dashboard Grafana dalam format JSON.

- `provisioning/`
  Berisi contoh file provisioning agar dashboard dan datasource bisa dimuat otomatis oleh Grafana.

## Dashboard Utama

File `dashboards/dual-isp-internet-monitoring.json` adalah export dari dashboard:

- `DUAL ISP INTERNET MONITORING`

Dashboard ini memantau:

- persentase traffic ISP_A,
- persentase traffic ISP_B,
- status internet ISP_A,
- status internet ISP_B,
- bandwidth download dan upload masing-masing ISP.

## Catatan Tentang Datasource UID

Dashboard JSON saat ini menggunakan UID datasource:

- `prometheus-main`

Karena itu file provisioning datasource dibuat memakai UID yang sama, agar import dashboard bisa langsung berjalan tanpa perlu mapping ulang datasource dari UI Grafana.

Pada versi repository ini, UID dan URL datasource sudah diubah menjadi nilai template agar tidak membocorkan detail produksi.
