# command

```
#重启pod
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -
#强制删除pod
kubectl delete pods <pod> --grace-period=0 --force
#同时获取命名空间和pod名
kubectl get pod -A|xargs -l bash -c 'echo $0 \|$1'
```

