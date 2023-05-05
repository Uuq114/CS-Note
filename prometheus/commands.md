[toc]

## Prometheus

config

```yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["localhost:9090"]
```

run

```bash
prometheus.exe --config.file=prometheus.yml
```

web page: `localhost:9090`



## Grafana

run

```bash
grafana-server.exe
```

web page: `localhost:3000`

