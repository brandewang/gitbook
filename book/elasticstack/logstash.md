# logstash

## 二进制部署

```bash
yum install java-1.8.0-openjdk -y
cd /opt/elk
tar zxvf logstash-7.11.1.tar.gz
mv logstash-7.11.1.tar.gz logstash

#logstash.service
[Unit]
Description=logstach

[Service]
ExecStart=/opt/elk/logstash/bin/logstash
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target

#logstash.conf
pipeline: # 管道配置
  batch:
    size: 125
    delay: 5
path.config: /opt/elk/logstash/conf.d #logstash -e模式下不允许使用该参数
#定期检查配置是否修改，并重新加载管道。也可以使用SIGHUP信号手动触发
#config.reload.automatic: false
#config.reload.interval: 3s
#http.enabled: true
http.host: 0.0.0.0
http.port: 9600-9700
log.level: info
path.logs: /opt/elk/logstash/logs

#debug
/opt/elk/logstash/bin/logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

## input

```bash
#文件
input{
  file {
    path => "/var/log/test/*.log"
    exclude => "error.log"
    tags => "nginx"
    tags => "web"
    type => "access"
    add_field => {
        "project" => "microservice"
        "app" => "product"
    }
  }
}

#beats
input {
  beats {
	  host => "0.0.0.0"
		port => 5044
	}
}
```

## filter

```bash
#JSON
JSON插件: 接收一个json数据,将其展开为Logstash事件中的数据结构,放到事件顶层。
常用字段:
- source 指定要解析的字段，一般是原始消息message字段
- target 将解析的结果放到指定字段, 如果不指定，默认在事件顶层

模拟数据:'{"remote_addr": "192.168.1.11","url":"/index","status":"200"}'
filter {
  json {
	  source => "message"
	}
}

将Nginx访问日志格式改为JSON:
log_format json  '{"@timestamp":"$time_iso8601", "remote_addr": "$remote_addr", "body_bytes_sent": "$body_bytes_sent",
"request_time": "$request_time", "status": "$status", "request_uri": "$request_uri", "request_method": "$request_method",
"http_referrer": "$http_referer", "http_x_forwarded_for": "$http_x_forwarded_for", "http_user_agent": "$http_user_agent"}';


#KV
KV插件: 接收一个键值数据，按照指定分隔符解析为Logstash事件中的数据结构，放到事件顶层.

模拟数据:'www.github.com?id=1&name=brande&age=32'
filter {
  kv {
    field_split => "&?"
  }
}

#Grok
Grok插件: 如果采集的日志格式是非结构化的，可以些正则表达式提取，grok是正则表达式支持的实现。
常用字段:
- match 正则匹配模式
- patterns_dir 自定义正则模式文件
Logstash内置的正则匹配模式，在安装目录下可以看到，路径：
vendor/bundle/jruby/2.5.0/gems/logstash-patterns-core-4.1.2/patterns/grok-patterns

正则匹配模式语法格式: %{SYNTAX:SEMANTIC}
- SYNTAX 模式名称，模式文件中的第一列
- SEMANTIC 匹配文件的字段名
例如: %{IP:client}

模拟数据: 192.168.1.10 GET /index.html 15824 0.043

示例: 正则匹配HTTP请求日志

filter {
  grok {
	  match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
		}
	}
}

多模式匹配,写多个正则表达式,只要满足其中一条就能匹配成功
filter {
  grok {
	  match => [
		  "message" , "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" %{CID:cid},
		  "message" , "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" %{CID:cid} %{TAG:tag}
		]
	}
}

#GeoIP
GeoIP插件: 根据Maxmind GeoLite2数据库中的数据添加有关IP地址位置信息
常用字段:
- source 指定要解析的IP字段,结果保存到geoip字段
- database GeoLite2数据库文件的路径
- fields 保留极细的指定字段
下载地址: https://www.maxmind.com/en/accounts/436070/geoip/downloads(需要登录)

示例1:
filter{
  grok {
	  patterns_dir => "/opt/patterns"
		match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
		}
	}
	geoip {
	  source => "client"
		database => "/opt/GeoLite2-City.mmdb"
	}
}

示例2: 保留解析的指定字段
filter{
  grok {
	  patterns_dir => "/opt/patterns"
		match => {
		  "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}"
		}
	}
	geoip {
	  source => "client"
		database => "/opt/GeoLite2-City.mmdb"
		target => "geoip"
		fields => ["city_name", "country_code2", "country_name", "region_name"]
	}
}
```

Output
```bash
#elasticsearch
output {
  elastichsearch {
	  hosts => ["192.168.3.71:9200", "192.168.3.72:9200", "192.168.3.73:9200"]
		index => "microservice-product-%{+YYYY.MM.dd}"
	}
}
```

