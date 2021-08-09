# 一、 Dockerfile 介绍

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。有了 Dockerfile，当我们需要定制自己额外的需求时，只需在 Dockerfile 上添加或者修改指令，重新生成 image 即可，省去了敲命令的麻烦。

# 二、 Dockerfile 常用指令

| 指令       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 构建新镜像是基于那个镜像                                     |
| LABEL      | 标签                                                         |
| RUN        | 构建镜像时运行的 shell 指令                                  |
| COPY       | 拷贝文件或目录到镜像中(相对路径)                             |
| ADD        | 解压压缩包并拷贝(相对路径)                                   |
| ENV        | 设置环境变量                                                 |
| USER       | 为 RUN、 CMD 和 ENTRYPOINT 执行命令指定运行用户              |
| EXPOSE     | 声明容器运行的服务端口                                       |
| WORKDIR    | 为 RUN、COPY 和 ADD 设置工作目录 (进入容器的默认路径)        |
| CMD        | 运行容器时默认执行，如果有多个 CMD 命令， 最后一个生效。 采用 json 必须双引号 |
| ENTRYPOINT | 如果与 CMD 一起用， CMD 将作为 ENTRYPOINT 的默认参数，如果有多个 ENTRYPOINT 指令，最后宇哥生效 |

```python
FROM centos:7
LABEL maintainer bosheng
WORKDIR /opt
COPY --chown=root:root a.sh .
RUN chmod +x a.sh
ADD b.tar.gz .
ENV ABC=123 NAME=gongbosheng
EXPOSE 88
CMD [ "/opt/a.sh 360000 $NAME"]
ENTRYPOINT [ "sh", "-c"]
```



# 三、 docker build 构建镜像

```python
# -t 指定标签 
docker build -t 7-study -< /root/docker/centos7-study.dockerfile


# 指定目录 但目录下必须名为 Dockerfile
docker build -t centos:7-study ./
  
# 指定 dockerfile 名字
docker build -t tag -f centos7-study.dockerfile  /root/docker/
  
  
# 从 git 构建
docker build -t tag github.com/creack/docker-firefox

# 从 tarball 构建
docker build - < context.tar.gz                         # 名字需要为 Dockerfile
docker build -< 2.tar.gz  -f centos7-study.dockerfile   # 指定名字


# 把需要的文件都打包然后通过压缩文件创建
[root@server-one docker]# tar zcf  2.tar.gz ./*
[root@server-one docker]# ls
2.tar.gz  a.txt  b.tar.gz  b.txt  centos7-study.dockerfile
[root@server-one docker]# mv 2.tar.gz /tmp/
[root@server-one docker]# cd /tmp/
[root@server-one tmp]# docker build -< 2.tar.gz  -f centos7-study.dockerfile -t tag
Sending build context to Docker daemon     516B
Step 1/9 : FROM centos:7
 ---> 8652b9f0cb4c
Step 2/9 : LABEL maintainer bosheng
 ---> Using cache
 ---> f15dc9f81d5b
Step 3/9 : RUN yum install wget curl -y
 ---> Using cache
 ---> e0aa0748274e
Step 4/9 : COPY a.txt /opt
 ---> dc354cabf27a
Step 5/9 : ADD b.tar.gz /opt
 ---> cc2129b8128e
Step 6/9 : ENV ABC=123
 ---> Running in e148a299adf8
Removing intermediate container e148a299adf8
 ---> 3c238d845444
Step 7/9 : EXPOSE 88
 ---> Running in 1143fd14cda8
Removing intermediate container 1143fd14cda8
 ---> 864db55aad57
Step 8/9 : WORKDIR /opt
 ---> Running in 5c4f8ba9d94a
Removing intermediate container 5c4f8ba9d94a
 ---> fc32bd230615
Step 9/9 : CMD ["sleep","360000"]
 ---> Running in dd9b0005343f
Removing intermediate container dd9b0005343f
 ---> 4bc0cf8fdb79
Successfully built 4bc0cf8fdb79
```



# 四、 CMD 与 ENTRYPOINT 区别

## CMD

**用法：**

```shell
CMD ["executable", "param1", "param2"]  # exec 形式 首选 但是无法解析变量
# 解析变量
ENV NAME=gongbosheng
CMD ["sh", "-c", "run.sh $NAME"]
# 覆盖 CMD $a 为宿主机的变量
docker run -d centos:t /opt/a.sh 123 $a  

CMD ["param1", "param2"]                # 作为 ENTRYPOINT 的默认参数
CMD command param1 param2               # shell 形式 可以直接识别变量
```



## ENTRYPOINT

**用法：**

```shell
# 无法被 docker run ... command 替代
# 如需结合 CMD 使用必须使用 exec 的形式
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2. 
```

# 五、 Nginx

```dockerfile
FROM centos:7
LABEL maintainer gongbosheng
WORKDIR /usr/local
RUN yum install -y gcc gcc++ make \
    openssl-devel pcre-devel gd-devel \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*

ADD nginx-1.18.0.tar.gz .
RUN cd nginx-1.18.0 && \
    ./configure --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_stub_status_module && \
    make -j 4 && make install && \
    mkdir /usr/local/nginx/conf/vhost && \
    cd /usr/local/ && rm -rf nginx-1.18.0 && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
ENV PATH $PATH:/usr/local/nginx/sbin
# COPY nginx.conf /usr/local/nginx/conf/nginx.conf
EXPOSE 80
CMD ["daemon off;"]
ENTRYPOINT ["nginx", "-g"]
```

# 六、 PHP 

```

```



# 七、 boke

# 八、 Tomcat

# 九、 微服务