Calico 是一个纯三层的数据中心网络方案，是目前Kubernetes主流的网络方案。



### 1 下载 YAML

```shell
wget https://docs.projectcalico.org/manifests/calico.yaml
```



### 2 修改 YAML

下载完后还需要修改里面定义Pod网络（CALICO_IPV4POOL_CIDR），与前面kubeadm init的 --pod-network-cidr指定的一样。

```shell
...
            - name: CALICO_IPV4POOL_CIDR
              value: "10.244.0.0/16"
...
```



### 3 部署

修改完成文件后，部署：

```shell
kubectl apply -f calico.yaml
kubectl get pods -n kube-system
```

等Calico Pod都Running，节点也会准备就绪。

```shell
# kubectl get nodes
NAME                STATUS   ROLES                  AGE     VERSION
kubernetes-master   Ready    control-plane,master   6d19h   v1.21.0
kubernetes-node-1   Ready    <none>                 6d19h   v1.21.0
kubernetes-node-2   Ready    <none>                 6d19h   v1.21.0
```



### 4 CoreDNS 问题处理

```shell
kubectl get pods -n kube-system
NAME                                     READY   STATUS             RESTARTS   AGE
calico-kube-controllers-8db96c76-z7h5p   1/1     Running            0          16m
calico-node-pshdd                        1/1     Running            0          16m
calico-node-vjwlg                        1/1     Running            0          16m
coredns-545d6fc579-5hd9x                 0/1     ImagePullBackOff   0          16m
coredns-545d6fc579-wdbsz                 0/1     ImagePullBackOff   0          16m
```

 在所有节点执行

```shell
docker pull registry.aliyuncs.com/google_containers/coredns:1.8.0
docker tag registry.aliyuncs.com/google_containers/coredns:1.8.0 registry.aliyuncs.com/google_containers/coredns/coredns:v1.8.0
```

过一会儿，CoreDNS Pod会自动恢复正常。