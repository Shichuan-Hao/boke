---
title: 软件的安装
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2018-01-10 21:33:35 +0800
categories: [博客]
tags: [linux]
math: true
mermaid: true
---

## 包管理器使用

Linux 下的应用程序多数以软件包的形式发布，用户拿到对应的包之后，使用包管理器进行安装。

说到包管理器，就要提到 `dpkg` 和 `rpm`。

先说说包。 Linux 下两大主流的包就是`rpm`和`dpkg`。

- `dpkg`（debian package），是`linux`一个主流的社区分支开发出来的。社区就是开源社区，有很多世界顶级的程序员会在社区贡献代码，比如 github。一般衍生于`debian`的 Linux 版本都支持`dpkg`，比如`ubuntu`；
- `rpm`（redhatpackage manager）。

> 先熟悉两种包之前，先了解下 RedHat 这家公司。
>
> RedHat 是一个做 Linux 的公司，可以把它理解成一家“保险公司”。 很多公司购买红帽的服务，是为了给自己的业务上一个保险。以防万一哪天公司内部搞不定 Linux 底层，或者底层有 Bug，再或者底层不适合当下的业务发展，需要修改等问题，红帽的工程师都可以帮企业解决。
>
> 再比如，RedHat 收购了JBoss，把 JBoss 改名为 WildFly。 像 WildFly 这种工具更多是面向企业级，比如没有大量研发团队的企业会更倾向使用成熟的技术。RedHat 公司也有自己的 Linux，就叫作 RedHat。RedHat 系比较重要的 Linux 有 RedHat/Fedora 等。



无论是`dpkg`还是`rpm`都抽象了自己的包格式，就是以`.dpkg`或者`.rpm`结尾的文件。

`dpkg`和`rpm`也都提供了类似的能力：

- 查询是否已经安装了某个软件包；
- 查询目前安装了什么软件包；
- 给定一个软件包，进行安装；
- 删除一个安装好的软件包。

关于 dpkg 和 rpm 的具体用法，可以用 man 杨希了解。

# 自动依赖管理

Linux 是一个开源生态，因此工具非常多。工具在给用户使用之前，需要先打成`dpkg`或者`rpm`包。 有的时候一个包会依赖很多其他的包，而`dpkg`和`rpm`不会对这种情况进行管理，有时候为了装一个包需要先装十几个依赖的包，过程非常艰辛！因此现在多数情况都在用`yum`和`apt`。

### yum

`yum` 的全名是 Yellodog Updator Modified。 

看名字就知道它是基于`Yellodog Updator`这款软件修改而来的一个工具。

`yum`是 Python 开发的，提供的是`rpm`包，因此只有`redhat`系的 Linux，比如 Fedora，Centos 支持`yum`。

`yum`的主要能力就是解决下载和依赖两个问题。

下载之所以是问题，是因为 Linux 生态非常庞大，有时候用户不知道该去哪里下载一款工具。比如用户想安装`vim`，只需要输入`sudo yum install vim`就可以安装了。`yum`的服务器收集了很多`linux`软件，因此`yum`会帮助用户找到`vim`的包。

另一方面，`yum`帮助用户解决了很多依赖，比如用户安装一个软件依赖了 10 个其他的软件，`yum`会把这 11 个软件一次性的装好。

关于`yum`的具体用法，可以使用 man 工具进行了解。

**apt**

因为我这次是用`ubuntu`Linux ，所以以 apt 为例子，yum 的用法是差不多的，具体的可以 man 一下。

`apt`全名是 Advanced Packaging Tools，是一个`debian`及其衍生 Linux 系统下的包管理器。

由于`advanced`（先进）是相对于`dpkg`而言的，因此它也能够提供和`yum`类似的下载和依赖管理能力。

比如在没有`vim`的机器上，可以用下面的指令安装`vim`。如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/os/1001.png)

然后用`dpkg`指令查看 vim 的状态是`ii`：

- 第一个`i`代表期望状态是已安装；
- 第二个`i`代表实际状态是已安装。

下面卸载`vim`，再通过`dpkg`查看，如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/os/1002.png)

![this is images](https://maxpixelton.github.io/images/assert/os/1003.png)

看到 vim 的状态从`ii`变成了`rc`，`r`是期望删除，`c`是实际上还有配置文件遗留。 

如果想彻底删除配置文件，可以使用`apt purge`，就是彻底清除的意思，如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/os/1005.png)

