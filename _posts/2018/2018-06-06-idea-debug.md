---
title: IDEA 的 Debug 模式
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2018-06-06 22:33:00 +0800
categories: [常用工具, IDEA]
tags:  [IDEA Debug]
math: true
mermaid: true
---


IDEA 几乎是 Java 程序员工作的标配，能够大大简化编码的额外工作，其中就包含对代码的 debug，即 IDEA 的 DEBUG 模式。只要程序出现问题（抛异常、结果不符合预期等等），几乎所有的人都会说一句：等我 debug 看看。但是，我在实际的工作、会使用 IDEA 做 debug 的同学少之又少。那么，这一节里，跟随我一起来看看怎样使用 IDEA debug 吧。本文用实例演示，怎样使用 IDEA 去 debug 代码，其中涉及基本用法、变量查看、计算表达式、断点调试等等

##  IDEA debug 模式下的界面

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0601.jpg)

这张图里面，我们需要关注三处（我用不同颜色的框进行了标记，且使用了编号）：

1. 启动类选择窗口（也能够对应用做配置），这里会记录当前工程我们曾经启动过的类（TestCase 也被包含在内）；
2. 运行按钮，针对你选择的启动类运行程序（注意，一定不要选错了，特别是历史记录较多的情况下）；
3. 调试按钮，当你点击之后，程序正常启动，几乎与 “运行按钮” 的行为一致

需要知道，IDEA 中几乎每一个按钮都有对应的快捷键，且快捷键支持自定义、修改。所以，如果很熟悉 IDEA 快捷键配置，完全可以使用快捷键而不是按钮的点击操作。

为了看到 IDEA debug 模式下的样子，先给 Controller 的方法打上一个断点（左键点击行号附近就可以），如下图红框中所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0602.jpg)

接着，点击调试按钮，以调试模式启动当前的工程，并发送请求到打了断点的那个工程，会发现，IDEA 界面变成了下面这样（注意，我在图中所做的标记）：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0603.jpg)

参照上图中所做的标记，其中 “每一项” 的含义是：

- 断点，左边行号附近单击左键可以给当前行添加或者去除断点
- debug tab（或者叫窗口），请求到达断点后，会自动激活 debug tab（idea 的默认行为）
- debug 按钮，debug 模式的核心实现，一共有8个按钮，可以实现方法进入、方法跳过、表达式计算等等，且把鼠标放置于按钮之上，会显示当前按钮功能说明及所对应的快捷键
- debug 模式功能按钮（相对的，如果是运行模式，则叫做运行模式功能按钮），包含了程序的启停、取消/恢复断点、查看线程快照等等
- 方法调用栈，显示了当前线程经过的所有方法，当勾选右上角的漏斗图标（Show All Frames）之后，就不会显示其它类库的方法了
- 变量区，在此区域内可以看到当前断点之前的当前方法内（包括方法的参数）的所有变量信息
- 变量查看区，可以新建或者是把变量区中的变量拖过去查看信息

### debug 按钮与 debug 模式功能按钮

#### debug 按钮

对照着下图，依次来看看这 8 个按钮都起到怎样的作用：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0604.jpg)

- `Show Execution Point`，点击这个按钮可以跳转到当前代码执行的行，这对于光标在其他页面时特别有用
- `Step Over`，步过，一行一行地往下走，如果这一行是调用方法，不会进入到方法中
- `Step Into`，步入，与 Over 类似，也是一行一行的往下走，但是，如果这一行是调用方法，则会进入到方法内部
- `Force Step Into`，强制步入，可以进入到任何方法中，一般用于查看底层源码实现
- `Step Out`，步出，从步入的方法内退出到方法调用处，此时方法已执行完毕，只是还没有完成赋值
- `Drop Frame`，断点回退，回退到上一个方法调用的开始处
- `Run to Cursor`，运行到光标处，可以将光标置于你想要查看的某一行，然后点击这个按钮，代码就会运行到这一行了
- `Evaluate Expression`，计算表达式，用于在 debug 过程中计算某个表达式的值，而不用去打印信息

#### debug 模式功能按钮

对照着下图，依次来看看这些功能按钮的作用是什么：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0605.jpg)

