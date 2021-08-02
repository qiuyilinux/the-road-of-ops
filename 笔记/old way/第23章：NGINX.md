### 类比Nginx与Apache

1. Nginx作为web服务器，相对Apache作为web服务器 占用的内存资源更更少，抗并发能力更强，处理请求时异步非阻塞的，而Apache则是阻塞型的，即使再高并发下Nginx依然能保持低资源低消耗高性能，高读模块话的设计，编写模块相对简单，社区活跃，各种高性能模块出品迅速。

   而Apache 相对Nginx 来说rewrite 功能更强大，模块超多，功能齐全，少bug，超稳定。

   一般来说，需要性能的web服务，用nginx。如果不需要性能只求稳定，那就apache吧。Nginx的各种功能模块实现得比Apache实现的要好，例如ssl的模块，可以配置项也会多一些。epoll网络IO模型时Nginx处理性能高的根本理由，但并不是所有的情况下兜售epoll大获全胜的，如果本实提供静态服务的就只有寥寥几个文件，apache的select模型或许比epool更高性能当然这只是根据网络IO模型的原理的一个假设，真正的应用还是需要实测了之后再说的。

2. 作为Web服务器：相比Apache，Nginx使用更少的资源，支持更多的并发连接，体现更高的效率，这点使Nginx尤其收到虚拟主机提供商的欢迎。在高连接高并发的情况下，Nginx是Apache服务器不错的替代品。Nginx在美国是做虚拟主机生意的老板们经常选择的软件平台之一，能够支持高达50000个并发连接数的响应。

   Nginx作为负载均衡服务器：Nginx既可以在内部直接支持Rails和PHP程序对外进行服务，也可以支持作为HTTP代理服务器对外进行服务，Nginx采用C进行编写，不论是系统资源开销还是CPU使用效率都比 Perlabl 要好很多

   作为邮件代理服务器：Nginx同时也是一个非常优秀的邮件代理服务器（最早开发这个产品的目的之一也是作为邮件代理服务器）Last.fm描述了成功并且美妙的使用经验，Nfinx是一个安装非常的简单，配置文件非常简洁（还能够支持perl语法，bugs非常少的服务器）：Nginx启动特别容易， 并且机会可以做到7*24小时不间断运行，即使运行数个月也不需要重新启动，你还能够不间断服务的情况下进行软件版本的升级。

3. Nginx配置简介 Apache复杂

   Nginx静态处理性能比Apache高三倍以上Apache对PHP支持比较简单，Nginx需要配合其他的后端用Apache的组件比Nginx多，现在Nginx才是web服务器的首选。

4. 最核心的区别在于apache是同步多进程模型，一个连接对应一个进程；nginx是异步非阻塞的，多个连接（万级别）可以对应一个进程

5. Nginx处理静态文件好，耗费内存少，但无一Apache仍然是目前的主流，有很多丰富的特性，所以还需要搭配着来，当然如果能确定Nginx就是和需求，那么使用Nginx回事更经济的方式。

6. 从个人国王的使用情况来看，Nginx的负载能力比Apache高很多。最新的服务器也该用Nginx了，而且Nginx改完配置能-t测试以下配置是否存在问题，Apache只有在重启的时候发现配置错误，会很崩溃，改的时候都会非常小心翼翼，现在看有好多集群站，前端Nginx抗并发，后端Apache集群，配合的也不错。

7. Nginx处理动态请求时鸡肋，一般动态请求要Apache去做，Nginx知识和做静态和反向。

8. 从我个人的经验来看，Nginx是很不错的前端服务器，负载性能很好，用webbench模拟1000个静态文件请求好不吃里。Apache对Php等语言的支持很好，此外Apache有强大的支持，发展时间相对Nginx更久bug更少的，但是Apache有先天不支持多核心处理负载鸡肋的缺点，建议使用Nginx做前端，后端用Apache，大型网站建议用Nginx自带的集群功能。

9. 你对Web server的需求决定你的选择。大部分情况下Nginx都有雨Apache，你如静态文件处理，Php-CGI的支持，反向代理共嗯你，前端Cache，长连接等等，在Apache+PHP（prefork）模式下，如果PHP处理慢或者前端压力很大的情况下，很容易出现Apache进程飙升，从而拒绝服务的现象。

从以上9点可以知道怎么选择Nginx与Apache了吧

