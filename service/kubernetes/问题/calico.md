## 1 calico 与 kubernetes 匹配问题

```
unable to recognize “calico.yaml”: no matches for kind “PodDisruptionBudget” in version “policy/v1”
```

这是k8s不支持当前calico版本的原因, calico版本与k8s版本支持关系可到calico官网查看:

```shell
https://projectcalico.docs.tigera.io/archive/v3.20/getting-started/kubernetes/requirements

#  curl https://docs.projectcalico.org/archive/v3.20/manifests/calico.yaml -O
```

