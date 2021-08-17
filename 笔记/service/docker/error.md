1 

```
# harbor 配置证书之后在 docker login www.bosheng.com -u admin -p Harbor12345 时候可能会遇到
Error response from daemon: Get "https://www.bosheng.com/v2/": x509: certificate relies on legacy Common Name field, use SANs or temporarily enable Common Name matching with GODEBUG=x509ignoreCN=0

这是因为 go 版本的问题
这不是 go 的缺陷，而是一个特性。该检查是在 1.11 中添加的，在 1.15 中默认启用，并且将在未来版本中删除选择退出。您需要更新注册表上的证书以更正属性。

这时修改证书支持 san 即可
https://www.huaweicloud.com/articles/464d60bd946abab9daaa8d438e08642f.html  # ctrl f san
```

