# command

```
#重启pod
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -
#强制删除pod
kubectl delete pods <pod> --grace-period=0 --force
#同时获取命名空间和pod名
kubectl get pod -A|xargs -l bash -c 'echo $0 \|$1'

#kubelet资源预留 
/etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2 --kube-reserved=cpu=500m,memory=1Gi,ephemeral-storage=1Gi --system-reserved=cpu=500m,memory=1Gi,ephemeral-storage=5Gi --eviction-hard=memory.available<5%,nodefs.available<10%,imagefs.available<10% --eviction-soft=memory.available<10%,nodefs.available<15%,imagefs.available<15% --eviction-soft-grace-period=memory.available=2m,nodefs.available=2m,imagefs.available=2m --eviction-max-pod-grace-period=30 --eviction-minimum-reclaim=memory.available=0Mi,nodefs.available=500Mi,imagefs.available=500Mi"
```

