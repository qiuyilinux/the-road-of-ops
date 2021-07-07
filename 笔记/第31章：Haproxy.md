# 常见的负载均衡



### DNS轮询解析

​	优点：dns轮询的配置是比较方便的，如果网络扩增，新增的web服务器只要增加一个公网IP即可。

​	缺点：

1. ​	如果某台服务器宕机或者更改IP地址，DNS服务器无法自动获知，仍旧会将访问分配到此服务器或者更新之前的IP地址。修改DNS记录全部生效一般要1-3小时，甚至更久。超出管理员的管理范围。会出现间歇的问题
2. 如果几台web服务器之间的硬件配置不同，导致能够承受的压力是不同的，但是DNS轮询解析目前不能很好的按权重进行分配。

### Nginx | Apache

​	应用层的负载均衡（nginx现在可以作4层的负载均衡  1.9版本+ 引入模块ngx_stream_core_module默认该模块是没有编译的，需要用到编译需添加--with-stream配置参数）

### LVS

​	详见第32章。

### Haproxy

​	是法国人willy Tarreau开发的一个开源软件，是一款应对客户端10000一闪改的并发的高性能TCP和HTTP负载均衡器。



### 四层负载均衡与七层负载均衡的对比

​	七层的负载均衡效率没有四层负载高（且四层可以跨平台）

​	四层负载均衡没有七层负载均衡支持的功能多，比如 url匹配 设置头部信息等（手机 电脑）

​	负载均衡是基于代理实现的一种形式。

# Haproxy

​	Haproxy提供高可用性，负载均衡以及基于TCP和HTTp应用的代理，支持虚拟主机，他是免费的，快速并且可靠的一种解决方案。Haproxy特别适用于那些负载特大的web站点，这些站点通常又需要会话保持或者七层处理。Haproxy运行在当前的硬件上完全可以支持数以万计的并发连接，并且他的运行模式使得他可以很简单安全的整合进您当前的架构中，同时可以保护你的web服务器不被暴露到网络上。

​	Haproxy实现了一种事件驱动，单一进程模型，此模型支持非常大的并发链接数。多进程或者多线程模型受内存限制，系统调度器限制以及无所不在的锁限制，很少能处理数千并发连接。

ps ： 什么是会话保持？

​	会话保持是指在负载均衡上的一种机制，可以识别客户端与服务器之间交互过程的关联性，在做负载均衡的同时还保证了一系列相关联的访问请求都会分配到同一台机器上。用人话来表述就是：再一次会话过程中发起的多个请求都会落到同一台机器上。

ps：会话（session）与连接（connection）之间的却别

​	连接：我们都知道TCP/IP协议中经常提到的“三次握手，四次挥手”的问题。自然也知道客户端 和服务器端是经过三次握手以后建立了连接。当他们建立了连接之后，那么客户端就可以向服务端多次发送请求。如果客户端和服务端需要断开连接，那么就需要经过四次挥手过程才能断开连接。

​	会话：如果用户需要登陆，那么可以裂解为经过三次握手以后，客户端与服务器端就是会话（session）。如果用户不需要登陆那么可以理解为经过三次握手以后，客户端与服务器端建立的就是连接。

​	连接（会话）删除：一般来说，所有厂商都会对负载均衡服务器的最大并发连接数量有限制，每一个连接都是有事件限制的。如果在规定的时间内某个连接不再有请求，那么这个连接就会被清除掉。删除了连接以后，客户端的请求将无法发送到同一个后端服务器，这时候就需要遵循负载均衡器的流量分发策略。换句话说，所谓的会话保持机制，就是将客户端与服务器端之间建立了多个连接，都发送到相同的服务器进行处理。如果客户端和和服务端部署了负载均衡设备，那么可能出现的一种情况就是这多个连接坑会被转发到不同的服务器尽行处理。这样就可能出现重复登陆购物车信息不准确的现象。

​	现通过用户来访问负载均衡器 即把他记录到一个表里面等下次来自这个IP的请求的时候继续请求 第一次记录的后台成员服务器的地址。会话保持的缺点就是当用户通过代理服务器访问的时候会出现单台后台成员服务器负载过高的情况出现。如果后台成员服务器宕机，那么这台服务器的session丢失，被分配到这台服务器的请求的用户还是需要重新登录。

