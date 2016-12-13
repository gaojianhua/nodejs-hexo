title: XenCenter虚拟机扩展磁盘空间
date: 2016-11-22 21:22:10
tags:
- 开始
- 我
- 日记
categories: xenserver
---

# 1.乡村销客
__ 乡村销客官网 __: http://www.vilsale.com 
乡村销客是面向化肥行业的企业互联网营销工具。通过“移动应用+云计算+应用市场”的互联网领先技术，帮助化肥生产销售企业快速实现  
<!-- more -->
1. __ 移动化市场营销及客户拜访，__  
1. __ 解决调度发运响应不畅  __  
1. __ 客户账户对账不准等问题， __  
1. __ 帮助肥料企业营销及客户管理 __”。 

乡村销客基于软件即服务的互联网理念，创建国内第一个专注__ 肥料行业的SAAS平台 __，为肥料企业打造性价比最高的企业互联网营销工具。
 
-----------这是广告结束的分割线--------------------------

----

# 2. 虚拟机扩展空间
使用XenCenter客户端增大虚拟机磁盘空间，首先停止虚拟机

点击虚拟机，存储，属性，大小和位置，调整磁盘到你需要的大小，不能减少。


# 3. 创建逻辑卷

```bash
df -h #可以看到此时根分区原来大小，没有变化

文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   48G  1.7G   46G    4% /
devtmpfs                 1.4G     0  1.4G    0% /dev
tmpfs                    1.4G     0  1.4G    0% /dev/shm
tmpfs                    1.4G  8.4M  1.4G    1% /run
tmpfs                    1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/xvda1               497M  124M  373M   25% /boot
tmpfs                    285M     0  285M    0% /run/user/0
```

```bash
fdisk -l #已经可以看到整个磁盘容量变大了


盘 /dev/xvda：107.4 GB, 107374182400 字节，209715200 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0008a81a

    设备 Boot      Start         End      Blocks   Id  System
/dev/xvda1   *        2048     1026047      512000   83  Linux
/dev/xvda2         1026048   104857599    51915776   8e  Linux LVM

磁盘 /dev/mapper/centos-root：51.0 GB, 50964987904 字节，99540992 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


```

创建新的磁盘分区

```bash
fdisk /dev/xvda #对磁盘/dev/xvda进行操作


迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：3
起始 扇区 (104857600-209715199，默认为 104857600)：
将使用默认值 104857600
Last 扇区, +扇区 or +size{K,M,G} (104857600-209715199，默认为 209715199)：+49G
分区 3 已设置为 Linux 类型，大小设为 49 GiB

命令(输入 m 获取帮助)：t
分区号 (1-3，默认 3)：3
Hex 代码(输入 L 列出所有代码)：8e
已将分区“Linux”的类型更改为“Linux LVM”

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘

```




p #查看当前分区
n #创建新分区
 
4 #创建第四个主分区
t #修改分区类型
8e #输入8e，代表分区使用LVM类型
p #查看当前分区状态
 
w #保存以上操作，否则不能新建分区

重新启动系统之后，再进行以下操作

# 把新创建的分区/dev/xvda3加入到与根分区/相同的LVM中

```bash
mkfs.ext4 /dev/xvda3 #格式化分区，需要等一会

mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
3211264 inodes, 12845056 blocks
642252 blocks (5.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=2162163712
392 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (32768 blocks): 完成
Writing superblocks and filesystem accounting information: 完成


 
pvcreate /dev/xvda3 #创建一个新的LVM分区

[root@h91 ~]# pvcreate /dev/xvda3
WARNING: ext4 signature detected on /dev/xvda3 at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/xvda3.
  Physical volume "/dev/xvda3" successfully created

```

```bash 
pvdisplay #查看已经存在的pv（物理卷）

[root@h91 ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/xvda2
  VG Name               centos
  PV Size               49.51 GiB / not usable 3.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              12674
  Free PE               11
  Allocated PE          12663
  PV UUID               pU4ffj-OGZI-6nMi-Bdtg-Iic7-dU2L-0Ws6Je
   
  "/dev/xvda3" is a new physical volume of "49.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/xvda3
  VG Name               
  PV Size               49.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               26KzIv-Wbvq-AmQF-widm-I7NM-Eojh-gLZmKj
```

```bash

vgdisplay #查看当前已经存在的vg（逻辑卷组）


[root@h91 ~]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               49.51 GiB
  PE Size               4.00 MiB
  Total PE              12674
  Alloc PE / Size       12663 / 49.46 GiB
  Free  PE / Size       11 / 44.00 MiB
  VG UUID               uIJF6W-u0Zp-IVdS-2mzF-0Mtt-4boz-0jRG69

```

```bash
lvdisplay #查看已经存在的lv（逻辑卷）

[root@h91 ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                IMlwQJ-uNxI-Wj8a-TRth-FV0V-vNSR-ALflQF
  LV Write Access        read/write
  LV Creation host, time bogon, 2016-11-04 14:50:24 +0800
  LV Status              available
  # open                 2
  LV Size                2.00 GiB
  Current LE             512
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                7CWWhk-KMGE-l155-bXaE-TFBj-UKXe-ZBdPN5
  LV Write Access        read/write
  LV Creation host, time bogon, 2016-11-04 14:50:24 +0800
  LV Status              available
  # open                 1
  LV Size                47.46 GiB
  Current LE             12151
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
```


```bash
vgextend centos /dev/xvda3 #把/dev/xvda3加入与根/目录相同的vg（逻辑卷组）
 [root@h91 ~]# vgextend centos /dev/xvda3
  Volume group "centos" successfully extended


lvextend -L +49GB -n /dev/centos/root #扩容lv（逻辑卷）root

[root@h91 ~]# lvextend -L +49GB -n /dev/centos/root
  Size of logical volume centos/root changed from 47.46 GiB (12151 extents) to 96.46 GiB (24695 extents).
  Logical volume root successfully resized.
```

```bash
e2fsck -f /dev/centos/root #检查
resize2fs /dev/centos/root #生效
``` 

centos7 生效命令

```bash
xfs_growfs  /dev/centos/root #生效

[root@h91 ~]# xfs_growfs  /dev/centos/root
meta-data=/dev/mapper/centos-root isize=256    agcount=4, agsize=3110656 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=12442624, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=6075, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 12442624 to 25287680
```

```bash
df -h #查看挂载状态，显示/分区已经扩容

[root@h91 ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   97G  1.7G   95G    2% /
devtmpfs                 1.4G     0  1.4G    0% /dev
tmpfs                    1.4G     0  1.4G    0% /dev/shm
tmpfs                    1.4G  8.4M  1.4G    1% /run
tmpfs                    1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/xvda1               497M  124M  373M   25% /boot
tmpfs                    285M     0  285M    0% /run/user/0

```

至此，XenServer虚拟机扩容LVM磁盘分区完成。







