# Popis řešení
Pro monitoring jsem se rozhodl použít Prometheus a Grafanu.
Shromažduji metriky ze všech zařízení které to umožnují.
![Alt text](https://github.com/jakubsindelar/monitoring/blob/main/img/title.jpeg?raw=true "title image")

## Co monitoruji?
- Turris router 
- Mikrotiky
- Synology NAS
- VPS server
- Docker

## Co k tomu používám?
- Prometheus - storage pro metriky
- Grafana - grafické zobrazení metrik
- cAdvisor - sbírá metriky z dockeru
- Node Exporter - sbírá metriky z serverů atd.

# Použité GitHub projekty:
- [snmp_exporter](https://github.com/prometheus/snmp_exporter) - export metrik z Synology
- [mktxp](https://github.com/akpw/mktxp/tree/main) - export metrik z Mikrotiků

# Konfigurace
## Monitoring docker-compose

```yaml
version: '2'

volumes:
    prometheus_data: {}
    grafana_data: {}

services:
  grafana:
    image: grafana/grafana-enterprise
    container_name: monitoring-grafana
    restart: unless-stopped
    user: root
    ports:
      - "3580:3000"
    volumes:
      - grafana_data:/var/lib/grafana
  prometheus:
    image: prom/prometheus
    container_name: monitoring-prometheus
    user: root
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=100y'
      - '--storage.tsdb.path=/prometheus'
    ports:
      - 9090:9090
    restart: unless-stopped
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    depends_on:
    - cadvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: monitoring-cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
  node-exporter:
    image: prom/node-exporter:latest
    container_name: monitoring-node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    expose:
      - 9100
    depends_on:
      - cadvisor
```

## Prometheus config 

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
- job_name: server-monitoring 
  scrape_interval: 15s
  honor_labels: true
  static_configs:
  - targets:
    - cadvisor:8080
    - node-exporter:9100
- job_name: home-network-monitoring 
  scrape_interval: 15s
  honor_labels: true
  static_configs:
  - targets:
    - 192.168.0.1:9100
- job_name: home-nas-monitoring 
  scrape_interval: 15s
  honor_labels: true
  static_configs:
  - targets:
    - 192.168.0.5
  metrics_path: /snmp
  params:
    module: [synology]
    auth: [public_v3]
  relabel_configs:
  - source_labels: [__address__]
    target_label: __param_target
  - source_labels: [__param_target]
    target_label: instance
  - source_labels: [__param_target]
    regex: (.*)
    replacement: ${1}:9116
    target_label: __address__
- job_name: 'Mikrotiks'
  static_configs:
  - targets:
    - 192.168.0.5:49090
```

# Co plánuji přidat
- monitoring VPN sítě wireguard
- monitoring SmartHome zařízení

# Závěr
Je to začátek, takže než přidám wireguard atd. tak to nejdív pořádně doladím a pak budu pokračovat. Ještě mi nefungují všechny grafy a trochu si ještě pohraju s nastavením Promethea.
