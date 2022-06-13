## 一、 环境准备

### 1 安装 go

```shell
cd /usr/local
wget https://go.dev/dl/go1.18.1.linux-amd64.tar.gz
tar zxvf go1.18.1.linux-amd64.tar.gz

vi /etc/profile
...
export GO_ROOM=/usr/local/go
export GO_PATH=$GO_ROOM/bin
export PATH=$PATH:$GO_PATH
```

### 2 安装依赖包

```shell
yum install -y wget pcre-devel gcc zlib-devel
```

## 二、 安装

#### 创建组和用户
```bash
    groupadd -g 2210 zabbix
    useradd -u 2210 zabbix -g zabbix
```


#### 创建目录及赋权

```bash
    [ ! -d /data/scripts/oss/zabbix ] && mkdir -p /data/scripts/oss/zabbix
    [ ! -d /data/logs/oss/zabbix ] && mkdir -p /data/logs/oss/zabbix
    [ ! -d /data/config/oss/zabbix ] && mkdir -p /data/config/oss/zabbix
    [ ! -d /data/run/oss/zabbix ] && mkdir -p /data/run/oss/zabbix
    
    chown zabbix:zabbix /data/config/oss/zabbix/* -Rf
    chown zabbix:zabbix /data/logs/oss/zabbix/* -Rf
    chown zabbix:zabbix /data/scripts/oss/zabbix/* -Rf
    chown zabbix:zabbix /data/run/oss/zabbix/* -Rf
    
    chown zabbix: -R /data/logs/oss/zabbix/
    chown zabbix: -R /data/run/oss/zabbix
```

#### 下载文件:
```bash
	cd /tmp
	wget https://cdn.zabbix.com/zabbix/sources/stable/5.4/zabbix-5.4.1.tar.gz
	tar zxvf zabbix-5.4.1.tar.gz
	cd zabbix-5.4.1
```

#### 编译安装agentd

```bash
    ./configure --prefix=/usr/local/zabbix-5.4.1 --enable-agent 
    make && make install 
```
#### 编译安装agent2
```bash
    yum install -y wget pcre-devel gcc zlib-devel
    ./configure --prefix=/usr/local/zabbix-5.4.1 --enable-agent2
    make && make install 
```

#### 目录软件agent2

```bash
	ln -s /usr/local/zabbix-5.4.1 /usr/local/zabbix
    ln -s /data/scripts/oss/zabbix /usr/local/zabbix/scripts
    ln -s /usr/local/zabbix/sbin/zabbix_agent2 /usr/local/sbin/zabbix_agent2
    
    cd /usr/local/zabbix
    mv etc/* /data/config/oss/zabbix/
    ln -s /data/config/oss/zabbix/ etc

```

#### 目录软件 agentd

```bash
	ln -s /usr/local/zabbix-5.4.1 /usr/local/zabbix
    ln -s /data/scripts/oss/zabbix /usr/local/zabbix/scripts
    ln -s /usr/local/zabbix/sbin/zabbix_agentd /usr/local/sbin/zabbix_agentd
    
    cd /usr/local/zabbix
    mv etc/* /data/config/oss/zabbix/
    ln -s /data/config/oss/zabbix/ etc

```

#### agentd 配置文件
```bash
    配置启动文件:/data/config/oss/zabbix/zabbix_agentd.conf
    	PidFile=/data/run/oss/zabbix/zabbix_agentd.pid
    	LogFile=/data/logs/oss/zabbix/zabbix_agentd.log
    	AllowKey=system.run[*]
    	ListenPort=31350   //监听端口
    	Hostname=glb-ubuntu  //被监控主机短名称呢个
    	BufferSize=1000
    	Timeout=3   // agentd端执行任务的超时时间
    	Include=/data/config/oss/zabbix/zabbix_agentd.conf.d/*.conf
    	ListenIP= 1.1.1.1  //被监控端的IP地址
    	ServerActive=2.2.2.2:31351  //服务端IP:服务端口
    	Server=2.2.2.2   //服务端IP
```

#### agent2 配置文件
```bash
PidFile=/data/run/oss/zabbix/zabbix_agent2.pid
LogFile=/data/logs/oss/zabbix/zabbix_agent2.log
Server=10.0.2.39
ListenPort=31350
ListenIP
ServerActive=10.0.2.39:31351
AllowKey=system.run[*]
Hostname
Timeout=3
Include=/data/config/oss/zabbix/zabbix_agentd.conf.d/*.conf
ControlSocket=/data/run/oss/zabbix/agent.sock
```

#### 设置开机启动
```bash
	chkconfig zabbix_agentd on
```

#### 相关命令
```bash
    service zabbix_agentd start
    service zabbix_agentd restart
    service zabbix_agentd stop
```

