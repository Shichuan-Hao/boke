---
title: java 线程池的体系结构
author: Shaun
date: 2022-01-21 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

> 基于 Java 8 版本
{: .prompt-tip }

## 体系结构

Java 的线程池中有 8 个非常重要的接口和类：

1. **Executor**，线程池顶级接口

2. **ExecutorService**，线程池次级接口，对 Executor 做了一些扩展，增加一些功能。

3. **ScheduledExecutorService**，对 ExecutorService 做了一些扩展，增加一些**定时任务**相关的功能。

4. **AbstractExecutorService**，抽象类，使用**模板方法设计模式**实现了一部分方法。

5. **ThreadPoolExecutor**，普通线程池类，也就常说的线程池，包含基本的一些线程池操作相关的方法实现。

6. **ScheduledThreadPoolExecutor**，定时任务线程池类，用来实现定时任务相关的功能。

7. **ForkJoinPool**，新型线程池类，Java 7 中新增的线程池类，基于**工作窃取理论**实现，运用于大任务拆小任务、任务无限多的场景。。

8. **Executors**，线程池工具类，定义了一些**快速实现线程池的方法【谨慎使用】**。

## Executor

**线程池顶级接口**，只定义了一个**执行无返回值任务的方法。**

```java
public interface Executor {
    // 执行无返回值任务
    void execute(Runnable var1);
}
```

## ExecutorService

**线程池次级接口**，对 Executor 做了一些扩展，主要增加了**关闭线程池**、**执行有返回值任务**、**批量执行任务的方法**。

```java
public interface ExecutorService extends Executor {
    
    // 关闭线程池，不再接受新任务，但已经提交的任务会执行完成
    void shutdown();

    // 立即关闭线程池，尝试停止正在运行的任务，未执行的任务将不再执行
    // 被迫停止及未执行的任务将以列表的形式返回
    List<Runnable> shutdownNow();

    // 检查线程池是否已经关闭
    boolean isShutdown();

    // 检查线程池是否已经终止，只有在 shutdown() 或 shutdownNow() 之后调用才有可能为 true
    boolean isTerminated();

    // 在指定时间内线程池达到终止状态了才会返回 true
    boolean awaitTermination(long var1, TimeUnit var3)
        throws InterruptedException;

    // 执行返回值的任务，任务的返回值是 task.call() 的结果
    <T> Future<T> submit(Callable<T> task);

    // 执行有返回值的任务，任务的返回值为这里传入的 result
    // 当然只有当任务执行完成并调用 get() 时才会返回
    <T> Future<T> submit(Runnable task, T result);

    // 执行有返回值的任务，任务的返回值为 null
    // 当然只有当任务执行完成并调用 get() 时才会返回
    Future<?> submit(Runnable task);

    // 批量执行任务，只有当这些任务都完成了，这个方法才会返回
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) 
        throws InterruptedException;

    // 在指定时间内批量执行任务，未执行完成的任务将被取消
    // 这里面的 timeout 是所有任务的总时间，不是单个任务的时间
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, 
                                    long timeout, TimeUnit unit) 
        throws InterruptedException;

    // 返回任意一个已完成任务的执行结果，未执行完成的任务将被取消
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) 
        throws InterruptedException, ExecutionException;

    // 在指定时间内，如果有任务完成，就返回任意一个已完成任务的执行结果，
    // 未执行完成的任务将被取消
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, 
                        long timeout, TimeUnit unit)   
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

## ScheduledExecutorService

对 ExecutorService 做了扩展，增加一些定时任务相关的功能，包含两大类：
1. 执行一次。
2. 重复执行。

```java
public interface ScheduledExecutorService extends ExecutorService {
    
    // 在指定延时后执行一次
    ScheduledFuture<?> schedule(Runnable command, 
                                long delay, TimeUnit unit);

    // 在指定延时后执行一次
    <V> ScheduledFuture<V> schedule(Callable<V> callable, 
                                long delay, TimeUnit unit);

    // 在指定延时后开始执行，并在之后以指定时间间隔重复执行
    // 相当于之后的延时以任务开始计算
    ScheduledFuture<?> scheduleAtFixedRate(Runnable command, 
                                            long initialDelay, 
                                            long period, 
                                            TimeUnit unit);

    // 在指定延时后开始执行，并在之后以指定延时重复执行
    // 相当于之后的延时以任务结束计算
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                long initialDelay, 
                                                long period, 
                                                TimeUnit unit);
}
```

## AbstractExecutorService

抽象类，运用模板方法设计模式实现了一部分方法，主要是**执行有返回值任务**、**批量执行任务**的方法。

```java
public abstract class AbstractExecutorService implements ExecutorService {

    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }

    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        return new FutureTask<T>(callable);
    }

    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Runnable task, T result) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task, result);
        execute(ftask);
        return ftask;
    }

    public <T> Future<T> submit(Callable<T> task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<T> ftask = newTaskFor(task);
        execute(ftask);
        return ftask;
    }

    /**
     * the main mechanics of invokeAny.
     */
    private <T> T doInvokeAny(Collection<? extends Callable<T>> tasks,
                              boolean timed, long nanos)
        throws InterruptedException, ExecutionException, TimeoutException {
        // 略...
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException {
        // 略...
    }

    public <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                           long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException {
        // 略...
    }

    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        // 略...
    }

    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                         long timeout, TimeUnit unit)
        throws InterruptedException {
       // 略...
    }
}
```
可以看到，这里的 `submit()` 方法对传入的任务都包装成了 FutureTask 来进行处理<font color="\#666600">（TODO:先放在这里，之后单独研究...）</font>

## ThreadPoolExecutor

普通线程池类，这是我们通常所说的线程池，包含**最基本的一些线程池操作相关的方法实现**。

线程池的主要逻辑都在这里实现，比如线程的创建、任务的处理、拒绝策略等。<font color="\#666600">（TODO:先放在这里，之后单独研究...）</font>

## ScheduledThreadPoolExecutor

定时任务线程池类，用于实现定时任务相关功能，将任务包装成定时任务，并按照定时策略来执行。<font color="\#666600">（TODO:先放在这里，之后单独研究...）</font>


> **Q：定时任务线程池类使用的是什么队列 ？**<br/>
A: 延时队列。定时任务线程池中没有直接使用并发集合中的 DelayQueue，而是自己又实现了一个 DelayWorkQueue，不过跟 DelayQueue 的实现原理是一样的。<br/>
ps: 延时队列使用的数据结构是**堆**（DelayQueue 中使用的是优先级队列，而优先级队列使用的堆；DelayedWorkQueue 直接使用的堆）。
{: .prompt-info }

## ForkJoinPool

新型线程池类，Java 7 中新增的线程池类，这个线程池与 Go 中的线程模型特别类型，都是基于**工作窃取理论**，特别适合处理归并排序这种先分后合的场景

![]()

## Executors

线程池工具类，定义了一系列快速实现线程池的方法 - newXXX()。

不过，《阿里手册》不建议使用这个类来新建线程池！但是我并不这么认为<font color="\#666600">（TODO:先放在这里，之后单独研究...）</font>

