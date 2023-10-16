---
title: 利用 Linux 指令分析 Web 日志
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2018-01-11 13:33:35 +0800
categories: [计算机基础, 操作系统]
tags: [Linux]
math: true
mermaid: true
---


著名的黑客、自由软件运动的先驱理查德.斯托曼说过，“编程不是科学，编程是手艺”。可见，要想真正搞好编程，除了学习理论知识，还需要在实际的工作场景中进行反复的锤炼。



### 第一步：能不能这样做 !

当想要分析一个线上文件的时候，首先要思考，能不能这样做？ 

这里可以先用`htop`指令看一下当前的负载。如果机器上没有`htop`，则可以考虑用`yum`或者`apt`去安装。

![this is images](https://maxpixelton.github.io/images/assert/os/1101.png)

如上图所示，我的机器上 8 个 CPU 都是 0 负载，`2G`的内存用了一半多，还有富余。 

用`wget`将目标文件下载到本地（如果没有 wget，可以用`yum`或者`apt`安装）。

```
wget 某网址
```

然后用`ls`查看文件大小。发现这只是一个 7M 的文件，因此对线上的影响可以忽略不计。

如果文件太大，建议用`scp`指令将文件拷贝到闲置服务器再分析

下图中我使用了`--block-size`让`ls`以`M`为单位显示文件大小。

![this is images](https://maxpixelton.github.io/images/assert/os/1102.png)

确定了当前机器的`CPU`和内存允许进行分析后，就可以开始第二步操作了。



### 第二步：LESS 日志文件

在分析日志前，记得要`less`一下，看看日志里面的内容。尽量使用`less`这种不需要读取全部文件的指令，**因为在线上执行`cat`是一件非常危险的事情，这可能导致线上服务器资源不足。**

![this is images](https://maxpixelton.github.io/images/assert/os/1103.png)

如上图所示，看到`nginx`的`access_log`每一行都是一次用户的访问，从左到右依次是：

- IP 地址；
- 时间；
- HTTP 请求的方法、路径和协议版本、返回的状态码；
- User Agent。

### 第三步：PV 分析

**PV（Page View），用户每访问一个页面就是一次`Page View`。**对于`nginx`的`acess_log`来说，分析 PV 非常简单，直接使用`wc -l`就可以看到整体的`PV`。

![this is images](https://maxpixelton.github.io/images/assert/os/1104.png)

### 第四步：PV 分组

通常一个日志中可能有几天的 PV，为了得到更加直观的数据，有时候需要按天进行分组。

为了简化这个问题，先来看看日志中都有哪些天的日志。

使用`awk '{print $4}' access.log | less`可以看到如下结果。

`awk`是一个处理文本的领域专有语言。这里就牵扯到领域专有语言这个概念，英文是 Domain Specific Language。

领域专有语言，就是为了处理某个领域专门设计的语言。比如 awk 是用来分析处理文本的DSL，html 是专门用来描述网页的DSL，SQL 是专门用来查询数据的DSL……还可以根据自己的业务设计某种针对业务的DSL。

![this is images](https://maxpixelton.github.io/images/assert/os/1105.png)

如果想要按天统计，可以利用 `awk`提供的字符串截取的能力。

![this is images](https://maxpixelton.github.io/images/assert/os/1106.png)

上图中，使用`awk`的`substr`函数，数字`2`代表从第 2 个字符开始，数字`11`代表截取 11 个字符。

接下来就可以分组统计每天的日志条数了。

![this is images](https://maxpixelton.github.io/images/assert/os/1107.png)

上图中，使用`sort`进行排序，然后使用`uniq -c`进行统计。可以看到从 2015 年 5 月 17 号一直到 6 月 4 号的日志，还可以看到每天的 PV 量大概是在 2000~3000 之间。

### 第五步：分析 UV

接下来分析 UV。UV（Uniq Visitor），也就是统计访问人数。通常确定用户的身份是一个复杂的事情，但是可以用 IP 访问来近似统计 UV。

![this is images](https://maxpixelton.github.io/images/assert/os/1108.png)

上图中，使用 awk 去打印`$1`也就是第一列，接着`sort`排序，然后用`uniq`去重，最后用`wc -l`查看条数。 这样就知道日志文件中一共有`2660`个 IP，也就是`2660`个 UV。



### 第六步：分组分析 UV

接下来尝试按天分组分析每天的 UV 情况。这个情况比较复杂，需要较多的指令，先创建一个叫作`sum.sh`的`bash`脚本文件，写入如下内容：

```shell
#!/usr/bin/bash
awk '{print substr($4, 2, 11) " " $1}' access.log |\
	sort | uniq |\
	awk '{uv[$1]++;next}END{for (ip in uv) print ip, uv[ip]}'
```

具体分析如下。

- 文件首部,使用`#!`，表示将使用后面的`/usr/bin/bash`执行这个文件。
- 第一次`awk`我们将第 4 列的日期和第 1 列的`ip`地址拼接在一起。
- 下面的`sort`是把整个文件进行一次字典序排序，相当于先根据日期排序，再根据 IP 排序。
- 接下来用`uniq`去重，**日期 + IP** 相同的行就只保留一个。
- 最后的`awk`再根据第 1 列的时间和第 2 列的 IP 进行统计。

> `awk`的原理：
>
> `awk`本身是逐行进行处理的。因此`next`关键字是提醒`awk`跳转到下一行输入。
>
>  对每一行输入，`awk`会根据第 1 列的字符串（也就是日期）进行累加。
>
> 之后的`END`关键字代表一个触发器，就是 END 后面用 {} 括起来的语句会在所有输入都处理完之后执行——当所有输入都执行完，结果被累加到`uv`中后，通过`foreach`遍历`uv`中所有的`key`，去打印`ip`和`ip`对应的数量。

编写完上面的脚本之后，保存退出编辑器。接着执行`chmod +x ./sum.sh`，给`sum.sh`增加执行权限。然后可以像下图这样执行，获得结果：

![this is images](https://maxpixelton.github.io/images/assert/os/1109.png)

如上图，`IP`地址已经按天进行统计好了。

> 根据access_log 分析出有哪些终端访问了这个网站，并给出分组统计结果：
>
> - access_log 中有 Debian 和 Ubuntu 等等。可以利用下面的指令看到，第 12 列是终端，如下所示：
>
>   ```shell
>   cat nginx_logs.txt | awk '{print $12}' | head -n 5
>   ```
>
> - 还可以使用sort和uniq查看有哪些终端，如下所示：
>
>   ```shell
>   cat nginx_logs.txt | awk '{print $12}' | sort | uniq | head -n 10
>   ```
>
> - 最后需要写一个脚本，进行统计：
>
>   ```shell
>   cat nginx_logs.txt |\
>   awk '{tms[$12]++;next}END{for (t in tms) print t, tms[t]}'
>   ```
>
>   



> 根据 access_log 分析出访问量 Top 前三的网页：
>
> - 如果不需要 Substring 等复杂的处理，也可以使用sort和uniq的组合。如下所示：
>
>   ```shell
>   cat nginx_logs.txt | awk '{print $7}' | sort | uniq -C
>   ```