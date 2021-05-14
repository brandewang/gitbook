# metrics-server

url: https://github.com/kubernetes-sigs/metrics-server
```
#下载manifests
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.4/components.yaml

#修改为国内镜像资源，添加对应args
      containers:
      - name: metrics-server
        image: bitnami/metrics-server:v0.4.4
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP

#创建资源
kubectl apply -f components.yaml
```