​	这种信息不同步的现象有三种解决方案

1. 会话保持（nginx（ip_hash url_hash） haproxy (源地址hash 使用cookie进行识别)lvs）

2. 会话复制（tomcat）

   他是基于IP组播来完成session的复制，tomcat的会话复制分为两种：

   全局会话复制：利用delta manager 复制会话中的变更信息到期群众的所有其他节点

   非全局复制：使用backuo manager进行复制他会把session复制给指定的一个备份节点。

   缺点为不适合大规模的生产集群，超过6个节点可能会出现各种问题。

3. 会话共享（Mencached Redis）

   ```shell
       session.save_handler = memcache
       session.save_path = "tcp://192.168.56.11:11211"
       （php） 默认将会话放到tmp目录下
   ```

   # haproxy配置文件

| 配置                      | 含义                                                 |
| ------------------------- | ---------------------------------------------------- |
| chroot<jail dir>          | 将工作目录切换到<jail dir>并执行chroot               |
| daemon                    | 后台工作模式                                         |
| uid                       | 进程账户id，建议设置为haproxy专用账户                |
| gid                       | 进程组id，建议设置为haproxy专用组                    |
| log<address><facility>    | 配置全局syslog，可以设置两台日志服务器               |
| nbproc<number>            | 指定后台进程数量                                     |
| pidfile<file>             | 指定pid文件                                          |
| ulimit-n<number>          | 设置每个进程最大文件描述符数量                       |
| maxconn<number>           | 每个进程支持的最大并发数                             |
| tune.bufsize<number>      | 设置buffer大小，默认16384B                           |
| mode                      | 可选tcp、http、health                                |
| timeout check<timeout>    | 设置检查超时时间                                     |
| contimeout<timeout>       | 设置连接超时时间                                     |
| balance roundrobin        | 设置轮询负载                                         |
| bind<address>:port        | 定义一个或者多个监听地址和端口                       |
| stats auth admin:admin    | 设置监控界面的用户名和密码                           |
| stats refresh<number>     | 统计页面刷新间隔时间                                 |
| option httplog            | 使用http日志                                         |
| cookie<name>              | 启用cookie的保持连接功能                             |
| option forwardfor         | 允许插入这种数据包头，可以让后端服务器获取客户端ip   |
| option abortonclose       | 负载高时，自动关闭处理时间长的请求                   |
| option allbackups         | 后端服务器宕机，是否激活全部备机，默认启动第一个备机 |
| option dontlognull        | 不记录空连接日志，主要用于不记录健康检查日志         |
| option redispatch         | 后端某个机器宕机，强制把请求转发给健康机器           |
| monitor-uri<URi>          | 检查uri文件是否存在，依次判断主机的健康状态          |
| monitor-fail if site_dead | 服务器宕机时，返回503代码                            |
| option httpchk<uri>       | 使用http协议检查服务器健康状态                       |
| retries<value>            | 服务器连接失败后的重试次数                           |
| timeout client            | 客户端最大超时时间，单位毫秒                         |
| timeout server            | 服务器最大超时时间，单位毫秒                         |
| timeout connect           | 最大连接超时时间，单位毫秒                           |
| default_backend           | 默认后端服务器组                                     |
| use_backend               | 当条件满足时，指定后端服务器组                       |
| acl<name><criterion>      | 定义访问控制列表                                     |

# 基本部署

```shell
yum install -y gcc systemd-devel openssl*  #安装编译工具等软件包
useradd haproxy -s /sbin/nologin -M        #创建haproxy用户nglogin 不创建家目录
https://www.haproxy.org/                   #haproxy源码包下载网址

wget http://download.openpkg.org/components/cache/haproxy/haproxy-1.8.10.tar.gz                              #下载源码包
tar xvf haproxy-1.8.10.tar.gz              #解压
cd haproxy-1.8.10                          #跳转目录
make TARGET=linux3110 PREFIX=/usr/local/haproxy  #指定路径编译
make install PREFIX=/usr/local/haproxy           #指定路径安装
cp -rf /usr/local/haproxy/sbin/haproxy /usr/sbin/#复制haproxy命令到/bin下
ldd /usr/bin/haproxy                             #查看命令使用的模块
cp /opt/haproxy-2.1.0/examples/haproxy.init /etc/init.d/haproxy  #修改复制并修改启动文件（指定配置文件路径 mkdir /etc/haproxy && touch /etc/haproxy/haproxy.cfg）

```

