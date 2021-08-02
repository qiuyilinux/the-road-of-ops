###### 1.介绍和安装

```shell
1.常用版本
	5.6.34 5.6.36 5.6.38 5.6.40
	5.7.18 5.7.20 5.7.24  #性能稳定性较高于5.6
2.MySQL的多种安装方式
	yum
	二进制（源码编译完成为二进制）  ****
	源码包（编译安装）
3.规划
	ip：192.168.50.135
	hostname：db01
	软件存放位置： /application/mysql
	数据存放位置： /data/mysql/data
4.1部署 (二进制安装)
	mkdir -p /application/mysql /data/mysql/data
	useradd mysql
	echo "export PATH=/application/mysql/bin:$PATH" >> /etc/profile && source /etc/profile
	#上传二进制文件
	tar xf mysql-5.7.26.tar.gz
	mv mysql-5.7.26.tar.gz mysql # 软连接 or 改名
	chown -Rf mysql:mysql /data /application/mysql
	
	rpm -qa | grep mariadb 
	yum remove mariadb-libs-5.5.60-1.el7_5.x86_64 # 可能会影响初始化
	
	libaio报错
	yum install -y libaio-devel
	
	mysqld --initialize-insecure --user=mysql --basedir=/application/mysql --datadir=/data/mysql/data  #初始化环境（警告无所谓）初始化文件不全可能是/etc/my.cnf 文件已经存在
	
4.2 部署（编译安装）
	yum install -y gcc* 
	mkdir -p /work/cmake && cd /work/cmake
	wget http://www.cmake.org/files/v2.8/cmake-2.8.10.2.tar.gz
	tar zxvf cmake-2.8.10.2.tar.gz
	mkdir /usr/local/cmake
	cd /work/cmake/cmake-2.8.10.2
	./bootstrap --prefix=/usr/local/cmake
	make && make install 
	echo "export PATH=$PATH:/usr/local/cmake/bin" >> /etc/profile 
	source /etc/profile
	
	mkdir -p /work/mysql /data/mysqldb /usr/local/mysql /var/lib/mysql
	chown -Rf mysql:mysql /var/lib/mysql/ /data/mysqldb
	
	cd /work/mysql
	wget https://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.45.tar.gz
	tar zxf mysql-5.6.45.tar.gz
	cd mysql-5.6.45
	
		
	useradd mysql
	[ -z "`rpm -qa | grep mariadb`" ] && yum remove mariadb-libs-5.5.60-1.el7_5.x86_64 -y # 可能会影响初始化
	yum install -y ncurses-devel  #可能会少这个包
	cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysqldb \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
	make install
	
	cat > /etc/my.cnf << EOF 
	[mysqld]
    user=mysql
    basedir=/usr/local/mysql
    datadir=/data/mysqldb
    socket=/var/lib/mysql/mysql.sock
	[mysql]
    socket=/var/lib/mysql/mysql.sock
EOF
    
    
    sed -i s@'export PATH=$PATH:/usr/local/cmake/bin'@'exportPATH=$PATH:/usr/local/cmake/bin:/usr/local/mysql/bin'@g /etc/profile
    [ $? -ne 0 ] && exit 888
    source /etc/profile	
	
	cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
	
	yum install -y perl*
	/usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysqldb
	
	/etc/init.d/mysqld start 
	[ $? -eq 0 ] && echo "oh my god" || echo oh shit!
	
	注：编译安装的信息和配置文件的信息一定要一致，否则可能无法启动，sock文件不一致则导致无法重置密码。
	
	
	
5.准备启动脚本
	sys-v: service mysqld start|stop|restart
		   /etc/init.d/mysqld start|stop|restart
		   
	vim /etc/my.cnf
    [mysqld] #服务器
    user=mysql
    basedir=/application/mysql
    datadir=/data/mysql/data
    socket=/tmp/mysql.sock
    [mysql] #客户端
    socket=/tmp/mysql.sock
    #如未有配置文件启动报错
	cp /application/mysql/support-files/mysql.server /etc/init.d/mysqld
	
    
	system: systemctl restart|start|stop mysqld
	
	vim /etc/my.cnf
    [mysqld] #服务器
    user=mysql
    basedir=/application/mysql
    datadir=/data/mysql/data
    socket=/tmp/mysql.sock
    [mysql] #客户端
    socket=/tmp/mysql.sock
    #如未有配置文件启动报错
	
	cat > /etc/systemd/system/mysqld.service <<EOF
	[Unit]
	Description=Mysql Server
	Documentation=man:mysqld(8)
	DOcumentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
	After=network.target
	After=syslog.target
	[Install]
	WantedBy=multi-user.target
	[Service]
	User=mysql
	Group=mysql
	ExecStart=/application/bin/mysqld --defaults-file=/etc/my.cnf
	LimitNOFILE = 5000
	EOF
```

###### 2.MySQL的基础管理

