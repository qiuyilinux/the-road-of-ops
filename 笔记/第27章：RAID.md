# RAID

​	  RAID (Redundant Arrays of Independent Disks)   独立磁盘冗余阵列。是利用多块物理硬盘来组成一个虚拟硬盘，并由这些虚拟的硬盘组成一个矩阵的存储系统。他的目的很简单却很重要。毕竟关系到数据，保证数据的安全性，提高数据读写效率。磁盘阵列主要分三种：外界式磁盘矩阵列柜，内接式磁盘矩阵列卡，软件模拟仿真（软raid）

- raid0 ：条状模式（两块以上的磁盘组成，磁盘利用率高 读写速度快 无容错）
- raid1 ：镜像卷（两块以上磁盘组成 磁盘利用率1/2 荣作性优 读写速度慢）
- raid5 ：raid5卷 分布式奇偶校验的独立硬盘结构（三块以上磁盘组成 磁盘利用率 2/3 读写速度优）
- raid6 ： 带有两种分布式存储的奇偶校验码的独立磁盘结构（四块 以上组成允许两块磁盘孙环）
- raid10： raid1+raid0

# 部署

```shell
1.yum install -y mdadm-4.0-5.el7.x86_64 #安装磁盘阵列管理工具mdadm
parted                                  #创建分区 
set 1 raid on                           #创建标签
mdadm -Cv /dev/md1 -l1 -n2 -c128 /dev/vd[b,c]1 #指定-l raid1 -n 盘量 -c 块大小
mdadm -Cv /dev/md5 -l5 -n3 -x1 -c128 /dev/vd[b,c,d,e]1 #-x 奇偶校验盘量
mdadm -D -s /dev/md1 >> /etc/mdadm.conf#生成raid配置文件

2.格式化挂载使用
mkfs.ext4 /dev/md127
mount /dev/md127 /raid/

3.移除添加故障盘
mdadm -r /dev/md1 /dev/vdc       #移除故障硬盘
mdadm -a /dev/md127 /dev/sdb1    #在read中添加新硬盘

4.移除组
mdadm --stop /dev/md0
mdadm --remove /dev/md0
```

