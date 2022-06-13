

### 1 部署 Dashborad

Dashboard是官方提供的一个UI，可用于基本管理K8s资源。

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.3/aio/deploy/recommended.yaml
```

默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：

```shell
vi recommended.yaml
# 大概在 32 行左右
...
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort
...

kubectl apply -f recommended.yaml
kubectl get pods -n kubernetes-dashboard
```

访问地址： https://nodeip:30001

创建 service account 并绑定默认 cluster-admin 管理员集群角色：

```shell
# 创建用户
kubectl create serviceaccount dashboard-admin -n kube-system
# 用户授权
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
# 获取用户Token
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

使用输出的 token 登陆 Dashborad



### 2 测试

在Kubernetes集群中创建一个pod，验证是否正常运行：

```shell
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc
# 它会启动一个随机端口映射到宿主机并开放访问 ?
```

