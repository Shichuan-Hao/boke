---
title: java.net.SocketException:Too many open files
author: shaun
date: 2019-07-11 21:33:11 +0800
categories: [文章]
tags:  [SpringBoot Coding and debug]
math: true
mermaid: true
---

在 Unix/Linux 系统中，如果遇到 <font color=red>java.net.SocketException:Too many open files</font> 错误，通常是由于我们的系统限制了文件打开数量。我们可以通过以下方法解决这个问题：

1. 修改 ulimit 设置（通过修改 ulimit 设置来提高文件打开数量的限制），命令是：`ulimit -n 2048`

2. 也可以修改 `/etc/launchd.conf` 来更改我们系统的 ulimit 限制，操作如下：
```shell
sudo nano /etc/launche.conf
```
然后添加以下行：
```conf
limit maxfiles 2048 2048
```
最后，重写系统。

3. 将参数 `-Djava.security.egd=file:/dev/./urandom`

以上，就可以解决 <font color=red>java.net.SocketException:Too many open files</font> 错误了。
