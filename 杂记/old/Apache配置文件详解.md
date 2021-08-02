**ServerTokens OS**                #在出现错误页的时候是否显示服务器操作系统名称ServerTokens Prod为不显示。

**ServerRoot "/etc/httpd"**      #用于指定Apache的运行目录，服务启动之后自动将目录改变为当前目录，在后面使用到的所有相对路径都是相对这个目录下。

**User apache**                        #apache的用户，部分版本默认为daemon。

**Group apache**                     #apache的用户组，部分版本默认为daemon。

**PidFile run/httpd.pid**          #记录httpd守护进程的PID号码，这是系统识别一个进程的方法，系统中httpd进程可以有多个，但这个PID对应的进程是其他的父进程

**Timeout 60**                          #服务器与客户端超时断开的时间

**KeepAlive off**                      #是否持续连接（因为每次连接都得三次握手，如果访问量不大，建议打开此项，如果网站访问量比较大关闭此项比较好） 修改为：KeepAlive on 表示允许持续性联机。

**MaxkeepAliveRequests 100** #表示一个持续连接的最大请求数

**KeepAliveTimeout 15**           #服务器与客户端持续连接超时断开时间

RLimitCPU 6 6                       #控制Apache进程占用CPU的时间（不知道6 6 啥意思 百度没查到 暂且认为软限制 和硬限制把）

RLimitMEM 810010000 810010000 #控制Apache进程占用CPU的内存大小

RLimitNPROC 10 20             #每个用户并发进程最大数量



<IfModule itk.c>

**StartServers 8**                       #StartServer开始服务时启动8个进程

**MinSpareServers 5**               #最小空闲进程5个进程

**MaxSpareServers 20**            #最多空闲进程20个进程

**ServerLimit 256**                    #服务器允许配置进程数的上线

**MaxClients 256**                     #限制同一时刻客户端的最大连接请求数量超过的要进入等候队列

**MaxRequestsPerChild 4000** #每个进程生存期内允许服务的最大请求数量，0表示永不结束

</IfModule>

ps ： itk模式的配置



LoadModule   ~                       #加载模块





以下内容来自网络

本指令 



一、一般的配置命令 



1、AccessFileName 



默认值：AccessFileName .htaccess 



此命令是针对目录的访问控制文件的名称； 



2、BindAddress 



默认值：BindAddress * 



设置服务器监听的IP地址； 



3、DefaultType 



默认值：DefaultType text/html 



服务器不知道文件类型时，用缺省值通知客户端； 



4、DocumentRoot 



默认值：DocumentRoot “/var/www/html/” 



设置Apache提供文件服务的目录； 



5、ErrorDocument 



设置当有问题发生时，Apache所做的反应； 



6、<IfModule>; 



使用不包含在Apache安装中的模块的命令 



7、Include 



包含其它的配置文件 



8、Listen 



默认值：所有能够连接到服务器的IP地址 



指定如何响应除去Port指定的端口地址外的地址请求； 



9、Options 



控制某个特定目录所能使用的服务器功能； 



其值有： 



None：表示只能浏览， 



　　FollowSymLinks：允许页面连接到别处， 



　　ExecCGI：允许执行CGI， 

　　 MultiViews：允许看动画或是听音乐之类的操作， 



　　Indexes：允许服务器返回目录的格式化列表， 



　　Includes：允许使用SSI。这些设置可以复选。 



　　All：则可以做任何事，但不包括MultiViews。 



　　AllowOverride：加None参数表示任何人都可以浏览该目录下的文件。 

　　 另外的参数有：FileInfo、AuthConfig、Limit。 



10、Port 



默认值：Port 80 



设置服务器监听的网络端口； 



11、ServerAdmin 



设定管理员的电子邮件地址； 



12、ServerName 



设定服务器的主机名称； 



13、ServerRoot 



默认值：ServerRoot /etc/httpd/ 



