

###### 1.简介

​	

###### 2.部署

```shell
清华大学开源镜像源：https://mirrors.tuna.tsinghua.edu.cn/

1.安装JDK
yum install -y java-1.8.0-openjdk.x86_64
2.安装Jenkins
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/jenkins-2.233-1.1.noarch.rpm
rpm -ivh jenkins-2.233-1.1.noarch.rpm 
3.修改配置
vi /etc/sysconfig/jenkins
4.解锁
http://192.168.50.135:8080/login?from=%2F
cat /var/lib/jenkins/secrets/initialAdminPassword
# 接下来可以选择插件安装(按需，后续也可以安装)
5.更新镜像源
Manage Jenkins --> Manafe Plugins --> Advanced --> Update Site 
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
6.安装插件
	# 下载安装
	6.1 Manage Jenkins --> Manafe Plugins --> Available --> 搜索pipeline --> Install without restart 
	# 手动安装（从源中把需要的组件下载到本地进行上传安装）
	6.2 Manage Jenkins --> Manafe Plugins --> Advanced --> Upload Plugin
    #汉化
    旧版本的jenkins下载这个插件就够了：locale plugin
	新版本的jenkins还需要下载Localization:Chinese(Simplified)这个插件
7.添加节点
Manage Jenkins --> Manage Nodes and Clouds --> NEW Node --> Node name 
build01

```

