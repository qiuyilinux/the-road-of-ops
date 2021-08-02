### 1.移除不用的模块

​	apache的功能非常强大，但不是每个项目中都能用到其所有的功能，其中加载了一些我们通常用不到的模块，这其实时是没有必要的。

​	去除不用的模块方法很简单，通过#注释httpd.conf问价下的加载的模块，切记在注释加载模块一个 一定一定要Test Configuration一下，否则到时候出差错了都不知道从哪里下手了，通常去掉的模块有：mod_include.so   mod_autoindex.so 等，可自行尝试。当然在性能运行的情况下，不执行该操作

### 2.设置线程参数

​	在httpd.conf配置中找到#LoadModule vhost_alias_module modules/mod_vhost_alias.so 在后面加入：

```shell
<IfModule mpm_winnt_module>
    ThreadsPerChild      300     #一个进程最多拥有的线程数range[100-500]
    MaxRequestsPerChild    3000  #一个线程最多可接受的链接数，默认为0 可能会导致内存泄漏，Apache存在轮询机制，会自动调节。
</IfModule>
```

### 3.启动Apache输出压缩

​	加载mod_deflate_so 模块

```shell
LoadModule deflate_module modules/mod_deflate.so
```

​	然后在  <IfModule mpm_winnt_module>...</IfModule>后面追加如下配置：

```
<IfModule mod_deflate.c>
 DeflateCompressionLevel 6
 AddOutputFilterByType DEFLATE text/html text/plain text/xml    application/x-httpd-php
 AddOutputFilter DEFLATE js css
</IfModule>
```

### 4.安全之Deny

​	通过Deny来禁止若干文件的访问（提高安全性），其用法如下：

```shell
<Directory "D:/deployment">
Order allow,deny
Deny from all
</Directory>
```

### 5.安全之防dos攻击

​	存在mod_dosevasive22.so 可防止网络上页面进行dos攻击（例如：机器人不断刷新指定网页，不断访问）下载该so文件后放置在modules，修改httpd.conf文件，在文件尾追加如下文件。

```shell
LoadModule dosevasive22_module modules/mod_dosevasive22.dll
DOSHashTableSize 3000       #黑名单总数量
DOSPageCount 3              #页面被攻击次数为该值时会判断为dos攻击
DOSSiteCount 50             #站点被攻击次数为该值时会判断为dos攻击
DOSPageInterval 1           #读取页面间隔
DOSSiteInterval 1           #读取站点间隔
DOSBlockingPeriod 10        #访问IP被封时间间隔

ps ： 时间单位为秒
```

### 6.修改默认配置参数

​	在httpd.conf中搜索httpd-defalut.conf，放开对其配置

```shell
Include conf/extra/httd-default.conf

ps : 该文件中存在了对apache的默认配置参数
ps ：对其更改为更适合当前站点
```

### 7.监听端口

​	默认情况下apache监听为80端口，在实际中最好指定监听具体的应用对应 ip地址:80即

```shell
Listen xxx.xxx.xxx.xxx:80
```

​	如服务器做代理 ，缓存 ，CDN ，负载均衡。建议将80端口转为其他端口如7564等奇怪端口，以增加服务器在网络上的安全性，并设置只允许代理服务器访问站点，并设置防火墙，配置如下：

```shell
<Directory "D:/deployment">
Order deny,allow
Deny from all
Allow from 代理IP
</Directory>

iptables -a ...
```

### 8.关联计算机Office软件

​	在应用中经常要打开或者下载附件，例如word，excel等 可通过一下设置关联到本地的Office软件打开 在Addtype application/x-gzip .gz .tgz 后追加

```
AddType application/vnd.openxmlformats  docx pptx xlsx doc xls ppt txt
```

### 9.请求注释

​	看的舒服。