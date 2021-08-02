文件传输协议，同时他也是一个应用程序，基于不同的操作系统有不同的FTP应用程序，而所有这些应用程序都要遵守同一种协议传输文件。即FTP。

### FTP的两种工作模式

PORT模式

1. 主动传输
2. 服务器准备完成后会建立TCP21端口等待客户端连接。（命令控制通道）监听
3. 客户端会在命令控制通道发送PORT命令（ip port （第五位*245+六位））
4. 服务器验证客户端的登陆信息
5. 服务器启用TCP20端口连接客户端

PASV模式

1. 被动传输
2. 服务器准备完成后会建立TCP21端口等待客户端连接。（命令控制通道）监听
3. 客户端会在命令控制通道发送Pasv命令
4. 服务器验证客户端的登陆信息
5. 启用1023-65535随机端口发送给客户端
6. 客户端启用随机端口进行连接

### FTP两种传输方式

ASCLL（文本模式传输）：适应对方主机，主要用于传输文本文件如html。

BINARY（二进制传输）：不适应对方主机，用于传输可执行文件，保持文件原样不会因对方主机的文本格式而改变。

### FTP三种用户

- 匿名用户：默认安装情况下vsftp使用匿名用户访问（默认只读权限 需要扩大）
- 本地用户：以本地用户为认证方式（权限过大 需要控制）
- 虚拟用户：灵活性较大，相对以上更安全（可对单独用户进行单独的权限授权）

### 配置

#### 	基本配置	

```shell
local_enable=YES				 #是否允许本地用户登陆
anonymous_enable=YES			  #是否允许匿名用户登陆
write_enable=YES				 #是否可写
dirmessage_enable=YES			  #当用户进入到某个目录的时候会显示某个目录需要注意的信息 默认配置文 message_file=.message 可以修改 当为YES的时候会检索这个文件
local_umask=022					 #文件被创建的时候的初始权限
pam_service_name=vsftpd			  #pam服务名称
xferlog_enable=YES				  #日志记录
xferlog_std_format=YES			  #日志记录格式
userlist_enable=YES				  #启用用户列表文件是否开启
connect_from_port_20=YES		  #是否开启20端口
tcp_wrappers=YES				 #tcp协议安全
listen=YES                         #是否监听IPV4的IP地址
listen_address=192.168.1.100       #监听具体哪一个IP（none所有）（本地的IP适用于多网卡）
#listen_ipv6=YES                   #是否监听IPV6的IP地址(与listen 不能同时开启)
listen_port=2121                   #绑定端口号
pasv_enable=YES                    #被动传输是否开启
pasv_min_port=1050                 #被动传输服务器端口号最小值
pasv_max_port=1060                 #被动传输服务器端口号最大值
pasv_promiscuous=YES               #默认值为NO。为YES时，取消PORT安全检查。该检查确保外出的数据只能连接到客户端上。
max_per_ip=1                       #限制一个IP地址最大连接数量为1
max_clients=10                     #所有IP地址打开连接数的总量
```

### 匿名用户

修改文件系统权限  chown ftp:ftp /var/ftp/pub   #匿名用户登陆目录    

#### 配置文件 (这个配置文件写法要求严格每一行的结尾不能有空格)

```shell
anonymous_enable=YES		 #允许匿名用户登录
anon_mkdir_write_enable=YES	  #允许匿名用户创建目录
anon_other_write_enable=YES	  #允许匿名用户写
anon_upload_enable=YES		 #控制是否允许匿名用户上传文件
anon_root=/mnt				 #更改匿名用户登录路径
anon_max_rate=8000			 #限制网速（byte单位）
no_anon_password=YES                   #匿名用户免密登陆
anon_world_readable_only=YES           #所有用户可写的文件允许下载
anon_umask=022                         #设置匿名用户创建文件时的反码值
chown_upload_mode=644                  #上传文件的权限
chown_uploads=YES                      #强制修改匿名用户上传的文件的所有者
chown_username=aaa                     #将上传文件的所有者切换为aaa
```

#### 开放SELINUX 安全上下文与bool值

```shell
setsebool -P ftpd_anon_write on        pu
setsebool -P ftpd_full_access 1        #接受所有访问
chcon -R -t puclic_content_rw_t qqq    #修改目录的安全上下文
```

### 本地用户

```shell
/etc/vsftpd/ftpusers		#配置不允许登陆的用户（黑名单）（这个文件比user_list优先级更高 在这里面deny掉即使在白名单里面也无法登录系统）
```

#### 配置文件	(这个配置文件写法要求严格每一行的结尾不能有空格)

