---
title: docker mysql8-Failed to access directory for --secure-file-priv
date: 2024-02-01 06:31:57 +0800
categories: [博客]
tags: [centos, ios] 
---

1. 在配置文件 my.cnf 下增加 `secure-file-priv=/var/lib/mysql`

2. 将此文件夹映射出来即可：`-v /usr/local/mysql3308/mysql-files:/var/lib/mysql-files`，具体如下：
```shell
docker run -p 3308:3306 --privileged=true --name mysql3308 -v /usr/local/mysql3308/mysql:/etc/mysql -v /usr/local/mysql3308/logs:/logs -v /usr/local/mysql3308/data:/var/lib/mysql -v /usr/local/mysql3308/mysql/mysql-files:/var/lib/mysql-files  -e MYSQL_ROOT_PASSWORD=root12 -d mysql:8.0.19
```