# 修改配置

```shell
vi /etc/haproxy/haproxy.cfg 

global
    log         127.0.0.1 local2 info
    chroot      /usr/local/haproxy
    pidfile     /usr/local/haproxy/haproxy.pid
    maxconn     455350
    user        haproxy
    group       haproxy
    daemon
defaults
    mode               http
    log                global
    option             httplog
    timeout connect    10s
    timeout client     30s
    timeout server     30s
frontend http-in
    bind *:80
    default_backend    backend_servers
    option             forwardfor
    option             httpclose
backend backend_servers
    balance            roundrobin
server web1 192.168.0.57:80 cookie 1 check inter 5000 fall 3 rise 2 weight 1
server web2 192.168.0.58:80 cookie 2 check inter 5000 fall 3 rise 2 weight 1

listen stats    
  mode http    
  bind *:10000    
  stats enable    
  stats uri /haproxy    
  stats realm HAProxy\ Statistics    
  stats auth admin:123.com


访问测试 ip 测试（windows复制到里linux可能会存在格式问题可使用dos2unix转换）
监控页面 ip:10000/haparoxy测试
```

# 配置文件详解

```shell

根据功能通途不同，其配置文件主要由五个部分组成，分别为global部分，defaults部分，rontend部分，backend部分，listen部分。
1）global部分
用于设置全局配置参数，属于进程级别的配置，通常与操作系统配置无关。
2）defaults部分
默认参数的配置部分。在这部分设置的参数，默认会自动引用到下面的frontend，backend和listen部分
3）frontend部分
用于设置接收用户请求的前端虚拟节点。frontend可以根据ACL规则直接指定要使用的后端backend
4）backend部分
用于设置集群后端服务集群的配置，也就是用来添加一组真实的服务器，以处理用户的请求应。
5）listen
此部分时frontend和backend部分的结合体

配置项说明

1）
	global
		log 127.0.0.1 local1 info
		maxconn 4096
		user nobody
		group nobody
		daemon
		nbproc 1
		pidfile /var/run/haproxy.pig
	log  #全局日志配置，local0是日志设备，info表示日志级别，其中日志级别有err warning info debug 4种。这个配置表示使用127.0.0.1上的rsyslog服务器中的loacl日志设备，记录日志等级为info
	maxconn #设置每个haproxy进程可接受的最大并发连接数量
	nbproc  #设置haproxy启动时可创建的进程数量，此参数要求将haproxy运行模式设置为daemon，默认之启动一个进程，建议该值小于CPU核数
	daemon  #设置haproxy进程进入后台运行，这是推荐的运行模式
	user/group #设置启动haproxy进程的用户和组
	pidfile    #指定haproxu进程id文件存放路径

2）
	defaults
		mode http
		retires 3
		timeout connect 10s
		timeout client 20s
		timeout server 30s
		timeout check 5s
	mode  #这是haproxy实例默认的运行模式，有tcp http health三个可选值 
	tcp模式：在此模式下。客户端和服务器间建立一个全双工的连接，不会对七层报文做任何检查，为默认模式，经常用于ssl ssh smtp 等应用
	http  #在此模式下，客户端请求再转发至后端服务器前将会被深度分析，所有不与RFC格式兼容的请求都会拒绝
	health #目前已被移除
	retires #设置连接后端服务器的重试次数，如果连接失败的次数超过该数值，haproxyu将会将对应的后端服务器标记为不可用。
	timeout connect   #设置成功连接到一台服务器的最长等待事件，默认单位为毫秒，但也可以使用其他的时间单位作为后缀
	timeout client    #设置连接客户端发送数据时最长等待事件，默认单位为毫秒，也可以使用其他的时间单位作为后缀。
	timeout server    #设置服务器端回应客户端数据发送的最长等待时间，默认单位为毫秒，但是也可以使用其他的时间作为后缀。
	timeout check     #设置对后端服务器的检测超时时间，默认单位为毫秒，但是也可以使用其他的时间作为后缀
	
	
3）frontend部分
	
	frontend www
		bind*:80
		mode http
		option httplog
		option forwardfor
		option httpclose
		log global
		default_backend htmpool
	#通过frontend关键字定义了一个名为“www”的前端虚拟节点
	bind  #此选项用于定义一个或者几个监听的套接字，只能在frontend和listen种定义格式如下：bind [<address>:[port_range]] [interface]
	option httplog  #默认情况下haproxy日志是不记录http请求的，此选项的作用是启用日志记录http请求。
	option fotwardfor #此选项的作用是保证后端服务器可以记录客户端的真实IP
	option httpclose  #此选项表示客户端和服务器完成一次连接请求之后，haprox将主动关闭此tcp连接，这是对性能非常有帮助的一个参数。
	log global               #表示使用global段种的定义日志的格式
	defalut_backend htmpool  #此选项用于指定后端默认的服务器池
	
4）backend部分
	backend htmpool
		mode http
		option redispatch
		option abortonclose
		balance roundrobin
		cookie SERVERID
		option httpchk GET /index.php
		server web1 192.168.0.57:80 cookie 1 check inter 5000 fall 3 rise 2 weight 1
		server web2 192.168.0.58:80 cookie 2 check inter 5000 fall 3 rise 2 weight 1
	#backend用于定义一个名称为htmpool的后端服务器组。根据需要可以定义多个
	option redispatch #此参数用于cookie保持的环境中，在默认情况下，haproxy会将其请求的换段服务器的serverid插入到cookie中，以保证会话的session持久性。而如果后端服务器出现故障，客户端cookie是不会刷新的，这就会造成无法访问。此时，如果设置此参数，就会将客户端的请求强制定向到另外一台将抗的后端服务器上，以保证服务正常
	option abortonclose #此参数可以在服务器负载较高的情况下，自动结束当前队列中处理时间较长的连接
	balance roundrobin  #指定负载均衡算法
	
	（haproxy支持的负载均衡算法
	roundrobin  #基于权重进行轮询调度的算法
	static-rr   #基于权重进行轮询调度算法，不过此算法为静态算法，在运行时调整其服务器权重不会生效
	source      #基于请求源IP算法，此算法先对请求的源IP进行HASH运算，然后将结果与后端服务器的权重总数相除后转发至某台匹配的后端服务器，这种方式可以使一个客户端IP的请求始终转发到某特地的后端服务器
	leastconn   #此算法会将新的连接请求转发至具有最少连接数的后端服务器。在会话时间较长的场景中推荐此算法，例如数据库负载均衡
	uri         #此算法会对部分或整个URI进行HASH运算，再经过服务器的总权重相除，最后转发到某台匹配的后端服务器上，使用与后端缓存服务器
	uri_param   #此算法会根据URL路径中的参数，这样可保证后端真是服务器数据不变时，同一个用户的请求始终分发到同一台服务器上
	hdr         #此算法根据HTTP头进行转发，如果指定的HTTP头名称不存在，则使用roundrobin算法进行策略转发
	cookie SERVERID #表示允许向cookie插入SERVERID，每台服务器的SERVERID可在下面的server关键字中使用cookie关键字定义
	option httpchk  #此选项表示启用HTTP的服务状态检测功能）
	
	version     #表示心跳检测时HTTP的版本号
	server web1 192.168.0.57:80 cookie 1 check inter 5000 fall 3 rise 2 weight 1   #server 用于定义后端真实服务器，不能用于frontend和listen端格式如下：server <name> <address>:[port] [param*]  name        为换段真实服务器指定一个内部名称（自定义）  address:port指定后端服务器的IP地址及端口  param*参数 （常用的参数：check 表示启用对此后端服务器执行健康状态检查  inter设置健康状态检查的间隔时间，单位为毫秒 rise检查多少次认为服务器可用 fall检查多少次认为服务器不可用   weight设置服务器的权重，默认时1，最大为256.设置为0表示不参与负载均衡  backup设置备份服务器，用于所有后端服务器全部不可用时 cookie为指定的后端服务器设置cookie值，此处指定的值将在请求入站的时候被检查，第一次为此值挑选的后端服务器将在后续的请求中一直被选中，其目的再于实现持久连接的功能）
	
5）listen部分

	listen admin_status
		bind 0.0.0.0:9188
		mode http
		log 127.0.0.1 local err
		stats refresh 30s
		stats uri /haproxy-status
		stats realm Welcom login
		stats auth admin:admin
		stats hide-version
		stats admin if TRUE
	#listen部分用于设置haproxy监控页面相关的参数
	stats refresh 30s          #设置haproxy监控统计界面的自动刷新时间
	stats uri /haproxy-status  #设置haproxy监控页面访问的uri路径
	stats realm Welcome login  #设置登陆监控页面时，密码框上的提示信息
	stats auth admin:admin     #设置登陆监控页面的用户名，密码。用户密码用冒号隔开，可以设置多个，每行一个
	stats hide-version         #设置再监控页面上隐藏haproxy的版本号
	stats admin if TRUE        #设置此选项，可在监控页面上启用，禁用后端服务器，尽在1.4.9版本以后生效
```

