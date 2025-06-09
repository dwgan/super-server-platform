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

新系统安装完成后，为了提升使用体验，通常要创建新用户、配置镜像源、配置ssh功能、远程桌面

### 创建新用户

创建一个新用户

```bash
root@ubuntu2204:~# adduser xd
Adding user `xd' ...
Adding new group `xd' (1001) ...
Adding new user `xd' (1001) with group `xd' ...
Creating home directory `/home/xd' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for xd
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y
```

授予用户权限

```bash
# 加入管理员用户组
usermod -aG sudo xd
```

### 更新ssh权限

Ubuntu22.04.5的版本默认不允许密码登陆，查看相关文件

```bash
xd@ubuntu2204:~$ cat /etc/ssh/sshd_config

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no

```

可以看到这里没有启用密码登陆，同时包含了一些自定义的文件，其中禁用了密码登陆

```bash
root@ubuntu2204:~# nano /etc/ssh/sshd_config.d/60-cloudimg-settings.conf 
PasswordAuthentication no
```

这里需要修改`/etc/ssh/sshd_config.d/60-cloudimg-settings.conf`文件，将其改为`PasswordAuthentication yes`，同时在`/etc/ssh/sshd_config`文件中开启`PasswordAuthentication yes`，即允许密码登陆

重启`ssh`

```bash
systemctl restart ssh
```

尝试`ssh`连接

```bash
root@ubuntu2204:~# ssh xd@127.0.0.1
xd@127.0.0.1's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.4.0-150-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jun  9 03:50:09 UTC 2025

  System load:    0.79      Temperature:           39.0 C
  Usage of /home: unknown   Processes:             28
  Memory usage:   0%        Users logged in:       0
  Swap usage:     0%        IPv4 address for eth0: 192.168.31.98


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '24.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Mon Jun  9 03:44:15 2025 from 127.0.0.1
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

xd@ubuntu2204:~$ 
```

成功，退出`root`用户，以后只从普通用户登陆

### 配置镜像源

```bash
# 创建备份文件
xd@ubuntu2204:~# sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
# 更新为清华源
xd@ubuntu2204:~# sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
EOF
# 更新源
xd@ubuntu2204:~# sudo apt update
```
