---
title: Java 线程堆栈
author:
  name: superhsc
  link: https://github.com/Shichuan-hao
date: 2018-06-07 22:33:00 +0800
categories: [博客]
tags:  [thread, heap, stack, jstack]
math: true
mermaid: true
---

## 使用 jstack 命令获取线程堆栈日志

jstack（stack trace for java）是 Java 虚拟机自带的一种堆栈跟踪工具，它用于打印出给定的 Java 进程、core file、远程调试服务的 Java 堆栈信息。也就是说，安装了 Java，这个工具顺带也就被安装了。

### 认识 jstack

总结下来，jstack 可以完成以下四种功能（其实说白了就是一种：打印当前程序的线程堆栈信息，只是在不同的场景下）：

- 对于正在运行的程序，查看其当前的堆栈信息（或者叫快照）
- 对于 hung（卡住）住的程序，查看其当前的堆栈信息
- 如果 Java 程序崩溃生成 core 文件（也就是 Linux 里面经常说的打 core），jstack可以用来获得 core 文件的堆栈信息，从而可以轻松地知道 Java 程序是如何崩溃和在程序何处发生问题
- 对于远端处于 debug 模式的服务，jstack 也可以用于查看其实时堆栈信息（不过，这个就不常用了）

### jstack 工具的使用

想要知道 jstack 怎么使用，有两种办法：

- `man jstack`（这对于 Linux 或者 Mac 用户来说一定不会陌生）
- 直接在命令行执行 `jstack` 或者 `jstack --help` 也会打印它的使用方法

个人更喜欢直接敲 `jstack` 命令然后回车，毕竟这种方法最简单。当执行了 `jstack` 之后命令行界面会打印什么：

```java
➜  ~ jstack
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```

Usage 部分基本上说明了 jstack 的功能，具体如下：

- pid，进程 id
- executable core，Java 程序的 core dump
- remote server IP or hostname，远程 debug 服务的主机名或 ip
- server_id，唯一id，对于一台主机上运行多个远程 debug 服务

Options 翻译过来是选项或者是命令参数的意思，我们也来看一下：

- -F，当 `jstack pid` 没有响应的时候，强制打印线程堆栈信息，一般情况不需要使用
- -m，打印 Java 和 native C/C++ 的所有堆栈信息，一般应用排查不需要使用
- -l，长列表，除了打印最基本的堆栈信息之外，还会打印关于锁的附加信息，一般应用排查不需要使用
- -h or -help，打印帮助信息

日常情况下，使用 jstack 命令的方式就是 `jstack pid`，即直接加上一个进程 id 就可以了。

Java 程序的进程 id 又该怎样去获取呢？也是有两种方法（仅限于 Linux 和 Mac 用户）：

```bash
# 使用 jps 命令查看
➜  ~ jps
59660 log-stack-02.jar

# 使用 ps 命令查看（Linux 和 Mac 上的用法稍有不同，注意区分自己的 OS）
➜  ~ ps -A |grep log-stack-02.jar
59660 ttys009    0:22.46 /usr/bin/java -jar log-stack-02.jar
```

有了进程 id 以后，就可以使用 jstack 命令去查看堆栈信息了，如下所示：

```bash
➜  ~ jstack 59660
2020-12-30 17:28:08
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.131-b11 mixed mode):

"Attach Listener" #31 daemon prio=9 os_prio=31 tid=0x00007ff40c8ad800 nid=0x5f03 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"DestroyJavaVM" #30 prio=5 os_prio=31 tid=0x00007ff409802800 nid=0x1903 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"http-nio-8000-Acceptor-0" #28 daemon prio=5 os_prio=31 tid=0x00007ff40dd5c000 nid=0x5d03 runnable [0x000070000ae73000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
	- locked <0x000000079bf3d8a8> (a java.lang.Object)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:448)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:70)
	at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:95)
	at java.lang.Thread.run(Thread.java:748)

"http-nio-8000-ClientPoller-1" #27 daemon prio=5 os_prio=31 tid=0x00007ff40a2fc800 nid=0x5c03 runnable [0x000070000ad70000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.KQueueArrayWrapper.kevent0(Native Method)
	at sun.nio.ch.KQueueArrayWrapper.poll(KQueueArrayWrapper.java:198)
	at sun.nio.ch.KQueueSelectorImpl.doSelect(KQueueSelectorImpl.java:117)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
	- locked <0x000000079c0e8758> (a sun.nio.ch.Util$3)
	- locked <0x000000079c0e8748> (a java.util.Collections$UnmodifiableSet)
	- locked <0x000000079c0e8628> (a sun.nio.ch.KQueueSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
	at org.apache.tomcat.util.net.NioEndpoint$Poller.run(NioEndpoint.java:743)
	at java.lang.Thread.run(Thread.java:748)

......
```

