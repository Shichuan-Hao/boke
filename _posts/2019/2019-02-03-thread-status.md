---
title:  线程的六个状态
author:
  name: superhsc
  link: https://github.com/happymaya
date: 2019-02-03 23:33:00 +0800
categories: [Java, 并发]
tags: [java, thread]
math: true
mermaid: true
---

就像生物从出生到长大、最终死亡的过程一样，线程也有自己的生命周期。在 Java 中，线程在运行的生命周期中的指定时刻只可能处于下面 6 种不同状态的其中一个状态：

<table>
   <tr>
      <td colspan="2">状态名称</td>
      <td>说明</td>
   </tr>
   <tr>
      <td colspan="2">New</td>
      <td>初始状态，线程被创建，但是此时还没有调用 start() 方法</td>
   </tr>
   <tr>
      <td colspan="2">Runnable</td>
      <td>运行状态，Java 线程将操作系统种的就绪（Ready）和<br/>运行（Running）两种状态笼统地称作“运行中”</td>
   </tr>
   <tr>
      <td rowspan="3">Block</td>
      <td>Blocked</td>
      <td>阻塞状态，表示线程阻塞于锁</td>
   </tr>
   <tr>
      <td>Waiting</td>
      <td>等待状态，表示线程进入等待状态，进入该状态表示当前线程<br/>需要等待其他线程做出一些特定动作（通知或中断）</td>
   </tr>
   <tr>
      <td>Timed Waiting</td>
      <td>计时（超时）状态，该状态不同于 Waiting，它可以在指定的时间自行返回</td>
   </tr>
   <tr>
      <td colspan="2">Terminated</td>
      <td>终止状态，表示当前线程已经执行完毕</td>
   </tr>
</table>

确定线程当前的状态，可以通过 `getState()` 方法，并且线程在任何时刻只可能处于一种状态。

