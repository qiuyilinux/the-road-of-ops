### 简介：

unison双向同步，是一款跨平台的文件同步工具，不仅支持本地同步，也支持ssh rsh socket等网络同步。

### 部署：

```shell
yum install -y ocaml ctags-etags glibc-static glib* inotify-tools  #安装相关依赖包
wget http://www.seas.upenn.edu/~bcpierce/unison//download/releases/stable/unison-2.48.4.tar.gz	-P /mnt #下载源码包
tar zxvf /mnt/unison-2.48.4.tar.gz  -C /mnt	#解压
cd /mnt/src && make UISTYLE=text THREADS=true STATIC=true && cp unison /usr/local/bin/  #编译

实例：
unison  /mnt/  /opt/ #同步两个目录中比较新的文件需要同步输入enter 不需要输入/ 跳过当前
unison /mnt/ /opt/ -auto=true  #默认同步（自动完成确认）
unison /mnt/ /opt/ -auto=true -batch=true #自动应答y
unison /mnt ssh://root@127.0.0.1//mnt -auto=true -batch=true #与远程主机同步
```

注：安全机制，当一个目录为空的时候系统会认为你误操作，跳过删除的文件。

（两台机器之间unison环境需要绝对一致）

### 配置文件

```shell
root = /mnt/                          #路径
root = ssh://root@192.168.1.100//mnt/ #目的路径
#force =  /opt/                       #强制指定以什么为准单向
#ignore = Path aaa                    #忽略同步指定的文件
batch = true                          #自动应答
maxthreads   = 300                    #最线程数量
repeat = 1                            #每1s检查一次同步
retry = 3                             #失败尝试次数
owner = true					    #保持同步过来的文件属主 
group = true					    #保持同步过来的文件组信息
perms = -1						    #保持同步过来的文件读写权限 
repeat = 1						    #失败重试
fastcheck = true					#表示同步时仅通过文件的创建时间来比较，如选项 为false，Unison则将比较两地文件内容
rsync = false
sshargs = -C                          #使用ssh进行压缩传输
xferbycopying = true			     #不变目录 扫描时可以忽略
log = true						    #表示在终端输出运行信息
logfile  = /root/.unison/unison.log	  #指定输出的log文件路径
confirmbigdel = false			     #默认值为true，表示当需要同步的两个目录一个为空时，unison将停止，这里设置为false，即便为空unison也不会停止运转
```

### 实例

```shell
unison default.prf	#执行
nohup unison default.prf &> /dev/null & #后台执行
```

番外：screen会话，在当前终端打开一个会话