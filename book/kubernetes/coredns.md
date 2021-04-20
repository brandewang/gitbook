# CoreDns

## 安装

```bash
helm repo add coredns https://coredns.github.io/helm
helm pull coredns/coredns
#安装前修改values.yaml中的副本数、clusterip、容忍节点等参数.
helm install coredns coredns/ -n kube-system
```

