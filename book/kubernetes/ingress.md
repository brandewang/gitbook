# ingress

## Ingress-nginx

github: https://github.com/kubernetes/ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml

- 修改国内镜像地址: lizhenliang/nginx-ingress-controller:0.30.0
- 建议直接宿主机网络暴露: hostNetwork: true
- 修改Deployment 为 DaemonSet
- 删除replicas

## traefik
#添加helm 仓库
helm repo add traefik https://helm.traefik.io/traefik
#下载traefik
helm pull traefik/traefik
#修改values
- Daemonset方式部署
- 禁用Service
- 将hostnetwork设置为true
- 设置nodeSelector: traefik: "true"
- 
```bash
#安装traefik
helm install traefik traefik/ -n traefik
#将指定k8s node 添加traefik为true的标签
kubectl label node gihtg-k8s-node01 traefik-true
```