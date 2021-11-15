# 一、 KVM 基本介绍



## 1 KVM是什么意思？

基于内核的虚拟机 Kernel-based Virtual Machine（KVM）是一种内建于 Linux中的开源虚拟化技术。具体而言，KVM 可帮助您将 Linux 转变为虚拟机监控程序，使主机计算机能够运行多个隔离的虚拟环境，即虚拟客户机或虚拟机（VM）。

KVM 是 Linux 的一部分。Linux 2.6.20 或更新版本包括 KVM。KVM 于 2006 年首次公布，并在一年后合并到主流 Linux 内核版本中。由于 KVM 属于现有的 Linux 代码，因此它能立即享受每一项新的 Linux 功能、修复和发展，无需进行额外工程。



## 2  KVM是如何运行的？

KVM 将 Linux 转变为一类（裸机恢复）虚拟机监控程序。所有虚拟机监控程序都需要一些操作系统层面的组件才能运行虚拟机，如

- 内存管理器
- 进程调度程序
- 输入/输出（I/O）堆栈
- 设备驱动程序
- 安全管理器
- 网络堆栈等。

由于 KVM 是 Linux 内核的一部分，因此所有这些组件它都有。每个虚拟机都像普通的 Linux 进程一样实施，由标准的 Linux 调度程序进行调度，并且使用专门的虚拟硬件，如网卡、图形适配器、CPU、内存和磁盘等。



## 3  Linux KVM 有什么优势？

KVM 是 Linux 的一部分。Linux 也是 KVM 的一部分。Linux 有的，KVM 全都有。然而，KVM 的某些特点让它成为了企业的首选虚拟机监控程序。主要体现为一下几个方面。

**（1） 安全性**

SELinux 和 sVirt

**（2） 存储**

KVM 能够使用 Linux 支持的任何存储，包括某些本地磁盘和NAS。还可以利用多路径 I/O 来增强存储并提供冗余能力。KVM 还支持共享文件系统，因此虚拟机镜像可以由多个主机共享。磁盘镜像支持精简置备，可以按需分配存储，不必预先备妥一切。

**（3） 硬件支持**

KVM 可以使用多种多样的认证 Linux 兼容硬件平台。由于硬件供应商经常助力内核开发，所以 Linux 内核中通常能快速采用最新的硬件功能。

**（4） 内存管理**

KVM 继承了 Linux 的内存管理功能，包括非统一内存访问和内核同页合并。虚拟机的内存可以交换，也可通过大型宗卷支持来提高性能，还可由磁盘文件共享或支持。

**（5） 实时迁移**

KVM 支持实时迁移，也就是能够在物理主机之间移动运行中的虚拟机，而不会造成服务中断。虚拟机保持开机状态，网络连接保持活跃，各个应用也会在虚拟机重新定位期间正常运行。KVM 也会保存虚拟机的当前状态，从而存储下来供日后恢复。

**（6） 性能和可扩展性**

KVM 继承了 Linux 的性能，针对客户机和请求数量的增长进行扩展，满足负载的需求。KVM 可让要求最苛刻的应用工作负载实现虚拟化，而这也是许多企业虚拟化设置的基础，如数据中心和私有云等

**（7） 调度和资源控制**

在 KVM 模型中，虚拟机是一种 Linux 进程，由内核进行调度和管理。通过 Linux 调度程序，可对分配给 Linux 进程的资源进行精细的控制，并且保障特定进程的服务质量。在 KVM 中，这包括完全公平的调度程序、控制组、网络命名空间和实时扩展。

**（8） 更低延迟 更高优先级**

Linux 内核提供实时扩展，允许基于虚拟机的应用以更低的延迟、更高的优先级来运行（相对于裸机恢复）。内核也将需要长时间计算的进程划分为更小的组件，再进行相应的调度和处理。



## 4 QEMU 是什么？

Qemu 是纯软件设计的虚拟化模拟器，几乎可以模拟任何硬件设备，我们最熟悉的就是能够模拟一台能够独立运行操作系统的虚拟机，虚拟机认为自己和虚拟机打交道，但其实是和Qemu模拟出来的硬件打交道，Qemu 将这些真正的指令转译给真正的硬件。

正因为Qemu是纯软件实现的，所有的指令都要经过qemu过一手，性能非常低，所以，在生产环境中，所以在生产环境中，Qemu配合KVM来完成虚拟化工作，因为kvm是硬件辅助的虚拟化技术，主要负责比较繁琐的cpu虚拟化和内存虚拟化，而QEMU则负责IO设备虚拟化，两者合作发挥自身的优势，相得益彰。



# 二、 部署



## 1 检查操作系统是否支持虚拟化

```shell
egrep -c 'vmx|xvm' /proc/cpuinfo

# 大于 0 则支持
```



## 2 检查系统是否支持 kvm 虚拟化

```shell
apt install cpu-checker

kvm-ok

# INFO: /dev/kvm exists
# KVM acceleration can be used
```



## 3 安装 KVM

```shell
apt install -y qemu qemu-kvm libvirt-daemon libvirt-clients bridge-utils virt-manager
```

- qemu包(快速模拟器)是一个允许执行硬件虚拟化的应用程序。
- qemu-kvm包是主要的KVM包。
- libvritd-daemon是虚拟化守护进程。
- bridge-utils包帮助您创建一个桥接连接，以允许其他用户访问主机系统以外的虚拟机。
- virt-manager是一个通过图形用户界面管理虚拟机的应用程序



## 4 检查进程是否运行

```shell
systemctl status libvirtd

systemctl is-enbaled libvirtd
```



## 5 检查 kvm 模块是否加载

```shell
lsmod | grep -i kvm
# kvm_intel             294912  0
# kvm                   823296  1 kvm_intel
```



# 三、 基本使用



# 四、 virsh 命令详解



## 1 介绍

virsh  是 kvm 一个管理工具包。 virsh 提供两种执行模式 

- 直接模式： 在直接模式中，你必须在 shell 中以参数，自变量的方式来执行 virsh 
- 互动模式： 如果在互动模式中，则 virsh 会提供一个提示字符串，你可以在提示字符串后，输入要执行的命令。如果 virsh 没有🔝任何参数或者自变量则默认就是进入互动模式。



## 2 语法

```shell
virsh [command] [args]
```



## 3 参数



- autostart                         自动加载指定的一个虚拟机 
- connect                           重新连接到 hypervisor
- console                           从一个 xml 文件 创建一个虚拟机
- start                                 开启一个非活跃的虚拟机
- destory                            删除一个虚拟机
- define                              从 XML 文件定义一个虚拟机
- domid                             把一个虚拟机名或 UUID 转换为 ID
- dominfo                          查看虚拟机信息
- domstate                       查看虚拟机状态
- domblkstat                    获取虚拟机设备块状态
- domifstat                        获取虚拟机网络接口状态
- dumpxml                        xml 中虚拟机的信息
- edit                                  编辑某个虚拟机的 xml 文件
- list                                  列出虚拟机
- migrate                           将虚拟机迁移到另一台主机
- reboot                            重新启动一个虚拟机
- resume                           重新恢复一个虚拟机
- save                                 把一个虚拟机的状态保存到一个文件
- dump                              把一个虚拟机的内核 dump 到一个文件中以便分析
- shutdown                       关闭一个虚拟机
- setmem                           改变内存的分配
- setmaxmem                   改变最大内存限制值
- suspend                          挂起一个虚拟机
- vcpuinfo                          虚拟机的 cpu 信息
- version                             限制 virsh 版本         



## 4 示例