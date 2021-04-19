# ssl

## cfssl

```bash
# ca-csr.json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "Shanghai",
      "ST": "Shanghai",
      "O": "GIHTG",
      "OU": "ops"
    }
  ],
  "ca": {
    "expiry": "876000h"
  }
}

# ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "server": {
        "expiry": "876000h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ]
      },
      "client": {
        "expiry": "876000h",
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ]
      }
    }
  }
}

# kub-apiserver-csr.json
{
  "CN": "kube-apiserver",
  "hosts": [
  "127.0.0.1",
    ],
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}


#初始化ca证书
cfssl gencert -initca ca-csr.json |cfssljson -bare ca -
#CA签发证书
gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=server kube-apiserver-csr.json |cfssljson -bare kube-apiserver
#续签ca证书, ca-csr.json和原始csr保持不变
cfssl gencert -renewca -ca ca.pem -ca-key ca-key.pem
```

## openssl

```bash
#自签证书
openssl genrsa -out dashboard.key 2048
openssl req -days 36000 -new -out dashboard.csr -key dashboard.key -subj '/CN=k8s-dashboard.corp.gihtg.com'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=whoami.gihtg.com"

#验证证书和私钥是否匹配
diff -eq <(openssl x509 -pubkey -noout -in cert.crt) <(openssl rsa -pubout -in cert.key)

#验证该证书是否ca签发
cert=/etc/kubernetes/pki/apiserver.crt
cacert=/etc/kubernetes/pki/ca.crt
expr $(openssl x509 -in ${cacert} -hash -noout) = $(openssl x509 -in ${cert} -issuer_hash -noout)
### 0 为 false， 1 为 true

#openssl 查看证书细节
- 打印证书的过期时间
openssl x509 -in signed.crt -noout -dates
- 打印出证书的内容：
openssl x509 -in cert.pem -noout -text
- 打印出证书的系列号
openssl x509 -in cert.pem -noout -serial
- 打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject
- 以RFC2253规定的格式打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject -nameopt RFC2253
- 在支持UTF8的终端一行过打印出证书的拥有者名字
openssl x509 -in cert.pem -noout -subject -nameopt oneline -nameopt -escmsb
- 打印出证书的MD5特征参数
openssl x509 -in cert.pem -noout -fingerprint
- 打印出证书的SHA特征参数
openssl x509 -sha1 -in cert.pem -noout -fingerprint
- 把PEM格式的证书转化成DER格式
openssl x509 -in cert.pem -inform PEM -out cert.der -outform DER
- 把一个证书转化成CSR
openssl x509 -x509toreq -in cert.pem -out req.pem -signkey key.pem
- 给一个CSR进行处理，颁发字签名证书，增加CA扩展项
openssl x509 -req -in careq.pem -extfile openssl.cnf -extensions v3_ca -signkey key.pem -out cacert.pem
- 给一个CSR签名，增加用户证书扩展项
openssl x509 -req -in req.pem -extfile openssl.cnf -extensions v3_usr -CA cacert.pem -CAkey key.pem -CAcreateserial
- 查看csr文件细节：
openssl req -in my.csr -noout -text
```

## letencrypt
```bash
#安装amce配置工具certbot
yum install certbot

#配置nginx验证路径，并将公网DNS指向该nginx服务器
    server {
        listen 80;
        server_name test.domain.com;
        access_log off;
        location ^~ /.well-known/acme-challenge/ {
            root /opt/nginx/etc/ssl/certbot/;
            #default_type "text/plain";
            #try_files $uri =404;
        }
	  }


#自动生成letencrypt颁发的证书
certbot certonly --agree-tos -d test.domain.com --webroot -w /opt/nginx/etc/ssl/certbot/ --dry-run

#忽略时间限制，强制更新证书
certbot renew --cert-name test.domain.com --force-renewal

#设置crontab，使证书每天自动保持续签尝试，并使nginx重新加载配置，保持证书最新
crontab -e
0 1 * * *  /usr/bin/certbot renew --renew-hook '/usr/bin/nginx -s reload'
```