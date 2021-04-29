# filebeat

## 二进制安装

```bash
mkdir -p /opt/elk/filebeat/


#filebeat.service
[Unit]
Description=FileBeat

[Service]
ExecStart=/opt/elk/filebeat/filebeat -c /opt/elk/filebeat/filebeat.yml
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target


```



## docker

```bash
#docker
docker run -idt \
  --name=filebeat \
  --user=root \
  --volume="/opt/logs:/opt/logs:ro" \
  --volume="/opt/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro" \
  docker.elastic.co/beats/filebeat:7.9.1
```



## 配置文件

```bash
#filebeat.yml
filebeat.inputs:
- type: log  #日志类型
  enabled: true  #是否启用
  paths:
    - /opt/logs/nginx/access.log #日志路径可同配
  fields_under_root: true  #是否将自定义fields直接加入到顶层
  fields:  #配置环境，项目，服务名，用于规划索引名称
    env: prd
    project: ops
    service: nginx01
  tags: ["proxy", "nginx", "access_log"]  #加入日志类型，用于匹配同日志类型字段

- type: log     #多类型输入配置
  enabled: true
  paths:
    - /opt/logs/helloworld01/*.log
  fileds_under_root: true
  fields:
    env: prd
    project: ms
    service: helloworld01
  tags: ["java", "springboot", "app_log"]
#java堆栈日志多行匹配
	multiline.pattern: '^\s'
	multiline.negate: false
	multiline.match: after

#常见的Java堆栈日志:
#Exception in thread "main" java.lang.NullPointerException
#  at com.example.myproject.Book.getTitle(Book.java:16)
#	 at com.example.myproject.Author.BookTitles(Author.java:25)
#	 at com.example.myproject.Bootstrap.main(Bootstrap.java:14)


- type: log
  enabled: true
  paths:
    - /opt/logs/helloworld02/*.log
  fields_under_root: true
  fields:
    env: prd
    project: ms
    service: helloworld02
  tags: ["java", "springboot", "app_log"]

#推送到Logstash
output.logstash:
  hosts: ["10.55.3.70:5044"]

#推送到redis
output.redis:
  enabled: true
  hosts: ["172.16.112.143:6379"]
  key: filebeat
  db: 1
  datatype: list


#推送到ES
setup.ilm.enabled: false
setup.template.name: "microservice-product"
setup.template.pattern: "microservice-product-*"
output.elasticsearch:
  hosts: ["10.55.3.71:9200"]
  index: "microservice-product-%{+yyy.MM.dd}"
```