设定服务器的根目录； 



14、User && Group 



指定服务器用来回答请求的用户ID和组ID； 



二、性能和资源配置命令 



1、进程控制 



1.1、MaxClients 



默认值：MaxClients 150 



设定能同时服务的请求数目； 



1.2、MaxRequestsPerChild 



默认值：MaxRequestsPerChild 0 



设置每个进程能够响应的最大请求数，0表示不限制； 



1.3、MaxSpareServers 



默认值：MaxSpareServers 10 



设定最大空闲服务进程数目；MinSpareServers设定最小空闲服务进程数目； 



1.4、ServerType 



默认值：ServerType standalone 



设定系统执行服务器的方式； 



1.5、StartServer 



默认值：StartServer 5 



服务器启动时建立的子进程的数目； 



1.6、ThreadsPerChild 



默认值：ThreadsPerChild 50 



设定Apache服务器使用的线程数； 



1.7、Timeout 



默认值：Timeout 300 



设定处理一个请求的超时值； 



2、建立持续连接 



2.1、KeepAlive 



默认值：KeepAlive ON 



设定在Apache中打开或者关闭TCP连接 



2.2、KeepAliveTimeout 



默认值： KeepAliveTimeout 5 



设定在Apache关闭TCP连接等待的时间； 



2.3、MaxKeepAliveRequests 



默认值：MaxKeepAliveRequests 100 



设定持续连接时每个连接的最大请求数； 



3、控制系统资源 



3.1、RLimitCPU 



控制Apache进程占用CPU的时间； 



3.2、RLimitMEM 



控制Apache进程的内存占用量； 



3.3、RLimitNPROC 



设定每个用户并发进程的最大数目； 



4、动态装载模块 



4.1、AddModule 



装入当前服务器不使用的预编译模块； 



4.2、ClearModuleList 



清除预编译模块列表； 



三、标准容器命令 



1、<Directory>; 



<Directory>;和</Directory>;容器指令，应用到指定的目录及其子目录上； 



2、<DirectoryMatch>; 



<DirectoryMatch>;和</DirectoryMatch>;除了使用规则表达式作为参数和不需要通配符～之外，和 



<Directory>;类似； 



3、<File>; 



<File>;和<File>;容器用来设置通过文件名访问； 



4、<FilesMatch>; 



和<File>;类似，但不能使用通配符～； 



5、<Location>; 



<Location>;和</Location>;容器命令用来提供通过URL的访问控制。 



6、<LocationMatch>; 



<LocationMatch>;和<Location>;相同，只是不能使用通配符～。 



四、虚拟主机命令 



1、<VirtualHost>; 



定义特定的虚拟主机。 



2、NameVirtualHost 



如果使基于名称的虚拟主机，则要使用此命令。 



3、ServerAlias 



针对含有多个IP名称的基于名称的虚拟主机。可以使用单独的虚拟主机命令定义所有的IP名称。 



五、日志命令 



  \1. 



​    ErrorLog 



默认值：ErrorLog logs/error_log 



设定错误日志文件。 



  \2. 



​    LockFile 



默认值：LockFile logs/accept.lock 



用来设置锁文件的路径，确保只有Apache服务器才拥有此文件的读写权限。 



  \3. 



​    PidFile 



默认值：PidFile logs/httpd.pid 



设定Apache服务器记录守护进程的进程ID的文件。 



  \4. 



​    ScoreBoardFile 



默认值：ScoreBoardFile logs/apache_status 



设置存储内部进程数据文件的路径。 



六、认证安全命令 



  \1. 



​    AllowOverride 



默认值：AllowOverride All 



当服务器找到AccessFileName指定的文件时，需要知道该文件中的那些指令可以覆盖在配置文件中稍早出现的指令。 



AllowOverride可以设为None，此时不会读取该文件的内容，如果设为All，服务器将允许所有的指令。 



  \2. 



​    AuthName 