# TLS加密代理

```shell
1.cd /etc/pki/tls/certs   #跳转到指定路径
openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout name.key -out name.crt   #按照配置文件分别生成证书的和私钥
cat name.key name.crt |tee server.pem             #整合证书

2.vi /etc/haproxy/haproxy.cfg
...
	daemon
	maxsslconn 65535               #定义ssl最大连接数
	tune.ssl.default-dh-param 2048 #ssl加密长度为2048
defaults
...
frontend http-in
	bind*:80
	bind*:443 ssl crt /etc/pki/tls/certs/server.pem  #定义证书路径与名称
	redirect scheme https if !{ ssl_fc }             #添加redirect规则
	将http重定向到https
default_backend backend_servers
...

```

# TCP穿透HTTPS

​	我们假设在两个不同的局域网后面分别由2台客户机A和B，AB所在的局域网都分别通过一个路由器接入互联网。互联网上有一台服务器S。

​	现在AB是无法直接和对方发送消息的，AB都不知道对方在互联网上的真正的IP和端口，AB所在的局域网的路由器只允许内部向外部主动发送的消息通过。对于B直接发送给A的路由器的消息，路由会认为其“不被信任”而直接丢弃。

​	要实现AB直接的通讯，就必须进行以下3步：A首先连接互联网上的服务器S并发送一条消息（对于UDP这种无连接的协议其实直接初始会话发送消息即可），这样S就获取了A在互联网上的实际终端（发送消息的IP和端口号 这是由内网地址在路由器上通过NAT映射而成的一个地址）。接着 B也进行同样的步骤，S就知道了AB在互联网上的终端（这就是“打洞”）。接着S分别告诉A和B对方的客户端在互联网上的实际终端，这样，在AB都知道对方的实际终端后，在使用UDP协议的时候就可以直接通过实际终端发送消息了。

