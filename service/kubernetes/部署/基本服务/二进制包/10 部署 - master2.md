### 1 安装 docker

> 参考 ./5 



### 2 创建 etcd 证书目录

```
mkdir -p /ysr/local/etcd/ssl
```



### 3 拷贝文件 

> master1
>
> 拷贝已经部署好的 node 相关文件到新节点

```
scp -r /usr/local/kubernetes root@kubernetes-master-2:/usr/local
scp -r /usr/local/etcd/ssl root@kubernetes-master-2:/usr/local/etcd/ssl
scp /usr/lib/systemd/system/kube* root@kubernetes-master-2:/usr/lib/systemd/system
scp /usr/bin/kubectl  root@kubernetes-master-2:/usr/bin
scp -r ~/.kube root@kubernetes-master-2:~
```



### 4 修改配置文件

```shell
sed -i 's/192.168.122.210/192.168.122.211/g' /usr/local/kubernetes/cfg/kube-apiserver.conf 
# etcd 的地址不能修改
sed -i 's/192.168.122.211:/192.168.122.210:/g' /usr/local/kubernetes/cfg/kube-apiserver.conf 

sed -i 's/192.168.122.210/192.168.122.211/g' /usr/local/kubernetes/cfg/kube-controller-manager.kubeconfig
sed -i 's/192.168.122.210/192.168.122.211/g' /usr/local/kubernetes/cfg/kube-scheduler.kubeconfig
sed -i 's/192.168.122.210/192.168.122.211/g' ~/.kube/config
```



### 5 启动开机自启

```shell
systemctl daemon-reload
systemctl start kube-apiserver kube-controller-manager kube-scheduler 
systemctl enable kube-apiserver kube-controller-manager kube-scheduler 
```



### 6 查看集群状态

```shell
# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true"}
etcd-2               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
```



