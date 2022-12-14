#                              top使用方法：

### **使用格式：**

top [-] [d] [p] [q] [c] [C] [S] [s] [n]

### 参数说明：

d：指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。
p:通过指定监控进程ID来仅仅监控某个进程的状态。
q:该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。
S：指定累计模式。
s：使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。
i：使top不显示任何闲置或者僵死进程。
c:显示整个命令行而不只是显示命令名。

### top 运行中可以通过 top 的内部命令对进程的显示方式进行控制。内部命令如下：

 s – 改变画面更新频率
 l – 关闭或开启第一部分第一行 top 信息的表示
 t – 关闭或开启第一部分第二行 Tasks 和第三行 Cpus 信息的表示
 m – 关闭或开启第一部分第四行 Mem 和 第五行 Swap 信息的表示
 N – 以 PID 的大小的顺序排列表示进程列表
 P – 以 CPU 占用率大小的顺序排列进程列表
 M – 以内存占用率大小的顺序排列进程列表
 h – 显示帮助
 n – 设置在进程列表所显示进程的数量
 q – 退出 top
 s – 改变画面更新周期

### 序号 列名 含义

 a PID 进程id
 b PPID 父进程id
 c RUSER Real user name
 d UID 进程所有者的用户id
 e USER 进程所有者的用户名
 f GROUP 进程所有者的组名
 g TTY 启动进程的终端名。不是从终端启动的进程则显示为 ?
 h PR 优先级
 i NI nice值。负值表示高优先级，正值表示低优先级
 j P 最后使用的CPU，仅在多CPU环境下有意义
 k %CPU 上次更新到现在的CPU时间占用百分比
 l TIME 进程使用的CPU时间总计，单位秒
 m TIME+ 进程使用的CPU时间总计，单位1/100秒
 n %MEM 进程使用的物理内存百分比
 o VIRT 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
 p SWAP 进程使用的虚拟内存中，被换出的大小，单位kb。
 q RES 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
 r CODE 可执行代码占用的物理内存大小，单位kb
 s DATA 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
 t SHR 共享内存大小，单位kb
 u nFLT 页面错误次数
 v nDRT 最后一次写入到现在，被修改过的页面数。
 w S 进程状态。（D=不可中断的睡眠状态，R=运行，S=睡眠，T=跟踪/停止，Z=僵尸进程）
 x COMMAND 命令名/命令行
 y WCHAN 若该进程在睡眠，则显示睡眠中的系统函数名
 z Flags 任务标志，参考 sched.h

### 默认情况下仅显示比较重要的 PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 列。可以通过下面的快捷键来更改显示内容。

通过 f 键可以选择显示的内容。按 f 键之后会显示列的列表，按 a-z 即可显示或隐藏对应的列，最后按回车键确定。
 按 o 键可以改变列的显示顺序。按小写的 a-z 可以将相应的列向右移动，而大写的 A-Z 可以将相应的列向左移动。最后按回车键确定。
 按大写的 F 或 O 键，然后按 a-z 可以将进程按照相应的列进行排序。而大写的 R 键可以将当前的排序倒转。

![image-20191118114009280](C:\Users\86155\AppData\Roaming\Typora\typora-user-images\image-20191118114009280.png)
1.3% us — 用户空间占用CPU的百分比。
1.0% sy — 内核空间占用CPU的百分比。
0.0% ni — 改变过优先级的进程占用CPU的百分比
97.3% id — 空闲CPU百分比
0.0% wa — IO等待占用CPU的百分比
0.3% hi — 硬中断（Hardware IRQ）占用CPU的百分比
0.0% si — 软中断（Software Interrupts）占用CPU的百分比