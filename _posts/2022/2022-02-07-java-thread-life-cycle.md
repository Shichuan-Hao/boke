---
title: java 线程的生命周期
author: Shaun
date: 2022-01-21 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

## 问题

1. 线程的状态有哪些 ？

2. synchronized 锁各阶段线程处于什么状态 ？

3. 重入锁、条件锁各阶段线程处于什么状态 ？

## 源码

关于线程的生命周期，我们可以看下 `java.lang.Thread.State` 这个类，它时线程的内部枚举类，定义了线程的各种状态，并且注释也很清晰。

```java
public enum State {
     /**
      * 新建状态，线程还未开始
      */
     NEW,

     /**
      * 可运行状态，正在运行或者在等待系统资源，比如 CPU
      */
     RUNNABLE,

     /**
      * 阻塞状态，在等待一个监视器锁（也就是常说的 synchronized）
      * 或者在调用 Object.wait() 方法且被 notify() 之后进入 BLOCKED 状态。
      */
     BLOCKED,

     /**
      * 等待状态，在调用了以下方法后进入此状态：
      * 
      * 1. Object#wait() 无超时的方法后且未被 notify() 前，如果被 notify() 了会进入 BLOCKED 状态。
      * 2. Thread.join() 无超时的方法后
      * 3. LockSupport.park() 无超时的方法后
      */
     WAITING,

     /**
      * 超时等待状态，在调用了以下方法后会进入超时等待状态：
      * 1. Thread.sleep() 方法
      * 2. Object.wait(timeout) 方法后且未到超时时间前，如果达到超时了或被 notify() 了会进入 BLOCKED 状态
      * 3. Thread.join(timeout) 方法后
      * 4. LockSupport.parkNanos(nanos) 方法后
      * 5. LockSupport.parkUntil(deadline) 方法后
      */
     TIMED_WAITING,

     /**
      * 终止状态，线程已经执行完毕
      */
     TERMINATED;
 }
```

## 流程图

1. 锁分为两大类：一类是 synchronized 锁，一类是基于 AQS 锁（以重入锁为例），即内部使用了 LockSupport.park() 或 LockSupport.parkNanos() 或 LockSupport.parkUntil 几个方法的锁。

2. 无论是 synchronized 锁还是基于 AQS 的锁，内部都是分成两个队列，一个是同步队列（AQS 的队列），一个是等待队列（Contition 的队列）

## 测试