### 配置Nginx基本环境

```shell
rpm -import http://nginx.org/keys/nginx_signing.key  #下载gpgkey
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm  #安装nginx源
yum install -y nginx           #安装nginx
yum install php php-mysql php-fpm php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap curl curl-devel phpmyadmin -y  #安装php相关软件包
yum install -y mariadb-server mariadb  #安装数据库
vi /etc/php-fpm.d/www.conf
user = nginx
group = nginx
```

### 虚拟目录

（如下配置 修改自default.conf生效行）

```shell
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
#虚拟目录
    location /alias {  #虚拟目录的名字
        alias /shell;  #虚拟目录的路径
        index index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### Nginx 配合 php

```shell
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
    }

    location /alias {
        alias /web;
        index index.php;
    }

    location ~ ^/alias/.+\.php$ {    #正则匹配^/alias/的php请求
        root /web;                   #网站目录/web
        rewrite /alias/(.*\.php?) /$1 break;  #跳转给php处理
        include fastcgi_params;               #加载配置文件
        fastcgi_pass 127.0.0.1:9000;          #所有请求由本主机的127.0.0.1处理
        fastcgi_index index.php;              #主页文件
        fastcgi_param SCRIPT_FILENAME /web$fastcgi_script_name; #匹配目录下的.php文件进行处理
    }
    location ~\.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;
        include fastcgi_params;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```

### nginx配合php原理

1. nginx的 worker进程 直接管理每一个请求到nginx的网络请求。

2. 对与php而言，由于整个网络请求的过程中php是一个cgi程序的角色，所以采用名为php-fpm的进程管理程序来对这些被请求的php程序进行管理。php-fpm程序也如同nginx一样，需要监听端口，并且有master 和worker 进程。 worker 进程直接管理每一个php进程。

3. 关于fastcgi ： fastcgi是一种进程管理器，管理cgi进程。市面上有多种实现了fastcgi功能的进程管理器，php-fpm就是其中一种。再提一点，php-fpm作为一种fast-cgi进程管理服务，会见厅端口，一般默认监听9000端口，并且是监听本机，也即是只接受来在本地的端口请求。

4. 关于fastcgi的配置文件，目前fastcgi的配置文件一般放在nginx.conf同级目录下，配置文件形式，一般有两种：fastcgi.conf和fast_params 不通的nginx版本会有不同的配置文件，这两个配置文件有一个非常重要的却别：fastcgi_parames文件中缺少下列配置：

   fastcgi_param SCRIPT_FILENAME  documentroot fastcgi_script_name;

   我们可以打开fastcgi_parames文件加上上述行，也可以再要使用配置的地方添加。

5. 当需要处理php请求的 时候，nginx的worker进程会将请求移交给php-fpm的worker进程进行处理，也就是最开头所说的nginx调用了php，其实严格得讲nginx简介调用php。

### Php四种运行模式

1. CGI       只是http服务器与动态脚本语言的一个通信端口而已，每次解析请求都会读取配置文件，加载环境变量，模块等 解析结束后就退出，周而复始，相对速度较慢。

2. Fast-cgi 他会启动一个进程管理器（php-fpm） Fast-cgi进程初始化 启动多个cgi进程等待连接 当web服务器接收到需要处理php请求的时候他会将请求转发给fast-cgi ，fast-cgi进程管理器收到请求的时候选择并连接一个cgi的解释器，web服务器将cgi环境变量和标准输入发送给cgi解释器， 解释器处理完成之后将标准错误，标准输出返回给web服务器。（可以理解为fast-cgi相当于是一个常驻内存当中的cgi在很大程度上提高了服务器性能）

3. CLI        cli是php的命令行运行模式，大家经常会使用它，但是可能并没有注意到（例如：我们在linux下经常使用 "php -m"查找PHP安装了那些扩展就是PHP命令行运行模式；有兴趣的同学可以输入php -h去深入研究该运行模式） 

4. 模块模式 模块模式是以mod_php5模块的形式集成，此时mod_php5模块的作用是接收Apache传递过来的PHP文件请求，并处理这些请求，然后将处理后的结果返回给Apache。如果我们在Apache启动前在其配置文件中配置好了PHP模块（mod_php5）， PHP模块通过注册apache2的ap_hook_post_config挂钩，在Apache启动的时候启动此模块以接受PHP文件的请求。

   ​     除了这种启动时的加载方式，Apache的模块可以在运行的时候动态装载，这意味着对服务器可以进行功能扩展而不需要重新对源代码进行编译，甚至根本不需要停止服务器。我们所需要做的仅仅是给服务器发送信号HUP或者AP_SIG_GRACEFUL通知服务器重新载入模块。但是在动态加载之前，我们需要将模块编译成为动态链接库。此时的动态加载就是加载动态链接库。  Apache中对动态链接库的处理是通过模块mod_so来完成的，因此mod_so模块不能被动态加载，它只能被静态编译进Apache的核心。这意味着它是随着Apache一起启动的。

### 基于用户IP的访问控制

```shell
location / {
	root	/web;
	index index.html
	allow 127.0.0.1;  #从上到下匹配规则，匹配到合适的规则不继续向下匹配，类比case。
	deny all;
}
```

### 基于用户身份的访问控制

```shell
auth_basic "welcome!";
auth_basic_user_file /etc/nginx/conf.d/.htpasswd;

