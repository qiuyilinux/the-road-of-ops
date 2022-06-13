### 1 kubeadm init 可能遇到错误

> 会有具体的报错提示

- swap 没关
- 内核参数没设置



### 2 kubeadm restet 清空下， 再 init



### 3 kubectl get cs 查看 master 节点组件错误

```
# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```

那是因为默认没有开机健康检查端口（部分版本），但是服务是正常的。

```shell
# 可以通过注释掉 --port=0 来启用健康检查
vi /etc/kubernetes/manifests/kube-scheduler.yaml

vi /etc/kubernetes/manifests/kube-controller-manager.yaml

```

当如上两个文件被修改时，会自动重新拉起这两个容器

```
# kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS   AGE
calico-kube-controllers-685b65ddf9-rhmtw    1/1     Running   0          7d6h
calico-node-5w6r5                           1/1     Running   0          7d6h
calico-node-bf47f                           1/1     Running   0          7d6h
calico-node-h288c                           1/1     Running   0          7d6h
coredns-545d6fc579-7xg8j                    1/1     Running   0          7d22h
coredns-545d6fc579-wcpxw                    1/1     Running   0          7d22h
etcd-kubernetes-master                      1/1     Running   0          7d22h
kube-apiserver-kubernetes-master            1/1     Running   0          7d22h
kube-controller-manager-kubernetes-master   1/1     Running   0          6m22s
kube-proxy-4fdsl                            1/1     Running   0          7d22h
kube-proxy-6kz9k                            1/1     Running   0          7d22h
kube-proxy-nqlml                            1/1     Running   0          7d22h
kube-scheduler-kubernetes-master            1/1     Running   0          7m2s
```





### 4 calico 网络组件有问题

> 可以尝试重新部署

```shell
kubectl delete -f calico.yaml
kubectl apply -f calico.yaml
```

