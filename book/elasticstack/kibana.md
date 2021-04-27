# Kibana

## 二进制安装

```bash
mkdir -p /opt/elk
cd /opt/elk
tar zxvf kibana-7.11.1-linux-x86_64.tar.gz
mv kibana-7.11.1-linux-x86_64 kibana

#config/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://10.55.3.71:9200"]
i18n.locale: "zh-CN"
#单位ms,设置过小可能会导致连接elasticsearch超时
elasticsearch.requestTimeout: 90000

#kibana.service
[Unit]
Description=kibana

[Service]
ExecStart=/opt/elk/kibana/bin/kibana --allow-root
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
