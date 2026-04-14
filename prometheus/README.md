# Prometheus Folder Notes

Folder ini berisi konfigurasi inti untuk monitoring internet dua ISP.

## File Ringkas

- `prometheus.yml`
  Menentukan job scrape Prometheus, Node Exporter, SNMP Exporter, dan Blackbox Exporter.

- `alert.rules.yml`
  Menentukan kondisi yang dianggap masalah operasional.

- `blackbox.yml`
  Mendefinisikan module probe ICMP untuk ISP_A dan ISP_B.

- `snmp.yml`
  Berisi definisi module SNMP Exporter. File ini auto-generated.

## Catatan Tentang `snmp.yml`

`snmp.yml` sengaja disimpan karena diperlukan oleh SNMP Exporter, tetapi file ini bukan file yang nyaman untuk diedit manual.

Jika di masa depan perlu penyesuaian besar pada module SNMP:

1. edit source generator SNMP Exporter,
2. generate ulang `snmp.yml`,
3. commit hasil akhirnya ke repository.