```shell
local_enable=YES					#允许本地用户登陆
local_umask=022						#本地用户创建文件默认反码值
userlist_enable=YES                   #启用用户列表文件
userlist_file=/etc/vsftpd/user_list   #指定用户列表文件（只有在列表里面的用户才能登陆）
userlist_deny=YES                     #为YES时user_list为用户黑名单，当NO时user_list为白名单
chroot_local_user=YES				#开启本地用户监牢机制
allow_writeable_chroot=YES		     #允许监牢机制可写，或者将用户家目录写权限去掉
chroot_list_enable=YES				#启用chroot列表文件
chroot_list_file=/etc/vsftpd/chroot_list#指定chroot列表文件路径
chroot_local_user=YES				#为YES时为白名单（白名单用户受监牢机制限制） NO时为黑名单（黑名单用户不受监牢机制限制）
local_max_rate=8000					#限制本地用户网速
user_config_dir=/etc/vsftpd/conf.d	 #指定分割文件路径里面直接 用 用户名 命名
write_enable=YES				    #开放本地用户写权限
pam_service_name=vsftpd             #本地用户登录没有这个不行
vi /etc/vsftpd/conf.d/gongbosheng	 #对单独用户进行配置如 进行限速 允许写入 等单用户特殊配置
```

### 虚拟用户

只能访问ftp服务，不能访问其他系统资源，强制chroot监牢机制，相对本地用户来说，安全性比较高。认证虚拟用户需要使用到单独的口令库文件，由可插入认证的模块PAM认证，设置虚拟用户的话本地用户和匿名用户都无法登陆FTP

#### 配置部署

```shell
1.yum install -y db4*
2.配置文件
guest_enable=YES                 #开启虚拟用户登陆
guest_username=vtest             #将虚拟用户映射为本地用户
virtual_use_local_privs=YES		#指定虚拟用户使用本地用户权限
chroot_local_user=YES			#开启chroot监牢机制
3.vi /etc/vsftpd/vusers			#添加虚拟用户列表
user1
123
user2
123
user3
123
4.db_load -T -t hash -f /etc/vsftpd/vusers /etc/vsftpd/vusers.db#将用户文件转换为数据库文件
5.chmod 600 /etc/vsftpd/vusers.db		#更改文件的权限
6.vi /etc/pam.d/vsftpd				   #修改认证方式(里面原来的认证方式需要注释掉 有合理需求再重新添加)
auth       required     pam_userdb.so db=/etc/vsftpd/vusers    #身份验证 
account    required     pam_userdb.so db=/etc/vsftpd/vusers    #访问控制
7.useradd -s /sbin/nologin vtest	#添加虚拟用户映射为的本地用户设置nologin
8.setsebool -P ftpd_connect_db 1	#允许ftp连接数据库文件进行身份验证布尔值
9.ststemctl restart vsftpd
```

#### 高级权限

```shell
user_config_dir=/etc/vsftpd/conf.d	#设置分割文件路径
#ssl_enable=YES					   #高级权限不支持加密

vi/etc/vsftpd/conf.d/user1
cmds_allowed=PASV,PWD,ABOR,CWD,DELE,LIST,MDTM,MKD,PORT,QUIT,NLST,PASS,RNFR,RNTO,SIZE,TYPE,USER,REST,CDUP,HELP,MODE,NOOP,REIN,STAT,STOU,SYST,FEAT,STOR,DELE
file_open_mode=0444

systemctl restart vsftpd

ABOR -取消文件传输
CWD  -更改目录
DELE -删除文件
LIST -列目录
MDTM -返回文件的更新时间
MKD  -新建文件夹
NLST -目录改名
PORT -打开一个传输端口
PWD  -显示当前工作目录
QUIT -退出
RETR -下载文件
SIZE -返回文件大小
STOR -上传文件
```

### FTP加密

安全传输层协议(TLS)用户在两个通信应用程序之间提供保密性和数据完整性，该协议由两部分组成，TLS记录协议，TLS握手协议。

```shell
yum install -y openssl
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout vsftpd.pem -out vsftpd.pem

-x509    	#自签发证书
-nodes   	#不给私钥加密
-days    	#证书生存周期
-newkey  	#加密类型
rsa:1024 	#加密类型与加密长度
-keyout  	#指定证书路径
out      	#输出到哪里
```

#### 配置文件

```shell
ssl_enable=YES                           #启用ssl加密
ssl_tlsv1_2=NO                     
ssl_tlsv1=YES                           
allow_anon_ssl=YES                       #允许匿名用户加密
force_anon_logins_ssl=YES                #强制匿名用户登录加密
force_anon_data_ssl=YES                  #强制匿名用户数据传输加密
force_local_data_ssl=YES                 #强制本地用户数据传输加密
force_local_logins_ssl=YES               #强制本地用户登录加密
rsa_cert_file=/etc/vsftpd/vsftpd.pem     #指定证书路径
```

