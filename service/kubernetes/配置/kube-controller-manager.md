### 1 启动参数详解

```
•--kubeconfig：连接apiserver配置文件
•--leader-elect：当该组件启动多个时，自动选举（HA）
•--cluster-signing-cert-file/--cluster-signing-key-file：自动为kubelet颁发证书的CA，与apiserver保持一致
```

### 2 启动参数实例

```shell
cat > /usr/local/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--leader-elect=true \\
--kubeconfig=/opt/kubernetes/cfg/kube-controller-manager.kubeconfig \\
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

