## LAMP

ps ：（本章主要深入了解apache）

​	LAMP指的是Linux Apache Mysql Php的简称，通过大量的生产环境的实践证明，在网络应用和开发环境方面，LAMP组合是非常棒的黄金搭档，他们的结合提供了非常强大的功能。

​	Linux作为底层的操作系统，提供灵活且可定制性的应用平台，为其他组件稳定高效的运行在其之上提供了保障。

​	Apache作为web服务器，提供了功能强大，稳定且支撑能力突出的web平台，为网站提供了强力支柱.

​	Mysql也是一款非常优秀的数据，从其产生的从多衍生数据库就可见的证明其强大

​	Php是一种开放源代码的多用途脚本语言，可嵌入html中，适用于web开发，且其编写的数据可访问mysql数据库及Linux提供的动态内容

## 工作模式

Apache有五种工作模式 beos event worker prefork mpmt_os2

LINUX上使用Apache三种工作模式

- Prefork MPM

  非线程，一个进程处理一个请求，进程提前生成等待客户端访问，没有线程安全库，预派生 

  优点：兼容所有模块，不用担心线程安全问题，进程相对独立，稳定性较高。

  缺点：在高并发情况下表现不是很优秀，一个请求占用一个进程，比较消耗内存。

- Worker MPM

  支持混合多线程 多进程，使用线程处理请求，支持高并发，进程提前生成等待客户端访问。每个进程可以创建多个进程和监听线程，服务线程。监听线程负责接收请求。系统会根据负载情况，增加减少基础南横数量。

  优点：支持高并发，占用资源小。

  缺点：稳定性较差，需要考虑线程安全问题，只要进程出现问题，贤臣给也会出现问题。不能结合PHP一起工作，配合不好。

- Event MPM

  可以把服务进程从连接中分离出来，可以用线程数量控制重要的连接资源。可以防止keepalive占用所有线程，支持高并发。

  优点：高并发相对稳定，可以处理更高的负载。

  缺点：不支持https

PS：工作模式不仅有以上这些 如 apache2-mpm-itk  这是一个补丁 插件，这种模块在虚机上应用广泛，因为他可以让每个网站使用不同的用身份运行，当虚拟主机中一个网站被入侵的时候，不会影响到其他的虚拟主机。

PS：可以通过 httpd -l 与 httpd -V 查看当前的工作模式。

PS：yum安装 缺省模式为Prefork 编译安装可以通过参数指定 默认也为Prefork

## Apache工作原理

**Apache生命周期**

![image-20191030150026997](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191030150026997.png)

- 启动阶段：Apache解析配置文件（如http.conf以及Include指令设定的配置文件等（Include指定的是分割文件）），模块加载（例如mod_php.so,mod_perl.so等）和系统资源初始化（例如日志文件，共享内存段等）工作。在这个阶段，Apache为了获得系统资源族弟啊使用权限，将以特权用户root（linux系统）或超级管理员administrator（windows系统）完成启动。
- 运行阶段：再以管理员身份完成启动后，分11个阶段处理用户请求。

**Apache处理请求阶段**

![image-20191030150839658](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191030150839658.png)

1. Post-Read-Request阶段: 在正常请求处理流程中，这是模块可以插入钩子的第一个阶段。对于那些想很早进入处理请求的模块来说，这个阶段可以被利用。 
2. URI Translation阶段：将请求的URL映射到本地的文件系统中，mod_alisa模块就是这个阶段工作
3. Header Parsing阶段：解析header头部，mod_setenvif在这个阶段工作
4. Access Contro阶段：根据配置文件检查是否允许访问请求的资源。Apache的标准逻辑实现了允许和拒绝指令。mod_authz_host就是利用这个阶段工作的。
5. Authentication阶段：按照配置文件设定的策略对用户进行认证，并设置用户名区域。模块可以在这个阶段插入钩子，实现一个认证方法。
6. Authorization阶段：根据配置文件检查是否允许认证过的用户执行请求的操作，模块可以在这阶段实现用户权限管理的方法。
7. MIME Type Checking阶段：根据请求的资源的MIME类型的相关规则，将文件交由相应的处理模块。
8. Fix Up阶段：模块在内容生成器之前，运行必要的处理流程
9. Response阶段：生成响应报文。
10. Logging阶段：在响应客户端后记录事务。
11. CleanUp阶段：清除请求后遗留的环境，如文件，目录的处理活着Socket的关闭等。

PS：什么是MPM？

MPM（多路处理模块）是Apache的核心组件之一，Apache通过MPM来使用操作系统的资源，对进程和线程池进行管理。Apache为了能够获得更好的运行性能，针对不同的平台提供了不同的MPM，用户可以根据实际的情况进行选择。

