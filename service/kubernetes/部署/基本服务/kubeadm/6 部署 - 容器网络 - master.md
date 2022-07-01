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



