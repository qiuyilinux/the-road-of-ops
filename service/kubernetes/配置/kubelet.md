### 1 启动参数详解

```shell
• --hostname-override：显示名称，集群中唯一
• --network-plugin：启用CNI
• --kubeconfig：空路径，会自动生成，后面用于连接apiserver
• --bootstrap-kubeconfig：首次启动向apiserver申请证书
• --config：配置参数文件
• --cert-dir：kubelet证书生成目录
• --pod-infra-container-image：管理Pod网络容器的镜像
```



### 2 启动参数实例

```shell
cat > /usr/local/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--hostname-override=kubernetes-node-1 \\
--network-plugin=cni \\
--kubeconfig=/usr/local/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/usr/local/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/usr/local/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/usr/local/kubernetes/ssl \\
--pod-infra-container-image=lizhenliang/pause-amd64:3.0"
EOF
```



### 3 配置参数文件

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