## 基本部署

```shell
1.yum install -y httpd php.x86_64 php-fpm phpmyadmin httpd mariadb mariadb-server memcached -y
php            #php主程序包
php-fpm        #守护进程
phpmyadmin     #数据库管理工具
httpd          #apache
mariadb        #数据库客户端
mariadb-server #数据库服务端
memcached      #内存数据库（数据库缓存工具）
2.vi /var/www/html/index.html    #静态页
<h1> hello world </h1>           #这个h* 代表的是字号
3.vi /var/www/html/index.php     #php探针
<?php
phpinfo()
?>
ps : apache 默认启动的是 fast-cgi 随 apache 一起启动 看不到进程 pid 包含在apache进程中。
4.systemctl start mariadb      #启动mariadb服务
  mysqladmin -u root password  #指定数据库设置本地登陆用户名和密码
  mysql -u root -p             #指定用户名和密码登陆数据库
  常用 SQL 语句
  show databases;              #查看有哪些数据库
  create database lamp;        #创建数据库
  use mysql;                   #进入数据库
  show tables;                 #查看有哪些数据库
  drop database lamp;          #删除数据库
  CREATE USER 'apache'@'127.0.0.1' IDENTIFIED BY '123';
  GRANT ALL PRIVILEGES ON lamp.* TO 'apache'@'%' IDENTIFIED BY '123' WITH      GRANT OPTION;               #开启远程数据连接并指定用户和密码(格式为GRANT privileges ON database.tablename TO ‘username’@‘host’ WITH GRANT OPTION;)
  FLUSH PRIVILEGES;            #重建授权表
```

## phpMyAdmin

```shell
1.vi /etc/my.cnf                     #开启数据库innodb引擎
innodb_file_per_table = 1
2.cd /etc/httpd/conf.d/              #apache分割文件路径
vi /etc/httpd/conf.d/phpMyAdmin.conf #配置phpmyadmin(这是全允许，可以按照原来的模板改 允许ip 从127.0.0.1 修改为当前IP即可 保证安全)

Alias /phpMyAdmin /usr/share/phpMyAdmin  #别名
Alias /phpmyadmin /usr/share/phpMyAdmin  #别名

<Directory /usr/share/phpMyAdmin/ >      
Options none
AllowOverride none
Require all granted
</Directory>
3. /etc/php.ini
date.timezone = Asia/Shanghai
4.systemctl reload httpd
```

## 自定义apache端口

```shell
1.vi /etc/httpd/conf/httpd.conf
Listen localhost:2000                      #修改apache监听端口号
2.semanage -a -t http_port_t -p tcp 20000  #添加端口绑定策略
3.systemctl reload httpd 
```

## 发布discuz

```shell
1.wget http://download.comsenz.com/DiscuzX/3.2/Discuz_X3.2_SC_UTF8.zip  #下载discuz包
2.unzip Discuz_X3.2_SC_UTF8.zip  #解压
3.cp upload/* /var/www/html/ -Rf #复制到网站目录下
4.chown apache:apache /var/www/html/* -Rf  #修改属主属组（discuz要求部分文件可写）
5.CREATE USER 'apache'@'127.0.0.1' IDENTIFIED BY '123'; #建立用户apache
6.GRANT ALL PRIVILEGES ON lamp.* TO 'apache'@'%' IDENTIFIED BY '123' WITH      GRANT OPTION;  #允许apache用户使用123密码 访问lamp下的所有表
7.FLUSH PRIVILEGES;            #重建授权表
8.设置selinux
chcon -t httpd_sys_content_rw_t /var/www/html/data/ -R  #修改可写安全上下文
chcon -t httpd_sys_content_rw_t /var/www/html/config/ -R
chcon -t httpd_sys_content_rw_t /var/www/html/uc_client/ -R
chcon -t httpd_sys_content_rw_t /var/www/html/uc_server/ -R
```

## 修改默认站点存放路径

```shell
1.DocumentRoot "/web/upload/"                 #修改站点存放路径
<Directory "/web/upload">                   #对站点存放路径做访问控制
    AllowOverride None
    # Allow open access:
    Require all granted                     #接收所有访问
</Directory>

2.chcon -t httpd_sys_content_t /web/upload -R
```

## 设置用户家目录站点

```shell
1.useradd www -s /usr/sbin/nologin  #创建用户
2.mkdir /home/www/public_html       #在指定路径下创建目录public_html
3.chmod 711 /home/www               #这个要求用户家目录权限必须为711 public_html目录权限最大为755
4.chcon -t httpd_sys_content_t /home/www -R  #配置安全上下文

http://106.13.128.217:2000/~www/   #访问测试
```