jstack 会直接把进程的堆栈信息打印到命令行上，但通常这些信息会非常多，不利于查看。所以，把结果重定向到文件中，例如：

```java
jstack 59660 > ~/log-stack-02.txt
```

如果程序出现卡顿、CPU 占用率过高等问题，首先想到打出当前的线程堆栈看看是怎么回事。

## 堆栈中的线程状态

Java 线程有一个非常重要的属性：线程状态，使用 jstack 也会把线程堆栈的状态打印出来。例如：running、waiting 等等。

### 线程状态

仔细阅读 jstack 线程堆栈日志，会发现，每一个线程都会打印当前所处的状态（java.lang.Thread.State: XXXXXX），如下所示：

```java
"http-nio-8000-Acceptor-0" #28 daemon prio=5 os_prio=31 tid=0x00007ff40dd5c000 nid=0x5d03 runnable [0x000070000ae73000]
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
	at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
	- locked <0x000000079bf3d8a8> (a java.lang.Object)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:448)
	at org.apache.tomcat.util.net.NioEndpoint.serverSocketAccept(NioEndpoint.java:70)
	at org.apache.tomcat.util.net.Acceptor.run(Acceptor.java:95)
	at java.lang.Thread.run(Thread.java:748)
```

所以，想要通过 jstack 命令来分析线程的情况的话，首先要知道线程都有哪些状态。可以打开 jdk 源码 `java.lang.Thread.State`，其中定义了线程的状态

```java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;
}
```

可以看到，JDK 中一共定义了6种线程状态，它们分别是：

- NEW，新创建的线程，尚未执行，这个状态不会出现在堆栈日志中
- RUNNABLE，运行中的线程，正在执行 run() 方法的 Java 代码
- BLOCKED，运行中的线程，因为某些操作被阻塞而挂起
- WATING，运行中的线程，因为某些操作在等待中（无限期等待另一个线程执行特定操作）
- TIMED_WAITING，运行中的线程，因为执行 sleep() 方法正在计时等待
- TERMINATED，线程已终止，run() 方法执行完毕

任何一个 JVM 中的线程，它的状态一定属于这6个其中的一个，且在线程的整个生命周期中，它的状态是不断变化的，即它会从某一个状态切换到另一个状态。

### NEW

```
/**
 * <h2>New</h2>
 * */
public static void newState() {

    System.out.println(new Thread().getState());
}
```

调用 `newState` 方法就会打印 “NEW”，所以，一个刚创建而尚未启动（尚未调用 Thread.start() 方法）的 Java 线程实例就处于 NEW 状态。



### RUNNABLE

RUNNABLE 状态下的线程在 Java 虚拟机中执行，但它可能执行等待操作系统的其他资源，例如处理器。看下面的这段代码，执行它输出的就是 RUNNABLE：

```java
/**
 * <h2>RUNNABLE</h2>
 * */
public static void runnableState() throws Exception {

    Thread thread = new Thread(()-> {
        while (true){
        }
    });

    thread.start();
    System.out.println(thread.getState());
}
```

当 Java 线程实例调用了 Thread.start()之后，就会进入 RUNNABLE 状态。RUNNABLE 状态可以认为包含两个子状态：READY 和 RUNNING：

