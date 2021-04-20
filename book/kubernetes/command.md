# command

```
#重启pod
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f -

```