## 基于客户端IP的目录访问控制

```shell
vi /etc/httpd/conf/httpd.conf    #编辑配置文件
Order deny，allow     #设置生效顺序
Deny from all         #禁止所有IP访问
Allow from 127.0.0.1  #允许127.0.0.1访问
```

## 虚拟目录

```
#写法1
Alias /shell /shell #名字  路径
<Directory /shell>
	order allow.deny
	allow from all
</Directory>

#写法2
<IfModule dir_module>
        DirectoryIndex index.html
        Alias /shell /shell
        <Directory /shell>
                order allow,deny
                allow from all
                #Require all granted
        </Directory>
</IfModule>

#见下图 apache 的虚拟目录是 当你别名字段被访问时 指向别名路径 但是当你在原站建一与别名同名的文件的时候他会做一个重定向 重定向到虚拟目录路径 原站则不生效
```

![image-20191111112302426](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191111112302426.png)

## 基于用户身份验证的访问控制

```shell
vi /etc/httpd/conf/httd.conf
<Directory "/var/www/html">
    AuthName "hello"                        #提示信息
    AuthType basic                          #验证方式
    AuthUserFile /etc/httpd/conf.d/.htpasswd#指定用户身份认证文件
    Require valid-user                      #只接收用户身份认证文件中的用户的访问
</Directory>

ps: 把此配置添加到针对此目录的全局配置下面 注意注释默认配置文件中的Require all granted(接受所有请求)或写到其下面。
```

## 虚拟主机

利用一个apache服务器发布多个站点

```shell
1.vi /etc/httpd/conf/httpd.conf
#DocumentRoot "/web/upload"
#<Directory "/web/upload">
#    AllowOverride None
#    # Allow open access:
#    Require all granted
#</Directory>

2.vi /etc/httpd/conf.d/vhost1.conf
Listen 2000
<VirtualHost 127.0.0.1:2000>
        ServerName 127..0.0.1:2000
        ServerAlias www.host1.com
        DocumentRoot "/shell/1"
<Directory "/shell/1">
        Order deny,allow
        deny from all
        allow from all
        Require all granted
</Directory>
ErrorLog "/var/log/httpd/one_error.log"
CustomLog "/var/log/httpd/one_access.log" combined
</VirtualHost>


3.vi /etc/httpd/conf.d/vhost2.conf
Listen 2001
<VirtualHost 172.16.0.4:2001>
        ServerName 172.16.0.4:2001
        ServerAlias www.host2.com
        DocumentRoot "/shell/2"
<Directory "/shell/2">
        Order deny,allow
        deny from all
        allow from all
        Require all granted
</Directory>
ErrorLog "/var/log/httpd/two_error.log"
CustomLog "/var/log/httpd/two_access.log" combined
</VirtualHost>

ps: 可基于域名 IP 端口



(基于域名)
<VirtualHost *:80>
        ServerName www.host1.com
        DocumentRoot "/shell/1"
<Directory "/shell/1">
        Order deny,allow
        deny from all
        allow from all
        Require all granted
</Directory>
ErrorLog "/var/log/httpd/one_error.log"
CustomLog "/var/log/httpd/one_access.log" combined
</VirtualHost>
<VirtualHost *:80>
        ServerName www.host2.com
        DocumentRoot "/shell/2"
<Directory "/shell/2">
        Order deny,allow
        deny from all
        allow from all
        Require all granted
</Directory>
ErrorLog "/var/log/httpd/two_error.log"
CustomLog "/var/log/httpd/two_access.log" combined
</VirtualHost>
```

## TLS加密

### 请求过程

1. 客户端请求服务器连接
2. 服务器返回证书公钥给客户端
3. 客户端验证证书是否有效
4. 有效则下一步 无效则弹出警告框
5. 生成一个随机数用证书公钥加密发送给服务器
6. 服务器用私钥解密之后并利用这个随机值生成 对称加密密钥
7. 用生成的对称加密的密钥加密要发送的内容
8. 用密钥来解密信息

### HTTPS一般使用的加密和HASH算法如下

- 非对称加密算法：RSA DSA DSS
- 非对称加密算法：AES RC4 3DES
- HSAH算法：MD5 SHA SHA256

### 部署

