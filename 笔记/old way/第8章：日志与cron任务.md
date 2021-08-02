​         
日志类型
​	auth			#安全认证相关信息，pam产生的日志
​	authpriv		#安全和认证信息（私有的），ssh，ftp等登陆信息的验证信息
​	corn			#系统定时任务cron和at产生的日志
​	kern			#内核产生的日志（不是用户进程产生的）
​	lpr				#打印产生的日志
​	mark（syslog）	#-rsyslog服务内部的信息，时间标识
​	news			#新闻组
​	user			#用户程序产生的相关信息
​	uucp unix to unix copy	#unix主机之间相关通讯
​	mail			#邮件
​	local0-6		#系统启动日志
​	
连接符
.		#只要比指定等级高的都记录下来
.=		#只记录等于指定等级的日志
.!		#代表不等于指定等级的日志都记录



日志等级

​	优先级					说明
​	none			不记录任何日志消息
​	emerg			紧急情况，系统不可用（如内核崩溃），一般通知所有用户
​	alert			需要立即修复的信息
​	crit			严重级别，组织整个系统或者整个软件不能正常工作的信息，例如硬盘错误。
​	err				错误，一般错误信息。组织某个功能或者模块不能正常工作的信息
​	warring			警告
​	notice			具有重要性普通文件的信息。不是错误，但是可能需要处理
​	info			通用性消息，一般用来提供有用信息
​	debug			调试程序产生的信息，日志信息最多
​	
​	
rsyslog日志服务	

日志存放路径/var/log
/etc/rsyslog.conf 或 /etc/rsyslog.d		#rsyslog服务的配置文件和分割文件

/var/log/dmesg							#记录了系统在开机时内核自检的信息，使用dmesg命令查看

/var/log/btmp							#记录错误的登录日志，这个文件是二进制文件，使用lastb命令查看

/var/log/lastlog						#记录系统中所有用户最后一次登陆的日志，使用lastlog命令查看

/var/log/message						#记录系统重要信息的日志，记录Linux系统的绝大多数重要信息，如果系统出现问题，首先要检查的就是这个日志文件

/var/run/utmp							#永久记录所有用户登陆，注销信息，同时记录系统的启动，重启，关机事件，使用last命令来查看

/var/run/utmp							#记录当前已经登陆的用户信息，这个文件会随着用户的登陆和注销不断地变化，要使用w，who，users等命令查看。


注：lastb/last
• -a 　把从何处登入系统的主机名称或IP地址显示在最后一行。
• -d 　将IP地址转换成主机名称。
• -f<记录文件> 　指定记录文件。
• -n<显示列数>或-<显示列数> 　设置列出名单的显示列数。
• -R 　不显示登入系统的主机名称或IP地址。
• -x 　显示系统关机，重新开机，以及执行等级的改变等信息。
	

修改日志默认存放路径

vi /etc/rsyslog.conf		#修改messages日志存放路径
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
ll -d -Z /opt/				#查看安全上下文
chcon -t var_log_t /opt		#修改/opt的安全上下文
systemctl restart rsyslog


网络日志服务器的配置（将多台服务器的日志放到一台服务器上进行集中管理）

服务器端配置
vi /etc/rsyslog.conf
$Modload imtcp			#引用tcp协议模块
$InputTCPServerRun 514	#指定使用的端口号
$template R,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"	#定义模板
*.*?R					#防止日志二次记录

losf -i:514				#查看端口是否启用
ss -tlanp				#查看端口是否启用

客户端配置
vi /etc/rsyslog.conf
*.* @@192.168.1.1		#指定本主机产生的所有日志都传输到192.168.1.1的TCP 514端口

vi /etc/bashrc			#定义日志的变量，追踪谁，在哪做过什么。
export PROMPT_COMMAND=' { msg=$(history 1 | { read x y; echo $y; });logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'

source /etc/bashrc		#使变量生效


日志转储
/etc/logrotate.conf		#配置文件
/etc/logrotate.d		#分割文件

参数定义

1

compress				#通过gzip压缩转储以后的日志
nocompress				#不必压缩时，用这个参数
delaycompress			#转储的日志文件到下一次转储时才压缩
nodelaycompress			#覆盖delaycompress选项，转储同时压缩

