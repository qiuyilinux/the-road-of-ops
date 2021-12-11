

# 一、 概览



dnsdist 是一个高度识别 DNS、 DoS 和滥用的负载均衡器。

它的目标是将流量路由到最好的服务器，在分流或组织滥用流量的同时为合法用户提供最佳性能。

**官方文档：**   https://dnsdist.org/index.html



# 二、 安装

## Debian

```shell
apt-get install -y dnsdist
```

## Redhat

```shell
yum install -y epel-release
yum install -y dnsdist
```



# 三、 基本使用

## 1 前台运行

```shell
disdist -l 127.0.0.1:5300 9.9.9.9 8.8.8.8 7.7.7.7

# 监听 IP 127.0.0.1
# 监听端口 5300
# 并将所有查询转发到三个列出的 IP 地址，并使用合理的负载均衡策略。
```



## 2 基础配置

```shell
vi /etc/dnsdist/dnsdist.conf

newServer({address="8.8.8.8", name="dns1", qps=10})
newServer({address="114.114.114.114", name="dns2", qps=1})
setServerPolicy(firstAvailable) -- first server within its QPS limit


# name 名称
# address 下游服务器地址
# qps 限制每秒查询量
```



## 3 从配置文件启动

```shell
dnsdist -C /etc/dnsdist/dnsdist.conf --local=0.0.0.0:53
```



## 4 控制台

dnsdist 提供非常多的控制台命令， 可以通过 help 查看

```shell
[root@dns dnsdist]# dnsdist -C /etc/dnsdist/dnsdist.conf --local=0.0.0.0:53
Added downstream server 8.8.8.8:53
Added downstream server 114.114.114.114:53
Listening on 0.0.0.0:53
dnsdist 1.3.3 comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it according to the terms of the GPL version 2
ACL allowing queries from: ::1/128, fc00::/7, 100.64.0.0/10, 169.254.0.0/16, 192.168.0.0/16, 10.0.0.0/8, fe80::/10, 172.16.0.0/12, 127.0.0.0/8
Console ACL allowing connections from: ::1/128, 127.0.0.1/8
Warning, this configuration can use more than 1064 file descriptors, web server and console connections not included, and the current limit is 1024.
You can increase this value by using LimitNOFILE= in the systemd unit file or ulimit.
Marking downstream dns1 (8.8.8.8:53) as 'up'
Marking downstream dns2 (114.114.114.114:53) as 'up'
> Polled security status of version 1.3.3 at startup, no known issues reported: OK
>
> help
addACL(netmask): add to the ACL set who can use this server
addAction(DNS rule, DNS action [, {uuid="UUID"}]): add a rule
addConsoleACL(netmask): add a netmask to the console ACL
addDNSCryptBind("127.0.0.1:8443", "provider name", "/path/to/resolver.cert", "/path/to/resolver.key", {reusePort=false, tcpFastOpenSize=0, interface="", cpus={}}): listen to incoming DNSCrypt queries on 127.0.0.1 port 8443, with a provider name of `provider name`, using a resolver certificate and associated key stored respectively in the `resolver.cert` and `resolver.key` files. The fifth optional parameter is a table of parameters
addDynBlocks(addresses, message[, seconds[, action]]): block the set of addresses with message `msg`, for `seconds` seconds (10 by default), applying `action` (default to the one set with `setDynBlocksAction()`)
addLocal(addr [, {doTCP=true, reusePort=false, tcpFastOpenSize=0, interface="", cpus={}}]): add `addr` to the list of addresses we listen on
addLuaAction(x, func [, {uuid="UUID"}]): where 'x' is all the combinations from `addAction`, and func is a function with the parameter `dq`, which returns an action to be taken on this packet. Good for rare packets but where you want to do a lot of processing
addLuaResponseAction(x, func [, {uuid="UUID"}]): where 'x' is all the combinations from `addAction`, and func is a function with the parameter `dr`, which returns an action to be taken on this response packet. Good for rare packets but where you want to do a lot of processing
addCacheHitResponseAction(DNS rule, DNS response action [, {uuid="UUID"}]): add a cache hit response rule
addResponseAction(DNS rule, DNS response action [, {uuid="UUID"}]): add a response rule
addSelfAnsweredResponseAction(DNS rule, DNS response action [, {uuid="UUID"}]): add a self-answered response rule
addTLSLocal(addr, certFile(s), keyFile(s) [,params]): listen to incoming DNS over TLS queries on the specified address using the specified certificate (or list of) and key (or list of). The last parameter is a table
...
```

示例

```shell
> showServers()
#   Name                 Address                       State     Qps    Qlim Ord Wt    Queries   Drops Drate   Lat Outstanding Pools
0   dns1                 8.8.8.8:53                       up     0.0      10   1  1          2       0   0.0   1.4           0
1   dns2                 114.114.114.114:53               up     0.0       1   1  1          0       0   0.0   0.0           0
All
```





# 四、  详细配置

```shell
-- 本地监听
addLocal("192.168.122.21:53",{reusePort=true})


-- 下游服务器
newServer({address="114.114.114.114"})               -- 默认
newServer({address="127.0.0.1", name="private", pool="private"})

-- 添加组
-- 办公
office = newNMG()
office:addMask("10.254.38.244/32")
-- 私人
private = newNMG()
private:addMask("192.168.122.0/24")

-- 劫持
-- 劫持到代理服务器
-- 如下实现 不同的请求地址 解析到不同的 ip
addAction(AndRule{NetmaskGroupRule(office), QNameRule("git.com")}, SpoofAction({"10.254.38.175"}))    -- 劫持到指定主机
addAction(AndRule{NetmaskGroupRule(private), QNameRule("git.com")}, SpoofAction({"192.168.122.99"}))    -- 劫持到指定主机
-- addAction("git.com", PoolAction("private"))          -- 使用指定 pool 解析

-- 分割文件
includeDirectory("/etc/dnsdist/conf.d")
```

