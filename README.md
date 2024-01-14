# 前言

这是一个关于super-server-platform搭建和升级的仓库，服务器搭建和升级过程中涉及到的材料将会被包含到这个仓库中，方便回溯和更新

# 多人在线服务器搭建方案

在多人共用一台GPU服务器的情况下，为了满足每个用户的使用方式和操作习惯特点，同时避免某些用户运行损害系统的命令导致整个服务器宕机，搭建一个支持多人同时在线、资源良好隔离、自愈性强的服务器平台成为了一个重要需求。一个较为推荐的解决方案是**通过虚拟化容器技术来隔离每个人的操作系统，并通过共享文件夹的形式达到多人共用的数据资源。**

针对多人服务器平台通常有以下几种方案：

1、原生系统上直接创建多用户

这种方案简单，只需要开放对应端口即可通过ssh和vnc实现远程操控。然而无法实现资源的隔离，例如无法按需分配GPU资源，无法个性化每个用户的开发环境，另外，某个用户运行损害系统的命令可能会导致整个系统宕机，影响其它用户使用。

2、原生系统上搭建虚拟机

该方案可以实现各个用户的开发环境隔离，同时各个用户的系统之间不会相互影响。然后虚拟机的引入会带来性能损耗，且硬件只能独占，不能共享，资源利用率低。

3、采用Docker容器

虽然是容器化的环境，但Docker是**应用级**容器，他更偏向于PaaS平台，还是没办法做到让每个用户拥有一个独立的操作系统。

4、基于LXC容器

LXC容器属于**系统级**虚拟化方案，用在单个主机上运行多个隔离的Linux系统容器，例如同时运行Ubuntu和CentOS，各个系统公用一套Linux内核，并且具有完全独立的开发环境。但LXC也有缺点：如无法有效支持跨主机之间的容器迁移、管理复杂等。而LXD很好地解决了这些问题。


![](https://gitee.com/dwgan/PicGo/raw/main/img/202401141944205.png)

4、基于LXD容器

LXD底层也是使用LXC技术，并可以提供更多灵活性和功能。因此LXD可以视作LXC的升级版。LXD的管理命令和LXC的管理命令大多相同。

# 基于LXD容器的服务平台配置

具体服务器配置细节参考文末链接，这里不再详细赘述。

### 配置方案

如图，Common-Server作为公共使用的容器平台，可以供用户临时使用。对于需要长期使用的用户，建议创建独立的容器，并且根据自己需要配置个性化的开发环境，例如gdw，具体创建方法待后续更新。

这里采用了独立ip配置，因此每个容器都有具有独立的ip。

![image-20240114201846610](https://gitee.com/dwgan/PicGo/raw/main/img/202401142018652.png)

### 资源隔离

如图，在宿主机上可存在4个4090GPU，通过对容器进行配置，可以指定宿主机上的GPU2和GPU3映射到容器系统的GPU0和GPU1上，并且其它的GPU资源对于容器是不可见的，因此可以实现良好的资源隔离。

![image-20240114203353541](https://gitee.com/dwgan/PicGo/raw/main/img/202401142033612.png)

![image-20240114203540145](https://gitee.com/dwgan/PicGo/raw/main/img/202401142035215.png)

![image-20240114203331560](https://gitee.com/dwgan/PicGo/raw/main/img/202401142033663.png)

### 文件共享

为了实现各个容器系统的文件共享，需要引入共享文件夹。如图，宿主机和Common-Server容器系统可以同时访问/home/xd/share文件夹

![image-20240114204049301](https://gitee.com/dwgan/PicGo/raw/main/img/202401142040334.png)

### 连接方式

#### 1、SSH

例如要连接Common-Server，可在局域网下使用以下命令连接

```
ssh 192.168.31.90
```

![image-20240114202701161](https://gitee.com/dwgan/PicGo/raw/main/img/202401142027198.png)

#### 2、VNC

由于某些技术上的原因，目前仅支持SSH连接。如需桌面显示请期待后续更新。。。

#### 3、非局域网下访问

为了实现非局域网下访问，引入了内网穿透，这里将Common-Server映射到公网IP，可实现非局域网下访问。注意非局域网下仅有10M带宽，因此传输大文件建议使用局域网。具体连接方式如图

![image-20240114204730196](https://gitee.com/dwgan/PicGo/raw/main/img/202401142047244.png)

#### 4、在PyCharm中使用SSH远程开发

使用该方案，可以解决服务器界面不友好的问题。详见[这里](https://dwgan.gitee.io/super-server-platform/How%20to%20develop%20using%20SSH%20in%20PyCharm.html)



# 参考文献

[基于LXD搭建多人共用GPU服务器，简单易用，全网最详细！](https://blog.csdn.net/u012558210/article/details/120243466)

[ubuntu使用VNC实现远程桌面](https://blog.csdn.net/weixin_44543463/article/details/113846220)

[LXD教程入门实践 配置独立ip 挂载gpu显卡驱动 制作镜像](https://blog.csdn.net/liuquan0071/article/details/103352574)

所需要的材料可以在[这里](https://pan.baidu.com/s/1cjyBnPFdct7Y91AS4KV5Xg?pwd=kg3j)找到
