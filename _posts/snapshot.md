---
layout: post
title: "How to create a Snapshot on LXC"
date: 2025-06-07
---

LXC平台提供了一个快照功能，支持像使用虚拟机一样给任何状态下的容器创建快照，在用户需要时可以方便的回溯到需要的快照，大大增强了系统的稳健性。

注意，快照功能必须要在宿主机上操作，同时保证容器已经关闭。若没有宿主机权限的容器用户，可以联系管理员


## 查看镜像列表

可以使用下面的命令来列出所有可用的镜像：

```bash
sudo lxc image list
```

## 创建快照

创建一个新的快照

```bash
xd@xd-Super-Server:~$ sudo lxc snapshot ubuntu2204 ubuntu2204-250609
```

查看容器的快照信息

···bash
xd@xd-Super-Server:~$ sudo lxc info ubuntu2204
Name: ubuntu2204
Remote: unix://
Architecture: x86_64
Created: 2025/06/09 03:32 UTC
Status: Running
Type: persistent
Profiles: default
Pid: 6334
Ips:
  eth0:	inet	192.168.31.98	vethTAGVH5
  eth0:	inet6	fe80::216:3eff:fe2f:b141	vethTAGVH5
  lo:	inet	127.0.0.1
  lo:	inet6	::1
Resources:
  Processes: 73
  Disk usage:
    root: 169.99MB
  CPU usage:
    CPU usage (in seconds): 38
  Memory usage:
    Memory (current): 331.97MB
    Memory (peak): 545.97MB
  Network usage:
    eth0:
      Bytes received: 43.15MB
      Bytes sent: 731.48kB
      Packets received: 37203
      Packets sent: 7311
    lo:
      Bytes received: 94.27kB
      Bytes sent: 94.27kB
      Packets received: 780
      Packets sent: 780
Snapshots:
  ubuntu2204-250609 (taken at 2025/06/09 03:54 UTC) (stateless)
```

设置对应的描述

```bash
xd@xd-Super-Server:~$ sudo lxc config set ubuntu2204   user.snapshot.description/ubuntu2204-250609   "支持ssh，配置了清华源"
```

查看对应的描述

```bash
xd@xd-Super-Server:~$ sudo lxc config get ubuntu2204   user.snapshot.description/ubuntu2204-250609
支持ssh，配置了清华源
```


## 恢复到快照

在开发中难免会遇到错误，当出现系统错误时，恢复到快照的状态就是一个后悔药

通过命令恢复到快照的位置

```bash
sudo lxc restore ubuntu2204 ubuntu2204-250609
```

## 基于快照创建新镜像

某些快照是完成系统重大更新的基础版本，我们希望基于这个版本继续开发，那么就需要基于这个快照创建新镜像

```bash
sudo lxc copy <容器名称>/<快照名称> <新镜像名称>
```
