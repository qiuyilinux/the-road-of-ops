![](../../image/7.png)



### 1 制作镜像

```
暂略
```



### 2 使用控制器部署镜像

```shell
kubectl create deployment web --image=lizhenliang/java-demo
kubectl get deploy,pods
```



### 3 使用 service 将 pod 暴露出去

```shell
 kubectl expose deployment web --port=80 --target-port=8080 --type=NodePort
--port 集群内部访问端口，暂时不用。
--target-port 镜像内服务运行端口
--type=NodePort service 类型，会生成一个随机端口， 通过 kubectl get service/svc 获取

# 访问：
# http://nodeip:port
```



### 4 日志、监控

```
暂略
```