```shell
1.yum install mod_ssl -y   #下载生成证书软件包
2.cd /etc/pki/tls/certs    #跳转到密钥路径
3.make gong.pem            #生成公钥和私钥
4.vi /etc/httpd/conf.d/vhost.conf
SSLStrictSNIVHostCheck on  #开启SNI验证
NameVirtualHost *:443      #启用加密端口
<VirtualHost 127.0.0.1:443>
        ServerName 127..0.0.1:443
        ServerAlias www.host1.com
        DocumentRoot "/shell/1"
        SSLEngine on #开启站点加密
        SSLCertificateFile "/etc/pki/tls/certs/gong.pem"    #指定公钥路径
        SSLCertificateKeyFile "/etc/pki/tls/certs/gong.pem" #指定私钥路径
<Directory "/shell/1">
        Order deny,allow
        deny from all
        allow from all
        Require all granted
</Directory>
ErrorLog "/var/log/httpd/one_error.log"
CustomLog "/var/log/httpd/one_access.log" combined
</VirtualHost>

ps ： httpd 不知道从哪个版本之后会有一个welcome.conf的分割文件 开启站点加密后这个文件会先生效 这个文件定义了默认的https 页面 需要把这个文件bak掉才能使自己的设置生效。
```

## Apache反向代理

​	反向代理：没有反向代理请求服务器时是直接请求服务器，有反向代理时就是请求代理服务器，再由代理服务i去将请求转发到具体的服务器。

​	正向代理：代理客户端请求服务器

​	反向代理：代理服务器接受客户端请求

```shell
vi /etc/httpd/conf/httpd.conf
ProxyRequests Off                             #开启反向代理  on为正向代理
ProxyPass / http://106.13.128.217:2001        #代理的地址 / 为代表目标服务器下的那个目录
ProxyPassReverse / http://106.13.128.217:2001 #为防止重定向之后没有代理而导致客户端无法访问所以需要设置
<proxy *:80>                                  #代理的段楼
AllowOverride None                            #忽略.htaccess文件
Order Deny,Allow                              #生效顺序
Allow from all                                #全部允许
</proxy>


基于域名虚拟主机反向代理
<VirtualHost *:80>
        ServerName www.gong.com
        DocumentRoot "/www"
        <Directory "/www">
                Order deny,allow
                deny from all
                allow from all
                Require all granted
        </Directory>

        <proxy>
                AllowOverride None
                Order Deny,Allow
                Allow from all
        </proxy>

        Proxyrequests off
        ProxyPass / http://106.13.128.217:2000
        ProxyPassReverse / http://106.13.128.217:2000
</VirtualHost>


<VirtualHost *:80>
        ServerName www.web.com
        DocumentRoot "/www"
        <Directory "/www">
                Order deny,allow
                deny from all
                allow from all
                Require all granted
        </Directory>

        <proxy>
                AllowOverride None
                Order Deny,Allow
                Allow from all
        </proxy>

        Proxyrequests off
        ProxyPass / http://106.13.128.217:2001
        ProxyPassReverse / http://106.13.128.217:2001
</VirtualHost>


ps ： 可根据用户访问的域名进行代理到不同的地址
```

## Apache负载均衡

```shell
        Proxyrequests off
        ProxyPass / balancer://mycluster/          #指定代理负载均衡一组服务器
        ProxyPassReverse / balancer://mycluster/

        <Proxy balancer://mycluster>               #对负载均衡服务器的相关配置
                ProxySet lbmethod=byrequests       #实现负载均衡的方式
                """
                	（byrequests按照请求次数分配权重
                     bytraffic 按照流量分配权重
                     bybusyness 按照服务器繁忙程度分配权重）
                """
                BalancerMember http://106.13.128.217:2000 loadfactor=1 smax=5 max=20 ttl=120 retry=300 timeout=15

"""
loadfactor=1  权重
smax 根据需要最多创建数量为柔性最大值。任何超过smax数量的链接都会指定一个生存时间ttl。
max  允许连接到后端服务器的硬性最大链接数。值为MPM中每个进程数量。在Prefork MPM 中该值总是为1 ，在worker MPM 中该值受ThreadPerChild 控制。
ttl  超出smax链接数的非活动链接的生存时间，单位为秒。Apache会关闭这段时间没有被使用过的所有链接。
timeout   链接超时时间，单位为秒。如果未设置，那么Apache会一直等到有可用的链接位置。该指令常和max参数一起使用来限制后端服务器链接。
retry  线程池工作单元重试的超时时间。如果到后端服务器的线程池工作单元状态是出错，Apache不会递交任何请求到该服务器，直到超时结束。这可以令后端服务器关闭进行维护，并稍后上线。如果值为0则表示总是重试错误状态的工作单元，不等待任何时间。
"""

                BalancerMember http://106.13.128.217:2001 loadfactor=1 smax=5 max=20 ttl=120 retry=300 timeout=15
        </Proxy>
```

