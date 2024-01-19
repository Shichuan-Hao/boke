---
title: Mojo 尝鲜
date: 2023-09-10 14:31:57 +0800
categories: [博客]
tags: [mojo] 
---


### 在 Linux 上安装

系统要求:
- Ubuntu 20.04 and later
- x86-64 CPU and minimun 4 GiB RAM
- Python 3.8 - 3.10
- g++ or clang++ compiler

as root:
```shell
apt-get install -y apt-transport-https &&
  keyring_location=/usr/share/keyrings/modular-installer-archive-keyring.gpg &&
  curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/gpg.0E4925737A3895AD.key' |  gpg --dearmor >> ${keyring_location} &&
  curl -1sLf 'https://dl.modular.com/bBNWiLZX5igwHXeu/installer/config.deb.txt?distro=debian&codename=wheezy' > /etc/apt/sources.list.d/modular-installer.list &&
  apt-get update &&
  apt-get install -y modular
```

安装 Modular CLI:
```shell
curl https://get.modular.com | \
  MODULAR_AUTH=mut_7bc2611ad94c4df4915c732ecd0c2521 \
  sh -
```

安装 Mojo SDK:
```shell
modular install mojo
```

在 VS Code 安装 Mojo 扩展参见[这里](https://marketplace.visualstudio.com/items?itemName=modular-mojotools.vscode-mojo)

Mojo 的入门参见[这里](https://docs.modular.com/mojo/manual/get-started/hello-world.html)

 
