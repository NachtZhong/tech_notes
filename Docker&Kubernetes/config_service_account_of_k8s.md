# 配置k8s集群的service account

## 生成证书和密钥

### 下载easyrsa3

```bash
# curl -L -O https://storage.googleapis.com/kubernetes-release/easy-rsa/easy-rsa.tar.gz
# tar xzf easy-rsa.tar.gz
# cd easy-rsa-master/easyrsa3
# ./easyrsa init-pki
```



### 创建根证书

```bash
# ./easyrsa --batch "--req-cn=192.168.120.121@`date +%s`" build-ca nopass
```



### 创建服务端证书和密钥(IP替换为apiserver的内网地址)

```bash
# ./easyrsa --subject-alt-name="IP:192.168.120.121" build-server-full server nopass
```



### 拷贝key到指定目录

```bash
# mkdir /etc/kubernetes/pki
# cp pki/ca.crt pki/issued/server.crt pki/private/server.key /etc/kubernetes/pki/
# chmod 644 /etc/kubernetes/pki/*
```



## 配置kube-apiserver服务

```bash
# vi /etc/kubernetes/apiserver
```

```bash
KUBE_API_ARGS="--client-ca-file=/etc/kubernetes/pki/ca.crt --tls-cert-file=/etc/kubernetes/pki/server.crt --tls-private-key-file=/etc/kubernetes/pki/server.key"
```



## 配置kube-controller-manager服务

```bash
# vi /etc/kubernetes/controller-manager
```

```bash
KUBE_CONTROLLER_MANAGER_ARGS="--service_account_private_key_file=/etc/kubernetes/pki/server.key --root-ca-file=/etc/kubernetes/pki/ca.crt"
```



## 删除旧secrets资源

```bash
# kubectl get secrets --all-namespaces
NAMESPACE     NAME                  TYPE                                  DATA      AGE
default       default-token-s1vfh   kubernetes.io/service-account-token   2         5m
kube-system   default-token-jct68   kubernetes.io/service-account-token   2         4m

# systemctl stop kube-controller-manager(删除旧secrets之前要停止kube-controller-manager服务)

# kubectl delete secret default-token-s1vfh
secret "default-token-s1vfh" deleted

# kubectl delete secret default-token-jct68 --namespace=kube-system
secret "default-token-jct68" deleted
```



## 重新启动kube-apiserver和kube-controller-manager服务

```bash
# systemctl restart kube-apiserver
# systemctl start  kube-controller-manager
```



## 检查新创建的secret是否包含根证书

```bash
# kubectl get secrets --all-namespaces
NAMESPACE     NAME                  TYPE                                  DATA      AGE
default       default-token-tv69r   kubernetes.io/service-account-token   3         3s
kube-system   default-token-27w5m   kubernetes.io/service-account-token   3         3s

# kubectl describe secret default-token-27w5m --namespace=kube-system
Name:       default-token-27w5m
Namespace:  kube-system
Labels:     <none>
Annotations:    kubernetes.io/service-account.name=default
        kubernetes.io/service-account.uid=80bc5d75-41e9-11e7-b90e-000c29f6f813

Type:   kubernetes.io/service-account-token

Data
====
ca.crt:     1233 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50I...
```

可以看到新创建的secret资源已包含`ca.crt`。

至此, k8s集群的serviceaccount验证配置完成