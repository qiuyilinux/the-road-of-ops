### 权限不足

报错

```
Error from server (Forbidden): Forbidden (user=etcd, verb=get, resource=nodes, subresource=proxy)
```

解决

> 临时给一个 admin 权限

```
kubectl create clusterrolebinding etcd   --clusterrole=cluster-admin   --user=etcd
```

