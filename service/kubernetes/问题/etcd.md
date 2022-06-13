### 1 etcd 成员错误问题

#### 问题

在启动 etcd 集群的时候错误的将其中一个节点的IP配置错误，如下：

> 其中 IP 192.168.122.211 为错误的节点地址，正确地址应为 192.168.122.221

```shell
...
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.122.210:2380,etcd-2=https://192.168.122.220:2380,etcd-3=https://192.168.122.211:2380"
...
```

当在 192.168.122.221 上尝试启用 etcd 的时候报错如下：

```shell
...
etcd cluster id mistmatch
...
```

etcd 无法启动

#### 解决

这是因为已经配置的成员对应关系和实际的不匹配导致的。

此时我们可以有两种方式解决这个问题

```shell
1. 在集群中其他正常的 etcd 节点上移除错误节点再重新添加正确节点
	etcdctl member list
	etcdctl member remove "ID"
	etcdctl member add etcd-3 --peer-urls=https://192.168.122.221:2080 
	注： 可能需要指定证书
2. 停止所有节点后删除集群所有节点中的 /var/lib/etcd/default.etcd/member 然后再重新启动
```