为某个要求认证的资源设定标号。 



  \3. 



​    Authtype 



为某个目录选择使用的认证类型。 



  \4. 



​    HostNameLookups 



默认值：HostNameLookups Off 



设置Apache允许或者禁止为每个请求而进行DNS查找。 



5、<Limit>; 



<Limit>;和<Limit>;包装起来的指令只应用到指定的任何有效的HTTP存取方式上。 



6、require 



这个指令选择经过验证可以存取目录的使用者。 



7、Satisfy 



如果已经建立了基本的HTTP认证配置，且同时使用了allow和require命令，则可用使用此命令来配置Apache在什么条件才能满足认证的要求。 





标准模块 



  \1. 



​    mod_access 



提供以客户端的主机名称或者IP地址为基础的存取控制。 



1.1、allow 



典型应用是在<Limit>;容器中，用来允许符合条件的主机存取。 



1.2、deny 



典型应用是在<Limit>;容器中，用来禁止符合条件的主机存取。 



1.3、order 



用来控制allow指令和deny指令的次序。 



  \2. 



​    mod_actions 



能够提供基于MIME类型的CGI脚本或HTTP请求方法。 



2.1、Action 



对特定的MIME-type指定一个操作。 



2.2、Script 



此命令和Action相似，但不是将操作和MIME-type联系起来，而是将操作和HTTP的请求方法联系起来，如GET、PUT等。 



  \3. 



​    mod_alias 



3.1、Alias 



该指令可以是文件存放在DocumentRoot之外的本地文件系统里。 



3.2、Redirect 



该指令将旧的URL重定向到新的URL。 



3.3、RedirectMatch 



与Redirect类似，但它使用规则表达式而不是简单的URL。 



3.4、RedirectTemp 



和Redirect类似，它让客户端知道重定向只是临时的。 



3.5、RedirectPermanent 



和Redirect类似，它让客户端知道重定向是永久的。 



3.6、ScriptAlias 



该指令除了把目标目录标记为包含CGI脚本以外，和Alias功能相同。 



3.7、ScriptAliasMatch 



该指令出使用规则表达式以外，和ScriptAlias命令相似。 



  \4. 



​    mod_asis 



允许定义文件的类型，这样，Apache可以不加HTTP头标（headers）传送它们。 



  \5. 



​    mod_auth 



5.1、AuthGroupFile 



该指令用来设置包含用来执行用户验证的使用者组列表的文本文件。 



5.2、AuthUserFile 



该指令用来设置包含用来执行用户验证的使用者以及密码列表的文本文件。 



  \6. 



​    mod_auth_anno 



允许以匿名方式访问需要认证的区域。 



  \7. 



​    mod_auth_db 



用户认证数据库 



  \8. 



​    mod_auth_external 



使Apache支持第三方认证。 



  \9. 



​    mod_autoindex 



当由DriectoryIndex指定的索引文件不存在时，该模块使Apache生成动态模块列表。 



9.1、AddAlt 



当FancyIndexing处于打开状态时，此命令设置代替图标的正文。 



9.2、AddAltByEncoding 



当FancyIndexing处于打开状态时，此命令用来为一个或多个MIME-encoding指定正文。 



9.3、AddAltByType 



此命令用于为文件设置代用正文以代替用于FancyIndexing的图标。 



9.4、AddDescription 



该命令设定某文件要显示的描述，供象征索引（FancyIndexing）使用。 



9.5、AddIcon 



该命令设定显示在文件名之后的图标供象征索引使用。 



9.6、AddIconByEncoding 



该命令设定显示在有MIME-encoding的文件名之后的图标供象征索引使用。 



9.7、AddIconByType 



该命令设定显示在MIME-type的文件名之后的图标供象征索引使用。 



9.8、DefaultIcon 



设定不知道指定的图标是什么的时候显示的图标。 



9.9、DirectoryIndex 



