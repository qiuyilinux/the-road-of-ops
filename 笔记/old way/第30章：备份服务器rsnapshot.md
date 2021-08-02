# 简介

以perl语言编写开源软件，可以对本地或者远程文件系统备份的实用工具。

ps ：配置文件使用制表符进行分割，如切换成空格可能使服务重启失败。

# 部署

```shell
1.yum install -y rsnapshot.noarch #安装软件包
2.vi /etc/rsnapshot.conf          #修改配置文件
config_version    1.2
snapshot_root    /date/  #指定存放备份的路径
cmd_cp        /usr/bin/cp
cmd_rm        /usr/bin/rm
cmd_rsync    /usr/bin/rsync
cmd_ssh    /usr/bin/ssh
cmd_logger    /usr/bin/logger
cmd_du        /usr/bin/du
retain    alpha    2  #指定不同级别的备份保存的数量
retain    beta        2
retain    gamma    2
retain    delta    2
Verbose             2
loglevel            3
logfile    /var/log/rsnapshot
lockfile    /var/run/rsnapshot.pid
ssh_args    -p 22
exclude_file    /date/file.exclude                  #指定用来排除备份的文件
backup    /home/        localhost/                  #对本地的文件系统进行备份
backup  root@192.168.1.114：/opt              ssh/  #对远程主机文件系统进行备份

3.vi /date/file.exclude
- /home/xinxiaoye

4.ssh-keygen

5.ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.114

6.rsnapshot configtest  #测试配置是否正确

7.rsnapshot -t alpha    #测试创建alpha级别的备份

8.rsnapshot alpha       #直接备份数据

9.rsnapshot beta        #创建beta级别的备份
```