htpasswd -cm /etc/nginx/conf.d/.htpasswd aaa
```

### Nginx站点加密

```shell
cd /etc/pki/tls/certs/
make nginx.pem

vi /etc/nginx/conf.d/bosheng.conf

	listen      443 ssl;
    server_name  localhost;
    ssl_certificate /etc/pki/tls/certs/nginx.pem;      #指定证书路径
    ssl_certificate_key /etc/pki/tls/certs/nginx.pem;  #指定私钥路径

    #ssl_session_cache shared:SSL:1m;  #设置会话缓存时间
    #ssl_session_timeout 5m;           #设置会话超时时间
    #ssl_ciphers HIGH:!aNULL:!MD5;     #指定加密算法
    #ssl_prefer_server_ciphers on;     #启用加密算法
```

### 虚拟主机

```shell
#基于域名 端口 IP地址
#基于IP地址 与 端口 的虚拟主机 访问优先级是高于域名的（因为域名最终解析到的也是IP地址）

vi /etc/nginx/conf.d/web1.conf
server{
        listen 127.0.0.1：80;
        server_name www.web1.com;
        location / {
                root /mnt/web1;
                index index.html;
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root /usr/share/nginx/html;
        }
}

vi /etc/nginx/conf.d/web2.conf
server{
        listen 172.16.0.4：81;
        server_name www.web2.com;
        location / {
                root /mnt/web2;
                index index.html;
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root /usr/share/nginx/html;
        }
}
```

### Tcp优化

```shell
vi /etc/sysctl.conf
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 65535
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
```

### Nginx反向代理

**优点：**

1. 降低原始web服务器负载
2. 安全保护后台服务器
3. web资源整合
4. 实现负载均衡集群

**4种反向代理软件：**

1. nginx
2. squid
3. apache
4. varnish

```shell
1.下载软件包
rpm -import http://nginx.org/keys/nginx_signing.key
#下载gpgkey

rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
#安装nginx专用源 里面含nginx的各个版本

yum install -y nginx                    
#安装nginx