设定当客户端没有指定以文件形式结尾的目录名称请求该目录的索引时所要找寻的来源列表。 



9.10、FancyIndexing 



设定目录的象征索引列表。 



9.11、HeaderNmae 



设定插入索引列顶部的文件名。 



9.12、IndexIgnore 



设定列目录时需要隐藏的文件。 



9.13、IndexOptions 



设定进行目录索引时的选项。 



9.14、ReadmeName 



设定要附加到索引文件后面的文件名。 



  \10. 



​    mod_cgi 



支持CGI。 



  \11. 



​    mod_dir 



其唯一的指令是DirectoryIndex，设定当客户端没有指定以文件形式结尾的目录名称请求该目录的索引时所要找寻的来源列表。 



  \12. 



​    mod_env 



将环境变量传递给CGI或SSI脚本。 



12.1、PassEnv 



此命令告诉模块从服务器的环境中传递一个或多个环境变量到CGI或SSI脚本。 



12.2、SetEnv 



设置环境变量，然后传递给CGI/SSI脚本。 



12.3、UnsetEnv 



从传递给CGI/SSI脚本的环境变量中删除一个或多个环境变量。 



  \13. 



​    mod_imap 



提供图形映射支持。 



  \14. 



​    mod_include 



使支持SSI。 



  \15. 



​    mod_log_config 



支持记录日志。 



  \16. 



​    mod_mime 



提供从文件名决定文件了类型的功能。 



16.1、AddEncoding 



该指令以指定的编码类型把可能作为文件名结尾的扩展名加入文件扩展名列表。 



16.2、AddLanguage 



该指令以指定的语言把可能作为文件名结尾的扩展名加入文件扩展名列表。 



16.3、AddType 



该指令以指定的内容类型把可能作为文件名结尾的扩展名加入文件扩展名列表。 



16.4、TypesConfig 



指定MIME类型配置文件所在的位置。 



  \17. 



​    mod_negotiation 



提供对内容协商的支持。 



  \18. 



​    mod_setenvif 



使你可以创建定制环境变量。 



18.1、BrowserMatch 



此命令用来在规则表达式与模式匹配时设置黄和删除定制环境变量。 



18.2、BrowserMatchNoCase 



在BrowserMatch的功能之外还提供与大小写无关的匹配。 



18.3、SetEnvIf 



能够设置和删除定制的环境变量，可用于所有的请求标题字段。 



18.4、SetEnvIfNoCase 



在SetEnvIf功能之外还提供与大小写无关的规则表达式匹配。 



  \19. 



​    mod_unique_id 



该模块为每个请求提供在非常特殊的条件下保证是唯一的标识符。 



  \20. 



​    mod_userdir 



该模块的指令是UserDir，提供代表使用者的目录。 





扩展模块测试 



  \21. 



​    mod_auth_dbm 



支持使用DBM文件存储基本HTTP认证。 



21.1、AuthDbmGroupFile 



设定包含代验证用户名组列表的文件名。 



21.2、AuthDbmUserFile 



设定包含代验证用户名和密码列表的文件名。 



  \22. 



​    mod_auth_digest 



使用MD5算法来进行用户的认证工作。使用它时，就不能使用mod_digest模块。 



  \23. 



​    mod_cern_meta 



提供对元信息的支持，元信息可以是HTTP头标。 



23.1、MetaFiles 



此命令允许或禁止元标题文件处理。 



23.2、MetaDir 



此命令用来指定存储元标题文件的目录的名称。 



23.3、Metasuffix 



此命令为元信息文件指定文件扩展名。 



  \24. 



​    mod_cookies 



唯一的指令是CookieLog，用来设定记录cookies用的文件名。 



  \25. 



​    mod_digest 



唯一的指令是AuthDigestFile，用来设定包含用户名和密码列表的文本文件的文件名。 



  \26. 



​    mod_dld 



该模块提供在启动时载入可执行文件及模块到服务器里去的功能。 



