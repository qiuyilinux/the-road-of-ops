### 1 命令概要



| 类型               | 命令           | 描述                                                         |
| ------------------ | -------------- | ------------------------------------------------------------ |
| 基础命令           | create         | 通过文件名或标准输入创建资源                                 |
|                    | expose         | 为 Deployment， Pod 创建 Service                             |
|                    | run            | 在集群中运行一个特定的镜像                                   |
|                    | set            | 在对象上设定特定的功能                                       |
|                    | explain        | 文档参考资料                                                 |
|                    | get            | 显示一个或多个资源                                           |
|                    | edit           | 使用系统编辑器编辑一个资源                                   |
|                    | delete         | 通过文件名，标准输入，资源名称活着标签选择器来删除资源       |
| 部署命令           | rollout        | 管理 Deployment, DFaemonset 资源发布（例如状态，发布记录，回滚等） |
|                    | rolling-update | 滚动升级，仅限 ReplicationController                         |
|                    | scale          | 对 Deployment,ReplicaSet,RC或Job资源扩容或缩容Pod数量        |
|                    | autoscale      | 为Deploy，RS，RC配置自动伸缩规则（依赖metrics-server和hpa）  |
| 集群管理命令       | certificate    | 修改证书资源                                                 |
|                    | cluster-info   | 显示集群信息                                                 |
|                    | top            | 查看资源利用率（依赖于metrics-server）                       |
|                    | cordon         | 标记节点不可调度                                             |
|                    | uncordon       | 标记几点可调度                                               |
|                    | drain          | 驱逐节点上的应用，准备下线维护                               |
|                    | taint          | 修改节点 taint 标记                                          |
| 故障诊断和调试命令 | describe       | 显示资源详细信息                                             |
|                    | logs           | 查看 Pod 容器日志，如果 Pod 有多个容器， -c 参数指定容器名称 |
|                    | attach         | 附加到 Pod 内的一个容器                                      |
|                    | exec           | 在容器内执行命令                                             |
|                    | port-forward   | 为 Pod 创建本地端口映射                                      |
|                    | proxy          | 为 Kubernetes Api Server 创建代理                            |
|                    | cp             | 拷贝文件或目录到容器中，或者从容器内向外拷贝                 |
| 高级命令           | apply          | 从文件名或者标准输入对资源创建/更新                          |
|                    | patch          | 使用布丁的方式修改，更新资源的某些字段                       |
|                    | replace        | 从文件名或者标准输入替换一个资源                             |
|                    | convert        | 在不同 API 版本之间转换对象定义                              |
| 设置命令           | label          | 给资源设置，更新标签                                         |
|                    | annotate       | 给资源设置，更新注解                                         |
|                    | completion     | kubectl 工具自动补全， source < (kubectl completion bash) (依赖于软件包 bash-completion) |
| 其他命令           | api-resources  | 查看所有资源                                                 |
|                    | api-versions   | 打印手支持的 api 版本                                        |
|                    | config         | 修改 kubeconfig 文件（用于访问 api ， 比如配置认证信息）     |
|                    | help           | 所有命令帮助                                                 |
|                    | version        | 查看 kubectl 和 kubernetes 版本                              |





### 2 使用

```shell
# 自动补全
yum install -y bash-completion
bash
source <(kubectl completion bash)

# 查看标签
kubectl get pods --show-labels
kubectl get pods,deployment,service --show-labels

# 使用控制器部署镜像
kubectl create deployment web --image=lizhenliang/java-demo

# 使用 service 将 pod 暴露出去
kubectl expose deployment web --port=80 --target-port=8080 --type=NodePort

# 获取所有 pod
kubectl get pods -A 

# 获取指定命名空间 pod
kubectl get pods -n default

# 创建指定资源 如命名空间
kubectl create namespace bosheng

# 根据 yaml 创建更新，删除指定资源
kubectl apply -f xxx.yaml
kubectl delete -f xxx.yaml

# 使用 create 生成 yaml
kubectl create deployment nginx --image=nginx:1.16 -o yaml --dry-run=client > ./deploy.yaml
--dry-run=client  直接生成
--dry-run=server  提交到 api 进行测试再生成
-o yaml 输出成 yaml 文件

# 使用 get 生成 yaml
kubectl get deployment nginx -o yaml > ./depoly.yaml

# 获取配置项 类似于 help
kubectl explain pods.spec.containers
kubectl explain deployment

# 登陆到 pod-net-test 的 web 容器中
kubectl exec -it pod-net-test -c web -n study  -- sh
```

