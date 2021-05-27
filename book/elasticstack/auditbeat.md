## auditbeat config
```
#auditbeat.yml
auditbeat.modules:
- module: file_integrity
  paths:
  - /bin
  - /usr/bin
  - /sbin
  - /usr/sbin
  - /etc
- module: system
  datasets:
    - host
    - login
    - package
    - user
  period: 1m
  state.period: 12h
  user.detect_password_changes: true
#- module: system
#  datasets:
#    - process
#    - socket
#  period: 1s
output.redis:
  enabled: true
  hosts: ["172.24.25.68:6379"]
  password: 'Zzredis@123!'
  key: auditbeat
  db: 1
  datatype: list
  timeout: 5
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
```


## logstash pipeline
```
#auditbeat.conf
input {
  redis {
    host => "172.24.25.68"
    port => 6379
    password => "Zzredis@123!"
    key => "auditbeat"
    data_type => "list"
    db =>1
  }
}
output{
  elasticsearch {
    hosts => ["172.24.25.51:9200", "172.24.25.52:9200", "172.24.25.53:9200"]
    user => elastic
    password => 'Zzelastic@123!'
    template_name => 'auditbeat-7.11.2'
    ilm_rollover_alias => 'auditbeat-7.11.2'
    ilm_pattern => "{now/d}-000001"
    ilm_policy => "auditbeat"
  }
}
```