​	为什么TCP不可以呢？因为TCP为状态协议，在连接过程中其他人不能插手，而且当TCP在断开连接后会有一段时间的保护期，不让这个端口进行下一次连接。而NAT为了合理的利用资源，当某一个映射在一段时间内没有发生数据交互，NAT就会认为这个地址没有人使用了，就会将这个映射销毁，回收端口号，这个时间叫老化时间。也就是说我们只要在老化时间没结束之前，连接映射出来的端口，就可以实现穿透。SOCKET编程中允许有一些特殊的选项，其中有一个叫SO_REUSEADDR的选项。它可以使TCP端口释放后立即就可以被再次使用，这就完美实现了TCP穿透（点到点连接）。

![image-20191211160115917](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191211160115917.png)

```shell
1.生成证书（证书保持一致）
cd /etc/pki/tls/certs                             #跳转到指定路径
openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout name.key -out name.crt   #按照配置文件分别生成证书的和私钥
cat name.key name.crt |tee server.pem             #整合证书（复制证书到apache端）

2.vi /etc/haproxy/haproxy.cfg
...
    bind *:443
#   bind *:443 ssl crt /etc/pki/tls/certs/gong.pem
#   redirect scheme https if !{ ssl_fc }
    default_backend    backend_servers
...
defaults
    mode               tcp
    log                global
...
server web1 192.168.1.101:443 cookie 1 check inter 5000 fall 3 rise 2 weight 1
server web2 192.168.1.102:443 cookie 2 check inter 5000 fall 3 rise 2 weight 1
...

3.访问测试

ps：操作为直接代理443端口
```

