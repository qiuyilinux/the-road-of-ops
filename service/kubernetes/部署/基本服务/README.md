生产环境部署 kubernetes 集群一般有两种方式

- kubeadm

  kubeadm 是一个 kubernetes 部署工具， 提供 kubeadn init 和 kubeadm join ， 用于快速部署 kubernetes 集群

- 二进制包

  从github下载发行版的二进制包，手动部署每个组件，组成Kubernetes集群。
  
- 第三方部署工具， 例如 kubespray

- 第三方 web 管理系统， 例如 rancher（rke）、 openshift、 kubesphere  最简单



小结：Kubeadm降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署Kubernetes集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。

