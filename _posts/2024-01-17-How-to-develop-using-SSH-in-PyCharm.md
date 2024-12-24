---
layout: post
title: "How to develop using SSH in PyCharm"
date: 2024-01-17
---

## 一、前言

对于开发者而言，通常更偏好图形界面，因为它直观简洁。然而，某些服务器提供商为了节约成本，通常默认仅提供一个SSH连接方式，这对于远程开发带来了一定挑战。

大多数开发工具为了满足这个需求，通常会提供一个远程SSH开发功能。以PyCharm为例，可以通过SSH实现服务器端和客户端的文件及编程指令同步，这为远程开发带来了极大的便利。

## 二、PyCharm的SSH开发原理

PyCharm中的远程SSH开发原理如图所示，本地客户端和服务器主机通过SSH和SFTP进行连接和文件传输

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212018049.png" alt="image-20240621201823008" style="zoom:100%;" />
</p>

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212016360.png" alt="Remote Execution in PyCharm – Random experiments in software engineering" style="zoom:80%;" />
</p>



## 三、PyCharm的SSH开发配置

PyCharm的SSH开发需要配置远程解释器和文件映射关系，主要步骤如下：

1、在PyCharm中打开工程文件夹

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212016543.png" alt="image-20240114214322456" style="zoom:50%;" />
</p>


2、在设置中选择配置SSH远程开发

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212016536.png" alt="image-20240114214500097" style="zoom: 50%;" />
</p>


3、输入远程服务器的IP和端口和用户名，这里的Host是非局域网下的域名，局域网下建议直接使用IP

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202407310159511.png" alt="image-20240731015911394" style="zoom:50%;" />
</p>


4、配置python解释器和远程服务器文件夹映射位置，解释器位置可在服务器中输入以下指令得到

```
which python
```

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212016565.png" alt="image-20240114214810509" style="zoom: 50%;" />
</p>


5、将本地文件同步到远程服务器

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202407310200266.png" alt="image-20240731020056007" style="zoom:80%;" />
</p>

6、选择比较本地和服务器端的文件大小和时间戳，将不同的部分进行同步即可

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202407310201238.png" alt="image-20240731020132073" style="zoom:80%;" />
</p>


7、开始Debug

<p align="center">
	<img src="https://cdn.jsdelivr.net/gh/dwgan/PicGo/img/202406212017271.png" alt="image-20240114215803132" style="zoom:80%;" />
</p>




## 参考文献

[搞AI开发，你不得不会的PyCharm技术](https://www.cnblogs.com/huaweiyun/p/16775793.html)

[Pycharm通过ssh远程连接服务器](https://blog.csdn.net/m0_45521766/article/details/126149339?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522170523956216800226520069%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=170523956216800226520069&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-126149339-null-null.142^v99^pc_search_result_base9&utm_term=pycharm%20ssh&spm=1018.2226.3001.4187)