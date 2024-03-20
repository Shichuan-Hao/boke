---
title: 【Linux】Centos 防火墙操作总结
date: 2024-02-01 06:31:57 +0800
categories: [博客]
tags: [centos, linux] 
---



## 基本使用

启动：`systemctl start firewalld`

关闭：`systemctl stop firewalld`

查看状态：`systemctl status firewalld`

开机禁用：`systemctl disable firewalld`

开机启用：`systemctl enable firewalld`

> systemctl 是 CentOS7 的服务管理工具中主要的工具，它融合之前 service 和 chkconfig 的功能于一体。
{: .prompt-tip }

启动一个服务：`systemctl start firewalld.service`

关闭一个服务：`systemctl stop firewalld.service`

重启一个服务：`systemctl restart firewalld.service`

显示一个服务的状态：`systemctl status firewalld.service`

在开机时启用一个服务：`systemctl enable firewalld.service`

在开机时禁用一个服务：`systemctl disable firewalld.service`

查看服务是否开机启动：`systemctl is-enabled firewalld.service`

查看已启动的服务列表：`systemctl list-unit-files | grep enabled`

查看启动失败的服务列表：`systemctl --failed`

## 配置 firewalld-cmd

查看版本：`firewalld-cmd --version`

查看帮助：`firewall-cmd --help`

显示状态：`firewall-cmd --state`

查看所有打开的端口：`firewall-cmd --zone=public --list-ports`

更新防火墙规则：`firewall-cmd --reload`

查看区域信息：`firewall-cmd --get-active-zones`

查看指定接口所属区域：`firewall-cmd --get-zone-of-interface=eth0`

拒绝所有包：`firewall-cmd --panic-on`

取消拒绝状态：`firewall-cmd --panic-off`

查看是否拒绝：`firewall-cmd --query-panic`

## 例子：开启防火墙端口

比如，需要打开防火墙 80 和 3306 端口

步骤 1: 设置开放的端口号

- `firewall-cmd --add-service=http --permanent`
- `firewall-cmd --add-port=80/tcp --permanent`
- `firewall-cmd --add-port=3060/tcp --permanent`

> `--permanent` 永久生效，没有此参数重启后失效
{: .prompt-tip }

步骤 2：重启防火墙 `firewall-cmd --reload`

步骤 3：查看开放端口 `fireall-cmd --list-all`