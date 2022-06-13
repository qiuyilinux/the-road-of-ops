### 1 启动参数详解

```
•--kubeconfig：连接apiserver配置文件
•--leader-elect：当该组件启动多个时，自动选举（HA）
```



### 2 启动参数实例

```
cat > /opt/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--leader-elect \\
--kubeconfig=/opt/kubernetes/cfg/kube-scheduler.kubeconfig \\
--bind-address=127.0.0.1"
EOF
```

