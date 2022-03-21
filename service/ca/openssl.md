# 自签证书

## （1） 创建根 CA 证书

### a. 创建根密钥

```shell
openssl ecparam -out contoso.key -name prime256v1 -genkey
```

### b. 创建根证书并进行自签名

```shell
openssl req -new -sha256 -key contoso.key -out contoso.csr

openssl x509 -req -sha256 -days 365 -in contoso.csr -signkey contoso.key -out contoso.crt
```

## （2） 创建服务器证书

### a. 创建证书的密钥

```shell
openssl ecparam -out fabrikam.key -name prime256v1 -genkey
```

### b. 创建 CSR 

```shell
openssl req -new -sha256 -key fabrikam.key -out fabrikam.csr
```

### c. 使用 CSR 和密钥生成证书，并使用 CA 的根密钥为该证书签名

```shell
openssl x509 -req -in fabrikam.csr -CA  contoso.crt -CAkey contoso.key -CAcreateserial -out fabrikam.crt -days 365 -sha256
```

### d. 验证新建的证书

```shell
openssl x509 -in fabrikam.crt -text -noout
```