- `Rerun Application`，重新运行程序，相当于 kill and run
- `Modify Run Configuration`，修改程序运行配置
- `Resume Program`，恢复程序，按照断点的设置去运行程序，点击则运行到下一个断点
- `Pause Program`，暂停程序
- `Stop Application`，关闭程序
- `View Breakpoints`，查看当前程序中设置的所有断点
- `Mute Breakpoints`，哑断点，让所有的断点失效（断点会变成灰色），再去点击 Resume Program，则这一次 debug 过程结束

### 变量查看

毋庸置疑，在 debug 模式下查看变量的变化过程是非常高频的需求之一。这里，我把所有在 debug 模式下查看变量的方法总结出来（强烈建议你照着去做一遍看看）。

#### IDEA 直接显示

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0606.jpg)

如图所示，IDEA 会直接在变量的后面显示其当前值，包括了给方法传递的参数，这种方式不需要做任何额外的操作，这是 IDEA 的默认行为。

#### 光标悬停

这是最方便也是使用频率最高的方式之一，当想要知道方法中某个变量的信息时，将光标悬停在变量上即可，如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0607.jpg)

注意到变量信息左边的 “+” 号，点击可以查看对象的详细信息（非常有用，且非常常用），如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0608.jpg)

#### 变量（查看）区中查看变量信息

debug 面板中有两块区域可以用来查看当前方法中的变量信息，其中，变量区会把当前方法中的所有变量都给列出来，可以寻找你想要查看的变量；变量查看区中，可以输入想要查看的变量或者直接从变量区中拖过来，也会显示出变量的详细信息。如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0609.jpg)

debug 模式下的变量查看功能是非常实用的，几乎所有的代码调试过程都离不开查看变量。以上所说的几种方式都是可行的。



### 计算表达式

之前介绍过 debug 按钮一共有8个，最后一个是 “计算表达式”。顾名思义，它的功能就是用来计算指定表达式的值的。这样的话，就不需要手动输出（sout 或者 log 出来）了。变量值的查看或者简单计算比较简单， “计算表达式” 的第一个核心功能，如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0610.jpg)

可以直接在窗口中输入计算方法，并查看其返回值。这对于某一行代码调用了多个方法时会特别有用，可以分别查看每个方法的返回值。

“计算表达式” 第二个核心功能非常强悍，它让我们不用构造参数多次发起请求，这就是设置变量的功能。例如，请求方法传递的 level 参数值是 debug，我可以使用 “计算表达式” 将其修改为 info。如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0611.jpg)

这个功能对于想要查看各种值的情况再好不过了，我在工作中做代码调试时，也是会经常使用这个功能，毕竟，请求接口的参数是多样化的，需要保证尽量多的覆盖不同的请求参数。

## 远程 Debug SpringBoot 工程

在工程上线之前，通常会把工程部署到测试（QA）环境进行功能、性能测试。但是，正是由于运行环境不在本地，让代码调试过程变得麻烦，只能通过不断的往代码中加更多的日志来查看系统的状态和变量的变化过程。

同时，每次日志代码变更，都需要重新编译、打包、部署，这无疑是非常低效的。IDEA 预见了这个问题，提供了远程调试的功能。

### 远程调试的说明与支持

远程调试使开发人员能够直接诊断服务器或其它线上进程上的问题，它提供了跟踪线上运行时错误并确定性能瓶颈和问题根源的方法，能够像在本地调试一样 Debug 远程服务器。

远程调试一定需要协议的支持，这里的协议就是：`jdwp`。

> JDWP 是 Java Debug Wire Protocol 的缩写，它定义了调试器（debugger）和目标虚拟机（target vm）之间的通信协议。
>
> Target vm 中运行着我们要调试的 Java 程序，它与一般运行的 JVM 没有什么区别，只是在启动时加载了 JDWP Agent 从而具备了调试功能。而 debugger 就是我们本地的调试器，它向运行中的 target vm 发送指令来获取 target vm 运行时的状态和控制远程 Java 程序的执行。Debugger 和 target vm 分别在各自的进程中运行，他们之间通过 JDWP 通信协议进行通信。

### 使用 IDEA 实现远程调试

不失一般性，还以 SpringBoot 工程为例，看一看 IDEA 是如何支持远程调试的（注意， 虽然Mac，Windows 或 Linux 的操作路径可能会有不一致的地方，但是功能一定是一样的）。

#### 远程调试的前提条件

远程调试的配置其实很简单，但是，为了使远程调试生效，需要保证这样几点：

