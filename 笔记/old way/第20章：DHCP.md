## 简介：

DHCP的前身是BOOTP，它工作在OSI的应用层，是一种帮助计算机从指定的DHCP服务器获取配置信息的自举的协议。

## DHCP有三种机制分配IP地址：

1. 自动分配方式：	DHCP服务器为主机指定一个永久性的IP地址， 一旦DHCP客户端第一次成功从DHCP服务器端租用到IP地址后，就可以永久性的使用该地址。 

2. 动态分配方式：    DHCP服务器给主机指定一个具有时间限制的IP地址，时间到期或主机明确表示放弃该地址时，该地址可以被其他主机使用。 

3. 手动分配方式：     客户端的IP地址是由网络管理员指定的，DHCP服务器只是将指定的IP地址告诉客户端主机。 

   三种地址分配方式种，只有动态分配可以重复使用客户端不再需要的地址。

   DHCP消息的格式是基于BOOTP消息格式的，这就要求设备具有BOOTP的中继代理功能，并能够与BOOTP客户端和DHCP服务器实现交互。BOOTP中继代理的功能，使得没有必要在每个物理网络都部署一台DHCP服务器

## 工作原理：

1. 发现阶段    客户端去查找DHCP服务器寻求的IP地址阶段。以广播的性治发送DHCP discover信息。
2. 提供阶段    在网络中接收到DHCP discover 的DHCP服务器会进行响应，向客户端发送一个包含IP地址及其他相关信息的offer给客户端。
3. 选择阶段    客户端选择某一台DHCP服务器发送的IP地址（先到先得）只接受第一个收到的IP地址信息 广播的方式回答request信息（位的是告诉所有DHCP服务器已经获取了）
4. 确认阶段    被选择的服务器接收request向客户端发送一个ack 包含IP地址租赁时间等相关信息。
5. 重新登陆    直接发送包含前一次所分配的IP地址的DHCP request信息，当DHCP服务器收到这个信息之后，他尝试让DHCP客户端继续使用此IP地址，并回答一个DHCP ACK信息。如果此IP地址已经无法再分配给原来的DHCP客户端使用，则DHCP服务器给客户端回答一个DHCP NACK信息，客户端重新发送请求。
6. 更新租约    每一个DHCP服务器出租的IP地址一般都有一个租赁期，期满后DHCP服务器便回收该IP地址。

## 端口：

服务器端口 UDP 67

客户端端口 UDP 68

## 配置部署：

```shell
yum install -y dhcp

vi /etc/dhcp/dhcpd.conf

ddns-update-style interim;  #指定DNS更新模式
ignore client-updates;      #忽略客户端设置
subnet 192.168.1.0 netmask 255.255.255.0 {             #指定子网
        option routers                  192.168.1.1;   #网关
        option subnet-mask              255.255.255.0; #子网掩码
        option domain-name              "bosheng.com"; #dns服务器
        option domain-name-servers      192.168.1.1;   #dns服务器IP
        option time-offset              -18000;        #漂移时间
#       option ntp-servers             192.168.1.1;    #时间服务器
        range dynamic-bootp 192.168.1.20 192.168.1.100;#IP地址范围
        default-lease-time 21600;                      #默认租赁时间
        max-lease-time 43200;                          #最长租赁时间
}
host fantasia {
          hardware ethernet 00:0c:29:ec:d7:80;    #指定客户端的mac地址
          fixed-address 192.168.1.99;             #为客户端绑定IP地址
        }

客户端测试


```

## DHCP超级作用域（将不同的子网看成相同的网络）

```shell
ddns-update-style interim;

ignore client-updates;

shared-network mylab    {                              #超级作用域名称为mylab

default-lease-time 43200;

max-lease-time 86400;

subnet 192.168.0.1 netmask 255.255.255.0 {

option routers 192.168.0.1;

range dynamic-bootp 192.168.0.10 192.168.0.250;

}

subnet 192.168.1.1 netmask 255.255.255.0; {

option routers 192.168.1.1;

range dynamic-bootp 192.168.1.10 192.168.1.250;

}

}

```

## DHCP中继代理

DHCP客户使用IP广播来寻找同一网段上的DHCP服务器。当服务器和客户段处在不同网段，即被路由器分割开来时，路由器是不会转发这种广播包的。因此可能需要在每个网段上设置一个DHCP服务器，虽然DHCP只消耗很小的一部分资源的，但多个 DHCP服务器，毕竟要带来管理上的不方便。DHCP中继的使用使得一个DHCP服务器同时为多个网段服务成为可能。

为了让路由器可以帮助转发广播请求数据包，使用ip help-address命令。通过使用该命令，路由器可以配置为接受广播请求，然后将其以单播方式转发到指定IP地址。

   在DHCP广播情况下，客户在本地网段广播一个 DHCP发现分组。网关获得这个分组，如果配置了帮助地址，就将DHCP分组转发到特定地址。

 ![img](http://p.ananas.chaoxing.com/star3/origin/931210c7dd4daddf87c533f333103e07.png) 

```shell
#网卡1
ifcfg-ens33
TYPE=Ethernet                 #网络类型为以太网
BOOTPROTO=none                #启动类型（none，staic，dhcp，bootp）
DEFROUTE=yes                  #是否开启路由支持
NAME=ens33                    #网卡名称
DEVICE=ens33                  #连接名称
ONBOOT=yes                    #是否自动激活
IPADDR=192.168.31.1           #IP地址
PREFIX=24                     #子网掩码
#网卡2
ifcfg-ens37
TYPE=Ethernet                 #网络类型为以太网
BOOTPROTO=none                #启动类型（none，staic，dhcp，bootp）
DEFROUTE=yes                  #是否开启路由支持
NAME=ens37                    #网卡名称
DEVICE=ens37                  #连接名称
ONBOOT=yes                    #是否自动激活
IPADDR=192.168.32.1           #IP地址
PREFIX=24                     #子网掩码
#开启IPV4转发
vi /etc/sysctl
net.ipv4.ip_forward = 1
sysctl -p
#开启中继
dhcrelay 192.168.1.100
#测试
```