- READY，该状态的线程可以被线程调度器进行调度使之更变为 RUNNING 状态
- RUNNING，该状态表示线程正在运行，线程对象的 run() 方法中的代码所对应的指令正在被 CPU 执行



### BLOCKED

此状态表示一个线程正在阻塞等待获取一个监视器锁。如果线程处于阻塞状态，该状态下的线程不会被分配 CPU 执行时间。同样，还是来看一个例子：

```java
/**
 * <h2>BLOCKED</h2>
 * */
public static void blockedState() throws Exception {

    Thread thread1 = new Thread(()-> {
        synchronized (MONITOR) {
            try {
                Thread.sleep(Integer.MAX_VALUE);
            } catch (InterruptedException e) {
            }
        }
    });

    Thread thread2 = new Thread(()-> {
        synchronized (MONITOR) {}
    });

    thread1.start();
    Thread.sleep(100);

    thread2.start();
    Thread.sleep(100);

    System.out.println(thread2.getState());
}
```

可以看到，线程正在等待一个监视器锁，只有获取监视器锁之后才能进入 synchronized 代码块或者 synchronized 方法，在此等待获取锁的过程的线程都处于阻塞（BLOCKED）状态。



### WAITING

一个线程进入等待状态是由于调用了下面的某个方法：

- 不带超时的 Object.wait()
- 不带超时的 Thread.join()
- LockSupport.park()

一个处于等待状态的线程总是在等待另一个线程进行一些特殊的处理，且是无限期的等待状态，这种状态下的线程不会被分配 CPU 执行时间。当一个线程执行了某个方法之后就会进入无限期等待状态，直到被显式唤醒，被唤醒后，线程状态由 WAITING 变成 RUNNABLE 然后继续执行。来看一段代码：

```java
/**
 * <h2>WAITING</h2>
 * */
public static void waitingState() throws Exception {

    Thread thread = new Thread(()-> {
        LockSupport.park();
        while (true) {
        }
    });

    thread.start();
    Thread.sleep(100);
    System.out.println(thread.getState());  // 这里输出 WAITING

    LockSupport.unpark(thread);
    Thread.sleep(100);
    System.out.println(thread.getState());  // 这里输出 RUNNABLE
}
```



### TIMED_WAITING

这种状态下的线程并不会无限期的等待，相反，等待时间是有限的。某个线程进入该状态是由于调用了下面的某个方法，且指定了超时时限：

- Thread.sleep()
- 带超时的 Object.wait()
- 带超时的Thread.join()
- LockSupport.parkNanos()
- LockSupport.parkUntil()

TIMED_WAITING 和 WAITING 是类似的，这种状态下的线程也不会被分配 CPU 执行时间，不过这种状态下的线程不需要被显式唤醒，只需要等待超时限期到达就会被 JVM 唤醒，有点类似于现实生活中的闹钟。下面，还是以实例进行演示：

```java
/**
 * <h2>TIMED_WAITING</h2>
 * */
public static void timedwaitingState() throws Exception {

    Thread thread = new Thread(()-> {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
        }
    });

    thread.start();
    Thread.sleep(100);
    System.out.println(thread.getState());
}
```



### ERMINATED

TERMINATED 状态表示线程已经终结，一个线程实例只能被启动一次，准确来说，只会调用一次 Thread.run() 方法，方法执行结束之后，线程状态就会更变为 TERMINATED，意味着线程的生命周期已经结束。这个状态下的演示代码就简单多了，如下所示：

```java
/**
 * <h2>TERMINATED</h2>
 * */
public static void terminatedState() throws Exception {

    Thread thread = new Thread(() -> {});
    thread.start();

    Thread.sleep(100);
    System.out.println(thread.getState());
}
```



### 线程状态切换关系

