---
title: java 线程池的构造方法
author: Shaun
date: 2022-01-21 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

> 基于 Java 8 版本
{: .prompt-tip }

## 问题

1. ThreadPoolExecutor 有几个构造方法 ？

2. ThreadPoolExecutor 最长的构造方法有几个参数 ？

3. keepAlivTime 是做什么用的 ？

4. 核心线程会不会超时关闭 ？能不能超时关闭 ？

5. ConcurrentLinkedQueue 能不能作为任务队列的参数 ？

6. 默认的线程是怎么创建的 ？

7. 如何实现自己的线程工厂 ？

8. 拒绝策略有哪些 ？

9. 默认的拒绝策略是什么 ？

## 构造方法

直接上代码：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         threadFactory, defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          RejectedExecutionHandler handler) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
         Executors.defaultThreadFactory(), handler);
}

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```
ThreadPoolExecutor 有四个构造方法，其中前三个最终都是调用最后一个，它有 7 个参数，分别是：

- `corePoolSize`

- `maximunPoolSize`

- `keepAliveTime`

- `unit`

- `workQueue`

- `threadFacotry`

- `handler`


### corePoolSize

核心线程数

当正在运行的线程数小于核心线程数时，来一个任务就创建一个核心线程。

当正在运行的线程数大于或等于核心线程数时，任务来了先不创建线程而是丢到任务队列中（排队）。

### maximunPoolSize

最大线程数

当任务队列满了，来一个任务就创建一个非核心线程，但是不能超过最大线程数。

### keepAliveTime + unit

线程保持空闲时间和单位。

默认情况下，这两个参数仅当在正在运行的线程数大于核心线程数时才有效，即只针对非核心线程。

但是，如果 allowCoreThreadTimeOut 被设置成了 true，针对核心线程也有效。

即当任务队列为空时，线程保持多久才会被销毁，内部主要是通过阻塞队列带超时的 `poll(timeout, unit)` 方法实现的。

### workQueue

任务队列。

当正在运行的线程数大于或等于核心线程数时，任务来了是先进入任务队列中的。

这个队列必须是阻塞队列，所以像 ConcurrentLinkedQueue 就不能作为参数，因为它虽然是并发安全的队列，但它不是阻塞队列。

```java
public class ConcurrentLinkedQueue<E> 
    extends AbstractQueue<E>
    implements Queue<E>, java.io.Serializable {
    // 略...
}
```

### threadFactory

线程工厂。

默认使用的是 Executors 工具类中的 DefaultThreadFacotry 类，这个类有个缺点，创建的线程名称是自动生成的，无法自定义以区分不同的线程池，且它们都是非守护线程。

```java
private static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

自定义一个线程工程，只需要实现 ThreadFactory，然后将名称和是否是守护线程当作构造方法的参数传进来就可以了，具体这方面可以参考 netty 中的默认工厂 `io.netty.util.concurrent.DefaultThreadFactory`或 google 中的线程工厂 `com.google.common.util.concurrent.ThreadFactoryBuilder`。

### handler

拒绝策略。

拒绝策略表示当任务队列满了并且线程数也达到最大了，这时候再新加任务，线程池已经无法承受了，这些新来的任务应该按什么逻辑来处理。

常用的拒绝策略有：**丢弃当前任务**，**丢弃最老的任务**，**抛出异常**、**调用者自己处理等待**。

默认的拒绝策略是：**抛出异常**，即线程池无法承载了，调用者再往里面添加任务会抛出异常。

虽然默认的拒绝策略简单除暴，但是相对于丢弃任务策略明显要好很多，因为最起码调用者自己可以捕获这个异常再进行二次处理。