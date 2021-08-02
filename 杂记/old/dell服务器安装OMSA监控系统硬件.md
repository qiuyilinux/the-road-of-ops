### 	事件背景

​	应公司要求，监测备份集raid盘阵当前状态，保证过年期间稳定运行。

```
dmidecode | grep "Product Name"    #查看服务器型号是否为dell

wget -q -O - http://linux.dell.com/repo/hardware/OMSA_7.2/bootstrap.cgi |bash -x   #安装dell的源（这个也是wget的一个小技巧 比较有意思）

yum install srvadmin-all -y       #安装所有监测组件，具体干啥的咱也不知道干就完了
 
yum install -y net-snmp net-snmp-devel net-snmp-utils wget perl OpenIPMI 

/opt/dell/srvadmin/sbin/srvadmin-services.sh start    #启动OMSA

ln -s /opt/dell/srvadmin/bin/omreport /usr/local/bin/ #把我们用的命令做个软链接



```

