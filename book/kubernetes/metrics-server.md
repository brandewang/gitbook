# metrics-server

url: https://github.com/kubernetes-sigs/metrics-server
```
#下载manifests
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

#修改为国内镜像资源，添加对应args
      containers:
      - name: metrics-server
        image: lizhenliang/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
          - --kubelet-preferred-address-types=InternalIP

#创建资源
kubectl apply -f components.yaml
```