再使用`dpkg -l`时，`vim`已经清除了：

![this is images](https://maxpixelton.github.io/images/assert/os/1006.png)

期待结果是`u`就是 unkonw（未知）说明已经没有了。实际结果是`n`，就是 not-installed（未安装）。

如果想查询`mysql`相关的包，可以使用`apt serach mysql`，这样会看到很多和`mysql`相关的包，如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/os/1007.png)

如果想精确查找一个叫作`mysql-server`的包，可以用`apt list`。

![this is images](https://maxpixelton.github.io/images/assert/os/1008.png)

这里找到了`mysql-server`包。

另外有时候国内的`apt`服务器速度比较慢，可以尝试使用阿里云的镜像服务器。具体可参考我下面的操作：

```
cat /etc/apt/sources.list

--以下是文件内容--
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```

镜像地址可以通过`/etc/apt/sources.list`配置，注意`focal`是我用的`ubuntu`版本，可以使用`sudo lsb_release`查看自己的 Ubuntu 版本。

如果用上面给出的内容覆盖`sources.list`，只需把版本号改成你自己的。注意，每个`ubuntu`版本都有自己的代号。

![this is images](https://maxpixelton.github.io/images/assert/os/1010.png)



## 编译安装 Nginx

编译安装 Nginx（发音是 engine X），是一个家喻户晓的 Web 服务器。 它的发明者是俄国的伊戈尔·赛索耶夫。

赛索耶夫 2002 年开始写 Nginx，主要目的是解决同一个互联网节点同时进入大量并发请求的问题。注意，大量并发请求不是大量 QPS 的意思，QPS 是吞吐量大，需要快速响应，而高并发时则需要合理安排任务调度。

后来塞索耶夫成立了 Nginx 公司， 2018 年估值到达到 4.3 亿美金。现在基本上国内大厂的 Web 服务器都是基于 Nginx，只不过进行了特殊的修改，比如淘宝用 Tengine。

在 Linux 上获取`nginx`源码，可以去搜索 [Nginx 官方网站](http://nginx.org/en/docs/)，一般都会提供源码包。

![this is images](https://maxpixelton.github.io/images/assert/os/1011.png)

如上图所示，可以看到 nginx-1.18.0 的网址是：http://nginx.org/download/nginx-1.19.2.tar.gz。然后用 wget 去下载这个包。 wget 是 GNU 项目下的下载工具，GNU 是早期`unix`项目的一个变种。`linux`下很多工具都是从`unix`继承来的，这就是开源的好处，很多工具不用再次开发了。可能很难想象`windows`下的命令工具可以在`linux`下用，但是`linux`下的工具却可以在任何系统中用。 因此，`linux`下面的工具发展速度很快，如今已成为最受欢迎的服务器操作系统。

当然也有同学的机器上没有`wget`，可以用`apt`安装一下。

1. 第一步：下载源码。使用`wget`下载`nginx`源码包：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1012.png)

2. 第二步：解压。解压下载好的`nginx`源码包：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1013.png)

   用 `ls` 发现包已经存在了，然后使用 tar 命令解压。

   tar 是用来打包和解压用的。之所以叫作 tar 有一些历史原因：

   - t 代表：tape（磁带）；
   - ar 代表：archive（档案）。

   因为早期存储介质很小，人们习惯把文件打包然后存储到磁带上，那时候 unix 用的命令就是tar。因为linux是个开源生态，所以就沿袭下来继续使用tar。

   - -x 代表：extract（提取）；
   - -z 代表：gzip，也就是解压 gz 类型的文件；
   - -v 代表 verbose（显示细节），如果不输入-v，就不会打印解压过程了；
   - -f 代表 file，这里指的是要操作文件，而不是磁带。

    所以 tar 解压通常带有 x 和 f，打包通常是c就是 create 的意思。

3. 第三步：配置和解决依赖。解压完，进入`nginx`的目录看一看。 如下图所示：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1014.png)

   可以看到一个叫作 configure 的文件是绿色的，也就是可执行文件。然后执行 configure 文件进行配置，这个配置文件来自一款叫作 autoconf 的工具，也是 GNU 项目下的，说白了就是bash（Bourne Shell）下的安装打包工具（就是个安装程序）。这个安装程序支持很多配置，你可以用./configure --help看到所有的配置项，如下图所示：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1015.png)

   这里有几个非常重要的配置项，叫作 prefix。prefix 配置项决定了软件的安装目录。如果不配置这个配置项，就会使用默认的安装目录。sbin-path 决定了nginx的可执行文件的位置。conf-path 决定了nginx配置文件的位置。我都使用了默认，然后执行./configure，如下图所示：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1016.png)

   autoconf 进行依赖检查的时候，报了一个错误，cc 没有找到。这是因为机器上没有安装 gcc 工具，gcc 是家喻户晓的工具套件，全名是 GNU Compiler Collection——里面涵盖了包括 c/c++ 在内的多门语言的编译器。

   用包管理器，安装 gcc，如下图所示。安装 gcc 通常是安装 build-essential 这个包。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1017.png)

   安装完成之后，再执行 ./configure，如下图所示：

   

   ![this is images](https://maxpixelton.github.io/images/assert/os/1018.png)

   看到配置程序开始执行。但是最终报了一个错误，如下图所示：

   ![this is images](https://maxpixelton.github.io/images/assert/os/1019.png)

   报错的内容是，nginx 的 HTTP rewrite 模块，需要 PCRE 库。 PCRE 是 per l语言的兼容正则表达式库。perl 语言一直以支持原生正则表达式，而受到广大编程爱好者的喜爱。我曾经看到过一个 IBM 的朋友用 perl 加上wget 就实现了一个简单的爬虫。接下来，开始安装PCRE。

   一般这种依赖库，会叫pcre-dev或者libpcre。用apt查询了一下，然后 grep。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1020.png)

   看到有 pcre2 也有 pcre3。这个时候可以考虑试试 pcre3。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1021.png)

   安装完成之后再试试 ./configure，提示还需要 zlib。然后用类似方法解决 zlib 依赖。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1022.png)

   zlib 包的名字叫zlib1g不太好找，需要查资料才能确定是这个名字。

   再尝试配置，终于配置成功了。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1023.png)

