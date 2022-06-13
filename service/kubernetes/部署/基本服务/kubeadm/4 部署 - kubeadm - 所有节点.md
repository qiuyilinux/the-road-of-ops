### 1 添加阿里云 yum 软件源

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```



### 2 安装 kubeadm 、kubelet 和 kubectl

```shell
yum install -y kubelet-1.21.0 kubeadm-1.21.0 kubectl-1.21.0
systemctl enable kubelet # 因为现在没有配置文件无法启动，所以只需要设置开机自启，稍后kubeadm部署完成会自动拉起。
```