2.修改配置
server {
    listen       80;
    server_name  localhost;
    location / {
        #root   /usr/share/nginx/html;
        #index  index.html index.htm;
	proxy_pass http://106.13.128.217:80;     #代理后台成员服务器
	proxy_set_header X-Real-IP $remote_addr; #记录客户端真实IP地址
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

3.记录客户端真实IP

apache
vi /etc/httpd/conf/httpd.conf
LogFormat "%{X-Real-IP}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined   #修改apache日志记录基础格式（为 真实访问的IP地址）

nginx
vi /etc/nginx/conf.d/bosheng.conf
location / {
	...
	set_real_ip_from 192.168.1.101;   #从代理服务器获取客户端真实IP
	real_ip_header X-Real-IP;
}
```

### 配置负载均衡集群

**调度算法**

1. RR轮询（默认）：轮询负载，一台成员服务器轮询一次，周而复始。
2. WRR  ：按照权重轮询，通过给定的权重值决定轮询次数。
3. ip_hash ：每个请求按照访问IP的hash结果分配，来自同一个IP的方可固定访问一个后端成员服务器
4. fair （默认不支持 需要安装upstream_fair）：只能，那个后端成员服务器响应速度最快，分配给那个
5. url_hash （需要安装Nginx的hash软件包）：按照访问的url的hash结果进行分配，使每个url定向到同一个后端成员服务器，可以进一步提高后端缓存服务器效率（每一个页面给一个后端成员服务器）

**部分状态参数**

```shell
down      #表示当前的server赞数不参与负载均衡（维护）
backup    #预留的备份机器，当其他所有的非backup机器出现故障或者忙的时候，才会请求back机器，因此这台机器的压力最轻。
max——fails#允许请求失败次数默认值为1
fail_timeout   #在经历了max_fails 暂停服务的时间
```

**配置**

```shell
upstream webservers {
server 106.13.128.217:80 weight=1;
server 106.13.128.217:81 weight=1;
server 106.13.128.217:82 weight=1 down;
}
server {
	listen 80;
	server_name localhost;
	location / {
		proxy_pass http://webservers;
		proxy_set_header X-Real-IP $remote_addr;
	}
	error_page 500 502 503 504 /50x.html;
	location = /50x.html {
		root /usr/share/nginx/html;
	}
}
```

### Stubstatus监控Nginx运行状态

```shell
vi /etc/nginx/conf.d/bosheng.conf
...
location /ngx_status {
	#启用状态监控模块
	stub_status on;
	#指定访问日志路径
	access_log /var/log/nginx/access.log;
	#启用用户身份验证
	auth_basic "aaa";
	#指定用户身份验证的用户列表文件
	auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
}
...
```

![image-20191120095236553](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191120095236553.png)

```shell
Active connection： 2            #当前活跃的连接数
server accepts handled requests  #accepts处理TCP链接数
handled                          #成功TCP连接数
requests                         #总共建立了多少个请求
Reading: 0 #读取客户端的头信息数量
Writing: 1 #返回给用户的头信息数量
Waiting: 1 #已经处理完，正在等待下一次请求的驻留连接数
```

### web压力测试工具

```shell
1.ab （创建多个并发访问线程）
yum install -y httpd-tools #安装相关软件包
ab -n 10 -c 10 http://123.56.96.231:80/   #-n请求数量 -c并发数量

2.webbench 
#下载源码包
wget http://home.tiscali.cz/~cz210552/distfiles/webbench-1.5.tar.gz
#解压
tar zxvf webbench-1.5.tar.gz
#跳转到解压目录
cd webbench-1.5/
#创建目录
mkdir -p /usr/local/man
#编译安装
make && make install
#-c 并发用户数 -t 测试时长
webbench -c 1000 -t 30 http://192.168.1.114/
```

### Nginx实现CDN网络加速

​	内容分发网络，尽可能避开互联网上影响数据传输速度和稳定性的瓶颈和缓解，使内容传输的更快，更稳定，简单来说，用户访问那台服务器最快就访问那台u武器。

优点：

1. 用户边缘性问题

2. 用户流量最合理分流处理

3. 用户访问网站的体验程度提升

   CDN优点：成本低 速度快 适合访问量比较大的网站

过程：

1. 用户在浏览器输入访问的域名
2. 浏览器向DNS服务器解析请求，得到缓存服务器的IP（根据用户当前的IP地址为用户）
3. 解析完成后浏览器得到CDN缓存服务器的IP地址，向缓存服务器发送访问请求
4. 缓存服务器根据浏览器提供的域名，通过cache内部专用的DNS解析得到此域名源服务器的真实IP地址，再由缓存服务器向真实IP地址提交访问请求。
5. 缓存服务器从真实IP地址得到内容后，一方面本地进行保存，已备后期使用，同时把得到的数据发送到客户端浏览器，完成访问的响应过程
6. 用户得到由缓存服务器传回的数据后显示处理，至此完成整个域名的访问过程。

配置：

```shell
proxy_cache_path /cache levels=1:2 keys_zone=servers:10m max_size=10g inactive=60m; #缓存路径 目录层次 缓存名字和共享内存大小 最大缓存空间
server{
	listen 80
	...
	location / {
		add_header X-Via $server_addr;
		add_header X-Cache $upstream_cacservershe_status; #使用add_header指定的头信息
		proxy_cache servers;                       #启用servers缓存
		proxy_cache_valid 200 2m;                  #200状态 缓存 2m
		proxy_cache_valid   any 10m;               #所有的 缓存 10m
	...
	}
}
```

### URL动静分离

### tomcat