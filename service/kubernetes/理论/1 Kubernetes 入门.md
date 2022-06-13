## 1 Kubernetes 核心概念

**有了 docker 为什么要用 kubernetes？**

> 企业需求： 为提高业务并发和高可用，会使用多台服务器，因此会面向这些问题。

- 多容器跨主机如何提供服务？
- 多容器如何分布式部署到节点？
- 多主机多容器怎么升级？
- 怎么高效管理这些容器？

为了解决这些问题，引入了容器编排系统：

- kubernetes
- swarm
- mesos marathon

![](../image/2.png)

## 2 Kubernetes 是什么

>  Kubernetes 是 Google 在 2014 年开园的一个容器集群管理系统， Kubernetes 简称 K8s
>
>  Kubernetes 用于容器化应用程序的部署，扩展和管理，目标是让部署容器化应用简单高效。
>
> 官方网站： http://www.kubernetes.io
>
> 官方文档： https://kubernetes.io/docs/home/

![](../image/3.png)



![](../image/4.png)

#### Master 组件

- kube-apiserver

kubernetes api， 集群的统一入口，各组件协调者，以RESTful API 提供接口服务，所有对象资源的增删改查和监听操作都交给 API server 处理后再提交给 Etcd 存储。

- kube-controller-manager

处理集群中常规后台任务，一个资源对应一个控制器，而 Controller Manager 就是负责管理这些控制器的。

- kube-scheduler

根据调度算法为新创建的 pod 选择一个 Node 节点，可以任意部署，可以部署在同一个节点，也可以部署在不通的节点上。

- etcd

分布式键值存储系统。用于保存集群状态数据，比如 Pod， Service 等对象信息。

#### Node 组件

- kubelet

kubelet 是 master 在 node 节点上的agent，管理本机运行容器的生命周期，比如创建容器， pod 挂载数据卷， 下载 secret， 获取容器和节点状态等工作。 kubelet 将每个 pod 转换成一组容器。

- kube-proxy

在 node 节点上实现 pod 网络代理，维护网络规则和四层负载均衡工作。

- 第三方容器引擎：docker 、 containerd、 podman

容器引擎，运行容器。



## 3 网络组件起什么作用

面临的问题：

- 容器都还有自己的内网，容器1 数据包如何转发到 容器2。
- 每个 docker 的网络是独立管理。

​	

![](../image/5.png)

部署网络组件的目的是打通 pod 到 pod 之间网络， node 和 pod 之间网络， 从而集群中的数据包可以任意传输，形成一个扁平化网络。

主流的网络组件有： Flannel ， Calico 等。

而所谓的 CNI （Container Network Interface） 就是 k8s 对接这些第三方网络组件的接口。



## 4 Kubernetes 将弃用 Docker

在 kubernetes 平台中，为了解决容器运行时（例如 docker） 集成问题， 在早期社区推出了 CRI（Container Runtime Interface 容器运行时接口），以支持更多的容器运行时。

当我们使用 docker 作为容器运行时之后，架构是这样的，如图所示。

![](../image/6.png)

Kubernetes 计划弃用就是 kubelet 中 dockershim。 即 kubernetes kubelet 实现中的组件之一，它能够与 docker engine 进行通信。

**为什么这么做？**

- docker 内部调用链比较复杂，多层封装和调用，导致性能降低，提升故障率，不易排查
- docker 还会在宿主机创建网络规则，存储卷，也带来安全隐患



**如何应对？**

在未来 kubernetes 版本彻底放弃 docker 支持之前，引入受支持的容器运行时。

除了 docker 之外， cri 还支持很多容器运行时，例如：

- containerd： containerd 与 docker 相兼容，相比 docker 轻量很多，目前较为成熟。
- cri-o，podman： 都是红帽（RedHat）项目，目前红帽主推 podman

