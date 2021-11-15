## rsync

###### 简介：

rsync 使用 rsync 算法提供一个客户机与远程文件系统同步的快速方法，而且可以通过ssh方式来传输文件

###### 特性：

1. 能更新整个目录树和文件系统
2. 有选择性的保持符号链接 硬链接 文件漏洞 权限 时间戳等信息
3. 对于安装来说 无任何特殊权限要求
4. 对多个文件来说 内部流水线减少文件等待的延迟
5. 能用rsh ssh 或者直接端口作为传入端口
6. 支持匿名 rsync 同步文件

###### 工作方式

1. 拷贝本地文件

2. 恢复本地文件

3. 远程数据传输，把本机的文件备份传到远程主机

4. 把远程主机的文件恢复到本主机

5. 作为独立的服务

6. 同步独立的服务

   注：简洁来说 可以在本地同步 本地与远程主机同步 还可以作为服务器

###### 基本命令参数

 1 -v, --verbose 详细模式输出
 2 -q, --quiet 精简输出模式
 3 -c, --checksum 打开校验开关，强制对文件传输进行校验
 4 -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD
 5 -r, --recursive 对子目录以递归模式处理
 6 -R, --relative 使用相对路径信息
 7 -b, --backup 创建备份，也就是对于目的已经存在有同样的文件名时，将老的文件重新命名为~filename。可以使用--suffix选项来指定不同的备份文件前缀。
 8 --backup-dir 将备份文件(如~filename)存放在在目录下。(其实就是将变动文件进行back)
 9 -suffix=SUFFIX 定义备份文件前缀
10 -u, --update 仅仅进行更新，也就是跳过所有已经存在于DST，并且文件时间晚于要备份的文件。(不覆盖更新的文件)
11 -l, --links 保留软链结
12 -L, --copy-links 想对待常规文件一样处理软链结
13 --copy-unsafe-links 仅仅拷贝指向SRC路径目录树以外的链结
14 --safe-links 忽略指向SRC路径目录树以外的链结
15 -H, --hard-links 保留硬链结
16 -p, --perms 保持文件权限
17 -o, --owner 保持文件属主信息
18 -g, --group 保持文件属组信息
19 -D, --devices 保持设备文件信息
20 -t, --times 保持文件时间信息
21 -S, --sparse 对稀疏文件进行特殊处理以节省DST的空间
22 -n, --dry-run现实哪些文件将被传输
23 -W, --whole-file 拷贝文件，不进行增量检测
24 -x, --one-file-system 不要跨越文件系统边界
25 -B, --block-size=SIZE 检验算法使用的块尺寸，默认是700字节
26 -e, --rsh=COMMAND 指定使用rsh、ssh方式进行数据同步
27 --rsync-path=PATH 指定远程服务器上的rsync命令所在路径信息
28 -C, --cvs-exclude 使用和CVS一样的方法自动忽略文件，用来排除那些不希望传输的文件
29 --existing 仅仅更新那些已经存在于DST的文件，而不备份那些新创建的文件
30 --delete 删除那些DST中SRC没有的文件
31 --delete-excluded 同样删除接收端那些被该选项指定排除的文件
32 --delete-after 传输结束以后再删除
33 --ignore-errors 及时出现IO错误也进行删除
34 --max-delete=NUM 最多删除NUM个文件
35 --partial 保留那些因故没有完全传输的文件，以是加快随后的再次传输
36 --force 强制删除目录，即使不为空
37 --numeric-ids 不将数字的用户和组ID匹配为用户名和组名
38 --timeout=TIME IP超时时间，单位为秒
39 -I, --ignore-times 不跳过那些有同样的时间和长度的文件
40 --size-only 当决定是否要备份文件时，仅仅察看文件大小而不考虑文件时间
41 --modify-window=NUM 决定文件是否时间相同时使用的时间戳窗口，默认为0
42 -T --temp-dir=DIR 在DIR中创建临时文件
43 --compare-dest=DIR 同样比较DIR中的文件来决定是否需要备份
44 -P 等同于 --partial
45 --progress 显示备份过程
46 -z, --compress 对备份的文件在传输时进行压缩处理
47 --exclude=PATTERN 指定排除不需要传输的文件模式
48 --include=PATTERN 指定不排除而需要传输的文件模式
49 --exclude-from=FILE 排除FILE中指定模式的文件
50 --include-from=FILE 不排除FILE指定模式匹配的文件
51 --version 打印版本信息
52 --address 绑定到特定的地址
53 --config=FILE 指定其他的配置文件，不使用默认的rsyncd.conf文件
54 --port=PORT 指定其他的rsync服务端口
55 --blocking-io 对远程shell使用阻塞IO
56 -stats 给出某些文件的传输状态
57 --progress 在传输时现实传输过程
58 --log-format=formAT 指定日志文件格式
59 --password-file=FILE 从FILE中得到密码
60 --bwlimit=KBPS 限制I/O带宽，KBytes per second
61 -h, --help 显示帮助信息

###### 使用实例

```shell
rsync -a -z -P -v -e ssh /mnt/ root@127.0.0.1:/opt/
rsync -a -z -P -v /mnt/ /opt/dir/
```

###### rsync服务器配置

​	配置文件

```shell
uid = root                         
gid = root                             
use chroot = no                             
read only = yes
hosts allow=*
hosts deny=*
max connections = 5
pid file = /var/run/rsyncd.pid
secrets file = /etc/rsyncd/rsyncd.secrets #用户密码文件
motd file = /etc/rsyncd/rsyncd.motd		 #提示信息文件
log file = /var/log/rsync.log
transfer logging = yes
log format = %t %a %m %f %b
syslog facility = local3
timeout = 300

[mnt]
path = /mnt/
list=yes
ignore errors
auth users = mnt
comment = mnt

[shell]
path = /shell/
list=yes
ignore errors
auth users = shell
comment = shell
```

​		客户端测试

```shell
rsync -v rsync://127.0.0.1	#查看rsyncd服务器输出的模块列表
rsync -a -v -P -z mnt@127.0.0.1::mnt /shell	#测试同步
echo "123" > /tmp/rsyncd.passwd	&& chmod 600 /tmp/rsyncd.passwd			#将rsyncd指定账户的密码追加的/tmp(硬性要求 只能时/tmp目录) 并修改权限为600
rsync -a -v -P -z --password-file=/tmp/rsyncd.passwd mnt@127.0.0.1::mnt /shell
注：服务的好处就是不用基于ssh协议 可以使用rsync协议 873 端口传输 缺点就是密码是明文 相对不安全。
```