4. 第四步：编译和安装。

   通常配置完之后，输入`make && sudo make install`进行编译和安装。

   - make 是 linux 下面一个强大的构建工具；
   - autoconf 是`./configure`会在当前目录下生成一个 MakeFile 文件；
   - make会根据 MakeFile 文件编译整个项目。编译完成后，能够形成和当前操作系统以及 CPU 指令集兼容的二进制可执行文件。然后再用make install安装。&& 符号代表执行完 make 再去执行make installl。

   ![this is images](https://maxpixelton.github.io/images/assert/os/1024.png)

   可以看到编译是个非常慢的活。等待了差不多 1 分钟，终于结束了。nginx 被安装到了/usr/local/nginx中，如果需要让 nginx 全局执行，可以设置一个软连接到 /usr/local/bin，具体如下：

   ```bash
   ln -sf /usr/local/nginx/sbin/nginx /usr/local/sbin/nginx
   ```

   

## 为什么会有编译安装？

原来使用 C/C++ 写的程序存在一个交叉编译的问题。就是写一次程序，在很多个平台执行。

而不同指令集的 CPU 指令，还有操作系统的可执行文件格式是不同的。因此，这里有非常多的现实问题需要解决。

一般是由操作系统的提供方，比如 RedHat 来牵头解决这些问题。可以用`apt`等工具提供给用户已经编译好的包。`apt`会自动根据用户的平台类型选择不同的包。

但如果某个包没有在平台侧注册，也没有提供某个 Linux 平台的软件包，就需要回退到编译安装，通过源代码直接在某个平台安装。



## 总结

**编译安装和包管理安装有什么优势和劣势了吗？**

- 包管理安装很方便，但是有两点劣势：
  - 需要提前将包编译好，因此有一个发布的过程，如果某个包没有发布版本，或者在某个平台上找不到对应的发布版本，就需要编译安装；
  - 如果一个软件的定制程度很高，可能会在编译阶段传入参数，比如利用`configure`传入配置参数，这种时候就需要编译安装。

**如果你在编译安装 MySQL 时，发现找不到 libcrypt.so，应该如何处理**？

- 遇到这类问题，首先应该去查资料。 比如查 StackOverflow，搜索关键词：libcrypt.so not found，或者带上自己的操作系统ubuntu；

- 下面关于 Stackoverflow 的一个解答：

  In Ubuntu 20.04 , `libcrypt.so.1` is provided by <u>libxcrypt</u> package, You need to install the `i386` variant of that：

  > sudo apt install libcrypt:i386

  You might need to enable `i386` first:

  > sudo dpkg --add-architercture i386
  >
  > sudo apt update