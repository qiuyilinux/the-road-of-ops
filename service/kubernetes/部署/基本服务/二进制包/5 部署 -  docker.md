> 所有节点执行以下步骤



### 1 安装

```shell
mkdir -p /tmp/kubernetes/component/docker
cd /tmp/kubernetes/component/docker
wget https://download.docker.com/linux/static/stable/x86_64/docker-19.03.9.tgz
tar zxvf docker-19.03.9.tgz
cp ./docker/* /usr/bin/
```



### 2 systemd 管理docker

```shell
cat > /usr/lib/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
EOF
```



### 3 创建配置文件

```shell
mkdir /etc/docker
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```



### 4 开机自启

```shell
systemctl daemon-reload
systemctl start docker
systemctl enable docker
```

