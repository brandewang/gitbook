# ElasticSearch-Swarm

```bash
#/etc/sysctl.conf
vm.max_map_count = 262144
sysctl -w vm.swappiness = 10


#elasticsearch.yml
node.name: node01
path.data: /opt/elasticsearch/data
path.logs: /opt/elasticsearch/logs
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
node.master: true
node.data: true

#Cluster
cluster.name: es-cluster01
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.unicast.hosts: ["brande-es01", "brande-es02", "brande-es03"]
#only initial master node
cluster.initial_master_nodes: ["node01"]

# Allow CORS
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-credentials: true


#swarm 部署
#manager节点运行以下操作

#创建overlay网络es, 在docker-compose.yml中外部调用，subnet网段需要小于ingress网段，不然会以ingress作为发现地址。
docker network create --driver overlay --subnet 10.0.6.0/24 es

#docker stack deploy es --compose-file=docker-compose.yml

#docker-compose.yml
version: '3'
services:
  elasticsearch1:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es01]
    environment:
      - cluster.name=es
      - node.name=es1
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9200:9200
    networks:
      - es

  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es02]
    environment:
      - cluster.name=es
      - node.name=es-2
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9201:9200
    networks:
      - es

  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.1
    deploy:
      replicas: 1
      placement:
        constraints: [node.hostname == brande-es03]
    environment:
      - cluster.name=es
      - node.name=es-3
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - discovery.zen.minimum_master_nodes=2
      - discovery.zen.fd.ping_timeout=120s
      - discovery.zen.fd.ping_retries=6
      - discovery.zen.fd.ping_interval=30s
      - cluster.initial_master_nodes=es1,es2,es3
      - "discovery.zen.ping.unicast.hosts=es_elasticsearch1,es_elasticsearch2,es_elasticsearch3"
      - "ES_JAVA_OPTS=-Xms2G -Xmx2G"
    volumes:
      - /opt/elasticsearch/data:/usr/share/elasticsearch/data
      - /opt/elasticsearch/logs:/usr/share/elasticsearch/logs
    ports:
      - 9202:9200
    networks:
      - es

networks:
  es:
    external:
      name: es
```