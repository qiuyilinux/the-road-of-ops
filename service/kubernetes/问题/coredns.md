###  CoreDNS 问题处理

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