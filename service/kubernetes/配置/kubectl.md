### 1 配置文件路径

> kubectl 使用 kubeconfig 认证认证文件链接 kubernetes 集群， 使用 kubectl config 指令生成 kubeconfig 文件。

```shell
～/.kube/

# 拥有这个配置文件的节点才可以查看集群状态
```



### 2 配置文件详解

```shell
apiVersion: v1
clusters:
# 集群
- cluster:
    certificate-authority-data: 
    server: https://10.0.9.112:6443
  name: kubernetes
# 上下文
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
# 当前上下文
current-context: kubernetes-admin@kubernetes

#
kind: Config
preferences: {}

# 客户端认证
users:
- name: kubernetes-admin
  user:
    client-certificate-data: 
    client-key-data: 
```

