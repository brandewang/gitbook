# Service Discovery

## Prometheus 自动发现
- file_sd_configs
- consul_sd_configs
- k8s_sd_configs

```bash
#file_sd_configs 基于文件的自动发现
#prometheus.yml
  - job_name: 'file_sd'
    file_sd_configs:
    - files: ['/opt/monitor/prometheus/sd_config/*.yml']
      refresh_interval: 5s
#sd_config/node.yml
- targets: ['10.55.5.25:9100']
  labels:
    team: 'team1'
    service: 'mysql'
- targets: ['10.55.3.100:9100']
  labels:
    team: 'team2'
    service: 'mongo'

#consul_sd_configs 基于consul的自动发现
#prometheus.yml
  - job_name: 'node_exporter'
    consul_sd_configs:
    - server: '10.55.5.25:8500'
      services: ['Linux']

#consul注册服务
curl -X PUT -d '{"id":"gihtg-prometheus","name":"Linux","address":"10.55.3.100","port":9100,"tags":["service"],"checks":[{"http": "http://10.55.3.100:9100","interval":"5s"}]}' http://10.55.5.25:8500/v1/agent/service/register
#consul注销服务
curl -X PUT http://10.55.5.25:8500/v1/agent/service/deregister/gihtg-spring-config


#k8s_sd_configs 基于k8s的自动发现
#新增prometheus rbac权限
#prometheus-rbac.yml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
    - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system

#将sa的token写入token.k8s
echo $(kubectl describe secrets $(kubectl get secrets -n kube-system |grep prometheus |cut -f1 -d ' ') -n kube-system |grep -E '^token' |cut -f2 -d':'|tr -d '\t'|tr -d ' ') > token.k8s

#prometheus.yml
  - job_name: 'kubernetes-cadvisor'
    scheme: https
    tls_config:
      insecure_skip_verify: true
    bearer_token_file: /opt/monitor/prometheus/token.k8s
    kubernetes_sd_configs:
    - role: node
      api_server: https://10.55.3.88:8443
      bearer_token_file: /opt/monitor/prometheus/token.k8s
      tls_config:
        insecure_skip_verify: true
```
