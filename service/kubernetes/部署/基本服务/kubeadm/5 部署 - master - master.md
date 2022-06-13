

### 1 部署 master （命令行）

```shell
kubeadm init \
  --apiserver-advertise-address=10.0.9.112 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.21.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --ignore-preflight-errors=all
```

• --apiserver-advertise-address 集群通告地址

• --image-repository 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址

• --kubernetes-version K8s版本，与上面安装的一致

• --service-cidr 集群内部虚拟网络，Pod统一访问入口

• --pod-network-cidr Pod网络，，与下面部署的CNI网络组件yaml中保持一致



### 2  部署 master （配置文件）

```shell
vi kubeadm.conf
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.21.0
imageRepository: registry.aliyuncs.com/google_containers 
networking:
  podSubnet: 10.244.0.0/16 
  serviceSubnet: 10.96.0.0/12 

kubeadm init --config kubeadm.conf --ignore-preflight-errors=all  
```

初始化完成后，最后会输出一个 join 命令， 注意保存， 以后会用。



### 3 拷贝认证文件

拷贝kubectl使用的连接k8s认证文件到默认路径

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```



### 4 查看工作节点

```shell
kubectl get nodes
kubectl get nodes
NAME               STATUS     ROLES            AGE   VERSION
kubernetes-master   NotReady   control-plane,master   20s   v1.21.0
kubernetes-node-1   NotReady   control-plane,master   20s   v1.21.0
kubernetes-node-2   NotReady   control-plane,master   20s   v1.21.0
```

注： 由于网络插件还没有部署。还没有准备就绪 not ready

> 参考资料：
>
> [https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file](#config-file) 
>
> [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node](#initializing-your-control-plane-node) 



### 5 加入 kubernetes node

在 node 节点上执行，向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

```shell
kubeadm join 10.0.9.112:6443 --token 7gqt13.kncw9hg5085iwclx \
--discovery-token-ca-cert-hash sha256:66fbfcf18649a5841474c2dc4b9ff90c02fc05de0798ed690e1754437be35a01
```

默认token有效期为24小时，当过期之后，该token就不可用了。这时就需要重新创建token，可以直接使用命令快捷生成：

```shell
kubeadm token create --print-join-command
```

> 参考资料：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-join/