26.1、LoadFile 



该指令在服务器启动时链接其所指的目的文件或程序库。 



26.2、LoadModule 



该指令链接目的文件或程序库的文件名并且把所指定的模块加入到使用中的模块列表。 



  \27. 



​    mod_example 



唯一指令是Example，该指令设置示例模块的内容句柄显示的说明标志。 



  \28. 



​    mod_expires 



让你确定服务器在响应请求时如何处理Expires HTTP标题。 



28.1、ExpiresActive 



禁止或允许Expires标题生成。 



28.2、ExpiresByType 



指定MIME类型文档的Expires HTTP标题值。 



28.3、ExpiresDefault 



为指定的使用范围内的所有文档设置缺省的过期时间。 



  \29. 



​    mod_headers 



操作HTTP应答标题；提供单独的名为Header的命令。 



  \30. 



​    mod_info 



该模块对服务器的配置提供了全面的描述，其中包括所有安装的模块及其在配置文件中使用的命令；唯一的命令是AddModuleInfo。 



  \31. 



​    mod_log_angent 



允许在单独的日志中存储用户代理的信息。 



31.1、AgentLog 



默认值：AgentLog logs/agent_log 



设定服务器记录及进入请求的文件名。 



  \32. 



​    mod_log_referrer 



提供了将请求中Referer头标写入日志的功能。 



  \33. 



​    mod_nmap_static 



提供经常访问的不改变的文件的列表。 



33.1、MmapFile 



该指令在Apache启动时，将一个或者多个文件映射进内存，Apache关闭时自动删除映射。 



  \34. 



​    mod_proxy 



提供对代理的支持。 



34.1、ProxyRequests 



默认值：ProxyRequests off 



开启或关闭代理服务。 



34.2、ProxyRemote 



将自己的代理服务器匹配接口于另一个代理服务器。 



34.3、ProxyPass 



把一个代理服务器的文档树映射到另一个代理服务器的文档空间。 



34.4、ProxyPa***everse 



建立反向代理。 



34.5、ProxyBlock 



屏蔽向某一主机或域发出的请求。 



34.6、NoProxy 



在内部网络环境下，对ProxyRemote指令产生一些控制，可以指定一个域名、子网、IP地址或主机名，让ProxyRemote指令所指定的代理服务器不对它们作处理服务。 



34.7、ProxyDomain 



设定代理服务器的缺省域名。 



34.8、ProxyReceiveBufferSize 



给所有代理服务器发出的请求设定了网络缓冲大小。 



34.9、CacheRoot 



打开磁盘缓冲功能，必须指定一个目录，以便代理服务器能够写入被缓存的文件。 



34.10、CacheSize 



默认值：CacheSize 5 



指定用作缓存的磁盘空间的大小。 



34.11、CacheGcInterval 



指定Apache每隔多长时间检查缓存目录，删除过期文件。 



34.12、CacheMaxExpire 



默认值：CacheMaxExpire 24 



指定缓存文档的过期时间，以小时计。 



34.13、CacheLastModifiedFactor 



默认值：CacheLastModifiedFactor 0.1 



指定了认为文档过期的时间系数。 



34.14、CacheDirLenth 



指定缓存文件的路径下子目录使用的字符数。 



34.15、CacheDirLevels 



默认值：CacheDirLevels 3 



指定缓存数据文家的子目录的层数。 



34.16、CacheDefaultExpire 



默认值：CacheDefaultExpire 1 



当不知道文件的最后修改时间，该指令提供了缺省的过期时间，以小时计。 



34.17、CacheForceCompletion 



默认值：CacheForceCompletion 90 



该指令告诉代理服务器，即使请求已经取消，也要续传从远端服务器发出的文档。后面的数字表示百分比，也就是文档的90%已经缓存时就续传。 



34.18、NoCache 



