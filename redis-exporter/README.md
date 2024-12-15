How to setup a Redis Exporter for Prometheus

**Phướng án 1:** Sử dụng docker

```
mkdir -p /root/redis-exporter
docker-compose.yml
version: "3.9"
services:

  redis-exporter:
    image: oliver006/redis_exporter
    ports:
      - 9121:9121
    restart: unless-stopped
    environment:
      REDIS_ADDR: "xx.xx.xx.xx:6379"
      REDIS_USER: null
      REDIS_PASSWORD: 5Kx98DsA#2024
```

Dashboard: https://grafana.com/grafana/dashboards/763

Ref: 
https://dev.to/nelsonmendezz_/how-to-monitor-redis-with-prometheus-and-grafana-docker-3lfb
https://ruan.dev/blog/2020/05/05/how-to-setup-a-redis-exporter-for-prometheus
