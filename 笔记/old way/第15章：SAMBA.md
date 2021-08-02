

## SMB协议

**samba实现linux与windows之间的文件共享（文件系统，打印机及其他的资源）**

smb通信协议是微软和因特尔在1987年制定的协议，主要作为Microsoft网络的通讯协议。smb是在会话层和表示层以及小部分应用层协议

smb使用了NetBIOS的应用程序接口。另外，他是一个开发性的协议，允许了协议扩展——使得它变得更大而且复杂；大约由65最上层的作业，而每个作业都超过120个函数，甚至Windows NT也没有全部支持到，最近微软又把SMB改名为CIFS，并且加入了许多心的特色。



Linux 访问 Windows共享

```shell
1. Window上设置一个共享目录
2. smbclient //192.168.1.28/web -U www  				#使用smbclient连接支持部分命令 不常用？查看支持命令（www为本地用户 交互模式）
3. mount.cifs //192.168.1.17/web /mnt -o username=ggg  	 #直接把目录挂载到/mnt 通过linux命令进行操作
```

Windows 访问 Linux 共享

```shell
1. yum install -y samba* #Linux 默认安装了samba客户端的服务 要安装samba服务器需要的软件包
   yum install -y cifs-utils  #安装.cifs需要软件包
2. systemctl start smb	#启动服务
3. pdbedit -a aaa	    #创建smb用户 设置口令 （基于本地用户）
   pdbedit -x #删除用户 
   pdbedit -L #查看smb用户
   pdbedit -c "[D]"	#暂停smb用户权限（禁止）
   pdbedit -c "[]"	#启用
4. setsebool samba_enale_home_dirs 1	#开启用户在家目录操作权限的布尔值
5. cmd   file\\127.0.0.1			   #windows连接（此时共享的为用户家目录）
（如果长期使用可以右键点击我的电脑点击映射网络驱动器）

##########################共享目录设置#######################
1.编写配置文件/etc/samba/smb.conf
[public]
	comment = public path
	path = /pub/
	public = yes
	writable = yes
	printable = no
	write list = +staff
	browseable = yes
2.setsebool -P smbd_anon_write 1 #更改共享文件夹科协的布尔值为开启
3.chmod 777 /pub			    #给目录权限为777
4.chcon -t public_content_rw_t /pub #修改安全上下文
5.访问
```