2

copytruncate			#用于还在打开的日志文件，把当前日志备份并截断
nocopytruncate			#备份日志文件但是不截断

3

create mode owner group	#转储文件，使用指定的文件模式创建新的
文件（权限 属主 属组）
nocreate				#不建立新的日志文件

4

errors address			#转储时的错误信息发送到指定的E-mail地址

5

ifempty					#即使是空文件的话也转储，这个是logrotate的缺省选项
notifempty				#如果是空文件的话，不转储

6

maxage count			#只转储保留多天内的日志文件，超过则删除

7

mail address			#把转储产生的日志文件发送到指定的E-mail地址
nomail					#转储时不发送日志文件

8

missingok				#如果日志不存在则忽略该警告信息

9

olddir directory		#转储后的日志文件放到指定的目录，必须和当前文件在同一个文件系统
noolddir				#转储后的日志文件和当前日志文文件放在同一个目录下

10

prerotate				#在转储以前需要执行的命令能放入这个对，这两个关键字必须独立成行
postrotate				#在转储以后需要执行的命令

11

daily					#指定转储周期为每天
weekly					#指定转储周期为每周
monthly					#指定转储周期为每月
rotate count			#指定日志文件删除之前的转储次数 0指没有备份 5 指保留5个备份
size					#当日志文件到达指定大小时才能转储
dateext					#使用日期作为命名格式
dateformat.%s			#配合dateext使用

**修改转储文件**
				
vi /etc/logrotate.d/mail

/var/log/mail/*.log
{
daily			#轮转周期（多久轮转一次）
dateext			#轮转下来的文件以时间命名
copytruncate	#截断转储文件
nocompress		#不压缩转储文件
rotate 3		#保留三份转储下来的日志
ifempty			#空日志文件也转储
}

logrotate /etc/logrotate.d/mail		#根据条件判断，转储符合条件的日志文件
logrotate -d /etc/logrotate.d/mail	#测试转储
logrotate -f /etc/logrotate.d/mail  #强制转储

注：logrotate自身的日志通常存放于/var/lib/logrotate/logrotate.status如果处于排障目的，我们想要logrotate记录到任何指定的文件，我们可以指定像下面这样从命令行指定。
	logrotate -vf -s /var/log/logrotate-status /etc/logrotate.d/log-file

​	logrotate启动脚本放在/etc/cron.daily/logrotate
​	
​	/etc/cron.daily/logrotate默认让Cron每天执行logrotate一次
​	
​	
**corn自动化任务**

Linux任务调度的主要工作分为两类
1.系统执行的工作：系统周期性所执行的工作，如备份系统数据，清理缓存
2.个人执行的工作:某个用户定期要做的工作，例如每隔十分钟检查邮件服务是否有新信，这些工作可由每个用户自行设置。

crontab格式共分为六段，前五段为时间设定段，第六段为所需要执行的命令段

第一段	代表分钟 0-59
第二段	代表小时 0-23
第三段	代表日期 1-31
第四段	代表月份 1-12
第五段	代表星期 0-6（0代表星期日）

/var/spool/cron		#cron的自动化任务文件存放路径
crontab -l			#查看自动化任务（-u 指定用户）
crontab -r			#为当前用户删除自动化任务
crontab -e			#为当前用户编辑自动化任务

配置日志文件自动轮转
 \* * */1 * *	/usr/sbin/logrotate /etc/logrotate.d/mail

 */1	#每一
 1-5	#从1到5
 1		#1
 1,15	#1和15

 分时日月周


注：
/etc/cron.d			#cron分割文件路径 常用来指定系统任务 crond 会读取下面脚本。

*/1 * * * * root run-parts /etc/cron.minly  #须指定执行的文件夹 通过run-parts 来读取指定文件夹 执行指定文件夹下的文件。

（通过crontab -l 无法查询）



cron.minly			#每分钟执行系统任务的文件夹
cron.hourly			#每小时
cron.daily			#每天
cron.weekly			#每周
cron.monthly		#每月
（一些服务会往这个下面写执行脚本 如logrotate）

crontab				#主配置文件
cron.deny			#该文件中所列出的用户不允许使用crontab命令