# Seznam použitých exportérů:
Zde je seznam exportérů plus docker-compose konfigurace (pokud běží v Dockeru)

- [Prometheus](https://prometheus.io) - storage pro metriky
- [Grafana](https://grafana.com) - grafické zobrazení metrik
- [cAdvisor](https://github.com/google/cadvisor) - sbírá metriky z dockeru
- [Node Exporter](https://github.com/prometheus/node_exporter) - sbírá metriky z serverů atd.
- [MKTXP](https://github.com/akpw/mktxp/tree/main) - sbírá metriky  z Mirotiků
- [OpenWrt exporter](https://grafana.com/blog/2021/02/09/how-i-monitor-my-openwrt-router-with-grafana-cloud-and-prometheus/) - sbírá metriky z OpenWrt
- [snmp_exporter](https://github.com/prometheus/snmp_exporter) - sbírá metrky ze Synology


## Node Exporter
Exporter pro exportování metrik z VPS serveru

``` yml
version: '3.3'

services:
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

# snmp exporter
Tento exporter používám pro export metrik ze Synology.
Konfigurace je možná pomocí souboru `snmp.yml`, který je přiložen v této složce.

``` yml
version: '3.3'

services:
  snmp-exporter:
    image: prom/snmp-exporter:latest
    command:
      - '--config.file=/etc/snmp_exporter/snmp.yml'
    container_name: Prometheus-SNMP
    hostname: prometheus-snmp
    mem_limit: 256m
    mem_reservation: 64m
    cpu_shares: 512
    security_opt:
      - no-new-privileges:true
    read_only: true
    user: 1026:100
    healthcheck:
      test: wget --no-verbose --tries=1 --spider http://localhost:9116/ || exit 1
    volumes:
      - /volume1/docker/snmp:/etc/snmp_exporter/:ro
    restart: on-failure:5
    ports:
      - 9116:9116
```

# mktxp
Tento exporter používám pro exportování metrik z Mikrotiků.
Konfiguruje se pomocí souboru `mktpx.conf` který je ve složce mktpx.

``` yml
version: '3.3'

services:
  mktxp:
    image: ghcr.io/akpw/mktxp:latest
    container_name: mikrotik-exporter
    volumes:
      - /volume1/docker/mktxp/mktxp:/home/mktxp/mktxp/
    restart: on-failure:5
    ports:
      - 49090:49090
```

# OpenWrt Exporter

Pro export metrik z OpenWrt používám tento [Projekt](https://grafana.com/blog/2021/02/09/how-i-monitor-my-openwrt-router-with-grafana-cloud-and-prometheus/)