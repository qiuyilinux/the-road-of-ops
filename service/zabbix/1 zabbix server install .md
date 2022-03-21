## 一、 环境

操作系统:  centos 7.9 

内核版本:  3.10.0-1160.el7.x86_64

zabbix: zabbix-6.0.0beta2

mysql: 5.7.28 + (硬性要求)





## 二、环境准备

### 1.1 gcc

```shell
yum install gcc gcc-c++ -y
```

### 1.2 libxml2-devel

```shell
yum install libxml2-devel -y
```

### 1.3 net-snmp-devel

```shell
yum install net-snmp-devel -y
# 肯能需要编译安装 正常安装
```

### 1.4 libevent-devel

```shell
yum install libevent-devel  -y
```

### 1.5 javac

```shell
yum java-1.8.0-openjdk-devel.x86_64 -y 
```

### 1.6 Curl library

```shell
yum install curl-devel -y
```

### 1.7 lib64hiredis0-0.12

```shell
rpm -ivh http://rpmfind.net/linux/mageia/distrib/5/x86_64/media/core/release/lib64hiredis0-0.12.1-1.mga5.x86_64.rpm
```

### 1.8 mysql-devel

```shell
rpm -ivh http://repo.mysql.com/yum/mysql-5.6-community/el/6/x86_64/mysql-community-release-el6-5.noarch.rpm
yum install mysql-devel -y
```

### 1.9 openIPMI

```shell
yum install OpenIPMI OpenIPMI-devel -y
```



### 1.10 创建目录、用户、赋权

```shell
groupadd zabbix
useradd zabbix -g zabbix

mkdir -p /data/config/oss/zabbix
mkdir -p /data/run/oss/zabbix
mkdir -p /data/logs/oss/zabbix
mkdir -p /data/scripts/oss/zabbix

chown zabbix:zabbix /data/config/oss/zabbix -R
chown zabbix:zabbix /data/logs/oss/zabbix -R
chown zabbix:zabbix /data/run/oss/zabbix -R
chown zabbix:zabbix /data/scripts/oss/zabbix -R
```



## 2 安装 zabbix

### 2.1 下载、解压、编译安装

```shell
wget https://cdn.zabbix.com/zabbix/sources/development/6.0/zabbix-6.0.0beta2.tar.gz
tar zxvf zabbix-6.0.0beta2.tar.gz
cd zabbix-6.0.0beta2
./configure --prefix=/usr/local/zabbix-6.0.0 --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --with-openipmi --enable-java
make install
```



### 2.2 软链

```shell
mv /usr/local/zabbix/etc/* /data/config/oss/zabbix/
rm -rf /usr/local/zabbix/etc
ln -s /data/config/oss/zabbix/ /usr/local/zabbix/etc
ln -s /data/scripts/oss/zabbix/ /usr/local/zabbix/scripts
rm -rf /usr/local/sbin/zabbix_*
ln -s /usr/local/zabbix/sbin/zabbix_agentd /usr/local/sbin/zabbix_agentd
ln -s /usr/local/zabbix/sbin/zabbix_server /usr/local/sbin/zabbix_server
```



### 2.3 创建数据库、用户、开启远程访问

```sql
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@"192.168.122.%" identified by "123456";
grant all on zabbix.* to zabbix@"192.168.122.%";
```



### 2.4 数据库文件导入

```
mysql -h 192.168.122.236 -uzabbix -p
use zabbix;
source /usr/local/zabbix-6.0.0beta2/database/mysql/schema.sql
source /usr/local/zabbix-6.0.0beta2/database/mysql/data.sql
source /usr/local/zabbix-6.0.0beta2/database/mysql/images.sql
```



### 2.4 agentd 配置文件

```shell
PidFile=/data/run/oss/zabbix/zabbix_agentd.pid
LogFile=/data/logs/oss/zabbix/zabbix_agentd.log
AllowKey=system.run[*]
ListenPort=31350
Hostname=zabbix-server
BufferSize=1000
Timeout=3
Include=/data/config/oss/zabbix/zabbix_agentd.conf.d/*.conf
LoadModulePath=/data/scripts/oss/zabbix/
ListenIP=192.168.122.192
ServerActive=192.168.122.192:31351
Server=192.168.122.192
```





### 2.5 agentd 启动文件

