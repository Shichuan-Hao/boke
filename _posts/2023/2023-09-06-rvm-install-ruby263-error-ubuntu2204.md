---
title: 在 Ubuntu 22.04 中使用 RVM 安装 Ruby 错误
date: 2023-09-06 14:31:57 +0800
categories: [文章]
tags: [rvm, ruby, ubuntu, openssl] 
---

## 背景
在 Ubuntu 22.04 中，使用 rvm 安装 ruby 2.6.3 遇到如下错误：
```bash
"Error running '__rvm_make -j16', please read /home/dylan9706/.rvm/log/1671650995_ruby-2.7.4/make.log
```

## 原因
由于 Ubuntu 22.04 LTS 使用不同版本的 openssl，好像是 3。我们需要版本 1 的 open ssl 来安装就版本的 ruby 二进制文件

## 解决办法
```bash
$ sudo apt install build-essential
$ cd ~/Downloads
$ wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
$ tar zxvf openssl-1.1.1g.tar.gz
$ cd openssl-1.1.1g
$ ./config --prefix=$HOME/.openssl/openssl-1.1.1g --openssldir=$HOME/.openssl/openssl-1.1.1g
$ make
$ make test (you might get a few tests failing but just ignore)
$ make install
$ rm -rf ~/.openssl/openssl-1.1.1g/certs
$ ln -s /etc/ssl/certs ~/.openssl/openssl-1.1.1g/certs
$ cd ~
$ rvm install ruby-X.X.X --with-openssl-dir=$HOME/.openssl/openssl-1.1.1g (where ruby-X.X.X is the version you want to install
```