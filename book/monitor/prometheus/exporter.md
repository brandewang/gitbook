# Exporter

## Linux监控 node_exporter
- git:https://github.com/prometheus/node_exporter
- grafana dashboard import id: 9276/8919

二进制部署
```bash
mkdir -p /opt/monitor/node_exporter

#node_exporter.service
[Unit]
Description=node_exporter

[Service]
ExecStart=/opt/monitor/node_exporter/node_exporter --collector.systemd --collector.systemd.unit-include=(sshd|docker).service
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## Nginx监控 VTS nginx-vts-exporter
- vts: https://github.com/vozlt/nginx-module-vts
- nginx-vts-exporter: https://github.com/hnlq715/nginx-vts-exporter
- grafana dashboard import id: 9913

二进制部署
```
mkdir -p /opt/monitor/nginx-vts-exporter
cd /opt/monitor/nginx-vts-exporter
wget https://github.com/hnlq715/nginx-vts-exporter/releases/download/v0.10.3/nginx-vts-exporter-0.10.3.linux-amd64.tar.gz
cp nginx-vts-exporter-0.10.3.linux-amd64/nginx-vts-exporter .

#nginx-vts-exporter.service
[Unit]
Description=nginx-vts-exporter
After=network.target

[Service]
Type=simple
ExecStart=/opt/monitor/nginx-vts-exporter/nginx-vts-exporter -nginx.scrape_uri=http://127.0.0.1/status/format/json
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]

WantedBy=multi-user.target
```



## Docker容器监控 cAdvisor
- git:https://github.com/google/cadvisor/
- grafana dashboard import id: 193

二进制部署
```bash
mkdir -p /opt/monitor/cadvisor

#cadvisor.service
[Unit]
Description=cAdvisor

[Service]
ExecStart=/opt/monitor/cadvisor/cadvisor --port=8080
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

docker部署
```bash
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest


在Ret Hat,CentOS, Fedora 等发行版上需要传递如下参数，因为 SELinux 加强了安全策略：
--privileged=true
启动后访问：http://127.0.0.1:8080查看页面，/metric查看指标
```

## MYSQL监控 mysqld_exporte
- git: https://github.com/prometheus/mysqld_exporter
- grafana dashboard import id: 7362

二进制部署
```bash
mkdir -p /opt/monitor/mysqld_exporter

#mysql user create
CREATE USER 'exporter'@'localhost' IDENTIFIED BY '123456' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';

#/opt/monitor/mysqld_exporter/my.cnf
[client]
user=exporter
password=123456

#mysqld_exporter.service
[Unit]
Description=mysqld_exporter

[Service]
ExecStart=/opt/monitor/mysqld_exporter/mysqld_exporter --config.my-cnf=/opt/monitor/mysqld_exporter/my.cnf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process

[Install]
WantedBy=multi-user.target
```
