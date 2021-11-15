	LVS 是 Linux Virtual Server 的简称，也就是 Linux 虚拟服务器。现在 LVS 已经是 Linux 标准内核的一部分，从 Linux2.4 内核以后，已经完全内置了 LVS 的各个功能模块，无需给内核打任何补丁，可以直接使用 LVS 提供的各种功能。

# LVS 的体系结构

![img](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eW51dQiaPwR6IDjdVmCcNA4zF7Viag55U1f5QNicpDXWSHWwWQvbajfEEPPHt4d4UD5ydvcUBGw0ibQFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

LVS 架设的服务器集群系统有三个部分组成：

(1) 最前端的负载均衡层，用 Load Balancer 表示

(2) 中间的服务器集群层，用 Server Array 表示

(3) 最底端的数据共享存储层，用 Shared Storage 表示

# LVS 负载均衡机制

​	LVS 不像 HAProxy 等七层软负载面向的是 HTTP 包，所以七层负载可以做的 URL 解析等工作，LVS 无法完成。

​	LVS 是四层负载均衡，也就是说建立在 OSI 模型的第四层——传输层之上，传输层上有我们熟悉的 TCP/UDP，LVS 支持 TCP/UDP 的负载均衡。因为 LVS 是四层负载均衡，因此它相对于其它高层负载均衡的解决办法，比如 DNS 域名轮流解析、应用层负载的调度、客户端的调度等，它的效率是非常高的。

​	所谓四层负载均衡 ，也就是主要通过报文中的目标地址和端口。七层负载均衡 ，也称为“内容交换”，也就是主要通过报文中的真正有意义的应用层内容。

​	LVS 的转发主要通过修改 IP 地址（NAT 模式，分为源地址修改 SNAT 和目标地址修改 DNAT）、修改目标 MAC（DR 模式）来实现。

# 三种模式

### 	NAT模式

NAT 模式下，网络数据报的进出都要经过 LVS 的处理。LVS 需要作为 RS（真实服务器）的网关。

当包到达 LVS 时，LVS 做目标地址转换（DNAT），将目标 IP 改为 RS 的 IP。RS 接收到包以后，仿佛是客户端直接发给它的一样。RS 处理完，返回响应时，源 IP 是 RS IP，目标 IP 是客户端的 IP。这时 RS 的包通过网关（LVS）中转，LVS 会做源地址转换（SNAT），将包的源地址改为 VIP，这样，这个包对客户端看起来就仿佛是 LVS 直接返回给它的。

![img](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eW51dQiaPwR6IDjdVmCcNA4zH5a5hdDapib8tL7alibn1pol3FtnVNxUdaJXyrwYwPQVxKibiczLhkeL8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 这种模式借助iptables的nat表来实现
- 用户的请求到分发器后，通过预设的iptables规则，把请求的数据包转发到后端的rs上去
- rs需要设定网关为分发器的内网ip
- 用户请求的数据包和返回给用户的数据包全部经过分发器，所以分发器成为瓶颈
- 在nat模式中，只需要分发器有公网ip即可，所以比较节省公网ip资源

### 	隧道模式

​	所谓隧道，实际上是[路由器](https://baike.baidu.com/item/路由器)把一种网络层协议封装到另一个协议中以跨过网络传送到另一个路由器的处理过程。发送路由器将被传送的协议包进行封装，经过网络传送，接受路由器解开收到的包，取出原始协议；而在传输过程中的中间路由器并不在意封装的协议是什么。这里的封装协议，称之为[传输协议](https://baike.baidu.com/item/传输协议)，是跨过网络传输被封装协议的一种协议，IP协议是IOS唯一选择的传输协议。而被封装的协议在此为IPX协议或者AppleTAlk协议，通常可以称之为乘客协议。需要特别注意的是：隧道技术是一种点对点的链接，因而必须在链接的两端配置隧道协议。

​	我们假设在站点A和B之间交换数据。在IP协议下，[数据包](https://baike.baidu.com/item/数据包)在路由器之间的传递直到到达目的地的过程，其线路是没有经过预先的设计和计划。然而在MPLS（多协议标记交换）协议下，在站点A和B之间传递的IP数据包必须沿由第一个[路由器](https://baike.baidu.com/item/路由器)预先建立起来的通路传送。这条通路在IP网络中就像一条中空的隧道，直接连接A和B两个站点。

​	它和NAT模式不同的是，它在LB和RS之间的传输不用改写IP地址（添加新的IP头）。而是把客户请求包封装在一个IP  tunnel里面，然后发送给RS节点服务器，节点服务器接收到之后解开IP   tunnel后，进行响应处理。并且直接把包通过自己的外网地址发送给客户不用经过LB服务器。IP隧道技术主要用于移动主机和虚拟私有网络（Virtual Private Network），在其中隧道都是静态建立的，隧道一端有一IP地址，令一端也有唯一的ip地址。

​	简单来说他是通过隧道模式把请求，分发到成员服务器。由于成员服务器上面有rip -> vip的路由（vip同时存在DS RS），所以直接返还给客户端，客户端依然能接受。

流程

1、 当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。此时报文的源IP为CIP，目标IP为VIP 。
2、 PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链。
3、IPVS比对数据包请求的服务是否为集群服务，若是，在请求报文的首部再次封装一层IP报文，封装源IP为为DIP，目标IP为RIP。然后发至POSTROUTING链。此时源IP为DIP，目标IP为RIP。
4、 POSTROUTING链根据最新封装的IP报文，将数据包发至RS（因为在外层封装多了一层IP首部，所以可以理解为此时通过隧道传输）。此时源IP为DIP，目标IP为RIP
5、RS接收到报文后发现是自己的IP地址，就将报文接收下来，拆除掉最外层的IP后，会发现里面还有一层IP首部，而且目标是自己的lo接口VIP，那么此时RS开始处理此请求，处理完成之后，通过lo接口送给eth0网卡，然后向外传递。此时的源IP地址为VIP，目标IP为CIP
6、 响应报文最终送达至客户端

注：在真实的环境下 RIP  DIP  VIP 都可以是公网IP so 隧道模式可以实现跨公网代理。

​	![image-20200108111200313](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20200108111200313.png)

### 	DR模式：直接路由

​	DR 模式下需要 LVS 和 RS 集群绑定同一个 VIP（RS 通过将 VIP 绑定在 loopback 实现），但与 NAT 的不同点在于：请求由 LVS 接受，由真实提供服务的服务器（RealServer，RS）直接返回给用户，返回的时候不经过 LVS。

​	详细来看，一个请求过来时，LVS 只需要将网络帧的 MAC 地址修改为某一台 RS 的 MAC，该包就会被转发到相应的 RS 处理，注意此时的源 IP 和目标 IP 都没变，LVS 只是做了一下移花接木。RS 收到 LVS 转发来的包时，链路层发现 MAC 是自己的，到上面的网络层，发现 IP 也是自己的，于是这个包被合法地接受，RS 感知不到前面有 LVS 的存在。而当 RS 返回响应时，只要直接向源 IP（即用户的 IP）返回即可，不再经过 LVS。

![img](https://mmbiz.qpic.cn/mmbiz_png/yNKv1P4Q9eW51dQiaPwR6IDjdVmCcNA4zemibr7dnicp6CicAuv3T8McIZXucfa5gice3uNEdxl9wK65UDHlZYZ8icJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​	DR 负载均衡模式数据分发过程中不修改 IP 地址，只修改 mac 地址，由于实际处理请求的真实物理 IP 地址和数据请求目的 IP 地址一致，所以不需要通过负载均衡服务器进行地址转换，可将响应数据包直接返回给用户浏览器，避免负载均衡服务器网卡带宽成为瓶颈。因此，DR 模式具有较好的性能，也是目前大型网站使用最广泛的一种负载均衡手段。

# LVS术语

1. DS：Director Server。指的是前端负载均衡器节点。
2. RS：Real Server。后端真实的工作服务器。
3. VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址。
4. DIP：Director Server IP，主要用于和内部主机通讯的IP地址。
5. RIP：Real Server IP，后端服务器的IP地址。6. CIP：Client IP，访问客户端的IP地址。

# 十种算法

1.静态算法（4种）：只根据算法进行调度 而不考虑后端服务器的实际连接情况和负载情况
①.RR：轮叫调度（Round Robin）
　 调度器通过”轮叫”调度算法将外部请求按顺序轮流分配到集群中的真实服务器上，它均等地对待每一台服务器，而不考虑服务器上实际的连接数和系统负载｡
②.WRR：加权轮叫（Weight RR）
　 调度器通过“加权轮叫”调度算法根据真实服务器的不同处理能力来调度访问请求。这样可以保证处理能力强的服务器处理更多的访问流量。调度器可以自动问询真实服务器的负载情况,并动态地调整其权值。
③.DH：目标地址散列调度（Destination Hash ）
　 根据请求的目标IP地址，作为散列键(HashKey)从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空。
④.SH：源地址 hash（Source Hash）
　 源地址散列”调度算法根据请求的源IP地址，作为散列键(HashKey)从静态分配的散列表找出对应的服务器，若该服务器是可用的且未超载，将请求发送到该服务器，否则返回空｡

动态算法（6种）：前端的调度器会根据后端真实服务器的实际连接情况来分配请求

①.LC：最少链接（Least Connections）
　 调度器通过”最少连接”调度算法动态地将网络请求调度到已建立的链接数最少的服务器上。如果集群系统的真实服务器具有相近的系统性能，采用”最小连接”调度算法可以较好地均衡负载。

②.WLC：加权最少连接(默认采用的就是这种)（Weighted Least Connections）
　 在集群系统中的服务器性能差异较大的情况下，调度器采用“加权最少链接”调度算法优化负载均衡性能，具有较高权值的服务器将承受较大比例的活动连接负载｡调度器可以自动问询真实服务器的负载情况,并动态地调整其权值。

③.SED：最短延迟调度（Shortest Expected Delay ）
　 在WLC基础上改进，Overhead = （ACTIVE+1）*256/加权，不再考虑非活动状态，把当前处于活动状态的数目+1来实现，数目最小的，接受下次请求，+1的目的是为了考虑加权的时候，非活动连接过多缺陷：当权限过大的时候，会倒置空闲服务器一直处于无连接状态。

④.NQ永不排队/最少队列调度（Never Queue Scheduling NQ）
　 无需队列。如果有台 realserver的连接数＝0就直接分配过去，不需要再进行sed运算，保证不会有一个主机很空间。在SED基础上无论+几，第二次一定给下一个，保证不会有一个主机不会很空闲着，不考虑非活动连接，才用NQ，SED要考虑活动状态连接，对于DNS的UDP不需要考虑非活动连接，而httpd的处于保持状态的服务就需要考虑非活动连接给服务器的压力。

⑤.LBLC：基于局部性的最少链接（locality-Based Least Connections）
　 基于局部性的最少链接”调度算法是针对目标IP地址的负载均衡，目前主要用于Cache集群系统｡该算法根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器;若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则用“最少链接”的原则选出一个可用的服务器，将请求发送到该服务器｡

⑥. LBLCR：带复制的基于局部性最少连接（Locality-Based Least Connections with Replication）
　 带复制的基于局部性最少链接”调度算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统｡它与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射｡该算法根据请求的目标IP地址找出该目标IP地址对应的服务器组，按”最小连接”原则从服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器；若服务器超载，则按“最小连接”原则从这个集群中选出一台服务器，将该服务器加入到服务器组中，将请求发送到该服务器｡同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度。

# 基本部署

### 	ipvsadm

```shell
-A        添加一个虚拟服务，使用ip地址、端口号、协议来唯一定义一个虚拟服务
-E        编辑一个虚拟服务
-D        删除一个虚拟服务
-C        清空虚拟服务列表
-R        从标准输入中还原虚拟服务列表
-S        保存虚拟服务规则至标准输出，输出规则可使用-R选项还原
-L        显示虚拟服务列表
-Z        虚拟服务器列表计数器清零（清空当前连接数）
-a        添加一台真实服务器
-e        编辑一台真实服务器
-d        减少一台真实服务器
-t        使用TCP服务，该参数后需加主机与端口信息
-u        使用UDP服务，该参数后需加主机与端口信息
-s        指定lvs的调度算法
-r        设置真实服务器IP与端口
-g        设置lvs工作模式为DR直连路由
-i        设置lvs工作模式为TUN隧道
-m        设置lvs工作模式为NAT地址转换模式
-w        指定真实服务器权重
-c        连接状态，配和-L使用
-n        数字格式显示
--stats   显示统计信息
--rate    显示速率信息
--sort    对虚拟服务器和真实服务器排序输出
--set tcp tcpfin udp  设置ipvs连接超时值，三个参数分别代表tcp会话超时时间、收到FIN包后tcp会话超时时间、udp超时时间
--timeout             显示tcp tcpfin udp的timeout值
--start-daemon        启动同步守护进程
--stop-daemon         停止同步守护进程
```

### 	nat模式

```shell
1.yum install -y ipvsadm  #安装软禁包
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf  #开启ipv4转发
sysctl -p               #使其生效

2.lvs添加网卡
vip为外网卡 192.168.50.135
dip为内网卡 192.168.49.138
将RS的网关指定为dip rs1：192.168.49.136 rs2：192.168.49.137
(vip 和 dip 不能在相同子网，因为客户端请求lvs，lvs将请求负载到了rs，rs如果和客户端相同子网，会尝试直接把包返回给客户端，而客户端并没有直接向rs请求，会认为这个包不合法，从而超时。即使指定了dip也不会走网关，因为相同子网直接返回不走网关。（把lvs理解为路由器问题就好理解的多）)

3.ipvsadm -A -t 192.168.50.135:80 -s wrr    #添加一个虚拟服务
ipvsadm -a -t 192.168.50.135:80 -r 192.168.49.136:80 -m -w 1 #添加成员服务器
ipvsadm -a -t 192.168.50.135:80 -r 192.168.49.137:80 -m -w 1 #添加成员服务器
ipvsadm -L -n           #查看规则
ipvsadm --set 1 5 300   #设置tcp超时时间
ipvsadm -L --timeout    #显示tcp设置

4.ipvsadm -ln --stats   #查看分发情况
ipvsadm -ln --rate      #查看速率
ipvsadm -C              #清空规则

5.保存为服务
ipvsadm -S -n > /etc/sysconfig/ipvsadm   #保存到配置文件
ipvsadm-restore < /etc/sysconfig/ipvsadm #从标准输入中读取
systemctl restart ipvsadm                #重启服务

#其实这个服务就是保存清空与恢复三条命令的交互运行（可以去看单元文件）

ps： iptables -t nat -A POSTROUTING -s 192.168.49.0/24 -o eth0 -j SNAT --to-source 192.168.50.135  #使内网服务器可以通过外网卡上网（我是没用到）
```

### 	tun模式

```shell
cat /opt/tun.server
echo 1 > /proc/sys/net/ipv4/ip_forward
/sbin/ifconfig tunl0 up
/sbin/ifconfig tunl0 192.168.50.105   192.168.50.105 netmask 255.255.255.255 up
/sbin/route add -host 192.168.50.105 dev tunl0
/sbin/ipvsadm -C
/sbin/ipvsadm --set 1 5 300
/sbin/ipvsadm -A -t 192.168.50.105:80 -s wrr
/sbin/ipvsadm -a -t 192.168.50.105:80 -r 192.168.50.135:80 -i -w 1
/sbin/ipvsadm -a -t 192.168.50.105:80 -r 192.168.50.136:80 -i -w 1

cat /opt/tun.client
/sbin/ifconfig tunl0 up
/sbin/ifconfig tunl0 192.168.50.105 broadcast 192.168.50.105 netmask 255.255.255.255 up
echo 1 > /proc/sys/net/ipv4/conf/tunl0/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/tunl0/arp_announce
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
echo 0 > /proc/sys/net/ipv4/conf/tunl0/rp_filter
echo 0 > /proc/sys/net/ipv4/conf/all/rp_filter
/sbin/route add -host 192.168.50.105 dev tunl0
```

### DR路由模式

```shell
cat dr.client
ifconfig lo:0 192.168.50.106 broadcast 192.168.50.106 netmask 255.255.255.255 up
route add -host 192.168.50.106 dev lo:0
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce

cat dr.server
ifconfig ens33:0 192.168.50.106 broadcast 192.168.50.106 netmask 255.255.255.255 up #broadcast 广播地址
route add -host 192.168.50.106 dev ens33:0
ipvsadm -C
ipvsadm --set 1 5 300
ipvsadm -A -t 192.168.50.106:80 -s wrr
ipvsadm -a -t 192.168.50.106:80 -r 192.168.50.135 -g -w 1
ipvsadm -a -t 192.168.50.106:80 -r 192.168.50.136 -g -w 1
```

