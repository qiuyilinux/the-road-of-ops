​			Tomcat服务器是一个免费的开源的Web服务器，属于轻量级服务器，是开发和调试JSP程序的首选。

# 1.安装JDK

​			（java开发出的产品）使整个java的核心，包括了java的运行环境，java攻击和java寄出的类库。

```shell
rpm -qa | grep java           #筛选出来java默认安装的软件包
yum erase -y java*            #卸载系统种的java重新配置环境
jdk-8u144-linux-x64.tar.gz    #下载相关软件包进行安装
tar zxvf jdk-8u144-linux-x64.tar.gz #解压
cp jdk1.8.0_144/ /usr/local/jdk -Rf #复制到/usr/local下

vim  /etc/profile #设置环境变量
export JAVA_HOME=/usr/local/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

# 2.安装Tomcat

```shell
mkdir /usr/local/tomcat                         #创建安装目录
tar zxvf apache-tomcat-9.0.27.tar.gz            #解压
cp apache-tomcat-9.0.27/* /usr/local/tomcat -Rf #把解压完成的文件复制到安装目录
/usr/local/tomcat/bin/startup.sh                #启动
http://106.13.128.217:8080/                     #访问测试
```

# 3.tomcat 相关文件

```shell
./bin    #用于启动，关闭tomcat或者其他功能的脚本（.bat or .sh 文件）
./conf   #用于配置Tomcat的XML及DTD文件
./logs   #Catalina和其他Web应用程序的日志文件
./temp   #临时文件
./webapps#web应用程序根目录
./work   #用以产生有JSP编译出的Servlet的.java和.class文件
```

# 4.修改Tomcat默认端口

```xml
/usr/local/tomcat/conf/server.xml   #全局配置
/usr/local/tomcat/conf/web.xml      #站点配置

vi /usr/local/tomcat/conf/server.xml
...(69行左右)
<Connector port="521" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <!-- A "Connector" using the shared thread pool-->
    <!--
    <Connector executor="tomcatThreadPool"
               port="521" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
 -->

...
(xml html <!-- 注释 -->)
```

#  5.日志

```shell
日志类型
catalina    #为命令行输出日志
localhost   #为localhost主机的命令输出日志
manager     #管理日志
admin       #访问日志
host-manager#虚拟主机日志

日志级别
WARNING > INFO > CONFIG > FINE > FINER > FINEST (lowest value)
```

# 6.设置tomcat开机启动

```shell
wget http://www.apache.org/dist/commons/daemon/source/commons-daemon-1.2.1-native-src.tar.gz                                #下载包（这个包的主要作用就是将java程序 作为 服务（后台程序））
tar zxvf commons-daemon-1.2.1-native-src.tar.gz  #解压
cd commons-daemon-1.2.1-native-src/              #跳转到工作目录
./configure && make -j4                          #编译安装
cp jsvc /usr/local/tomcat/bin/                   #把编译完的启动文件放到指定路径

vi /usr/local/tomcat/bin/daemon.sh      #配置启动脚本
JAVA_HOME=/usr/local/jdk                #指定java的家目录
CATALINA_HOME=/usr/local/tomcat         #指定tomcat的家目录
CATALINA_OPTS="-Xms768m -Xmx1024m -XX:PermSize=128m -XX:MaxPermSize=256m"  #启动参数
cp daemon.sh /etc/init.d/tomcat
chmod 755 /etc/init.d/tomcat   
useradd -s /sbin/nologin tomcat   #（启动脚本那种设置了启动用户tomcat）
chown tomcat:tomcat /usr/local/tomcat/ -Rf
chkconfig tomcat on       #设置开机自启
chkconfig --list tomcat   #查看启动项里面是否有
service tomcat start      #启动
```

# 7.配置tomcat加密访问

```xml
vi server.xml    

cd /etc/pki/tls/certs
make tomcat.pem
chmod +x tomcat.pem
chown tomcat:tomcat tomcat.pem

...
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true">
        <SSLHostConfig>
            <Certificate certificateKeyFile="/etc/pki/tls/certs/tomcat.pem"
                         certificateFile="/etc/pki/tls/certs/tomcat.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
...

```

# 8.修改网站默认存放路径

```shell
vi server.xml
...
 <Host name="localhost"  appBase="/webapps"
            unpackWARs="true" autoDeploy="true">
...

mkdir /webapps/ROOT -p
vi /webapps/ROOT/index.html
<h1> hello world </h1>
```

# 9.虚拟目录

```xml
vi server.xml

<Host name="localhost"  appBase="/webapps"
            unpackWARs="true" autoDeploy="true">
...
<Context path="/cc" docBase="/web/web1" debug="0"/> 
...
</Host>
```

# 10.虚拟主机

```xml
<!-- 基于域名：-->
<Host name="www.bosheng.com"  appBase="/web/web1"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
<!-- 基于IP： -->
<Host name="127.0.0.1"  appBase="/web/web2"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

# 11.freecms

```shell
wget http://www.freeteam.cn/freecms1.5.rar                 #下载freeteam
wget https://www.rarlab.com/rar/rarlinux-x64-5.7.1.tar.gz  #下载rar格式的压缩包
tar zxvf rarlinux-x64-5.7.1.tar.gz -C /usr/local/rar       #解压到指定目录
cd /usr/local/rar && make                                  #编译
unrar x freecms1.5.rar                                     #解压

mysql -u root -p123
source /opt/freecms-1.5/DB/mysql/freecms.sql;

mv /usr/local/tomcat/webapps/ROOT{,.bak}
cp /home/freecms-1.5/ROOT/ /usr/local/tomcat/webapps/ -Rf

vi /usr/local/tomcat/webapps/ROOT/WEB-INF/classes/db.properties #修改网站指定数据库文件
mysql.driver=org.gjt.mm.mysql.Driver
mysql.url=jdbc:mysql://localhost:3306/freecms
mysql.username=root
mysql.password=123

vi server.xml
<Connector port="521" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               URIEncoding="UTF-8"/>
```

# 12.动静分离

```shell
server {
        listen 2000;
        server_name localhost;
        location / {
                proxy_pass http://106.13.128.217:800;
                proxy_redirect off;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|htm)$ {
            proxy_pass http://106.13.128.217:800;
        proxy_redirect off;
        }

        location ~ .*(jsp|jspx|do|action|html)$ {
            proxy_pass http://106.13.128.217:521;
        proxy_redirect off;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
                root /usr/share/nginx/html;
        }
}

后续继续测！
```