![this is images](https://Shichuan-hao.github.io/images/assert/java/log-stack/0805.jpg)

## 总结

### 什么是线程堆栈，可以用来做什么

线程堆栈也称作线程调用堆栈。Java 线程堆栈是虚拟机中线程（包括锁）状态的一个瞬间快照，即系统在某个时刻所有线程的运行状态，包括每一个线程的调用堆栈，锁的持有情况等信息。线程堆栈包含的信息有：

- 线程的名字、id、线程的数量等等
- 线程的运行状态，锁的状态（锁被哪个线程持有，哪个线程再等待锁等等）
- 调用堆栈（即函数的调用层次关系），调用堆栈包含完整的类名、所执行的方法、源代码的行数等信息

借助线程堆栈，可以分析许多问题，如线程死锁、锁争用、死循环、识别耗时操作等等。在多线程场合下的稳定性问题分析和性能问题分析，线程堆栈分析是最有效的方法，在多数情况下甚至无需对系统了解就可以进行相应的分析。

### 使用 jstack 工具打印线程堆栈

jstack 主要用来查看某个 Java 进程内的线程堆栈信息。语法格式如下：

```
jstack [option] pid
```

虽然 jstack 提供了一些选项（或者说参数），不过，那些只是在特定的场合下才需要，绝大多数情况下，直接使用如下的命令就足够了：

```
# 由于线程堆栈会打印很多内容，一般我们都会重定向到文件中
jstack pid > /stack_log.txt
```

### 线程状态

JDK 中一共定义（java.lang.Thread.State）了6种线程状态，它们标识了当前线程的活动情况，且随着线程的运行，线程的状态是不断切换的。不过，万变不离其宗，最核心的还是要理解 Java 定义的这些状态的含义：

- NEW，新创建的线程，尚未执行，这个状态不会出现在堆栈日志中
- RUNNABLE，运行中的线程，正在执行 run() 方法的 Java 代码
- BLOCKED，运行中的线程，因为某些操作被阻塞而挂起
- WATING，运行中的线程，因为某些操作在等待中（无限期等待另一个线程执行特定操作）
- TIMED_WAITING，运行中的线程，因为执行 sleep() 方法正在计时等待
- TERMINATED，线程已终止，run() 方法执行完毕



### 使用线程堆栈排查、解决问题

借助线程堆栈，可以排查、分析这样的问题（其实也就是线程堆栈最善于解决的问题）：

- 系统无缘无故 CPU 过高
- 系统挂起，无响应
- 系统运行越来越慢，性能瓶颈
- 线程死锁、死循环，饥饿等等

#### 死锁问题

死锁是多线程特有的问题：线程间相互等待资源，而又不释放自身的资源，导致无穷无尽的等待，其结果是系统任务永远无法执行完成。

这样的问题分两步去定位：

- jps/top 命令查找 Java 进程 pid
- jstack pid 把线程堆栈打印出来

下面是一份真实的死锁线程堆栈日志：

```java
"Thread2" #12 prio=5 os_prio=31 tid=0x00007fb87205b800 nid=0xa303 waiting for monitor entry [0x000070000873f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.imooc.log.stack.chapter7.DeadLock.lambda$deadLockExample$1(DeadLock.java:47)
	- waiting to lock <0x000000076ad9dad0> (a java.lang.Object)
	- locked <0x000000076ad9dae0> (a java.lang.Object)
	at com.imooc.log.stack.chapter7.DeadLock$$Lambda$2/81628611.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

"Thread1" #11 prio=5 os_prio=31 tid=0x00007fb87205a800 nid=0xa503 waiting for monitor entry [0x000070000863c000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at com.imooc.log.stack.chapter7.DeadLock.lambda$deadLockExample$0(DeadLock.java:33)
	- waiting to lock <0x000000076ad9dae0> (a java.lang.Object)
	- locked <0x000000076ad9dad0> (a java.lang.Object)
	at com.imooc.log.stack.chapter7.DeadLock$$Lambda$1/1989780873.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)Found one Java-level deadlock:

Found one Java-level deadlock:
=============================
"Thread2":
  waiting to lock monitor 0x00007fb86f034ca8 (object 0x000000076ad9dad0, a java.lang.Object),
  which is held by "Thread1"
"Thread1":
  waiting to lock monitor 0x00007fb86f032418 (object 0x000000076ad9dae0, a java.lang.Object),
  which is held by "Thread2"
===================================================
```

从日志可以看出：两个线程处于 BLOCKED 状态是因为在等相互之间的锁，且结尾部分很明确的告诉了你发现了一个 Java 级别死锁，Thread2 在等待 Thread1 上的锁，Thread1 在等待 Thread2 上的锁。



#### CPU 使用异常问题

当系统中出现了死循环、复杂的科学计算、频繁的 Young、Full GC 时，都会让 CPU 的使用率瞬间飙升，此时，把这些问题都归类为 CPU 使用异常问题。但是，需要注意，对于死循环这类问题来说，是代码出现了 bug 造成的，实际上它是一种错误。

定位这类问题有自己的套路可循（但是要注意，下面给出的套路命令只限于 Linux 系统），四个步骤：

- top 命令查看占用 CPU 的进程 pid
- ps -mp `pid` -o THREAD,tid,time
- jstack `pid` > thread.txt
- printf %x 10086 && echo

此时，再去查询线程堆栈日志，并根据堆栈日志中的类、方法调用堆栈定位并解决问题。

#### 资源不足问题

日常工作、学习中见到的资源不足问题基本上都是资源池分配失败，等待连接造成的。例如：系统中存在着大量的短暂瞬时连接 MySQL、Redis 的情况，那么，很可能会出现资源不足、超时的问题出现。

下面，可以看到一个真实的资源不足线程堆栈：

```bash
"happymaya-hsc-200" #253 prio=5 os_prio=31 tid=0x00007fd68b2e3800 nid=0x29803 waiting on condition [0x0000700013c0b000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x000000076b99a808> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at org.apache.commons.pool2.impl.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:590)
        at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:425)
        at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:346)
        at redis.clients.util.Pool.getResource(Pool.java:49)
        at redis.clients.jedis.JedisPool.getResource(JedisPool.java:226)
        at redis.clients.jedis.JedisPool.getResource(JedisPool.java:16)
        at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.fetchJedisConnector(JedisConnectionFactory.java:271)
        at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.getConnection(JedisConnectionFactory.java:464)
        at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:132)
        at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:95)
        at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:82)
        at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:211)
        at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:184)
        at org.springframework.data.redis.core.RedisTemplate.execute(RedisTemplate.java:171)
        at com.imooc.log.stack.controller.InsufficientResourceController.getValue(InsufficientResourceController.java:64)
        at com.imooc.log.stack.controller.InsufficientResourceController$$Lambda$610/1636065031.get(Unknown Source)
        at java.util.concurrent.CompletableFuture$AsyncSupply.run(CompletableFuture.java:1604)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

很显然，线程在等待 Redis 连接池分配连接资源给它。如果这样的线程堆栈是大量的，那么，这就是资源不足的问题了。解决这类问题有两种策略：

- 资源池设计的不合理，需要根据业务需求重新配置资源池
- 代码逻辑不合理，大量的瞬时短暂连接需要避免



#### 大量的 WAITING 线程

线程处于 WAITING 状态的定义是：一个正在无限期等待另一个线程执行一个特别的动作的线程处于这一状态。`java.lang.Thread.State` doc 中给出了详细的说明，一个线程进入 WAITING 状态是因为调用了以下方法：

- 不带时限的 Object.wait 方法
- 不带时限的 Thread.join 方法
- LockSupport.park

然后会等其它线程执行一个特别的动作，比如：

- 一个调用了某个对象的 Object.wait 方法的线程会等待另一个线程调用此对象的 Object.notify() 或 Object.notifyAll()
- 一个调用了 Thread.join 方法的线程会等待指定的线程结束

一定要避免系统中长时间存在大量的 WAITING 线程，因为这样会对系统产生两类负面影响：

- 浪费虚拟机内存，可能会导致以后的操作、创建对象内存不足
- 频繁的上下文切换，导致 CPU 使用率升高