```shell
ldd /usr/bin/haproxy   #查看程序需要的库信息

linux-vdso.so.1 =>  (0x00007ffeb1bed000)
libc.so.6 => /lib64/libc.so.6 (0x00007fb3b6c25000)
/lib64/ld-linux-x86-64.so.2 (0x00007fb3b6ff2000)

#第一列：程序需要的库（libc.so.6）
#第二列：系统提供的程序需要的库的对应库（/lib64/libc.so.6）
#第三列：库加载的开始地址(0x00007fb3b6c25000)
```

