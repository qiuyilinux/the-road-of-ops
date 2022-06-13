### 1 生成kube-apiserver证书

> 依赖于之前在部署 etcd 时下载的生成证书工具

```shell
mkdir /tmp/kubernetes/ssl/kubernetes
cd /tmp/kubernetes/ssl/kubernetes


# 生成证书
cat >  certs.sh  << EOF
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

cat > ca-csr.json << EOFCONTENT
{
    "CN": "kubernetes",
    "key":{
        "algo": "rsa",
        "size": 2048
    },
    "names":[
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ],
    "ca":{
        "expiry": "87600h"
    }
}
EOFCONTENT

# 根据 json 生成 ca 证书
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
cat > server-csr.json << EOFCONTENT
{
    "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "10.0.0.1",
      "192.168.122.210",
      "192.168.122.211",
      "192.168.122.220",
			"192.168.122.221",
			"192.168.122.240",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Beijing",
            "L": "Beijing"
        }
    ]
}
EOFCONTENT

# 使用 ca 证书 根据 json 生成 证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
EOF


sed -i 's/EOFCONTENT/EOF/g' /tmp/kubernetes/ssl/etcd/certs.sh
chmod +x /tmp/kubernetes/ssl/etcd/certs.sh

# 生成证书 server.pem server-key.pem
sh certs.sh 
```



### 2 下载二进制包

> https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md
> 注：打开链接你会发现里面有很多包，下载一个server包就够了，包含了Master和Worker Node二进制文件。

```
mdkir /tmp/kubernetes/component/kubernetes
cd /tmp/kubernetes/component/kubernetes
wget https://dl.k8s.io/v1.20.15/kubernetes-server-linux-amd64.tar.gz
tar zxvf kubernetes-server-linux-amd64.tar.gz
```



### 3 解压二进制包

```
mkdir -p /usr/local/kubernetes/{bin,cfg,ssl,logs} 
cd kubernetes/server/bin
cp kube-apiserver kube-scheduler kube-controller-manager /usr/local/kubernetes/bin
cp kubectl /usr/bin/
```





### 4 部署 kube-apiserver

#### 4.1 创建配置文件

```shell
cat > /usr/local/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--etcd-servers=https://192.168.122.210:2379,https://192.168.122.220:2379,https://192.168.122.221:2379 \\
--bind-address=192.168.122.210 \\
--secure-port=6443 \\
--advertise-address=192.168.122.210 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/usr/local/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/usr/local/kubernetes/ssl/server.pem \\
--kubelet-client-key=/usr/local/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/usr/local/kubernetes/ssl/server.pem  \\
--tls-private-key-file=/usr/local/kubernetes/ssl/server-key.pem \\
--client-ca-file=/usr/local/kubernetes/ssl/ca.pem \\
--service-account-key-file=/usr/local/kubernetes/ssl/ca-key.pem \\
--service-account-issuer=api \\
--service-account-signing-key-file=/usr/local/kubernetes/ssl/server-key.pem \\
--etcd-cafile=/usr/local/etcd/ssl/ca.pem \\
--etcd-certfile=/usr/local/etcd/ssl/server.pem \\
--etcd-keyfile=/usr/local/etcd/ssl/server-key.pem \\
--requestheader-client-ca-file=/usr/local/kubernetes/ssl/ca.pem \\
--proxy-client-cert-file=/usr/local/kubernetes/ssl/server.pem \\
--proxy-client-key-file=/usr/local/kubernetes/ssl/server-key.pem \\
--requestheader-allowed-names=kubernetes \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--enable-aggregator-routing=true \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/usr/local/kubernetes/logs/k8s-audit.log"
EOF
```

#### 4.2 拷贝证书

```shell
cp /tmp/kubernetes/ssl/kubernetes/ca*pem /tmp/kubernetes/ssl/kubernetes/server*pem /usr/local/kubernetes/ssl/
```

#### 4.3 创建 token 文件

```shell
cat > /usr/local/kubernetes/cfg/token.csv << EOF
c47ffb939f5ca36231d9e3121a252940,kubelet-bootstrap,10001,"system:node-bootstrapper"
EOF
# 格式：token，用户名，UID，用户组
# token也可自行生成替换： head -c 16 /dev/urandom | od -An -t x | tr -d ' '
```

#### 4.4 单元文件管理 kube-apiserver

