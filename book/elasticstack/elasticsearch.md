# ElasticSearch

- url: elastic.co

  

## 启动先决条件

- 调整进程最大打开文件数数量

```bash
#临时设置
ulimit -n 65535
#永久设置
vi /etc/security/limits.conf
* hard nofile 65535
* soft nofile 65535
```
- 调整进程最大虚拟内存区域数量
```bash
#临时设置
sysctl -w vm.max_map_count=262144
#永久设置
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p
```

## elasticsearch二进制部署

```bash
mkdir -p /opt/elk
cd /opt/elk
tar zxvf elasticsearch-7.11.2-linux-x86_64.tar.gz
mv elasticsearch-7.11.2 elasticsearch
useradd es
chown -R es:es elasticsearch

#elasticsearch.yml
cluster.name: elk-cluster    # 集群名称
node.name: node-1           # 集群节点名称
#path.data: /path/to/data   # 数据目录
#path.logs: /path/to/logs   # 日志目录
network.host: 0.0.0.0       # 监听地址
http.port: 9200             # 监听端口
#transport.tcp.port: 9300   # 内部节点之间通信端口
discovery.seed_hosts: ["10.55.3.71", "10.55.3.72", "10.55.3.73"] # 集群节点列表
cluster.initial_master_nodes: ['node-1']  # 首次启动指定的master节点

#elasticsearch.service
[Unit]
Description=elasticsearch
[Service]
User=es
LimitNOFILE=65535
ExecStart=/opt/elk/elasticsearch/bin/elasticsearch
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

## helm
```
helm repo add elastic https://helm.elastic.co
mkdir /opt/helm && cd /opt/helm
helm pull elastic/elasticsearch

#vim values.yaml
imageTag: "7.11.2"

service:
  labels: {}
  labelsHeadless: {}
  #type: ClusterIP
  type: NodePort
  nodePort: 30009

volumeClaimTemplate:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: 'dynamic-cephfs'
  resources:
    requests:
      storage: 5Gi

extraInitContainers:
  - name: do-something
    image: busybox
    command: ['chown', '-R', '1000:1000', '/usr/share/elasticsearch/data']
    #command: ['id']
    securityContext:
      runAsUser: 0

#vim statefulset.yaml 将数据卷以同样的形式挂载到initContainer,以给文件夹赋权
      {{- if .Values.extraInitContainers }}
      # Currently some extra blocks accept strings
      # to continue with backwards compatibility this is being kept
      # whilst also allowing for yaml to be specified too.
      {{- if eq "string" (printf "%T" .Values.extraInitContainers) }}
{{ tpl .Values.extraInitContainers . | indent 6 }}
      {{- else }}
{{ toYaml .Values.extraInitContainers | indent 6 }}
      {{- end }}
      {{- end }}
        volumeMounts:
          {{- if .Values.persistence.enabled }}
          - name: "{{ template "elasticsearch.uname" . }}"
            mountPath: /usr/share/elasticsearch/data
          {{- end }}
      {{- end }}
```


## Command
```bash
#查看集群节点: curl -XGET 'http://127.0.0.1:9200/_cat/nodes?pretty'
#查询集群状态: curl -XGET 'http://127.0.0.1:9200/_cluster/health?pretty'
```

## UI

- elasticsearch-head

```bash
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install --registry=https://registry.npm.taobao.org
npm run start
curl http://localhost:9100/
```

- cerebro

```bash
#github
https://github.com/lmenezes/cerebro

#docker
docker run -idt --name cerebro -p 9000:9000 lmenezes/cerebro
```

- ElasticHD

```bash
docker run -p 9800:9800 -d --name elastichd  containerize/elastichd
```

