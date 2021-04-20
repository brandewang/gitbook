# CNI

- CNI (Container Network Interface, 容器网络接口): 是一个容器网络规范，Kubernetes网络采用的就是这个CNI规范，负责初始化infra容器的网络设备。


- CNI 二进制程序默认路径： /opt/cni/bin/
  项目地址: https://github.com/containernetworking/plugins/releases

- CNI 配置文件默认路径：/etc/cni/net.d


## flannel
- 项目地址：https://github.com/coreos/flannel
- k8s-v1.17+配置清单: https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
- 网络模式: hostgateway、vxlan

#更改相关配置
```
  net-conf.json: |
    {
      "Network": "172.8.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```
#应用配置文件 (flannel不会自动生成cni二进制，需要提前准备)

```
kubectl apply -f kube-flannel.yml
```

## calico
- 项目地址：https://github.com/projectcalico/calico


calico包括如下重要组件：calico/node，Typha，Felix，etcd，BGP Client，BGP Route Reflector。

- calico/node：把Felix，calico client， confd，Bird封装成统一的组件作为统一入口，同时负责给其他的组件做环境的初始化和条件准备。

- Felix：主要负责路由配置以及ACLS规则的配置以及下发，它存在在每个node节点上。

- etcd：存储各个节点分配的子网信息，可以与kubernetes共用；

- BGPClient(BIRD), 主要负责把 Felix写入 kernel的路由信息分发到当前 Calico网络，确保 workload间的通信的有效性；

- BGPRoute Reflector(BIRD), 大规模部署时使用，在各个节点之间不是mesh模式，通过一个或者多个 BGPRoute Reflector 来完成集中式的路由分发；当etcd中有新的规则加入时，Route Reflector 就会将新的记录同步。
BIRD Route Reflector负责将所有的Route Reflector构建成一个完成的网络，当增减Route Reflector实例时，所有的Route Reflector监听到新的Route Reflector并与之同步交换对等的路由信息.

- Typha：在节点数比较多的情况下，Felix可通过Typha直接和Etcd进行数据交互，不通过kube-apiserver，既降低其压力。生产环境中实例数建议在3~20之间，随着节点数的增加，按照每个Typha对应200节点计算。
如果由kube-apiserver转为Typha。需要将yaml中typha_service_name 修改calico-typha，同时replicas不能为0 ，否则找不到Typha实例会报连接失败。

### 部署

https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

#### Install Calico with Kubernetes API datastore, 50 nodes or less

1.Download the Calico networking manifest for the Kubernetes API datastore.
curl https://docs.projectcalico.org/manifests/calico.yaml -O
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.Customize the manifest if desired.
4.kubectl apply -f calico.yaml

#### Install Calico with Kubernetes API datastore, more than 50 nodes

1.Download the Calico networking manifest for the Kubernetes API datastore.
curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.Modify the replica count to the desired number in the Deployment named, calico-typha.

``` bash
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: calico-typha
  ...
spec:
  ...
  replicas: <number of replicas>
```
4.Customize the manifest if desired.
5.kubectl apply -f calico.yaml

#### Install Calico with etcd datastore

1.Download the Calico networking manifest for etcd.
curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml
2.change CALICO_IPV4POOL_CIDR in calico.yaml
3.In the ConfigMap named, calico-config, set the value of etcd_endpoints to the IP address and port of your etcd server.
4.Customize the manifest if desired.
5.kubectl apply -f calico.yaml

PS: 修改calico.yaml中 defaultMode: 0040

```bash
#修改calicoctl直连数据库
cat > /etc/calico/calicoctl.cfg << EOF
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "https://10.55.3.41,https://10.55.3.42:2379,https://10.55.3.43:2379"
  etcdKeyFile: "/opt/etcd/ssl/server-key.pem"
  etcdCertFile: "/opt/etcd/ssl/server.pem"
  etcdCACertFile: "/opt/etcd/ssl/ca.pem"
EOF
```




## calicoctl
https://github.com/projectcalico/calicoctl

#优化网络模式，调整为BGP模式，为降低局域网网络连接数量，选取两个node作为route reflector

#调整calico模式为BGP
```
cat << EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 26
  cidr: 172.8.0.0/16
  ipipMode: Never
  natOutgoing: true
EOF
```

#禁用全局Full-mesh
``` bash
cat << EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
 logSeverityScreen: Info
 nodeToNodeMeshEnabled: false
 asNumber: 64512
EOF
```

#导出节点1和几点2的配置并修改
``` bash
kubectl label node gihtg-k8s-master01 route-reflector=true

calicoctl get node gihtg-k8s-master01 -o yaml > rr01.yaml

apiVersion: projectcalico.org/v3
kind: Node
metadata:
  creationTimestamp: null
  name: gihtg-k8s-master01
  labels:
    # 增加标签，将rr标签置为true 。建议在kubectl label 中添加，可以保持一致以免发生歧义。
    i-am-a-route-reflector: true
spec:
  bgp:
    ipv4Address: 192.168.2.1/24
    # 增加标签，确保同一个反射簇配置ID一致，即rr01与rr02一致，用于冗余和防环
    routeReflectorClusterID: 224.0.0.1
  orchRefs:
  - nodeName: 192.168.2.1
    orchestrator: k8s

calicoctl apply -f rr01.yaml
```
#建立BGP node 与 Route Reflector 的连接规则
```
$ cat << EOF | calicoctl create -f -
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: peer-to-rrs
spec:
  # 规则1：普通 bgp node 与 rr 建立连接
  nodeSelector: !has(i-am-a-route-reflector)
  peerSelector: has(i-am-a-route-reflector)

---
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: rr-mesh
spec:
  # 规则2：route reflectors 之间也建立连接
  nodeSelector: has(i-am-a-route-reflector)
  peerSelector: has(i-am-a-route-reflector)
EOF
```
#查看bgp连接，及各个节点的路由
```
calicoctl node status
route -n
```