线程在生命周期中并不是固定处于某一个状态而是随着代码的执行在不同状态之间切换。Java 线程状态变迁如下图所示：
![线程的生命周期](https://images.happymaya.cn/assert/java/thread/java-thread-life-cycle.png)

## New 新建

New 表示线程被创建但尚未启动的状态

![新建状态](https://images.happymaya.cn/assert/java/thread/java-thread-life-cycle-new-status.png)

当用 `new Thread()` 新建一个线程时，如果线程没有开始运行 `start()` 方法，所以也没有开始执行 `run()` 方法里面的代码，那么此时它的状态就是 New。

而一旦线程调用了 `start()`，它的状态就会从 New 变成 Runnable，也就是状态转换成上图中中间大方框里的内容。

## Runnable 运行
![运行状态](https://images.happymaya.cn/assert/java/thread/java-thread-life-cycle-runnable-status.png)

Java 中的 Runable 状态对应**操作系统线程状态**中的两种状态，分别是 **Running** 和 **Ready**，也就是说，Java 中处于 Runnable 状态的线程有可能正在执行，也有可能没有正在执行，正在等待被分配 CPU 资源。

所以，如果一个正在运行的线程是 Runnable 状态，当它运行到任务的一半时，执行该线程的 CPU 被调度去做其他事情，导致该线程暂时不运行，它的状态依然不变，还是 Runnable，因为它有可能随时被调度回来继续执行任务。

> 在操作系统层面，线程有 READY 和 RUNNING 状态；而在 JVM 层面，只能看到 RUNNABLE 状态（图源：HowToDoInJava：Java Thread Life Cycle and Thread States），所以 Java 系统一般将这两个状态统称为 RUNNABLE（运行中） 状态 。
为什么 JVM 没有区分这两种状态呢？ （摘自：java线程运行怎么有第六种状态？ - Dawell的回答 ） 现在的时分（time-sharing）多任务（multi-task）操作系统架构通常都是用所谓的“时间分片（time quantum or time slice）”方式进行抢占式（preemptive）轮转调度（round-robin式）。这个时间分片通常是很小的，一个线程一次最多只能在 CPU 上运行比如 10-20ms 的时间（此时处于 running 状态），也即大概只有 0.01 秒这一量级，时间片用后就要被切换下来放入调度队列的末尾等待再次调度。（也即回到 ready 状态）。线程切换的如此之快，区分这两种状态就没什么意义了。
{: .prompt-tip }

## 阻塞状态

![阻塞状态](https://images.happymaya.cn/assert/java/thread/java-thread-life-cycle-block-status.png)

在 Java 中阻塞状态通常不仅仅是 Blocked，实际上它包括三种状态（如上图的红框部分），分别是 **Blocked(被阻塞）**、**Waiting(等待）**、**Timed Waiting(计时等待）**，这三种状态统称为**阻塞状态**。

### Blocked 被阻塞

最简单的是 Blocked，从箭头的流转方向，可以看出，从 Runnable 状态进入 Blocked 状态只有一种可能：就是进入 synchronized 保护的代码，但没有抢到 monitor 锁，无论是进入 synchronized 代码块，还是 synchronized 方法，都是一样的。

再往右看，当处于 Blocked 的线程抢到 monitor 锁，就会从 Blocked 状态回到Runnable 状态。

### Waiting 等待

线程进入 Waiting 状态有三种可能性。

1. 没有设置 Timeout 参数的 `Object.wait()` 方法；
2. 没有设置 Timeout 参数的 `Thread.join()` 方法；
3. `LockSupport.park()` 方法。

> Blocked 仅仅针对 synchronized monitor 锁，可是在 Java 中还有很多其他的锁，比如 ReentrantLock，如果线程在获取这种锁时没有抢到该锁，就会进入 Waiting 状态，因为本质上它执行了 `LockSupport.park()` 方法，所以会进入 Waiting 状态。同样，`Object.wait()` 和 `Thread.join()` 也会让线程进入 Waiting 状态。
{: .prompt-info }

**Blocked 与 Waiting 的区别是: Blocked 在等待其他线程释放 monitor 锁，而 Waiting 则是在等待某个条件，比如 join 的线程执行完毕，或者是 `notify()/notifyAll()`。**

### Timed Waiting 限期等待

在 Waiting 和 Timed Waiting 状态非常相似，区别仅在于**有没有时间限制**，Timed Waiting 会等待超时，由系统自动唤醒，或者在超时前被唤醒信号唤醒。

以下情况会让线程进入 Timed Waiting 状态。
1. 设置了时间参数的 `Thread.sleep(long millis)/Thread.sleep(long millis)` 方法；
2. 设置了时间参数的 `Object.wait(long timeout)/Object.wait(long timeout)` 方法；
3. 设置了时间参数的 `Thread.join(long millis)/Thread.join(long millis)` 方法；
4. 设置了时间参数的 `LockSupport.parkNanos(long nanos)/LockSupport.parkNanos(long nanos)` 方法和 `LockSupport.parkUntil(long deadline)/LockSupport.parkUntil(long deadline)` 方法。


### 三种状态阻塞状态间流转

#### Blocked -> Next Status

从 Blocked 状态进入 Runnable 状态，要求线程获取 monitor 锁；

#### Waiting -> Next Status
从 Waiting 状态流转到其他状态则比较特殊，由于 Waiting 是不限时的，也就是无论过了多长时间它都不会主动恢复，只有当执行了 `LockSupport.unpark()` 或者 join 的线程运行结束，或者被中断时才可以进入 Runnable 状态。

如果其他线程调用 `notify()` 或 `notifyAll()` 来唤醒它，它会直接进入 Blocked 状态，这是因为唤醒 Waiting 线程的线程如果调用 `notify()` 或 `notifyAll()`，要求必须首先持有该 monitor 锁，所以处于 Waiting 状态的线程被唤醒时拿不到该锁，就会进入 Blocked 状态，直到执行了 `notify()/notifyAll()` 的唤醒它的线程执行完毕并释放 monitor 锁，才可能轮到它去抢夺这把锁，如果它能抢到，就会从 Blocked 状态回到 Runnable 状态。

#### Timed Waiting -> Next Status
同样在 Timed Waiting 中执行 `notify()` 和 `notifyAll()` 也是一样的道理，它们会先进入 Blocked 状态，然后抢夺锁成功后，再回到 Runnable 状态。

当然对于 Timed Waiting 而言，如果它的**超时时间到了且能直接获取到锁/join的线程运行结束/被中断/调用了 `LockSupport.unpark()`**，会直接恢复到 Runnable 状态，而无需经历 Blocked 状态。

## Terminated 终止

![终止状态](https://images.happymaya.cn/assert/java/thread/java-thread-life-cycle-terminated-status.png)

最后一种状态是 Terminated 终止状态，要想进入这个状态有两种可能：
- `run()` 方法执行完毕，线程正常退出。
- 出现一个没有捕获的异常，终止了 `run()` 方法，最终导致意外终止。

## 注意点

线程转换的两个注意点：
1. 线程的状态是需要按照箭头方向来走的，比如线程从 New 状态是不可以直接进入 Blocked 状态的，它需要先经历 Runnable 状态。
2. 线程生命周期不可逆：一旦进入 Runnable 状态就不能回到 New 状态；一旦被终止就不可能再有任何状态的变化。所以一个线程只能有一次 New 和 Terminated 状态，只有处于中间状态才可以相互转换。
