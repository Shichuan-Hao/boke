---
title: 进程、重定向和管道指令
author:
  name: superhsc
  link: https://github.com/happymaya
date: 2018-01-07 22:33:35 +0800
categories: [计算机基础, 操作系统]
tags: [图灵机, 图灵机的构造, 图灵机执行程序, 冯诺依曼模型]
math: true
mermaid: true
---


理解管道相关内容之前，有一些概念需要了解，比如：==进程==、==标准流==和==重定向==。

## 进程

**应用的可执行文件是放在文件系统里，把可执行文件启动，就会在操作系统里（具体来说是内存中）形成一个==应用的副本==，这个副本就是进程（操作系统分配资源的最小单位，作用）。**

ps

使用 ps 指令，查看当前进程。

p 代表 processes，就是进程；s 代表 snapshot，就是快照（像拍照一样）。

```bash
[root@ca001 ~]# ps
  PID TTY          TIME CMD
 5679 pts/0    00:00:00 bash
11200 pts/0    00:00:00 ps
```

如上，启动了两个进程，ps 和 bash：

- ps 是刚刚启动的，被 ps自己捕捉到了；
- bash 是因为开了这个控制台，执行的shell是bash。

当然操作系统不可能只有这么几个进程，这是因为不带任何参数的 ps 指令显示的是同一个电传打字机（TTY上）的进程。TTY 这个概念是一个历史的概念，过去用来传递信息，现在已经被传真、邮件、微信等取代。

操作系统上的 TTY 是一个==输入输出终端的概念==，比如用户打开 bash，操作系统就为用户分配了一个输入输出终端。没有加任何参数的 ps 只显示在同一个 TTY 的进程。

如果想看到所有的进程，可以用ps -e，-e 没有特殊含义，只是为了和 -A 区分开。

通常不直接用ps -e，而是用 ps -ef，因为 - f 可以带上更多的描述字段，如下图所示：

```bash
[root@ca001 ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  2021 ?        01:13:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2     0  0  2021 ?        00:00:00 [kthreadd]
root         4     2  0  2021 ?        00:00:00 [kworker/0:0H]
root         6     2  0  2021 ?        00:01:01 [ksoftirqd/0]
root         7     2  0  2021 ?        00:00:31 [migration/0]
root         8     2  0  2021 ?        00:00:00 [rcu_bh]
root         9     2  0  2021 ?        02:35:46 [rcu_sched]
root        10     2  0  2021 ?        00:00:00 [lru-add-drain]
```

- UID，进程所有者；
- PID，进程唯一标识；
- PPID，进程父进程 ID；
- C , CPU 利用率（就是 CPU 占用）；
- STIME，开始时间；
- TTY 是进程所在的 TTY，如果没有 TTY 就是 ？号；
- TIME，
- CMD，进程启动时的命令，如果不是一个 Shell 命令，而是用方括号括起来，那就是系统进程或者内核过程。



另外一个用得比较多的是`ps aux`，和 ps -ef 能力差不多，是 [BSD]() 风格的（加州伯克利分校研发的 Unix 分支版本的衍生风格），如下：

```bash
[root@ca001 ~]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  51756  2776 ?        Ss    2021  73:01 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2  0.0  0.0      0     0 ?        S     2021   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        S<    2021   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S     2021   1:02 [ksoftirqd/0]
root         7  0.0  0.0      0     0 ?        S     2021   0:31 [migration/0]
root         8  0.0  0.0      0     0 ?        S     2021   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        S     2021 155:46 [rcu_sched]
root        10  0.0  0.0      0     0 ?        S<    2021   0:00 [lru-add-drain]
root        11  0.0  0.0      0     0 ?        S     2021   1:20 [watchdog/0]
```

在 BSD 风格中有些字段的叫法和含义变了。

### top

还有一个和 ps 能力差不多，但是显示是实时更新数据的 top 指令。

因为自带的 top 显示的内容有点少， 所以我喜欢用一个叫作 htop 的指令如下图所示：

以上，进程只是一个皮毛，更多的之后研究！

## 管道（Pipeline）

管道（Pipeline）的作用是：==**在命令和命令之间，传递数据。**==

比如说：**一个命令的结果，可以作为另一个命令的输入。**

==**这里说的命令就是进程。更准确地说，管道在进程间传递数据。**==

## 标准输入/输出流

每个进程拥有自己的标准输入流、标准输出流、标准错误流。

这几个标准流说起来很复杂，但其实都是文件：

- 标准输入流（用 0 表示），作为进程执行的上下文（进程执行可以从输入流中获取数据）；
- 标准输出流（用 1 表示），写入的结果会被打印到屏幕上；
- 如果进程在执行过程中发生异常，那么异常信息会被记录到标准错误流（用 2 表示）中。

## 重定向

执行一个指令，比如 ls -l，结果会写入标准输出流，进而被打印。

这时可以**用重定向符将结果重定向到一个文件，**

比如说：

```
ls -l > out
```

