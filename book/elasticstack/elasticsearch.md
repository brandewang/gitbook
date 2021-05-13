# ElasticSearch

- url: elastic.co

c : code node
d : data node
f : frozen node
h : hot node
i : ingest node
l : machine learning node
m : master eligible node
r : remote cluster client node
s : content node
t : transform node
v : voting-only node
w : warm node
-  : coordinating node only
  

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

./bin/elasticsearch-certutil ca
./bin/elasticsearch-certutil cert -ca elastic-stack-ca.p12
mv elastic-stack-ca.p12 config/
mv elastic-certificates.p12 config/

useradd -r es
chown -R es:es elasticsearch

#elasticsearch.yml
#cluster
cluster.name: shidc01-es-cluster01
node.name: shidc01-es-master01
path.data: /opt/elk/elasticsearch/data
path.logs: /opt/elk/elasticsearch/logs
network.host: 10.55.3.71
http.port: 9200
transport.tcp.port: 9300
discovery.seed_hosts: ["10.55.3.71", "10.55.3.72", "10.55.3.73"] 
discovery.zen.minimum_master_nodes: 2
cluster.initial_master_nodes: ['shidc01-es-master01']
#跨站访问
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization
#X-Pack
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate 
xpack.security.transport.ssl.keystore.path: /opt/elk/elasticsearch/config/elastic-certificates.p12 
xpack.security.transport.ssl.truststore.path: /opt/elk/elasticsearch/config/elastic-certificates.p12 

#path
cat > /etc/profile.d/elk.sh <<EOF
export PATH=/opt/elk/elasticsearch/bin:${PATH}
EOF

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

#初始x-pack用户
./bin/elasticsearch-setup-passwords interactive
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

## API
```
#修改ilm轮询间隔
PUT _cluster/settings
{
  "persistent" : { 
    "indices.lifecycle.poll_interval": "10s"
  },
  "transient" : { }
}
```