```shell
#!/bin/bash
#
# chkconfig: - 90 10
# description:  Starts and stops Zabbix Agent using chkconfig
#				Tested on Fedora Core 2 - 5
#				Should work on all Fedora Core versions
#
# @name:	zabbix_agentd
# @author:	Alexander Hagenah <hagenah@topconcepts.com>
# @created:	18.04.2006
#
# Modified for Zabbix 2.0.0
# May 2012, Zabbix SIA
#
# Source function library.
. /etc/init.d/functions

# Variables
# Edit these to match your system settings

	# Zabbix-Directory
	BASEDIR=/usr/local

	# Binary File
	BINARY_NAME=zabbix_agentd

	# Full Binary File Call
	FULLPATH=$BASEDIR/sbin/$BINARY_NAME

	# PID file
	PIDFILE=/tmp/$BINARY_NAME.pid

	# Establish args
	ERROR=0
	STOPPING=0

#
# No need to edit the things below
#

# application checking status
if [ -f $PIDFILE  ] && [ -s $PIDFILE ]
	then
	PID=`cat $PIDFILE`

	if [ "x$PID" != "x" ] && kill -0 $PID 2>/dev/null && [ $BINARY_NAME == `ps -e | grep $PID | awk '{print $4}'` ]
	then
		STATUS="$BINARY_NAME (pid `pidof $APP`) running.."
		RUNNING=1
	else
		rm -f $PIDFILE
		STATUS="$BINARY_NAME (pid file existed ($PID) and now removed) not running.."
		RUNNING=0
	fi
else
	if [ `ps -e | grep $BINARY_NAME | head -1 | awk '{ print $1 }'` ]
		then
		STATUS="$BINARY_NAME (pid `pidof $APP`, but no pid file) running.."
	else
		STATUS="$BINARY_NAME (no pid file) not running"
	fi
	RUNNING=0
fi

# functions
start() {
	if [ $RUNNING -eq 1 ]
		then
		echo "$0 $ARG: $BINARY_NAME (pid $PID) already running"
	else
		action $"Starting $BINARY_NAME: " $FULLPATH
		touch /var/lock/subsys/$BINARY_NAME
	fi
}

stop() {
	echo -n $"Shutting down $BINARY_NAME: "
	killproc $BINARY_NAME
	RETVAL=$?
	echo
	[ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$BINARY_NAME
	RUNNING=0
}


# logic
case "$1" in
	start)
		start
		;;
	stop)
		stop
		;;
	status)
		status $BINARY_NAME
		;;
	restart)
		stop
		sleep 10
		start
		;;
	help|*)
		echo $"Usage: $0 {start|stop|status|restart|help}"
		cat <<EOF

			start		- start $BINARY_NAME
			stop		- stop $BINARY_NAME
			status		- show current status of $BINARY_NAME
			restart		- restart $BINARY_NAME if running by sending a SIGHUP or start if not running
			help		- this screen

EOF
	exit 1
	;;
esac

exit 0
```



### 2.6 agentd 启动

```shell
/etc/init.d/zabbix_agentd start 
chkconfig zabbix_agentd on
```





### 2.5 server 配置文件

```shell
ListenPort=31351
LogFile=/data/logs/oss/zabbix/zabbix_server.log
LogFileSize=1024
DebugLevel=3
PidFile=/data/run/oss/zabbix/zabbix_server.pid
DBHost=192.168.122.236
DBName=zabbix
DBUser=zabbix
DBPassword=123456
DBPort=3306
StartPollers=10
StartPollersUnreachable=10
StartTrappers=5
StartPingers=5
StartDiscoverers=5
JavaGateway=127.0.0.1
JavaGatewayPort=31352
StartJavaPollers=10
StartDBSyncers=20
Timeout=30
AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts
ExternalScripts=/usr/local/zabbix/share/zabbix/externalscripts
# FpingLocation=/usr/sbin/fping
Include=/data/config/oss/zabbix/zabbix_server.conf.d/*.conf
LoadModulePath=/data/scripts/oss/zabbix
HistoryCacheSize=100M
HistoryIndexCacheSize=100M
TrendCacheSize=100M
ValueCacheSize=100M
CacheSize=100M
LogSlowQueries=1000
CacheUpdateFrequency=1
```



### 2.7 server 启动文件

```shell
#!/bin/bash
#
# chkconfig: - 90 10
# description:  Starts and stops Zabbix Server using chkconfig
#                               Tested on Fedora Core 2 - 5
#                               Should work on all Fedora Core versions
#
# @name:        zabbix_server
# @author:      Alexander Hagenah <hagenah@topconcepts.com>
# @created:     18.04.2006
#
# Modified for Zabbix 2.0.0
# May 2012, Zabbix SIA
#
# Source function library.
. /etc/init.d/functions

# Variables
# Edit these to match your system settings

        # Zabbix-Directory
        BASEDIR=/usr/local

        # Binary File
        BINARY_NAME=zabbix_server

        # Full Binary File Call
        FULLPATH=$BASEDIR/sbin/$BINARY_NAME

        # PID file
        PIDFILE=/tmp/$BINARY_NAME.pid

        # Establish args
        ERROR=0
        STOPPING=0

#
# No need to edit the things below
#

# application checking status
if [ -f $PIDFILE  ] && [ -s $PIDFILE ]
        then
        PID=`cat $PIDFILE`

        if [ "x$PID" != "x" ] && kill -0 $PID 2>/dev/null && [ $BINARY_NAME == `ps -e | grep $PID | awk '{print $4}'` ]
        then
                STATUS="$BINARY_NAME (pid `pidof $APP`) running.."
                RUNNING=1
        else
                rm -f $PIDFILE
                STATUS="$BINARY_NAME (pid file existed ($PID) and now removed) not running.."
                RUNNING=0
        fi
else
        if [ `ps -e | grep $BINARY_NAME | head -1 | awk '{ print $1 }'` ]
                then
                STATUS="$BINARY_NAME (pid `pidof $APP`, but no pid file) running.."
        else
                STATUS="$BINARY_NAME (no pid file) not running"
        fi
        RUNNING=0
fi

# functions
start() {
        if [ $RUNNING -eq 1 ]
                then
                echo "$0 $ARG: $BINARY_NAME (pid $PID) already running"
        else
                action $"Starting $BINARY_NAME: " $FULLPATH
                touch /var/lock/subsys/$BINARY_NAME
    fi
}

stop() {
        echo -n $"Shutting down $BINARY_NAME: "
        killproc $BINARY_NAME
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$BINARY_NAME
        RUNNING=0
}


# logic
case "$1" in
        start)
                start
                ;;
        stop)
                stop
                ;;
        status)
                status $BINARY_NAME
                ;;
        restart)
                stop
                sleep 10
                start
                ;;
        help|*)
        echo $"Usage: $0 {start|stop|status|restart|help}"
                cat <<EOF

                        start           - start $BINARY_NAME
                        stop            - stop $BINARY_NAME
                        status          - show current status of $BINARY_NAME
                        restart         - restart $BINARY_NAME if running by sending a SIGHUP or start if not running
                        help            - this screen

EOF
        exit 1
        ;;
esac

exit 0
```



