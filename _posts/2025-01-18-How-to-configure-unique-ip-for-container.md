---
layout: post
title: "How to configure unique ip for container"
date: 2025-01-18
---





在Super-Server-Platform使用过程中，为了最好的实现资源隔离，需要给每个容器用户配置独立的IP。

采用的方案是，在宿主机上建立一个桥接网络，将所有的容器都桥接到这个网络上，容器系统就可以通过这个桥接网络连接到路由器，从而通过路由器的HDCP服务获取独立的IP。

## 实现方法

参考[这个](https://blog.csdn.net/liuquan0071/article/details/103352574)新建一个叫做`br0`的虚拟桥接网络

然后将`br0`绑定到实际的物理网络接口`eno1`上，同时添加一个回环网络，通过修改`/etc/network/interfaces`这个文件实现

```shell
(base) xd@xd-Super-Server:~$ sudo cat /etc/network/interfaces
# /etc/network/interfaces

# 配置回环接口
auto lo
iface lo inet loopback

# 配置桥接接口 br0
auto br0
iface br0 inet dhcp
    bridge_ports eno1
```

然后将容器连接到`br0`这个网络上，通过修改`/etc/lxc/default.conf`这个网络接口实现

```sh
(base) xd@xd-Super-Server:~$ sudo cat /etc/lxc/default.conf
lxc.net.0.type = veth
lxc.net.0.link = br0
lxc.net.0.flags = up
lxc.net.0.hwaddr = fe:c0:01:21:f0:bb
```

最终的网络状态应该如下，宿主机通过`eno1`这个接口与路由器通信，`br0`这个接口桥街道`eno1`上，因此它们的接口参数是一致的。容器桥街道`br0`上，最终和路由器通信。

```shell
(base) xd@xd-Super-Server:~$ ifconfig
br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.105  netmask 255.255.255.0  broadcast 192.168.31.255
        inet6 fe80::3eec:efff:febc:fc46  prefixlen 64  scopeid 0x20<link>
        ether 3c:ec:ef:bc:fc:46  txqueuelen 1000  (以太网)
        RX packets 1579  bytes 111132 (111.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 898  bytes 130366 (130.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:45:e1:cc:90  txqueuelen 0  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.105  netmask 255.255.255.0  broadcast 192.168.31.255
        ether 3c:ec:ef:bc:fc:46  txqueuelen 1000  (以太网)
        RX packets 16417  bytes 2315897 (2.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9432  bytes 1678660 (1.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enxb03af2b6059f: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether b0:3a:f2:b6:05:9f  txqueuelen 1000  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2  bytes 308 (308.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 6652  bytes 729004 (729.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6652  bytes 729004 (729.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lxcbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.0.3.1  netmask 255.255.255.0  broadcast 0.0.0.0
        ether 00:16:3e:00:00:00  txqueuelen 1000  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lxdbr0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.128.199.1  netmask 255.255.255.0  broadcast 0.0.0.0
        inet6 fd42:43e2:487f:c26e::1  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::9417:99ff:fe8d:3ad6  prefixlen 64  scopeid 0x20<link>
        ether 96:17:99:8d:3a:d6  txqueuelen 1000  (以太网)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 127  bytes 15896 (15.8 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethDELE7P: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fcb7:17ff:fe0d:c46f  prefixlen 64  scopeid 0x20<link>
        ether fe:b7:17:0d:c4:6f  txqueuelen 1000  (以太网)
        RX packets 234  bytes 31150 (31.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 425  bytes 74739 (74.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethSOJUPG: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc75:88ff:fe99:4a21  prefixlen 64  scopeid 0x20<link>
        ether fe:75:88:99:4a:21  txqueuelen 1000  (以太网)
        RX packets 45  bytes 5831 (5.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 381  bytes 32626 (32.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethXH30GT: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc27:6ff:fe50:efd4  prefixlen 64  scopeid 0x20<link>
        ether fe:27:06:50:ef:d4  txqueuelen 1000  (以太网)
        RX packets 157  bytes 21879 (21.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 432  bytes 67713 (67.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

注意不同的物理接口对应不同的MAC，更换物理的接口需要在系统上进行相应的配置，目前使用的物理接口如下图

![image-20250118114645458](https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/image-20250118114645458.png)

![微信图片_20250118115135](https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250118115135.png)