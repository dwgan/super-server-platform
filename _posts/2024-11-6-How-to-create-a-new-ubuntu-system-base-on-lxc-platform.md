---
layout: post
title: "How to create a new LXD based system"
date: 2024-01-17
---

## 一、新增一个系统

#### 创建系统

新增一个ubuntu22.04的系统

```shell
(base) xd@xd-Super-Server:~$ sudo lxc launch ubuntu:22.04 build-server
Creating build-server
Starting build-server
(base) xd@xd-Super-Server:~$ sudo lxc list
+---------------+---------+-----------------------+------+------------+-----------+
|     NAME      |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+---------------+---------+-----------------------+------+------------+-----------+
| Common-Server | RUNNING | 192.168.31.90 (eth0)  |      | PERSISTENT | 0         |
+---------------+---------+-----------------------+------+------------+-----------+
| build-server  | RUNNING | 192.168.31.252 (eth0) |      | PERSISTENT | 0         |
+---------------+---------+-----------------------+------+------------+-----------+
```

连接到系统

```shell
(base) xd@xd-Super-Server:~$ sudo lxc exec build-server -- /bin/bash
```

#### 创建新用户

[【ubuntu】ubuntu添加或删除用户_ubuntu删除子用户-CSDN博客](https://blog.csdn.net/u011119817/article/details/100709942)

#### 配置镜像源

[Ubuntu22.04更换阿里镜像源_ubuntu22.04更换阿里源-CSDN博客](https://blog.csdn.net/weixin_43996864/article/details/136823530)

#### 配置SSH

将客户端密钥复制到服务端

```shell
$ cat ~/.ssh/id_rsa.pub # 显示客户端密钥
```

粘贴到如下文件中

```shell
xd@build-server:~$ sudo vi ~/.ssh/authorized_keys
```

添加权限

```shell
sudo chown xd:xd /home/xd/.ssh
sudo chown xd:xd /home/xd/.ssh/authorized_keys
```

#### 配置内网穿透