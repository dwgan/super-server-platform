---
layout: post
title: "How to expand storage pool"
date: 2024-07-08
---







## 前言

LXC系统给每个用户提供了一个独立、完整的存储管理系统，这个用户的数据被系统打包放到一个存储池被LXC平台统一管理，所有用户通过抢占的方式共享这个存储池。

然而，随着用户数量的增多这个存储池占用的空间会显著的变大，有时候需要添加额外的存储空间才能满足用户的需求。

在LXC平台中，可以存在多个存储池，每个存储池又可以包含多个分区，系统有一套自动机制来管理数据的存储。因此要扩充存储池既可以直接增加存储池的个数，也可以增加包含的分区个数来增加存储池的大小，另外还可以直接扩大分区的大小来增加存储池的大小。这三种方式分别对应不同层级的操作，同时具有不同的灵活性、复杂性。

由于我需要添加的是同一块硬盘上的其他分区，且考虑到实现的复杂性，这里选择直接增加分区个数来增加存储池的大小。

![image-20240708145857133](https://pic.dwgan.top/img/image-20240708145857133.png)

查看存储池状态

```shell
(base) xd@xd-Super-Server:~$ sudo zpool status default #查看存储池状态
  pool: default
 state: ONLINE
  scan: scrub repaired 0B in 0h4m with 0 errors on Sun Jun  9 00:28:19 2024
config:

	NAME         STATE     READ WRITE CKSUM
	default      ONLINE       0     0     0
	  nvme0n1p5  ONLINE       0     0     0
```

可以看到这个存储池在Linux系统上是独占一个分区的，因此拓展存储池有两种方式，**一是**拓展这个分区的大小，**二是**添加一个新的分区到这个存储池。由于拓展分区大小需要要求新的空间和zfs存储池所在分区在物理位置上是连续的且新的空间位置在zfs分区之后，所以这里采用第二种方式，即将空闲空间新建一个分区添加到存储池。

查看当前的存储设备分布

```shell
(base) xd@xd-Super-Server:~$ sudo fdisk -l　＃查看当前的存储设备分布
Disk /dev/nvme0n1：1.8 TiB，2000398934016 字节，3907029168 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
磁盘标识符：E297036A-738F-4B0D-BBCD-57C909CC08DB

设备                 起点       末尾       扇区   大小 类型
/dev/nvme0n1p1       2048     206847     204800   100M EFI 系统
/dev/nvme0n1p2     206848     239615      32768    16M Microsoft 保留
/dev/nvme0n1p3     239616 1079834743 1079595128 514.8G Microsoft 基本数据
/dev/nvme0n1p4 1079836672 1285015551  205178880  97.9G Microsoft 基本数据
/dev/nvme0n1p5 2077870080 2497298431  419428352   200G Microsoft 基本数据
/dev/nvme0n1p6 2497300480 2530856959   33556480    16G Linux swap
/dev/nvme0n1p7 2541342720 3905832959 1364490240 650.7G Linux 文件系统
/dev/nvme0n1p8 3905832960 3907026943    1193984   583M Windows 恢复环境
```

可以看到整个服务器上有一块1.8TB的nvme硬盘（实际是2T，系统换算过来是1.8T），安装了Windows和Linux双系统。其中`Linux Swap`是Linux的交换分区，相当于Windows的虚拟内存。注意其中的`/dev/nvme0n1p5`就是存储池的所在的分区了，由于当时是通过Windows进行分区再通过Linux添加的，所以显示它是Microsoft 基本数据，实际上他确实LXC容器的存储池，可见200G的空间确实偏小了。其中的Linux文件系统就是宿主机占用的空间了。

查看当前的存储池实际使用情况

```shell
(base) xd@xd-Super-Server:~$ zpool list
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
default   199G   169G  30.3G         -    48%    84%  1.00x  ONLINE  -
```

可见当前所剩的空间确实不多

为了拓展`/dev/nvme0n1p5`，需要从系统上的其他位置寻找空闲空间

```shell
(base) xd@xd-Super-Server:~$ sudo parted /dev/nvme0n1 print free
Model: Samsung SSD 980 PRO 2TB (nvme)
磁盘 /dev/nvme0n1: 2000GB
Sector size (logical/physical): 512B/512B
分区表：gpt
Disk Flags: 

数字  开始：  End     大小    文件系统        Name                          标志
      17.4kB  1049kB  1031kB  剩余空间
 1    1049kB  106MB   105MB   fat32           EFI system partition          启动, esp
 2    106MB   123MB   16.8MB                  Microsoft reserved partition  msftres
 3    123MB   553GB   553GB   ntfs            Basic data partition          msftdata
      553GB   553GB   987kB   剩余空间
 4    553GB   658GB   105GB   ntfs            Basic data partition          msftdata
      658GB   1064GB  406GB   剩余空间
 5    1064GB  1279GB  215GB   zfs             Basic data partition          msftdata
      1279GB  1279GB  1049kB  剩余空间
 6    1279GB  1296GB  17.2GB  linux-swap(v1)  Linux swap partition
      1296GB  1301GB  5369MB  剩余空间
 7    1301GB  2000GB  699GB   ext4
 8    2000GB  2000GB  611MB   ntfs                                          隐藏分区, diag
      2000GB  2000GB  1122kB  剩余空间
```

可以看到还有`406GB`剩余空间可用，可以将它添加到zfs存储池

```shell
(base) xd@xd-Super-Server:~$ sudo parted /dev/nvme0n1 #使用 parted 工具创建一个新的分区
GNU Parted 3.2
使用 /dev/nvme0n1
欢迎使用 GNU Parted! 输入 'help'可获得命令列表.
(parted) mkpart primary 658GB 1064GB                                      
(parted) quit                                                             
信息: You may need to update /etc/fstab.
(base) xd@xd-Super-Server:~$ sudo fdisk -l /dev/nvme0n1 #查看当前的存储设备分布
Disk /dev/nvme0n1：1.8 TiB，2000398934016 字节，3907029168 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
磁盘标识符：E297036A-738F-4B0D-BBCD-57C909CC08DB

设备                 起点       末尾       扇区   大小 类型
/dev/nvme0n1p1       2048     206847     204800   100M EFI 系统
/dev/nvme0n1p2     206848     239615      32768    16M Microsoft 保留
/dev/nvme0n1p3     239616 1079834743 1079595128 514.8G Microsoft 基本数据
/dev/nvme0n1p4 1079836672 1285015551  205178880  97.9G Microsoft 基本数据
/dev/nvme0n1p5 2077870080 2497298431  419428352   200G Microsoft 基本数据
/dev/nvme0n1p6 2497300480 2530856959   33556480    16G Linux swap
/dev/nvme0n1p7 2541342720 3905832959 1364490240 650.7G Linux 文件系统
/dev/nvme0n1p8 3905832960 3907026943    1193984   583M Windows 恢复环境
/dev/nvme0n1p9 1285015552 2077870079  792854528 378.1G Linux 文件系统

分区表记录没有按磁盘顺序。
```

这时候将它原来的空闲的406G空间映射到`/dev/nvme0n1p9`分区，注意到Linux系统显示的大小只有378.1G，这个可能是由于计算方式和对齐引起的。

```shell
(base) xd@xd-Super-Server:~$ sudo dd if=/dev/zero of=/dev/nvme0n1p9 bs=1M status=progress #删除新分区上的数据
405802057728 bytes (406 GB, 378 GiB) copied, 226 s, 1.8 GB/s
dd: 写入'/dev/nvme0n1p9' 出错: 设备上没有空间
记录了387137+0 的读入
记录了387136+0 的写出
405941518336 bytes (406 GB, 378 GiB) copied, 256.674 s, 1.6 GB/s
(base) xd@xd-Super-Server:~$ sudo wipefs -a /dev/nvme0n1p9 #清除分区上的文件系统签名
(base) xd@xd-Super-Server:~$ sudo zpool add default /dev/nvme0n1p9 #添加新分区到ZFS存储池
(base) xd@xd-Super-Server:~$ sudo zpool status default #验证存储池状态
  pool: default
 state: ONLINE
  scan: scrub repaired 0B in 0h4m with 0 errors on Sun Jun  9 00:28:19 2024
config:

	NAME         STATE     READ WRITE CKSUM
	default      ONLINE       0     0     0
	  nvme0n1p5  ONLINE       0     0     0
	  nvme0n1p9  ONLINE       0     0     0

errors: No known data errors
```

已经成功地将新分区 `/dev/nvme0n1p9` 添加到现有的 ZFS 存储池 `default` 中。现在存储池中包含两个分区 `nvme0n1p5` 和 `nvme0n1p9`。

```shell
(base) xd@xd-Super-Server:~$ sudo zpool list default #验证存储池容量
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
default   577G   169G   408G         -    16%    29%  1.00x  ONLINE  -
(base) xd@xd-Super-Server:~$ sudo zfs list #查看详细使用情况
NAME                                                                              USED  AVAIL  REFER  MOUNTPOINT
default                                                                           169G   390G    24K  none
default/containers                                                                168G   390G    24K  none
default/containers/Common-Server                                                 25.6G   390G  22.9G  /var/lib/lxd/storage-pools/default/containers/Common-Server
default/containers/gdw-server                                                     127G   390G  56.8G  /var/lib/lxd/storage-pools/default/containers/gdw-server
default/containers/hwt-server                                                    13.4G   390G  25.3G  /var/lib/lxd/storage-pools/default/containers/hwt-server
default/containers/zjy-server                                                    1.95G   390G  24.7G  /var/lib/lxd/storage-pools/default/containers/zjy-server
default/custom                                                                     24K   390G    24K  none
default/deleted                                                                    48K   390G    24K  none
default/deleted/images                                                             24K   390G    24K  none
default/images                                                                    453M   390G    24K  none
default/images/64509028accfe5a2727603687820708af03fa9a04f6821d40d1734a620cd587d   227M   390G   227M  none
default/images/fb684aaaea88810de6fd7873eb05e818f66c9128b33edb5d81663fc4626e9197   227M   390G   227M  none
default/snapshots                                                                  96K   390G    24K  none
default/snapshots/Common-Server                                                    24K   390G    24K  none
default/snapshots/gdw-server                                                       24K   390G    24K  none
default/snapshots/hwt-server                                                       24K   390G    24K  none
```

至此，可以看到剩余空间拓展到了390Ｇ