该指令指定了由空格分隔的主机、域名、IP地址的列表，对于列表中的对象不执行缓存操作。 



  \35. 



​    mod_rewrite 



提供URL重写功能。 



35.1、RewriteEngine 



默认值：RewriteEngine off 



提供了mod_rewrite模块内的URL重写引擎开关。 



35.2、RewriteOptions 



用指定的选项改变重写引擎的属性。 



35.3、RewriteRule 



定义重写规则。 



35.4、RewriteCond 



给RewriteRule指令定义的重写规则添加额外的条件。 



35.5、RewriteMap 



利用映射图方便了关键字到值的查找。 



35.6、RewriteBase 



仅当在每一路径配置文件中使用重写规则时才起作用。 



35.7、RewriteLog 



设定重写的日志。 



35.8、RewriteLoglevel 



默认值：RewriteLoglevel 0 



指定重写文件中记录什么类容，0表示不记录任何内容。 



35.9、RewriteLock 



如果想用外部映射程序来生成重写映射图，就可以使用该指令指定一个用作同外部映射程序同步通信的锁定文件。



  \36. 



​    mod_so 



提供在服务器启动时装载可执行代码和模块的功能。 



36.1、LoadFile 



在服务器启动时链接目标文件或库，还可以用来加载一些模块工作时必须的代码。 



36.2、LoadModule 



该指令链接目标文件或库，并将模块添加到活动模块列表中。 



  \37. 



​    mod_speling 



处理含有错误拼写或错误大小写的URL请求。 



  \38. 



​    mod_status 



允许管理员通过Web监视Apache。 



  \39. 



​    mod_usertrack 



该模块用来产生记录用户在一个站点使用cookies活动的“clickstream”日志。 



39.1、CookieExpires 



该指令设置mod_usertrack模块生成的cookie的生存时间，以秒计。 



39.2、CookieName 



默认值：CookieName Apache 



让用户更改用于跟踪目的的cookie的名称。 



39.3、CookieTracking 



对所有的请求，Apache均发送一个用户跟踪的cookie,该指令可用来在服务器或者目录的基础上禁止该行为。 



  \40. 



​    mod_vhost_alias 



支持动态配置批量虚拟主机。 



40.1、VirtualDocumentRoot 



设置基于服务器名的Apache文档。 



40.2、VirtualDocumentRootIP 



除了用IP地址代替主机名以外，其它功能和VirtualDocumentRoot类似。 



40.3、VirtaulScriptAlias 



该指令告诉Apache在什么地方找到CGI脚本。 



40.4、VirtaulScriptAliasIP 



除了用IP地址代替主机名以外，其它功能和VirtaulScriptAlias类似。



## 配置文件示例

