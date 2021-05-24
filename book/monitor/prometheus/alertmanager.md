# AlertManager
- git: https://github.com/prometheus/alertmanager

## alertmanager 服务配置
```bash
mkdir /opt/monitor/alertmanager

#alertmanager.service
[Unit]
Description=alertmanager

[Service]
ExecStart=/opt/monitor/alertmanager/alertmanager --config.file=/opt/monitor/alertmanager/alertmanager.yml
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## alertmanager.yml 全局配置参考
```bash
global:
  resolve_timeout: 5m #当alertmanager持续多长时间未接受到告警后，标记告警状态为resolved (已解决)
  #邮箱服务器
  smtp_smarthost: "smtphz.qiye.163.com:465"
  smtp_from: "ops@gihtg.com"
  smtp_auth_username: "ops@gihtg.com"
  smtp_auth_password: ""
  smtp_require_tls: false

# 告警模版
templates:
- "template/*.tmpl"


route:
  group_by: ['instance'] #对告警进行分组，将标签相同对告警合并为一个通知
  group_wait: 10s #触发告警后,等待10s,再发送通知,设置这个时间是为了尽可能多的将告警合并为一个通知。
  group_interval: 10s #相同Group之间发送告警通知的时间间隔
  repeat_interval: 10m #之前已经发送的告警,如果还没恢复,即一直处于FIRING状态,等10m后再次发送。
  receiver: 'admin'
  routes:
  - receiver: 'database-admin'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  - receiver: 'frontend-admin'
    group_by: [product, environment]
    match:
      team: frontend
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
- name: 'admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
    send_resolved: true
- name: 'database-admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
- name: 'frontend-admin'
  email_configs:
  - to: 'brande.wang@gihtg.com'
inhibit_rules:
  - source_match:
      severity: 'critical' #指定告警级别
    target_match:
      severity: 'warning'  #指定意志告警级别
    equal: ['alertname', 'instance'] #设置匹配相同的标签
```
