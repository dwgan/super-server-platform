---
layout: post
title: How to develop using SSH in PyCharm
date: 2024-06-10
---

# Welcome to the Super Server Platform


# 前言

对于团队开发而言，资源共享是是提高生产效率的有效手段之一，尤其是在当今国际贸易制裁下，高端服务器资源公用将会成为常态。如何合理、高效地利用服务器资源，是本项目要解决的问题。

在多人共用一台GPU服务器的情况下，为了满足不同用户的使用方式和操作习惯特点，同时避免某些用户运行损害系统的命令导致整个服务器宕机，搭建一个支持多人**同时在线、资源良好隔离且自愈性强**的服务器平台成为了一个重要需求。

多人在线服务器有多种方案，其中一个较为推荐的解决方案是**通过虚拟化容器技术来隔离每个人的操作系统，并通过共享文件夹的形式达到多人共用的数据资源。**

该仓库包含了关于super-server-platform的搭建、升级记录和使用建议，服务器搭建和升级过程中涉及到的材料将会被包含到这个仓库中，方便回溯和更新。



# 多人在线服务器搭建方案

针对多人服务器平台通常有以下几种方案：

#### 1、原生系统上直接创建多用户

这种方案简单，只需要开放对应端口即可通过ssh和vnc实现远程操控。然而无法实现资源的隔离，例如无法按需分配GPU资源，无法个性化每个用户的开发环境，另外，某个用户运行损害系统的命令可能会导致整个系统宕机，影响其它用户使用。

#### 2、原生系统上搭建虚拟机

该方案可以实现各个用户的开发环境隔离，同时各个用户的系统之间不会相互影响。 虚拟机包含完整的操作系统和应用程序，因此通常需要更多的内存和存储资源。虚拟机的引入会带来性能损耗，且硬件只能独占，不能共享，资源利用率低。

#### 3、采用Docker容器

Docker容器提供了一个隔离的开发环境，在不同容器中的用户资源不会受到影响，同时由于容器共享主机操作系统的内核，因此它们更轻量级且占用更少的资源，使用容器几乎不会带来性能损失。虽然是容器化的环境，但Docker是**应用级**容器，他更偏向于PaaS平台，还是没办法做到让每个用户拥有一个独立的操作系统。

#### 4、基于LXC (LinuX Container) 容器

LXC容器属于**系统级**虚拟化方案，用在单个主机上运行多个隔离的Linux系统容器，例如同时运行Ubuntu和CentOS，各个系统公用一套Linux内核，并且具有完全独立的开发环境。LXC是基于Linux内核的容器技术，与Linux操作系统更加天然集成，提供更好的性能。但LXC也有缺点：如无法有效支持跨主机之间的容器迁移、管理复杂等。而LXD很好地解决了这些问题。