```shell
ServerTokens Prod
ServerRoot "/usr/local/apache2"
PidFile logs/httpd.pid
Timeout 20
KeepAlive On
EnableSendfile on
MaxKeepAliveRequests 100
KeepAliveTimeout 15

#RLimitMEM 810010000 810010000
RLimitCPU 6 6
RLimitNPROC 10 20

<IfModule itk.c>
    StartServers       30
    MinSpareServers    30
    MaxSpareServers   50
    ServerLimit      1500
    MaxClients       1500
    MaxRequestsPerChild  10000
</IfModule>

ServerName localhost:80
Listen 80
Listen 8000

LoadModule auth_basic_module modules/mod_auth_basic.so
#LoadModule auth_digest_module modules/mod_auth_digest.so
LoadModule authn_file_module modules/mod_authn_file.so
LoadModule authn_alias_module modules/mod_authn_alias.so
#LoadModule authn_dbm_module modules/mod_authn_dbm.so
LoadModule authn_default_module modules/mod_authn_default.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule authz_user_module modules/mod_authz_user.so
LoadModule authz_owner_module modules/mod_authz_owner.so
#LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
LoadModule authz_default_module modules/mod_authz_default.so
LoadModule include_module modules/mod_include.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule logio_module modules/mod_logio.so
LoadModule env_module modules/mod_env.so
LoadModule ext_filter_module modules/mod_ext_filter.so
LoadModule mime_magic_module modules/mod_mime_magic.so
LoadModule expires_module modules/mod_expires.so
LoadModule deflate_module modules/mod_deflate.so
LoadModule headers_module modules/mod_headers.so
LoadModule usertrack_module modules/mod_usertrack.so
LoadModule setenvif_module modules/mod_setenvif.so
LoadModule mime_module modules/mod_mime.so
#LoadModule dav_module modules/mod_dav.so
LoadModule status_module modules/mod_status.so
LoadModule autoindex_module modules/mod_autoindex.so
LoadModule info_module modules/mod_info.so
#LoadModule dav_fs_module modules/mod_dav_fs.so
LoadModule vhost_alias_module modules/mod_vhost_alias.so
LoadModule negotiation_module modules/mod_negotiation.so
LoadModule dir_module modules/mod_dir.so
LoadModule actions_module modules/mod_actions.so
LoadModule speling_module modules/mod_speling.so
LoadModule userdir_module modules/mod_userdir.so
LoadModule alias_module modules/mod_alias.so
LoadModule substitute_module modules/mod_substitute.so
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule cgi_module modules/mod_cgi.so
LoadModule fastcgi_module modules/mod_fastcgi.so
LoadModule cband_module modules/mod_cband.so

User apache
Group apache

ServerAdmin root@localhost
UseCanonicalName Off
DocumentRoot "/var/www/html/error"

<ifmodule mod_deflate.c>
DeflateCompressionLevel 9
SetOutputFilter DEFLATE
AddOutputFilterByType DEFLATE text/html text/plain text/css application/x-httpd-                                                                                                 php text/javascript application/x-javascript text/css
AddOutputFilter DEFLATE js css
</ifmodule>

<ifmodule mod_headers.c>
SetEnvIf Range (,.*?){5,} bad-range=1
RequestHeader unset Range env=bad-range
</ifmodule>

<Directory /webHome>
    Options FollowSymLinks
    AllowOverride All
    RewriteEngine on
    RewriteCond %{QUERY_STRING} ^[^=]*$
    RewriteCond %{QUERY_STRING} %2d|\- [NC]
    RewriteRule ^(.*) $1? [L]
    RewriteCond %{HTTP:range} !(^bytes=[^,]+(,[^,]+){0,4}$|^$)
    RewriteRule .* - [F]
</Directory>

<Directory "/var/www/html/error">
    Options FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>

<Directory "/var/www/error/stop">
   Options FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>

<Directory "/usr/share/awstats/wwwroot">
        Options FollowSymLinks
        RewriteEngine on                                                                                                                                                         
    RewriteCond %{QUERY_STRING} ^[^=]*$                                                                                                                                          
    RewriteCond %{QUERY_STRING} %2d|\- [NC]                                                                                                                                      
    RewriteRule ^(.*) $1? [L]                                                                                                                                                    
    RewriteCond %{HTTP:range} !(^bytes=[^,]+(,[^,]+){0,4}$|^$)                                                                                                                   
    RewriteRule .* - [F]
</Directory>

<IfModule mod_userdir.c>
    UserDir disabled
</IfModule>

DirectoryIndex index.php index.html index.html.var
AccessFileName .htaccess

<Files ~ "^\.ht">
    Order allow,deny
    Deny from all
    Satisfy All
</Files>

TypesConfig conf/mime.types
DefaultType text/plain

<IfModule mod_mime_magic.c>
    MIMEMagicFile conf/magic
</IfModule>

HostnameLookups Off
ErrorLog /webHome/webLog/syslog/error_log
LogLevel warn
#LogFormat "%v %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" com                                                                                                 bined
LogFormat "%h \"%{X-FORWARDED-FOR}i\" \"%{REMOTE_PORT}e\" %l %u %t \"%r\" %>s %b                                                                                                  \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h \"%{X-FORWARDED-FOR}i\" \"%{REMOTE_PORT}e\" %l %u %t \"%r\" %>s %b                                                                                                 " common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
SetEnvIf Remote_Addr "::1" loopback
LogFormat "%v|%{%F %T}t|%m %I %O|%v %h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"                                                                                                 %{User-Agent}i\"" netiolog2
#CustomLog "|/shell/logger" netiolog2 env=!loopback
#CustomLog "|/usr/local/sbin/cronolog /webHome/%v/log/access_log.%Y%m%d" netiolo                                                                                                 g2 env=!loopback
ServerSignature Off

Alias /logs/ "/webHome/webLog/"
Alias /icons/ "icons/"
Alias /zhengju/ "/var/www/html/zhengju/"
Alias /gethost/ "/var/www/html/"
<Directory "/usr/local/apache2/icons">
    Options MultiViews FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>

<IfModule mod_dav_fs.c>
    # Location of the WebDAV lock database.
    DAVLockDB /var/lib/dav/lockdb
</IfModule>

ScriptAlias /cgi-bin/ "/usr/local/apache2/cgi-bin/"

<Directory "/usr/local/apache2/cgi-bin">
    Options MultiViews FollowSymLinks
    AllowOverride all
    Options None
    Order allow,deny
    Allow from all
</Directory>

IndexOptions FancyIndexing VersionSort NameWidth=* HTMLTable Charset=UTF-8
AddIconByEncoding (CMP,/icons/compressed.gif) x-compress x-gzip
AddIconByType (TXT,/icons/text.gif) text/*
AddIconByType (IMG,/icons/image2.gif) image/*
AddIconByType (SND,/icons/sound2.gif) audio/*
AddIconByType (VID,/icons/movie.gif) video/*
ReadmeName README.html
HeaderName HEADER.html
IndexIgnore .??* *~ *# HEADER* README* RCS CVS *,v *,t
#AddDefaultCharset UTF-8
AddType application/x-compress .Z
AddType application/x-gzip .gz .tgz
AddHandler type-map var
AddType text/html .shtml
AddOutputFilter INCLUDES .shtml
AddType text/vnd.wap.wml .wml
AddType image/vnd.wap.wbmp .wbmp
AddType application/vnd.wap.wmlc .wmlc
AddType text/vnd.wap.wmls .wmls
AddType application/vnd.wap.wmlsc .wmlsc

Alias /error/ "/var/www/html/error/"

<IfModule mod_negotiation.c>
    <IfModule mod_include.c>
        <Directory "/var/www/html/error">
            AllowOverride None
            Options IncludesNoExec
            AddOutputFilter Includes html
            AddHandler type-map var
            Order allow,deny
            Allow from all
            LanguagePriority en es de fr
            ForceLanguagePriority Prefer Fallback
        </Directory>
    </IfModule>
</IfModule>

NameVirtualHost *:80

<VirtualHost *:80>
ErrorDocument 403 /error/xinnet.html
ErrorDocument 404 /error/xinnet.html
ErrorDocument 400 " security test"
AddHandler php-fpm .php
AddType application/x-httpd-php .php
Action php-fpm /php5-fcgi/php-fpm-test
ScriptAlias /php5-fcgi /php/phpfarm-0.1.0/inst/bin
FastCGIExternalServer /php/phpfarm-0.1.0/inst/bin/php-fpm-test -host 127.0.0.1:9                                                                                                 000
</VirtualHost>
<VirtualHost *:8000>
ErrorDocument 403 /error/xinnet.html
ErrorDocument 404 /error/xinnet.html
ErrorDocument 400 " security test"
</VirtualHost>
TraceEnable off
include /vhost/*.conf
RewriteEngine On
RewriteRule ^(.*)-htm-(.*)$ $1.php?$2

```

