​	实现ftp上传下载追加文件的方法有很多，但是curl绝对是最简单的哪一种！

```shell
 curl --USER IPSecList:bZEjBLQe0a22 -a -T filename "ftp://vhctrl.xincache.cn/"
 #指定用户名IPSecList密码bZEjBLQe0a22 -a 上传文件的时候追加文件 -T 上传文件  filename  文件名  ftp://vhctrl.xincache.cn/目的路径  （ftp://vhctrl.xincache.cn/filename（这个filename为你想追加到那个文件中））
  curl --USER IPSecList:bZEjBLQe0a22 "ftp://vhctrl.xincache.cn/"
 #列出指定路径下的文件信息
```

