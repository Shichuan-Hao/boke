---
title: Mojo 安装
date: 2023-09-10 14:31:57 +0800
categories: [文章]
tags: [Mojo] 
---


### Install on Linux
system requirements:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694135194770-f48aa247-7173-4443-b31e-834ed01770e8.png#averageHue=%23fcfaf7&clientId=u207a5411-fa2c-4&from=paste&height=107&id=u49e37f41&originHeight=107&originWidth=307&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9924&status=done&style=none&taskId=ufc445b81-d6c5-4096-8357-16566e20a61&title=&width=307)
as root:
```shell
apt-get install -y apt-transport-https &&
  keyring_location=/usr/share/keyrings/modular-installer-archive-keyring.gpg &&
  curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/gpg.0E4925737A3895AD.key' |  gpg --dearmor >> ${keyring_location} &&
  curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/config.deb.txt?distro=debian&codename=wheezy' > /etc/apt/sources.list.d/modular-installer.list &&
  apt-get update &&
  apt-get install -y modular
```
Install the Modular CLI:
```shell
curl https://get.modular.com | \
  MODULAR_AUTH=mut_7bc2611ad94c4df4915c732ecd0c2521 \
  sh -
```
Install the Mojo SDK:
```shell
modular install mojo
```
install the [Mojo extension for VS Code.](https://marketplace.visualstudio.com/items?itemName=modular-mojotools.vscode-mojo)
Get starte with [Hello World!](https://docs.modular.com/mojo/manual/get-started/hello-world.html)

 
