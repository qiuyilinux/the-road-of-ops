## netcat

### 定义

功能强大网络工具，被称为瑞士军刀，方便快捷，体积小，现已有人将其缩小为10k。

### 安装

```shell
wget https://sourceforge.net/projects/netcat/files/netcat/0.7.1/netcat-0.7.1.tar.gz #下载相关源码包
tar -zxvf netcat-0.7.1.tar.gz -C /usr/local #指定解压到/usr/local
cd /usr/local/netcat-0.7.1	                #跳转到指定目录
test -d /usr/local/netcat || mkdir /usr/local/netcat #创建安装目录
./configure --prefix=/usr/local/netcat	             #编译指定安装路径
make && make install  #安装
cat /etc/profile      #添加环境变量
...
NC_HOME=/usr/local/netcat
PATH=$PATH:$NC_HOME/bin
...

ps：yum安装nc的时候安装的ncat 可以通过 ls -l `which nc` 来查看。
```

### 参数

```shell
语法：nc [-hlnruz][-g<网关...>][-G<指向器数目>][-i<延迟秒数>][-o<输出文件>][-p <通信端口>][-s<来源地址>][-v...][-w<超时秒数>][主机名称][通信端口...]

-c            #close connection on EOF from stdin（输入结束之后关闭连接）
-e            #连接之后要执行的程序
-g<网关>      #设置路由器跃程通信网关 最多可设置8个
-G<指向器条目> #设置来源路由指向器，其数值为4的倍数
-h            #在线帮助
-i<延迟秒数>   #设置时间间隔，以便传送信息及扫描端口
-l            #使用监听模式，管控传入的资料
-n            #直接使用ip地址，而不通过域名服务器
-o<输出文件>   #指定文件名称，把往来传输的数据以16禁止子码倾倒成文件保存
-p<通信端口>   #设置本地主机使用的通信端口
-r            #乱数指定本地与远程主机的通信端口
-s<来源地址>   #设置本地主机送出的数据包的IP地址
-u            #使用UDP传输协议
-v            #详细输出 （用两个-v 可以显示更详细的内容）
-w<超时秒数>   #设置等待连线的时间
-z            #使用0输入/输出模式，只在扫描端口时使用
```

## 实例

```shell
1.端口扫描
nc -v -v -w 2 106.13.128.217 -z 1-100 #两个-v 可以查看这个端口默认服务是什么  一个-v 可以查看端口是否open 默认tcp协议  指定udp协议的时候不会显示端口默认服务

2.不同机器之间拷贝文件
node 1： nc -l -p 1234 > test.txt       #把1234端口接收到数据都写入到test.txt(-p指定端口)
node 2:  nc -c 106.13.128.217 < mm.txt  #-c指定传输完成之后断开连接
ps:拷贝目录配合tar

3.简单聊天工具
node 1： nc -l -p 1234 #监听本主机1234端口
node 2： nc 106.13.128.217 1234 #连接远程主机1234端口聊天

4.建立本地1234端口到远程主机80端口的连接
nc -p 1234 -w 5 106.13.128.217 8090   #5秒超时

ps : 以上为基本用法 
```