这样 out 文件就会有 ls -l 的结果，屏幕上也不会再打印 ls -l 的结果。

具体来说：

- `>` 符号，覆盖重定向，每次都会把目标文件覆盖；
- `>>` 符号，追加重定向。会在目标文件中追加。

比如每次启动一个程序日志都写入`/var/log/somelogfile`中，可以这样操作，如下所示：

```bash
start.sh >> /var/log/somelogfile
```

经过这样的操作后，每次执行程序日志就不会被覆盖了。

另外还有一种情况，比如输入：

```
ls1 > out
```

结果并不会存入out文件，因为 ls1 指令是不存在的。结果会输出到标准错误流中，仍然在屏幕上。这里可以==**把标准错误流也重定向到标准输出流，然后再重定向到文件。**==

```bash
ls1 & > out
```

这个写法等价于：

```bash
ls1 > out 2>&1
```

相当于把 ls1 的标准输出流重定向到 out，因为 ls1 > out 出错了，所以标准错误流被定向到了标准输出流。

& 代表一种引用关系，具体代表的是 ls1 >out 的标准输出流。

## 管道的作用和分类

晓得了进程和重定向，管道的作用呼之欲出。

**管道（Pipeline）将一个进程的输出流定向到另一个进程的输入流，就像水管一样，作用就是把这两个文件接起来。**如果一个进程输出了一个字符 X，那么另一个进程就会获得 X 这个输入。

**管道和重定向很像，但是管道是一个连接一个进行计算，重定向是将一个文件的内容定向到另一个文件，这二者经常会结合使用。**

Linux 中的管道也是文件，有两种类型的管道：

- 匿名管道（Unnamed Pipeline），这种管道也在文件系统中，但是它只是一个存储节点，不属于任何一个目录。说白了，就是没有路径；
- 命名管道（Named Pipeline），这种管道就是一个文件，有自己的路径。

### FIFO

管道具有 FIFO（First In First Out），FIFO 和排队场景一样，先排到的先获得。所以先流入管道文件的数据，也会先流出去传递给管道下游的进程。

## 使用场景

### 排序

比如用 ls，希望按照文件名排序倒序，可以使用匿名管道，将 ls 的结果传递给 sort 指令去排序。

这样 ls 的开发者就不用关心排序问题了。

```
[root@localhost maya]# ls | sort -r
out
hello
a.txt
[root@localhost maya]# ls | sort
a.txt
hello
out
```



### 去重

比如有一个字典文件，里面都是词语。如下所示：

```bash
[root@localhost maya]# touch b.txt
[root@localhost maya]# vim b.txt 
[root@localhost maya]# cat b.txt 
apple
Banana
Apple
Banana
```

如果想要去重可以使用 uniq 指令，uniq 指令能够找到文件中相邻的重复行，然后去重。

由于上面的文件重复行是交替的，所以不可以直接用uniq，需要先 sort 这个文件，然后利用管道将 sort 的结果重定向到 uniq 指令。指令如下：

```bash
[root@localhost maya]# sort b.txt | uniq
apple
Apple
Banana
```

### 筛选

有时候想根据正则模式筛选对应的内容。

比如说想找到项目文件下所有文件名中含有 Spring 的文件。就可以利用 grep 指令，操作如下：

```bash
[root@localhost maya]# find ./ | grep Spring
```

- `find ./` 递归列出当前目录下所有目录中的文件；

- `grep` 从 find 的输出流中找出含有 Spring 关键字的行。

如果希望包含 Spring 但不包含 MyBatis 可以这样操作：

```bash
find ./ | grep Spring | grep -v MyBatis
```

- grep -v是匹配不包含 MyBatis 的结果。

### 数行数

还有一个比较常见的场景是数行数。比如想知道一个文件里面有多少行，就可以使用`wc -l`指令，如下所示：

```
[root@localhost maya]# wc -l c.txt 
3 c.txt
```

但是如果想知道当前目录下有多少个文件，可以用 `ls | wc -l`，如下所示：

```bash
[root@localhost var]# ls | wc -l
30
```

想知道当前 java 的项目目录下有多少行代码，可以使用下面这个指令：

```
find -i ".java" ./ | wc -l
```



### 中间结果

管道一个接着一个，是一个计算逻辑。有时候想要把中间的结果保存下来，这就需要用到 `tee` 指令。

`tee` 指令从标准输入流中读取数据到标准输出流。

`tee` 还有一个能力，就是自己利用这个过程把输入流中读取到的数据存到文件中。比如下面这条指令：

```
find ./ -i "*.java" | tee JavaList | grep Spring
```

这句指令的意思是从当前目录中找到所有含有 Spring 关键字的 Java 文件。

`tee` 本身不影响指令的执行，但是 `tee` 会把 find 指令的结果保存到 JavaList 文件中。

`tee` 这个执行就像英文字母中的 T 一样，连通管道两端，下面又开了口。这个开口，在函数式编程里面叫作副作用。

### xargs

`xargs` 指令**从标准数据流中构造并执行一行行的指令。**