###  2.8 server 启动

```shell
/etc/init.d/zabbix_server start
chkconfig zabbix_server on
```





## 3 web端配置

### 3.1 软件包安装及首页配置

```
yum install nginx -y 

mkdir -p /var/www/html/zabbix

cp -rf /usr/local/zabbix-6.0.0beta2/ui/* /var/www/html/zabbix/

chown nginx:nginx /var/www/html/zabbix/ -Rf

yum install epel-release

yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm -y

yum install yum-utils -y

yum-config-manager --enable remi-php72 -y

yum install php72 -y

yum install php72-php-fpm php72-php-gd php72-php-json php72-php-mbstring php72-php-mysqlnd php72-php-xml php72-php-xmlrpc php72-php-opcache php72-php-bcmath -y
```



### 3.2 nginx 配置

#### 3.2.1 环境准备

```shell
mkdir -p  /var/log/nginx/zabbix
chown nginx:nginx /var/www/html/zabbix -R
```

#### 3.2.2  nginx.conf

```shell
user                                                    nginx;
worker_processes                                        auto;
error_log                                               /var/log/nginx/error.log;
pid                                                     /var/run/nginx.pid;
worker_rlimit_nofile                                    65535;

events {
        use                                             epoll;
        worker_connections                              65535;
}


http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

        log_format  zabbixformat  '$remote_addr - $remote_user [$time_local] "$request_method $scheme://$http_host$uri $server_protocol" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for" "$upstream_cache_status" "$upstream_addr"';
        access_log                                      /var/log/nginx/access.log  main;
        sendfile                                        on;
        tcp_nopush                                      on;
        #tcp_nodelay                                     on;
        keepalive_timeout                               65;
        keepalive_requests                              2048;
        types_hash_max_size                             2048;
        client_max_body_size                            100M;
        charset                                         utf8;
        include                                         /etc/nginx/mime.types;
        default_type                                    application/octet-stream;
        server_tokens                                   off;

        gzip                                            on;
        gzip_min_length                                 1k;
        gzip_buffers                                    4 32k;
        gzip_http_version                               1.1;
        gzip_comp_level                                 2;
        gzip_types                                      text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php;
        gzip_vary                                       on;
        gzip_disable                                    "MSIE [1-6]\.";

    	include 					conf.d/*.conf;
}
```

#### 3.2.3 conf.d/zabbix.conf

```shell
server {
	listen       				80;
	server_name  				_;
	include                                 conf.d/zabbix_public;
}
```

#### 3.2.4 conf.d/zabbix_public

```shell
    access_log /var/log/nginx/zabbix/zabbix-access.log zabbixformat;
    access_log off;
    error_log  /var/log/nginx/zabbix/zabbix-error.log warn;

    client_body_buffer_size 5m;

    fastcgi_buffers 16 256k;
    fastcgi_busy_buffers_size 256k;
    fastcgi_temp_file_write_size 256k;
    fastcgi_intercept_errors on;

    location / {
        root   /var/www/html/zabbix;
        index index.php  index.html index.htm;
	add_header Cache-Control no-store;

    }

    location /status {
        stub_status on;
        access_log   off;
        allow 127.0.0.1/32;
        deny all;
    }

    error_page  404              /404.html;
    location = /404.html {
        root   /usr/share/nginx/html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /var/www/html/zabbix;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME   $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```

#### 3.2.5 启动 nginx

```shell
systemctl start nginx 
systemctl enable nginx
```

###  3.3 配置 php

```
cd /etc/opt/remi/php72/

sed -i 's/post_max_size.*/post_max_size =16M/g' php.ini
sed -i 's/max_execution_time.*/max_execution_time =300/g' php.ini
sed -i 's/max_input_time.*/max_input_time =300/g' php.ini
sed -i 's/date.timezone.*/date.timezone = Asia\/Shanghai/g' php.ini


systemctl start php72-php-fpm
systemctl enable php72-php-fpm.service
```

