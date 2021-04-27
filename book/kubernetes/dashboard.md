# Dashboard

## install
```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml

默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：

$ vi recommended.yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
$ kubectl apply -f recommended.yaml
$ kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-6b4884c9d5-gl8nr   1/1     Running   0          13m
kubernetes-dashboard-7f99b75bf4-89cds        1/1     Running   0          13m
```
访问地址：https://NodeIP:30001

## helm
```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
mkdir /opt/helm && cd /opt/helm
helm pull kubernetes-dashboard/kubernetes-dashboard
#vim values.yaml
service:
  type: NodePort
  nodePort: 30001
  externalPort: 443

helm install elasticsearch ./elasticsearch -n elk
```

## dashboard (master01 上执行), 自签证书。
```
cd /opt/cert/
openssl genrsa -out dashboard.key 2048
openssl req -days 36000 -new -out dashboard.csr -key dashboard.key -subj '/CN=k8s-dashboard.corp.gihtg.com'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kubernetes-dashboard
```

## 创建service account并绑定默认cluster-admin管理员集群角色：

```
# 创建用户
$ kubectl create serviceaccount dashboard-admin -n kube-system
# 用户授权
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
# 获取用户Token
$ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```
使用输出的token登录Dashboard。

## 创建登陆dashboard的配置文件
```
kubectl create sa dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')
DASHBOARD_LOGIN_TOKEN=$(kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}')
echo ${DASHBOARD_LOGIN_TOKEN}

kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/cert/ca.pem \
  --embed-certs=true \
  --server=https://{{ k8s_master_vip }}:8443 \
  --kubeconfig=dashboard.kubeconfig

kubectl config set-credentials dashboard_user \
  --token=${DASHBOARD_LOGIN_TOKEN} \
  --kubeconfig=dashboard.kubeconfig

kubectl config set-context default \
  --cluster=kubernetes \
  --user=dashboard_user \
  --kubeconfig=dashboard.kubeconfig

kubectl config use-context default --kubeconfig=dashboard.kubeconfig
```
