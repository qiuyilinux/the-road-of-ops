### 1 搭建 NFS 服务器

```shell
yum -y install nfs-utils
mkdir /ifs/kubernetes
cat > /etc/exports << EOF
/ifs/kubernetes 0.0.0.0/32(rw,async,no_root_squash,no_subtree_check)
EOF
systemctl start nfs
systemctl enable nfs
```

注： 需要在每个node 上都安装 nfs-utils 这个包。 否则在 pod 分配到没有 nfs 挂载类型的节点上，会出现挂在错误。



### 2 通过 yaml 部署 nfs

> yaml 文件见 /yaml/nfs/*
>
> 上传 yaml 文件到任意目录

```yaml
# deployment
# 修改如下地址和路径
...

              value: 10.0.9.112 
            - name: NFS_PATH
              value: /ifs/kubernetes
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.0.9.112
            path: /ifs/kubernetes
```

部署

```shell
kubectl apply -f .
```

