### 1 创建工作目录并拷贝必要文件

```shell
mkdir -p /usr/local/kubernetes/{bin,cfg,ssl,logs} 

# 拷贝二进制文件
cd /tmp && wget https://dl.k8s.io/v1.20.15/kubernetes-server-linux-amd64.tar.gz
tar zxvf kubernetes-server-linux-amd64.tar.gz
cd /tmp/kubernetes/server/bin
cp kubelet kube-proxy /usr/local/kubernetes/bin 

# 拷贝证书到到 node （master 操作）
scp /usr/local/kubernetes/ssl/ca.pem kubernetes-node-1:/usr/local/kubernetes/ssl/ca.pem
```



### 2 部署 kubelet

#### 2.1创建配置文件

```shell
cat > /usr/local/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--hostname-override=kubernetes-node-2 \\
--network-plugin=cni \\
--kubeconfig=/usr/local/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/usr/local/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/usr/local/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/usr/local/kubernetes/ssl \\
--pod-infra-container-image=lizhenliang/pause-amd64:3.0"
EOF
```



#### 2.2 配置参数文件

```shell
cat > /usr/local/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS:
- 10.0.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /usr/local/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```



#### 2.3 生成 kubelet 初次加入集群引导 kubeconfig 文件

> master 操作

```shell
KUBE_CONFIG="/tmp/bootstrap.kubeconfig"
KUBE_APISERVER="https://192.168.122.210:6443" # apiserver IP:PORT
TOKEN="c47ffb939f5ca36231d9e3121a252940" # 与token.csv里保持一致

# 生成 kubelet bootstrap kubeconfig 配置文件
kubectl config set-cluster kubernetes \
  --certificate-authority=/usr/local/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=${KUBE_CONFIG}
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}

scp /tmp/bootstrap.kubeconfig kubernetes-node-2:/usr/local/kubernetes/cfg/bootstrap.kubeconfig
```



#### 2.4 systemd 管理 kubelet

```shell
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service

[Service]
EnvironmentFile=/usr/local/kubernetes/cfg/kubelet.conf
ExecStart=/usr/local/kubernetes/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```



#### 2.5 启动并设置开机自启

```shell
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
```



#### 2.6 批准 kubelet 证书申请并加入集群

> master 操作

```shell
# 查看kubelet证书请求
[root@kubernetes-master-1 cfg]# kubectl get csr
NAME                                                   AGE   SIGNERNAME                                    REQUESTOR           CONDITION
node-csr-h9LVTZcbAPS1IUUkdv0S0EBOQBD-odLUBb2lDz28sCk   30s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

# 批准申请
[root@kubernetes-master-1 cfg]# kubectl certificate approve node-csr-h9LVTZcbAPS1IUUkdv0S0EBOQBD-odLUBb2lDz28sCk
certificatesigningrequest.certificates.k8s.io/node-csr-h9LVTZcbAPS1IUUkdv0S0EBOQBD-odLUBb2lDz28sCk approved

# 查看节点
[root@kubernetes-master-1 cfg]# kubectl get nodes
NAME                STATUS     ROLES    AGE   VERSION
kubernetes-node-1   NotReady   <none>   4s    v1.20.15

# 注：由于网络插件还没有部署，节点会没有准备就绪 NotReady
```

### 3 部署 kube-proxy

#### 3.1 创建配置文件

```shell
cat > /usr/local/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--config=/usr/local/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

#### 3.2 配置参数文件

```shell
cat > /usr/local/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /usr/local/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kubernetes-node-1
clusterCIDR: 10.244.0.0/16
EOF
```

#### 3.3 生成 kube-proxy.kubeconfig 文件

> master
>
> 如果没有生成过 proxy 证书需要进行的操作

```shell
# 切换工作目录
cd /usr/local/kubernetes/ssl

# 创建证书请求文件
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF


# 生成 config.json
cat > ca-config.json << EOFCONTENT
{
    "signing":{
        "default":{
            "expiry": "87600h"
        },
        "profiles":{
            "kubernetes":{
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOFCONTENT

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```

> master 
>
> 必须操作

```shell
# 生成kubeconfig文件：
KUBE_CONFIG="/tmp/kube-proxy.kubeconfig"
KUBE_APISERVER="https://192.168.122.210:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/usr/local/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-credentials kube-proxy \
  --client-certificate=/usr/local/kubernetes/ssl/kube-proxy.pem \
  --client-key=/usr/local/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=${KUBE_CONFIG}
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}

# 拷贝到 node 节点
scp /tmp/kube-proxy.kubeconfig kubernetes-node-2:/usr/local/kubernetes/cfg/kube-proxy.kubeconfig

# 删除多余文件
rm -rf /usr/local/kubernetes/ssl/kube-proxy-csr.json
```



#### 3.4 systemd 管理 kube-proxy

```shell
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target

[Service]
EnvironmentFile=/usr/local/kubernetes/cfg/kube-proxy.conf
ExecStart=/usr/local/kubernetes/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

#### 3.5 启动并设置开机自启

```shell
systemctl daemon-reload
systemctl start kube-proxy
systemctl enable kube-proxy
```

