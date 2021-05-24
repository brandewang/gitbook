# Prometheus

- 官网: prometheus.io
- git: https://github.com/prometheus/prometheus
- alert: https://awesome-prometheus-alerts.grep.to/rules

## prometheus服务配置
```bash
mkdir -p /opt/monitor/prometheus

#prometheus.service
[Unit]
Description=prometheus

[Service]
ExecStart=/opt/monitor/prometheus/prometheus --config.file=/opt/monitor/prometheus/prometheus.yml --storage.tsdb.path=/opt/monitor/prometheus/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
Type=simple

[Install]
WantedBy=multi-user.target

#prometheus.yml 全局配置
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

#Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 127.0.0.1:9093

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
```
