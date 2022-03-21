## 一、 环境

操作系统:  centos 7.9 

内核版本:  3.10.0-1160.el7.x86_64

glibc: glibc-2.17-317.el7.x86_64



## 二、 环境准备

```shell
yum install -y wget 
yum install -y gcc gcc-c++ cmake ncurses-devel autoconf perl perl-devel
```



## 三、 安装



### 1 下载源码包

```shell
cd /usr/local
wget https://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.40.tar.gz
```



### 2 创建用户、 用户组

```shell
groupadd mysql
useradd -r -g mysql mysql
```



### 3 创建数据目录

```shell
mkdir -p /data/data/mysql/5.6.40
mkdir -p /data/config/mysql/5.6.40
mkdir -p /data/log/mysql/5.6.40
mkdir -p /data/run/mysql/5.6.40

chown mysql:mysql /data/data/mysql -R
chown mysql:mysql /data/config/mysql -R
chown mysql:mysql /data/log/mysql -R
chown mysql:mysql /data/run/mysql -R
```





### 4 解压、重命名、跳转到目录

```shell
tar zxvf mysql-5.6.40.tar.gz
mv mysql-5.6.40 mysql5.6.40
cd mysql5.6.40
```



### 5 编译安装

```shell
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql-5.6.40 \
-DMYSQL_DATADIR=/data/data/mysql/5.6.40 \
-DSYSCONFDIR=/data/config/mysql/5.6.40 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DMYSQL_TCP_PORT=3308


make -j4 && make install
```



### 6 编辑配置文件

```shell
vi /data/config/mysql/5.6.40/my.cnf
[mysqld]
basedir = /usr/local/mysql-5.6.40
datadir = /data/data/mysql/5.6.40
socket = /data/run/mysql/5.6.40/mysql.sock
port = 3306
```



### 7 拷贝启动文件 

```shell
cp /usr/local/mysql-5.6.40/support-files/mysql.server  /etc/init.d/mysqld-5.6.40
```



### 8 初始化

```shell
/usr/local/mysql-5.6.40/scripts/mysql_install_db --basedir=/usr/local/mysql-5.6.40 --datadir=/data/data/mysql/5.6.40 --user=mysql
```



### 9 环境变量

```shell
vi /etc/profile
...
export MYSQL_HOME=/usr/local/mysql-5.6.40
export MYSQL_BIN=$MYSQL_HOME/bin
export PATH=$PATH:$MYSQL_BIN
```



### 10 启动、 开机自启

```shell
/etc/init.d/mysql-5.6.40 start
chkconfig mysqld-5.6.40 on
```



### 11 设置密码

```shell
mysqladmin  -u root password -S /data/run/mysql/5.6.40/mysql.sock
```



### 12 测试连接

```shell
mysql -u root -p -S /data/run/mysql/5.6.40/mysql.sock
```



### ps

如果安装失败重新编译时，需要清除旧的对象文件和缓存信息 

```shell
make clean
rm -rf CMakeCache.txt
rm -rf /etc/my.cnf
```