- 本地机器与部署机器（想要远程调试的机器）之间的网络需要是互通的，这是最基本的条件
- 保证本地 debug 的代码与远程部署的代码完全一致，不能发生任何的修改，否则，断点将会无法命中

#### 添加远程调试（Remote JVM Debug）配置

打开 IDEA 的 `Run/Debug Configurations` 面板（启动按钮的旁边），如下图所示：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0612.jpg)

点击左上角的 “+” 号，新增加一个配置，并从配置模板中选择 `Remote JVM Debug`，确定之后弹出如下所示的面板：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0613.jpg)

默认情况下，模板都已经帮助填好了所需要的信息，唯一要做的就是给当前的远程调试配置起个名字。最后，我们需要注意，这里的配置告诉我们在启动远程程序的时候，需要加上给出的 JVM 参数（自动生成的）：

```bash
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

- `agentlib:jdwp`，通知 JVM 使用 jdwp 来运行调试环境
- `transport=dt_socket`，监听 socket 端口连接方式（也可以使用 dt_shmem 共享内存方式，但限于 windows 机器，并且服务提供端和调试端只能位于同一台机）
- `server=y`，表示当前是调试服务端，=n 表示当前是调试客户端
- `suspend=n`，表示启动时不中断（如果启动时中断，一般用于调试启动不了的问题）
- `address=5005`，表示本地监听 5005 端口，可以修改，但是，直接使用默认值就好

### 远程调试实践

根据之前的配置，如果应用程序名是 `log-stack-02.jar`，想要远程调试它的话，启动的命令就需要修改成（address 端口号可以随意定义，但是也要同步 IDEA 调试配置）：

```bash
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar log-stack-02.jar
```

执行命令，启动工程，我们可以看到控制台（如果你对日志输出做过配置，则以你的配置为准）打印如下所示的日志：

```java
➜ java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar log-stack-02.jar
Listening for transport dt_socket at address: 5005

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.4.RELEASE)

14:16:22.863 [main] INFO  com.imooc.log.stack.Application -  No active profile set, falling back to default profiles: default
14:16:26.695 [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer -  Tomcat initialized with port(s): 8000 (http)
14:16:26.753 [main] INFO  o.a.coyote.http11.Http11NioProtocol -  Initializing ProtocolHandler ["http-nio-8000"]
14:16:26.792 [main] INFO  o.a.catalina.core.StandardService -  Starting service [Tomcat]
14:16:26.793 [main] INFO  o.a.catalina.core.StandardEngine -  Starting Servlet engine: [Apache Tomcat/9.0.17]
14:16:27.083 [main] INFO  o.a.c.c.C.[.[localhost].[/api] -  Initializing Spring embedded WebApplicationContext
14:16:27.084 [main] INFO  o.s.web.context.ContextLoader -  Root WebApplicationContext: initialization completed in 4076 ms
14:16:28.654 [main] INFO  o.s.s.c.ThreadPoolTaskExecutor -  Initializing ExecutorService 'applicationTaskExecutor'
14:16:29.446 [main] INFO  o.s.b.a.e.web.EndpointLinksResolver -  Exposing 15 endpoint(s) beneath base path '/actuator'
14:16:30.004 [main] INFO  o.a.coyote.http11.Http11NioProtocol -  Starting ProtocolHandler ["http-nio-8000"]
14:16:30.334 [main] INFO  o.s.b.w.e.tomcat.TomcatWebServer -  Tomcat started on port(s): 8000 (http) with context path '/api'
14:16:30.346 [main] INFO  com.imooc.log.stack.Application -  Started Application in 8.562 seconds (JVM running for 10.039)
```

可以看到，工程像原来一样正常启动（服务器端口号仍然是 8000），只是第一句打印了：调试端口在 5005 上监听。接着，我们就可以在本地对远程的工程进行调试了，如下图所示，点击 “绿色的小虫” 按钮：

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0615.jpg)

如果顺利的话，在 debug tab 面板上会显示如下内容，代表着连接远程工程成功。

```bash
Connected to the target VM, address: 'localhost:5005', transport: 'socket'
```

那么，在本地设置好了断点之后，发请求到远端的程序就可以进行本地调试了，非常的方便。最后，有两点还需要说明：

- 如果防火墙禁止了远程访问，还需要修改防火墙的配置（远程调试时关闭，调试之后再打开就可以了）
- 一定不要在线上（生产）环境进行远程调试，一定记住，远程调试只适合测试环境.