`xargs` 从输入流获取字符串，然后利用空白、换行符等切割字符串，在这些字符串的基础上构造指令，最后一行行执行这些指令。

比如，重命名当前目录下的所有 .a 的文件，想在这些文件前面加一个前缀 prefix_。比如说 x.a文件需要重命名成prefix_x.a，就可以用 xargs 指令构造模块化的指令。

现在有`x.a`、`y.a`、`z.a`三个文件，如下图所示：

```bash
[root@localhost lala]# ls
x.a  y.a  z.a
```

然后使用下面指令，构造需要的指令：

```bash
[root@localhost lala]# ls | xargs -I GG echo "mv GG prefix_GG"
mv x.a prefix_x.a
mv y.a prefix_y.a
mv z.a prefix_z.a
```

- ls，找到所有的文件；
- -I 参数，查找替换符，这里用 GG 替代 ls 找到的结果；-I GG 后面的字符串 GG 会被替换为x.a``x.b或x.z；
- echo，是一个在命令行打印字符串的指令。使用 echo 主要是为了安全，帮助检查指令是否有错误。

用 xargs 构造了 3 条指令。（如果没有用xargs指令，而是用一条条mv指令去敲，这样就构成了样板代码）。

最后去掉 echo，就是最终想要的结果，如下所示：

```bash
[root@localhost lala]# ls | xargs -I GG mv GG prefix_GG
[root@localhost lala]# ls
prefix_x.a  prefix_y.a  prefix_z.a
[root@localhost lala]# 
```

## 管道文件

匿名管道，用==|==就可以创造和使用。

匿名管道也是利用了文件系统的能力，是一种文件结构，它拥有一个自己的 inode，但不属于任何一个文件夹。

还有一种管道叫作**命名管道（Named Pipeline）**。命名管道是要挂到文件夹中的，因此需要创建。

用`mkfifo`指令可以创建一个命名管道，下面创建一个叫作 pipe1 的命名管道，如所示：

```bash
[root@localhost lala]# mkfifo pipe1
[root@localhost lala]# ls
pipe1  prefix_x.a  prefix_y.a  prefix_z.a
[root@localhost lala]# 
```

命名管道和匿名管道能力类似，可以连接一个输出流到另一个输入流，也是 First In First Out。

当执行cat pipe1的时候，你可以观察到，当前的终端处于等待状态。因为 cat pipe1 的时候pipe1中没有内容。

如果此时，再找一个终端去写一点东西到 pipe 中，比如说:

```bash
echo "XXX" > pipe1
```

这个时候，cat pipe1就会返回，并打印出xxx，如下所示：

```
[root@localhost lala]# cat pipe1 &

xxx
```

以像上图那样演示这段程序，在 cat pipe1 后面增加了一个 ==&== 符号，代表指令在后台执行，不会阻塞用户继续输入。然后通过echo 指令往 pipe1 中写入东西，接着就会看到 xxx 被打印出来。

## 总结

- 匿名管道帮助把 Linux 指令串联起来形成很强的计算能力。特别是 xargs 指令支持模板化的生成指令，拓展了指令的能力。

- 命名管道，命名管道让我们可以真实拿到一个管道文件，让多个程序之间可以方便地进行通信。

>  xargs 的作用：
>
> - xargs 将标准输入流中的字符串分割成一条条子字符串，然后再按照我们自己想要的方式构建成一条条指令，大大拓展了 Linux 指令的能力。
> - 比如可以用来按照某种特定的方式逐个处理一个目录下所有的文件；根据一个 IP 地址列表逐个 ping 这些 IP，收集到每个 IP 地址的延迟等。
>



> 下面 Shell 程序的作用是：
>
> ```bash
> mkfifo pipe1
> mkfifo pipe2
> echo -n run | cat - pipe1 > pipe2 &
> cat < pipe2 > pipe1
> ```
>
> - 前 2 行代码创建了两个管道文件；
> - 从第 3 行开始，代码变得复杂。`echo -n run` 就是向输出流中写入一个run字符串（不带回车，所以用-n）。通过管道，将这个结果传递给了cat。cat是 concatenate 的缩写，意思是把文件粘在一起。
> - 当 cat 用 > 重定向输出到一个管道文件时，如果没有其他进程从管道文件中读取内容，cat 会阻塞。
> - 当 cat 用 < 读取一个管道内容时，如果管道中没有输入，也会阻塞。
>
> 从这个角度来看，总共有 3 次重定向：
>
> - 将 - 也就是输入流的内容和 pipe1 内容合并重定向到 pipe2；
>
> - 将 pipe2 内容重定向到 cat；
> - 将 cat 的内容重定向到 pipe1。
>
> 观察下路径：pipe1 -> pipe2 -> pipe1，构成了一个循环。 这样导致管道 pipe1 管道 pipe2 中总是有数据（没有数据的时间太短）。于是，就构成了一个无限循环。打开执行这个程序后，可以用 htop 查看当前的 CPU 使用情况，会发现 CPU 占用率很高。