---
layout: post
title: "How to create a new LXC system"
date: 2025-06-07
---

某些开发环境下，只能采用特定版本的系统系统。

基于LXC的架构，宿主机已经采用了Ubuntu18.04的版本，但是用户可以自己选择需要的系统版本，例如Ubuntu22.04

## 创建新系统

LXC中创建新系统，

```bash
sudo lxc launch <系统版本> <容器名称>
```

这里以ubuntu22.04为例，这里会默认安装最新的ubuntu22.04.5

```bash
xxx@xxx-Super-Server:~$ sudo lxc launch ubuntu:22.04 ubuntu2204
Creating ubuntu2204
Starting ubuntu2204
xxx@xxx-Super-Server:~$
```

查看宿主机的版本信息

```bash
xd@xd-Super-Server:~$ cat /etc/os-release 
NAME="Ubuntu"
VERSION="18.04.6 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.6 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

查看当前的容器列表

```bash
xxx@xxx-Super-Server:~$ sudo lxc list
+---------------+---------+-----------------------+------+------------+-----------+
|     NAME      |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+---------------+---------+-----------------------+------+------------+-----------+
| ubuntu2204    | RUNNING | 192.168.31.176 (eth0) |      | PERSISTENT | 0         |
```

由于没有配置ssh，因此只能通过lxc bash连接

```bash
xxx@xx-Super-Server:~$ sudo lxc exec ubuntu2204 -- bash
# 查看容器的版本信息
root@ubuntu2204:~# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.5 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.5 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

可以看到，宿主机是`ubuntu18.04.6 LTS`，而容器是`Ubuntu 22.04.5 LTS`

## 配置新系统

新系统安装完成后，为了提升使用体验，通常要配置镜像源、配置ssh功能、远程桌面

配置镜像源

```bash
# 创建备份文件
root@ubuntu2204:~# sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 更新为清华源
root@ubuntu2204:~# sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
EOF
# 更新源
root@ubuntu2204:~# sudo apt update
```