![](https://gitee.com/dwgan/PicGo/raw/main/img/202401141944205.png)

#### 5、基于LXD容器

LXD底层也是使用LXC技术，并可以提供更多灵活性和功能。LXD支持Web图形用户界面，使得容器的管理变得更加直观和易用。因此LXD可以视作LXC的升级版。LXD的管理命令和LXC的管理命令大多相同。



# 基于LXD容器的服务平台特性

基于LXD容器平台的核心是LXC，它允许建立多个系统级别的容器，每个容器即是一个完整的Linux操作系统。一个容器既可以支持多个用户共享，也可以一个用户独享。每个容器通过网络与共享文件夹的方式与宿主机进行通讯，每个容器可以独享一个IP地址。LXD支持Web图形用户界面，且兼容所有LXC的命令操作。通过配置LXC容器，可以实现对不同容器进行硬件资源（例如GPU）分配。宿主机系统和LXC容器系统共享一个Linux内核，相同内核的不同Linux发行版本可以同时运行。

![image-20240117221213953](https://gitee.com/dwgan/PicGo/raw/main/img/202401172212108.png)

具体服务器配置细节参考文末链接，这里不再详细赘述，以下简要介绍该平台使用方案。

## 配置方案

LXD支持多个用户，这里配置了Common-Server作为公共使用的容器平台，可以供所有用户临时使用。

对于需要长期使用的用户，建议创建独立的容器，并且根据自己需要配置个性化的开发环境，例如gdw，具体创建方法见文末链接。

这里采用了独立IP配置，因此每个容器在局域网下都有具有独立的IP。

![image-20240114201846610](https://gitee.com/dwgan/PicGo/raw/main/img/202401142018652.png)



## 资源隔离

如图，在宿主机上可存在4个4090GPU，通过对容器进行配置，可以指定宿主机上的GPU2和GPU3映射到容器系统的GPU0和GPU1上，并且其它的GPU资源对于容器是不可见的，因此可以实现良好的资源隔离。

![image-20240114203353541](https://gitee.com/dwgan/PicGo/raw/main/img/202401142033612.png)

![image-20240114203540145](https://gitee.com/dwgan/PicGo/raw/main/img/202401151147634.png)

![image-20240115114805293](https://gitee.com/dwgan/PicGo/raw/main/img/202401151148380.png)

![image-20240114203331560](https://gitee.com/dwgan/PicGo/raw/main/img/202401142033663.png)



## 文件共享

为了实现各个容器系统的文件共享，需要引入共享文件夹。如图，宿主机和Common-Server容器系统可以同时访问/home/xd/share文件夹

![image-20240114204049301](https://gitee.com/dwgan/PicGo/raw/main/img/202401142040334.png)



# 基于LXD容器的服务平台使用方案及操作指南

## 连接方式

#### 局域网下的SSH访问

例如要连接Common-Server，可在局域网下使用以下命令连接

```
ssh xd@192.168.31.90
```

![image-20240114202701161](https://gitee.com/dwgan/PicGo/raw/main/img/202401142027198.png)

#### 非局域网下的SSH访问

为了实现非局域网下访问，引入了内网穿透，这里将Common-Server映射到公网IP，可实现非局域网下访问。注意非局域网下仅有10M带宽，因此传输大文件建议使用局域网。具体连接方式如图

![image-20240115105916908](https://gitee.com/dwgan/PicGo/raw/main/img/202401151059053.png)

#### 在PyCharm中使用SSH远程开发

使用该方案，可以解决服务器界面不友好的问题。详见[How to develop using SSH in PyCharm](./How to develop using SSH in PyCharm)



## 镜像和快照

由于容器是运行在宿主机上的，宿主机具有对容器系统进行备份、恢复的能力。LXD容器提供了镜像和快照功能，将当前系统生成镜像可以将其快速部署到新的系统上（需要LXD环境）；通过生成系统快照，可以在系统出现问题时快速恢复到原先正常的状态。具体操作参考[How to create LXD snapshot and image](https://dwgan.github.io/super-server-platform/How to create LXD snapshot and image)，**注意：这一操作需要有权限访问宿主机。**



## 新增用户

基于LXD容器的系统最大的好处就是能够为每个用户提供一个完整的独立的开发环境（Linux内核是所有用户公用的），每个用户都可以根据自己的需求修改系统配置而不会影响到其他用户。

然而，更强大的方案往往意味着更高的技术门槛。因此，在不损失可玩性的同时，保证所有水平的用户拥有一个较好的体验，这里提供了两种使用方式。

#### 在Common-Server上新增用户

第一种方式是在一个公用容器（已经建立好了一个Common- Server）上直接建立多个用户，然后每个用户分配对应的权限（目前按照最高用户权限来）。这种方式的好处是操作简单，无需重新配置环境，对于新手或者对于无特殊环境要求的开发者友好（Common-Server的内网穿透、PyTorch环境已经配置好了）。具体操作参考[How to create a new LXD-based system](https://dwgan.github.io/super-server-platform/How to create a new LXD-based system)，**注意：这一操作需要有权限访问宿主机。**

#### 新增LXC容器用户

某些用户对开发环境有特殊要求，且不希望和其它用户共享环境配置，则建议新增一个LXD容器系统。对于新增LXD容器系统，目前有两种方式，其中一种是直接根据Common-Server的镜像克隆一个系统（系统版本是Ubuntu18.04）。对于需要其它版本系统的用户，需要重新下载新的镜像安装，此时可根据需要配置开发环境。具体操作参考[[How to create a new LXD-based system](https://dwgan.github.io/super-server-platform/How to create a new LXD-based system)，**注意：这一操作需要有权限访问宿主机。**



## TODO

### LXD管理界面

 LXD相比于LXC的好处就是多了界面管理，但是目前由于某些技术上的原因，还未能实现界面

### VNC桌面

由于某些技术上的原因，目前仅支持SSH连接。如需桌面显示请期待后续更新。。。



# 参考文献

[LXD vs Docker](https://ubuntu.com/blog/lxd-vs-docker)

[基于LXD搭建多人共用GPU服务器，简单易用，全网最详细！](https://blog.csdn.net/u012558210/article/details/120243466)

[ubuntu使用VNC实现远程桌面](https://blog.csdn.net/weixin_44543463/article/details/113846220)

[LXD教程入门实践 配置独立ip 挂载gpu显卡驱动 制作镜像](https://blog.csdn.net/liuquan0071/article/details/103352574)

所需要的材料可以在[这里](https://pan.baidu.com/s/1cjyBnPFdct7Y91AS4KV5Xg?pwd=kg3j)找到

