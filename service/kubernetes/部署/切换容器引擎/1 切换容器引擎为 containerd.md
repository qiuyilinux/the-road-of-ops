> 参考资料：[https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#containerd](#containerd)



### 1 配置先决条件

```shell
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 如果之前装过 docker 需要进行如下操作
# systemctl stop docker 
# systmectl stop docker.scoket

# 设置必需的 sysctl 参数，这些参数在重新启动后仍然存在。
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```



### 2 安装 containerd

```shell
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install -y containerd.io
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```



### 3 修改配置文件

• pause镜像设置过阿里云镜像仓库地址

• cgroups驱动设置为systemd

• 拉取Docker Hub镜像配置加速地址设置为阿里云镜像仓库地址

```shell
vi /etc/containerd/config.toml
   [plugins."io.containerd.grpc.v1.cri"]
       sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.2"  
         ...
   [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
       SystemdCgroup = true
         ...
#   可能会没有这个配置项
#   [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
#       endpoint = ["https://b9pmyelo.mirror.aliyuncs.com"]
          
systemctl restart containerd
```



### 4 配置 kubelet 使用 containerd

```shell
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver=systemd

systemctl restart kubelet
```



### 5 验证

```shell
kubectl get node  -o wide
NAME                STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION          CONTAINER-RUNTIME
kubernetes-master   Ready    control-plane,master   8d    v1.21.0   10.0.9.112    <none>        CentOS Linux 7 (Core)   4.14.141-1.el7.x86_64   docker://20.10.16
kubernetes-node-1   Ready    <none>                 8d    v1.21.0   10.0.9.113    <none>        CentOS Linux 7 (Core)   4.14.141-1.el7.x86_64   docker://20.10.16
kubernetes-node-2   Ready    <none>                 8d    v1.21.0   10.0.9.114    <none>        CentOS Linux 7 (Core)   4.14.141-1.el7.x86_64   containerd://1.6.4
```





### 6 管理容器工具

containerd提供了ctr命令行工具管理容器，但功能比较简单，所以一般会用crictl工具检查和调试容器。

项目地址：https://github.com/kubernetes-sigs/cri-tools/

**设置crictl连接containerd：**

```shell
vi /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```



**下面是docker与crictl命令对照表：**

| 镜像相关功能     | Docker                  | Containerd               |
| ---------------- | ----------------------- | ------------------------ |
| 显示本地镜像列表 | docker images           | crictl images            |
| 下载镜像         | docker pull             | crictl pull              |
| 上传镜像         | docker push             | 无，例如buildk           |
| 删除本地镜像     | docker rmi              | crictl rmi               |
| 查看镜像详情     | docker inspect IMAGE-ID | crictl inspecti IMAGE-ID |

 

| 容器相关功能 | Docker         | Containerd     |
| ------------ | -------------- | -------------- |
| 显示容器列表 | docker ps      | crictl ps      |
| 创建容器     | docker create  | crictl create  |
| 启动容器     | docker start   | crictl start   |
| 停止容器     | docker stop    | crictl stop    |
| 删除容器     | docker rm      | crictl rm      |
| 查看容器详情 | docker inspect | crictl inspect |
| attach       | docker attach  | crictl attach  |
| exec         | docker exec    | crictl exec    |
| logs         | docker logs    | crictl logs    |
| stats        | docker stats   | crictl stats   |

 

| POD相关功能   | Docker | Containerd      |
| ------------- | ------ | --------------- |
| 显示 POD 列表 | 无     | crictl pods     |
| 查看 POD 详情 | 无     | crictl inspectp |
| 运行 POD      | 无     | crictl runp     |
| 停止 POD      | 无     | crictl stopp    |





### 7 切换回 docker 引擎

注：练习完后，建议还切回Docker引擎，就是把kubelet配置参数去掉即可。

```shell
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=
systemctl restart docker
systemctl restart kubelet
```

 

 