# haproxy日志转储

```shell
1.vi /etc/rsyslog.conf
# Provides UDP syslog reception
$ModLoad imudp      #允许UDP协议接收514端口转发过来的日志
$UDPServerRun 514
$AllowedSender UDP, 127.0.0.1   #允许发送者为UDP 127.0.0.1（允许自己）
...
*.info;mail.none;authpriv.none;cron.none;local2.none    /var/log/messages

loacl2.*                                                /var/log/haproxy.log#(local2与haproxy日志global段指定的日志设备保持一致)
...

2.systemctl restart rsyslog
systenctl restart haproxy
tailf /var/log/haproxy.log

ps:设置 apache 的日志格式
LogFormat "\"%{X-Forwarded-For}i\" %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

# 数据库负载均衡

```shell
1.yum instlal -y mariadb.x86_64 mariadb-devel.x86_64 mariadb-server.x86_64
systemctl start mariadb   #启动服务
mysqladmin -u root -p     #设置用户密码
mysql -u root -p123       #登陆交互模式
CREATE USER 'aaa'@'127.0.0.1' IDENTIFIED BY '123';     #建立用户apache
GRANT ALL PRIVILEGES ON lamp.* TO 'aaa'@'%' IDENTIFIED BY '123' WITH      GRANT OPTION;             #允许apache用户使用123密码 访问lamp下的所有表
FLUSH PRIVILEGES;         #重建授权表

ps ： mysql 3306 oracle 1521 sqlite  1433

2.vi /etc/haproxy/haproxy.cfg
...
defaults
    mode               tcp
...
frontend mysql-in
    bind *:3306
...
server db01 192.168.1.155:3306 cookie 1 check inter 5000 fall 3 rise 2 weight 1
server db02 192.168.1.156:3306 cookie 2 check inter 5000 fall 3 rise 2 weight 1
...

3.测试
mysql -u aaa -p123  -h 192.168.1.21  -e "show databases;" 
```

# 动静分离URL访问控制

```shell
1.

vi /etc/haproxy/haproxy.cfg
...
default_backend  dynamic              #默认backend为dynamic
  acl url_static   path_end -i .jpg   #访问控制列表, 匹配结尾为.jpg的资源
  use_backend  static  if url_static  #如果结尾为.jpg, 则使用backend为static
backend dynamic
  balance  roundrobin  #负载均衡算法roundrobin
  server   dynamic 172.16.1.5:80 check
  backend  static
  balance  uri         #这里使用uri算法
  server   static  172.16.1.4:80 check

2.

vi /etc/haproxy/haproxy.cfg
...
frontend web *:80   # *表示haproxy监听所有地址，监听的端口为80
   # 定义访问控制，表示以url以.css .js .html .php结尾的分别调度到哪台服务器上访问
    acl url_static       path_end       -i .css .js .html    
    acl url_dynamic      path_end       -i .php

    # usr_backend表示使用backend服务，if表示如果满足url_static这个条件就调度到这台服务器上
    use_backend        static          if url_static    
    default_backend    dynamic

backend static   # 定义调用后端的静态页面的服务器上
    server node1 192.168.27.18:80 check inter 3000 rise 2 fall 2 maxconn 5000
backend dynamic  # 定义调用后端的动态页面的服务器上
    server node2 192.168.27.19:80 check inter 3000 rise 2 fall 2 maxconn 5000  
```

