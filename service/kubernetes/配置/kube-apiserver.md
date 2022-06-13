### 1 启动参数详解

```shell
•--logtostderr：启用日志
•---v：日志等级
•--log-dir：日志目录
•--etcd-servers：etcd集群地址
•--bind-address：监听地址
•--secure-port：https安全端口
•--advertise-address：集群通告地址
•--allow-privileged：启用授权
•--service-cluster-ip-range：Service虚拟IP地址段
•--enable-admission-plugins：准入控制模块
•--authorization-mode：认证授权，启用RBAC授权和节点自管理
•--enable-bootstrap-token-auth：启用TLS bootstrap机制
•--token-auth-file：bootstrap token文件
•--service-node-port-range：Service nodeport类型默认分配端口范围
•--kubelet-client-xxx：apiserver访问kubelet客户端证书
•--tls-xxx-file：apiserver https证书
•1.20版本必须加的参数：--service-account-issuer，--service-account-signing-key-file
•--etcd-xxxfile：连接Etcd集群证书
•--audit-log-xxx：审计日志
•启动聚合层相关配置：--requestheader-client-ca-file，--proxy-client-cert-file，--proxy-client-key-file，--requestheader-allowed-names，--requestheader-extra-headers-prefix，--requestheader-group-headers，--requestheader-username-headers，--enable-aggregator-routing
```

### 2 启动参数实例

```shell
cat > /usr/local/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--etcd-servers=https://192.168.122.210:2379,https://192.168.122.220:2379,https://192.168.122.221:2379 \\
--bind-address=192.168.122.210 \\
--secure-port=6443 \\
--advertise-address=192.168.122.210 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/opt/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \\
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/opt/kubernetes/ssl/server.pem  \\
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \\
--client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--service-account-issuer=api \\
--service-account-signing-key-file=/opt/kubernetes/ssl/server-key.pem \\
--etcd-cafile=/opt/etcd/ssl/ca.pem \\
--etcd-certfile=/opt/etcd/ssl/server.pem \\
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
--requestheader-client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--proxy-client-cert-file=/opt/kubernetes/ssl/server.pem \\
--proxy-client-key-file=/opt/kubernetes/ssl/server-key.pem \\
--requestheader-allowed-names=kubernetes \\
--requestheader-extra-headers-prefix=X-Remote-Extra- \\
--requestheader-group-headers=X-Remote-Group \\
--requestheader-username-headers=X-Remote-User \\
--enable-aggregator-routing=true \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
EOF
```

