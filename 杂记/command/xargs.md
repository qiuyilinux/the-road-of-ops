## xarg

###### 详解：

- xargs 是命令传递参数的一个过滤器，也是组合多个命令的一个工具。
- xargs 可以将管道或者标准输入（stdin）数据转换成命令行参数，也能够从文件的输出中读取数据。
- xargs 也可以将单行或者多行文本输入转换为其他格式，例如多行变单行，单行变多行。
- xargs 默认的命令是echo，这意味着通过管道产地给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。
- xagrs 是一个强有力的命令，它能够捕捉一个命令的输出，然后传递给另外一个命令，之所以能用到这个命令，关键是由于很多命令不支持 | 管道来传递参数，而日常工作中有这个必要所以就有了xargs这个命令，例如：

```shell
find /sbin/ -perm /700 | ls -l       #这个命令是错误的
find /sbin/ -perm /700 |xargs ls -l  #这样才是正确大的
```

xargs 一般是和管道一起使用

###### 命令格斯

```shell
somecommand |xargs -item command
```

###### 参数

```shell
-a file		#从文件中读入作为stdin
-e flag		#注意有可能回事-E，flag必须是一个以空格分割的标志，当xargs分析到含有flag这个标志的时候就停止。
-p		    #每当执行各异argument的时候询问一次用户
-n num		#后面加次数
-t		    #表示先打印命令然后再执行
-r 		    #no-run-if-empty  当xargs的输入为空的时候则停止xargs，不用去执行了
-i		    #或者是-I，这个得看linux支持了，将xargs的每项名称，一般是一行一行的赋值给{},可以用{}代替。
-s num		#命令行的最大字符数，指得是 xargs 后面那个命令的最大命令字节符数
-L num		#从标准输入一次读取num行送给command命令 or -l
-d 		    #delim分割符，默认的xargs的分隔符是回车，argument的分隔符是空格，这个理修改的是xargs的分割符。
-x		    #exit的意思，主要配合-s使用
-P		    #修改最大的进程数，默认是1 为0的时候为 as mant as it can ，这个例子我没有想到，应该平时用不到的吧
```

###### 实例

xargs 用作替换工具，读取输入数据重新格式化后输出

定义一个测试文件内有多行文本数据：

```shell
#cat test.txt
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z
```

多行输入单行输出

```shell
# cat test.txt | xargs
a b c d e f g h i j k l m n o p q r s t u v w x y z
```

-n 选项多行输出

```shell
#cat test.txt | xargs -n3
a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z
```

-d 选项可以自定义一个定界符

```shell
#echo "nameXnameXnameXname" | xargs -dX
name name name name

```

结合-n 使用

```shell
#echo "nameXnameXnameXname" | xargs -dX -n 2 
name name
name name

```

读取stdin 将格式化后的参数传递给命令

假设一个命令为sk.sh和一个保存参数的文件arg.txt

```shell
#!/bin/bash
#sk.sh命令内容，打印所有参数 
echo $*
```

arg.txt内容

```shell
#cat arg.txt
aaa
bbb
ccc
```

xargs的一个选项-l,使用-l指定一个替换字符串{},这个字符串在xargs扩展时会被替换掉，当-i与xargs结合使用，每一个参数命令都会被执行一次：

```shell
#cat arg.txt |xargs -I {} ./sk.sh {}
aaa
bbb
ccc
```

复制所有txt文件到/data/images 目录下：

```shell
ls *.txt | xargs -n1 -I {} cp {} /opt
```

查找.txt结尾的文件 统计其行数

```shell
find ./ -name "*.txt" |xargs wc -l
```

###### 总结

其实最主要的作用就是把管道之前的值作为参数传递给之后的命令  可以把参数进行一定的格式转换 （-n 两个值为一组  或者 -d 指定分割符 -i {}取参数 （linux 支持为-I））