### 1 配置文件

```shell
cat > /usr/local/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/usr/local/kubernetes/logs \\
--config=/usr/local/kubernetes/cfg/kube-proxy-config.yml"
EOF
```



### 2 配置参数文件

```shell
cat > /usr/local/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /usr/local/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: kubernetes-node-1
clusterCIDR: 10.244.0.0/16
EOF
```

