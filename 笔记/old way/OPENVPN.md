###### 部署

```shell
# 配置阿里云源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all && yum makecache

# 安装依赖包
yum install rpm-build gcc lsof -y
yum install -y pam-devel*
yum install -y lzo lzo-devel openssl openssl-devel pam pam-devel
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum install -y pkcs11-helper pkcs11-helper-devel

# 确认安装结果
rpm -qa lzo lzo-devel openssl openssl-devel pam pam-devel pkcs11-helper pkcs11-helper-devel
lzo-2.06-8.el7.x86_64
pam-1.1.8-22.el7.x86_64
lzo-devel-2.06-8.el7.x86_64
openssl-devel-1.0.2k-16.el7.x86_64
pam-1.1.8-22.el7.i686
openssl-1.0.2k-16.el7.x86_64
pkcs11-helper-1.11-3.el7.x86_64
pam-devel-1.1.8-22.el7.x86_64
pkcs11-helper-devel-1.11-3.el7.x86_64

# 下载OPENVPN源码包
mkdir -p /work/openvpn
wget -O /work/openvpn/openvpn-2.2.2.tar.gz http://oss.aliyuncs.com/aliyunecs/openvpn-2.2.2.tar.gz

# 使用rpmbuild将源码包编译成rpm包来进行安装
cd /work/openvpn && rpmbuild -tb openvpn-2.2.2.tar.gz
ps:执行上面这条命令以后就会正常开始编译了，编译完成以后会在/root/rpmbuild/RPMS/x86_64?目录下生成 openvpn-2.2.2-1.x86_64.rpm 安装包。

# 安装
rpm -ivh /root/rpmbuild/RPMS/x86_64/openvpn-2.2.2-1.x86_64.rpm

# 初始化PKI
cd /usr/share/doc/openvpn-2.2.2/easy-rsa/2.0
# 找到 vars 证书环境文件，修改以下几行 export 定义的参数值
export KEY_COUNTRY="CH"
export KEY_PROVINCE="SH"
export KEY_CITY="SHANGHAI"
export KEY_ORG="xinnet"
export KEY_EMAIL="1918175266@qq.com"

# 生成服务端的证书
# 清除并删除keys目录下的所有key
ln -s openssl-0.9.6.cnf openssl.cnf
ll openssl*
source ./vars
./clean-all
# 生成CA证书，刚刚上面已经在vars文件中配置了默认参数值
./build-ca # 一路回车 ca.key ca.crt
# 生成服务器证书
./build-key-server boshengvpn # 一路回车 + y
ls ./keys # boshengvpn.crt  boshengvpn.csr  boshengvpn.key
# 创建vpn登陆用户的秘钥与证书
./build-key xin #xin.crt  xin.csr  xin.key
# 如果创建用户证书时报错，可以将keys整个目录删除，然后从source ./vars这一步开始重新操作（慎重，否则之前在keys目录里的用户数据就会都删除）

# 生成Diffie Hellman参数
# 执行了./build-dh后，会在 keys 目录下生成 dh 参数文件 dh1024.pem。该文件客户端验证的时候会用到
./build-dh

# 将/usr/share/doc/openvpn-2.2.2/easy-rsa/2.0/keys 目录下的所有文件复制到 /etc/openvpn下：
cp -a /usr/share/doc/openvpn-2.2.2/easy-rsa/2.0/keys/* /etc/openvpn/
# 复制openvpn服务端配置文件 server.conf 到 /etc/openvpn/ 目录下：
cp -a /usr/share/doc/openvpn-2.2.2/sample-config-files/server.conf /etc/openvpn/


# 修改配置 
cat > /etc/openvpn/server.conf << EOF
local  172.16.0.4              # 监听地址(内网或外网地址)，最好填写openvpn服务器的公网IP地址（使用"curl ifconfig.me"命令查看）。或者这一行直接注释掉！(我在线上配置的就是注释这行)
port 1194                          # 端口
proto udp                          # 协议
dev tun
ca ca.crt                          # CA证书路径
cert boshengvpn.crt                # 此处crt以及下一行的key，请填写生成服务器端证书时用户自定义的名称
key boshengvpn.key
dh dh1024.pem                      # 秘钥交换协议文件
server 10.8.0.0 255.255.255.0      # 给vpn客户机分配的地址池。最好别和openvpn部署机的内网ip在一个网段内
ifconfig-pool-persist ipp.txt
push "route 0.0.0.0 255.0.0.0"     #可以通过本机连接到那
client-to-client
keepalive 10 120
comp-lzo
user nobody
group nobody
persist-key
persist-tun
status openvpn-status.log
log openvpn.log
verb 3
EOF


# 配置内核参数
cat > /etc/sysctl.conf << EOF
vm.min_free_kbytes = 409600
vm.vfs_cache_pressure = 200
vm.swappiness = 0
fs.file-max = 1024000
fs.aio-max-nr = 1024000
kernel.sysrq = 1
net.ipv4.ip_forward = 1
EOF
sysctl -p

# 添加iptables 规则 确保 可以转发包到外网
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE

# 启动
service openvpn start
```

###### 新建用户

```shell
# 后续给同事开vpn账号，只需要下面几步（比如给xin同事开vpn）

cd /usr/share/doc/openvpn-2.2.2/easy-rsa/2.0
./build-key xin
/etc/init.d/openvpn restart
cat > config.ovpn <<EOF
client
dev tun
proto udp
remote 106.13.128.217 1194
resolv-retry infinite
nobind
mute-replay-warnings
ca  ca.crt
cert xin.crt
key xin.key
comp-lzo
EOF

# 然后将ca.crt、config.ovpn、wuwei.crt、wuwei.csr、wuwei.key这五个文件打包给用户即可
```