```shell
1.用户和权限管理
	mysqladmin -uroot -p[旧密码] password 123   #修改超级管理员用户密码
	mysql -u root -p123  #登陆
	
	1.1用户定义：用户名@'白名单'
	例：
	oldsheng@'10.0.0.2'	     #允许10.0.0.2这个IP通过oldsheng用户连接
	oldsheng@'10.0.0.%'		 #允许10.0.0.任意IP 通过oldsheng用户连接			
	oldsheng@'10.0.0.5%'     #允许10.0.0.5任意IP 通过oldsheng用户连接
	oldsheng@'10.0.0.0/255.255.254.0' #允许10.0.0.0 掩码为255.255.254.0通过oldsheng用户连接（不支持/24）
	oldsheng@'%'             #允许任意IP地址通过oldsheng用户连接（危险）
	oldsheng@'db03'          #允许db03主机名通过oldsheng连接
	oldsheng@'www.baidu,com' #允许www.baidu.com 通过oldsheng连接
	
	1.2用户管理操作（8.0以后必须要创建用户授权之前可以直接授权）
	#创建用户
	create user oldsheng@'%';     
    #创建用户设置密码
    create user oldbo@'%' identified by '123'; 
    #查询用户
    select user,host,authentication_string from mysql.user;  
    （5.7authentication_string 下面为密码的hash值 5.6没有显示）
    #5.6修改密码
    set password for oldsheng@'%'=password('123');
    #5.7修改密码
    alter user oldsheng@'%' identified by '123';
    (mysql5.7可以使用5.6的修改方式，但5.6无法使用5.6的修改方式)
    #用户删除
    drop user oldbo@'%';
    drop user oldsheng@'%';
    
    1.3权限的基本管理
    1.3.1 grant 命令使用
    grant 权限 on 权限范围 to 用户  identified by 密码 英文部分为固定
    权限：
    ALL：所有权限
SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,RELOAD,SHUTDOWN,PROCESS,FILE,REFERENCES,INDEX,ALTER,SHOW DATABASES,SUPER,CREATE,TEMPORARY TABLES,LOCK TABLES,EXECUTE,REPLICATION SLAVE,REPLOCATION CLIENT,CREATE VIEW,SHOW VIEW,CREATE ROUTINE,ALTER ROUTINE,CREATE USER,EVENT,TRIGGER,CREATE TABLESPACE (8.1可以自己定义权限集合 rules)
    with grant option #超级管理员才具备的，给别的用户授权功能。
    
    1.3.2 权限范围：
    *.*              #所有 
    wordpress.*      #wordpress库下所有授权
    wordpress.user   #对wordpress下user表授权
    1.3.3 例：
    #使sheng对wordpress下所有表拥有所有权限密码为1（如果存在密码会修改）
    grant ALL on wordpress.* to sheng@'%' identified by '1';
    #使app用户对wordpress下所有表拥有所有权限密码为123（5.7版本以前（包括5.7）可以不用创建用户直接授权）
    grant select,update,delete,insert on wordpress.* to app@'%' identified by '123'; 
    
    1.3.4 show grant for #查看用户权限
    show grant for sheng@'%'
    show grant for sheng  
    
    1.3.5  revoke #回收权限（mysql不会覆盖用户权限,用grant添加权限会一直添加）
    #回收app用户对wordpress的删除权限
    revoke delete on wordpress.* from app@'%';
```

###### 2.SQL

```shell
2.1 学习MySQL主要还是学习通用的SQL语句，那么SQL语句包括增删改查，SQL语句怎么分类呢。
	DQL：（数据查询语言）查询语句，凡是select语句都是DQL。
	DML：（数据操作语言）insert，delete，update 对表当中的数据进行增删改。
	DDL：（数据定义语言）create，drop，alter 对表结构增删改。
	TCL：（事务控制语言）commit提交事务，rollback回滚事务。
	DCL：（数据控制语言）grant授权，revoke撤销权限等。	
2.2 导入数据
	show databases;   (这个不是SQL语句，数据MySQL命令)
	create databaes bbj; (这个不是SQL语句，数据MySQL命令)
	use bbj; (这个不是SQL语句，数据MySQL命令)
	show tables; (这个不是SQL语句，数据MySQL命令)
2.3 SQL脚本
	2.3.1什么是SQL脚本
		当一个文件的扩展名是.sql,并且该文件中编写了大量的sql语句，我们称之为sql脚本。
	2.3.2如何使用
		source /home/aaa.sql
2.4 查看表结构
	desc bbj
2.5	查看当前使用那个数据库
	select datebase();
	查看使用那个数据库版本
	select version();
2.6 常用语句
	2.6.1 查看创建表的语句
	show create table emp;
	2.6.2 SQL语句
		2.6.3 简单查询语句（DQL）
			语法格式
				select 字段1 字段2 字段3 ... from 表名;
			提示：
				1.任何一条sql语句以";"为结尾
				2.sql语句不区分大小写
			# 查询sal字段乘12 （字段可以参与数学运算）
			select ename,sal * 12 from emp;	
			# 查询ename对字段重命名为姓名 查询 sal对字段重命名为hello对数据乘12（sql语句以单引号包含起来（双引号只适用于MySQL））
			select ename as '姓名',sal * 12 as hello from emp;	
			#查看表里面所有数据 
			select * from emp;
		2.6.4 条件查询
	
   
	
```

