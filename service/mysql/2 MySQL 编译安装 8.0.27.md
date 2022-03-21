## 一、 环境

操作系统:  centos 7.9 

内核版本:  3.10.0-1160.el7.x86_64

glibc: glibc-2.17-317.el7.x86_64



## 二、 环境准备

### 2.1 wget

```shell
yum -y install wget
```



### 2.2 cmake

> cmake 要求 3.5.1 +

```shell
cd /usr/local
wget https://cmake.org/files/v3.8/cmake-3.8.2-Linux-x86_64.tar.gz
tar zxvf cmake-3.8.2-Linux-x86_64.tar.gz
ln -s /usr/local/cmake-3.8.2-Linux-x86_64 /usr/local/cmake


vi /etc/profile
export CMAKE_HOME=/usr/local/cmake
export CMAKE_BIN=$CMAKE_HOME/bin
export PATH=$PATH:$CMAKE_BIN

source /etc/profile

cmake --version
```



### 2.3 gcc

> gcc 要求 5.3+

```shell
yum -y install bzip2

cd /usr/local
wget https://ftp.gnu.org/gnu/gcc/gcc-8.3.0/gcc-8.3.0.tar.gz  --no-check-certificate

tar zxvf gcc-8.3.0.tar.gz

# 下载依赖包
./contrib/download_prerequisites

# 编译安装
mkdir build && cd build
../configure --prefix=/usr/ --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j4 && make install
## 注意了，上面那个 prefix 必须用 usr/，以便覆盖掉旧版的 gcc ，以免编译程序找不到新版 gcc

gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/x86_64-pc-linux-gnu/8.3.0/lto-wrapper
目标：x86_64-pc-linux-gnu
配置为：../configure --prefix=/usr/ --enable-checking=release --enable-languages=c,c++ --disable-multilib
线程模型：posix
gcc 版本 8.3.0 (GCC)
```



### 2.4 ncurses-devel、bison、openssl-devel

```shell
yum -y install ncurses-devel bison openssl-devel
```



### 2.5 创建用户组

```shell
groupadd mysql
useradd -r -g mysql mysql
```



### 2.6 创建工作目录

```shell
mkdir -p /data/config/mysql/8.0.27
mkdir -p /data/log/mysql/8.0.27
mkdir -p /data/run/mysql/8.0.27


chown mysql:mysql /data/config/mysql -R
chown mysql:mysql /data/log/mysql -R
chown mysql:mysql /data/run/mysql -R
```



## 三、 安装

### 3.1 下载、解压

> https://dev.mysql.com/downloads/mysql/ 

```shell
cd /usr/local
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-boost-8.0.27.tar.gz
tar zxvf mysql-boost-8.0.27.tar.gz
mv mysql-8.0.27 mysql8.0.27
cd mysql8.0.27/
```



### 3.2 编译安装

```shell
cmake . \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=./boost \
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1 \
-DWITHOUT_ARCHIVE_STORAGE_ENGINE=1 \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql-8.0.27 \
-DMYSQL_DATADIR=/data/data/mysql/8.0.27 \
-DSYSCONFDIR=/data/config/mysql/8.0.27 \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DMYSQL_TCP_PORT=3307 \
-DCMAKE_C_COMPILER=/usr/bin/gcc \
-DCMAKE_CXX_COMPILE=/usr/bin/g++ \
-DFORCE_INSOURCE_BUILD=1

make -j4 && make install
```





### 3.3 拷贝启动文件

```shell
cp /usr/local/mysql-8.0.27/support-files/mysql.server /etc/init.d/mysqld-8.0.27
```



### 3.4 环境变量

```shell
vi /etc/profile

export MYSQL_HOME_8027=/usr/local/mysql-8.0.27
export MYSQL_BIN_8027=$MYSQL_HOME_8027/bin
export PATH=$PATH:$MYSQL_BIN_8027
...
```



### 3.5 初始化

```shell
/usr/local/mysql-8.0.27/bin/mysqld --initialize-insecure
chown mysql:mysql /data/data/mysql -R
```



### 3.6 配置文件

```shell
[mysqld]
basedir = /usr/local/mysql-8.0.27
datadir = /data/data/mysql/8.0.27
socket = /data/run/mysql/8.0.27/mysql.sock
default_authentication_plugin=mysql_native_password   # 注1
port = 3307

# 注1:可以看到MySQL8.0.11版本默认的认证方式是caching_sha2_password ，而在MySQL5.7版本则为mysql_native_password。若想在MySQL8.0版本中继续使用旧版本中的认证方式需要在my.cnf 文件中配置并重启，
```



### 3.7 启动

```shell
/etc/init.d/mysqld-8.0.27 start
```



### 3.8 设置密码

```shell
/usr/local/mysql-8.0.27/bin/mysqladmin -u root password -S /data/run/mysql/8.0.27/mysql.sock
```



### 3.9 测试登录

```shell
/usr/local/mysql-8.0.27/bin/mysql -u root -p -S /data/run/mysql/8.0.27/mysql.sock
```

