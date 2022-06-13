### 1 准备 cfssl 证书生成工具

> 记得修改证书的签发的地址

```shell
cd /tmp/kubernetes/ssl/
# 下载 cfssl 工具
cat > get_cfssl.sh << EOF
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl*
mv cfssl_linux-amd64 /usr/bin/cfssl
mv cfssljson_linux-amd64 /usr/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
EOF

chmod +x /tmp/kubernetes/ssl/get_cfssl.sh
sh /tmp/kubernetes/ssl/get_cfssl.sh

mkdir -p /tmp/kubernetes/ssl/etcd
cd /tmp/kubernetes/ssl/etcd

# 生成 证书脚本
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
            "ST": "Beijing"
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
        "192.168.122.210",
        "192.168.122.211",
        "192.168.122.220",
        "192.168.122.221"
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



### 2 部署 etcd 集群

> 需要 etcd 集群到各个节点上，注意配置文件变化
>
> 依赖于上一步证书， 各个节点使用的证书一致。

```shell
# 下载软件包到临时目录并解压
mkdir -p /tmp/kubernetes/component
cd /tmp/kubernetes/component
mkdir etcd/
cd etcd/
wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz
tar zxvf etcd-v3.4.9-linux-amd64.tar.gz

# 创建工作目录解压二进制包
mkdir -p /usr/local/etcd/{bin,cfg,ssl}
cp /tmp/kubernetes/component/etcd/etcd-v3.4.9-linux-amd64/{etcd,etcdctl} /usr/local/etcd/bin/
ln -s /usr/local/etcd/bin/{etcd,etcdctl} /usr/local/bin/

# 创建配置文件
cat > /usr/local/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.122.210:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.122.210:2379"

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.122.210:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.122.210:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.122.210:2380,etcd-2=https://192.168.122.220:2380,etcd-3=https://192.168.122.221:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

# 创建启动文件
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
EnvironmentFile=/usr/local/etcd/cfg/etcd.conf
ExecStart=/usr/local/etcd/bin/etcd \
--cert-file=/usr/local/etcd/ssl/server.pem \
--key-file=/usr/local/etcd/ssl/server-key.pem \
--peer-cert-file=/usr/local/etcd/ssl/server.pem \
--peer-key-file=/usr/local/etcd/ssl/server-key.pem \
--trusted-ca-file=/usr/local/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/usr/local/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# 拷贝证书
cp /tmp/kubernetes/ssl/etcd/ca*pem /tmp/kubernetes/ssl/etcd/server*pem /usr/local/etcd/ssl/

# 启动开机自启
systemctl daemon-reload
systemctl start etcd
systemctl enable etcd
```



集群配置修改

```shell
vi /opt/etcd/cfg/etcd.conf
#[Member]
ETCD_NAME="etcd-1"   # 修改此处，节点2改为etcd-2，节点3改为etcd-3
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.122.210:2380"   # 修改此处为当前服务器IP
ETCD_LISTEN_CLIENT_URLS="https://192.168.122.210:2379" # 修改此处为当前服务器IP

#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.122.210:2380" # 修改此处为当前服务器IP
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.122.210:2379" # 修改此处为当前服务器IP
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.122.210:2380,etcd-2=https://192.168.122.220:2380,etcd-3=https://192.168.122.221:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```



### 3 查看集群状态

```shell
[root@kubernetes-master-1 cfg]# etcdctl --cacert=/usr/local/etcd/ssl/ca.pem --cert=/usr/local/etcd/ssl/server.pem --key=/usr/local/etcd/ssl/server-key.pem --endpoints="https://192.168.122.210:2379,https://192.168.122.220:2379,https://192.168.122.221:2379" endpoint health --write-out=table
+------------------------------+--------+-------------+-------+
|           ENDPOINT           | HEALTH |    TOOK     | ERROR |
+------------------------------+--------+-------------+-------+
| https://192.168.122.220:2379 |   true | 12.674716ms |       |
| https://192.168.122.210:2379 |   true | 12.762085ms |       |
| https://192.168.122.221:2379 |   true | 15.137042ms |       |
+------------------------------+--------+-------------+-------+
```