```shell
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/usr/local/kubernetes/cfg/kube-apiserver.conf
ExecStart=/usr/local/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

#### 4.5 启动开机自启

```shell
systemctl daemon-reload
systemctl start kube-apiserver 
systemctl enable kube-apiserver
```



### 5 部署 kube-controller-manager

#### 5.1 创建配置文件

```shell
cat > /usr/local/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--leader-elect=true \\
--kubeconfig=/usr/local/kubernetes/cfg/kube-controller-manager.kubeconfig \\
--bind-address=127.0.0.1 \\
--allocate-node-cidrs=true \\
--cluster-cidr=10.244.0.0/16 \\
--service-cluster-ip-range=10.0.0.0/24 \\
--cluster-signing-cert-file=/usr/local/kubernetes/ssl/ca.pem \\
--cluster-signing-key-file=/usr/local/kubernetes/ssl/ca-key.pem  \\
--root-ca-file=/usr/local/kubernetes/ssl/ca.pem \\
--service-account-private-key-file=/usr/local/kubernetes/ssl/ca-key.pem \\
--cluster-signing-duration=87600h0m0s"
EOF
```

#### 5.2 生成 kuberconfig 文件

##### 5.2.1 生成kube-controller-manager证书

```shell
 cd /tmp/kubernetes/ssl/kubernetes/
 cat > kube-controller-manager-csr.json << EOF
{
  "CN": "system:kube-controller-manager",
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
      "O": "system:masters",
      "OU": "System"
     }
  ]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

# 拷贝证书到指令路径
cp /tmp/kubernetes/ssl/kubernetes/kube-controller-manager*.pem  /usr/local/kubernetes/ssl/
```

##### 5.2.2 生成kubeconfig文件

> 以下是 shell 命令直接在终端执行

```shell
cd /usr/local/kubernetes/ssl/
KUBE_CONFIG="/usr/local/kubernetes/cfg/kube-controller-manager.kubeconfig"
KUBE_APISERVER="https://192.168.122.210:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/usr/local/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-credentials kube-controller-manager \
  --client-certificate=./kube-controller-manager.pem \
  --client-key=./kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-controller-manager \
  --kubeconfig=${KUBE_CONFIG}
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 5.3 单元文件管理 kube-controller-manager

```shell
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/usr/local/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/usr/local/kubernetes/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target 
EOF
```

#### 5.4 启动开机自启

```shell
systemctl daemon-reload
systemctl start kube-controller-manager
systemctl enable kube-controller-manager
```

### 6 部署 kube-scheduler

#### 6.1 创建配置文件

```shell
cat > /usr/local/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--leader-elect \\
--kubeconfig=/usr/local/kubernetes/cfg/kube-scheduler.kubeconfig \\
--bind-address=127.0.0.1"
EOF
```

#### 6.2 生成kubeconfig文件

##### 6.2.1 生成 kube-scheduler证书

```shell
 cd /tmp/kubernetes/ssl/kubernetes/
 cat > kube-scheduler-csr.json << EOF
{
  "CN": "system:kube-scheduler",
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
      "O": "system:masters",
      "OU": "System"
     }
  ]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler

# 拷贝证书到指令路径
cp /tmp/kubernetes/ssl/kubernetes/kube-scheduler*.pem  /usr/local/kubernetes/ssl/
```

##### 6.2.2 生成 kubeconfig 文件

```shell
cd /usr/local/kubernetes/ssl/
KUBE_CONFIG="/usr/local/kubernetes/cfg/kube-scheduler.kubeconfig"
KUBE_APISERVER="https://192.168.122.210:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/usr/local/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-credentials kube-scheduler \
  --client-certificate=./kube-scheduler.pem \
  --client-key=./kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-scheduler \
  --kubeconfig=${KUBE_CONFIG}
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 6.3 单元文件管理 kube-scheduler

```shell
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
EnvironmentFile=/usr/local/kubernetes/cfg/kube-scheduler.conf
ExecStart=/usr/local/kubernetes/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

#### 6.4 启动开机自启

```shell
systemctl daemon-reload
systemctl start kube-scheduler
systemctl enable kube-scheduler
```

### 7 查看集群状态

#### 7.1 生成kubectl连接集群的证书

```shell
cd /tmp/kubernetes/ssl/kubernetes/
cat > admin-csr.json <<EOF
{
  "CN": "admin",
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
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin

cp /tmp/kubernetes/ssl/kubernetes/admin*.pem  /usr/local/kubernetes/ssl/
```

#### 7.2 生成 kubeconfig 文件

```shell
cd /usr/local/kubernetes/ssl/
mkdir /root/.kube

KUBE_CONFIG="/root/.kube/config"
KUBE_APISERVER="https://192.168.122.210:6443"

kubectl config set-cluster kubernetes \
  --certificate-authority=/usr/local/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-credentials cluster-admin \
  --client-certificate=./admin.pem \
  --client-key=./admin-key.pem \
  --embed-certs=true \
  --kubeconfig=${KUBE_CONFIG}
kubectl config set-context default \
  --cluster=kubernetes \
  --user=cluster-admin \
  --kubeconfig=${KUBE_CONFIG}
kubectl config use-context default --kubeconfig=${KUBE_CONFIG}
```

#### 7.3 通过kubectl工具查看当前集群组件状态

```shell
[root@kubernetes-master-1 ssl]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
# 如上输出说明Master节点组件运行正常。
```

### 8 授权kubelet-bootstrap用户允许请求证书

```shell
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

