# monitor

## kube-state-mertics

将k8s各类资源对象暴露为prometheus指标的简单服务

- git: https://github.com/kubernetes/kube-state-metrics.git

#部署

```
git clone https://github.com/kubernetes/kube-state-metrics.git
cd kube-state-metrics
git checkout v1.9.8
cd examples/standard
#change manifests
kubectl apply -f .
```

## kube-prometheus

https://github.com/prometheus-operator/kube-prometheus