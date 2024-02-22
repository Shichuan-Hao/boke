---
title: java 线程池的生命周期
author: Shaun
date: 2022-01-22 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

> 基于 Java 8 版本
{: .prompt-tip }

> 线程池源码指的是 ThreadPoolExecutor 类
{: .prompt-tip }

## 问题

1. 线程池的状态有哪些 ？

2. 各种状态下对于任务队列中的任务有何影响 ？

## 源码

在整理线程池的体系结构是，看到一些方法，比如 shutDown/shutDownNow()，它们都是与线程池生命周期相关联的。

线程池 ThreadPoolExecutor 中定义的生命周期中的状态以及相关方法：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;     // =29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;   // =000 11111...

// runState is stored in the high-order bits
private static final int RUNNING    = -1 << COUNT_BITS; // =111 00000...
private static final int SHUTDOWN   =  0 << COUNT_BITS; // =000 00000...
private static final int STOP       =  1 << COUNT_BITS; // =001 00000...
private static final int TIDYING    =  2 << COUNT_BITS; // =010 00000...
private static final int TERMINATED =  3 << COUNT_BITS; // =011 00000...

// Packing and unpacking ctl

// 线程池的状态
private static int runStateOf(int c)     { return c & ~CAPACITY; }
// 线程池中工作线程的数量
private static int workerCountOf(int c)  { return c & CAPACITY; }
// 计算 ctl 的值，等于运行状态“加上”线程数量
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

从上面这段代码，可以得出：

1. 线程池的状态和工作线程的数量共同保存在控制变量 ctl 中，类似于 AQS 中的 state 变量，不过这里直接使用的 AtomicInteger，这里换成 unsafe + volatile 也是可以的。

2. ctl 的高三位保存运行状态，低 29 位保存工作线程的数量，也就是说线程的数量最多只能有 $2^{29} - 1$ 个，也就是上面的 CAPACITY。

3. 线程池一共有五种状态，分别是：
    - **RUNNING**，表示可接受新任务，且可执行队列中的任务。

    - **SHUTDOWN**：表示不可接受新任务，但可执行队列中的任务。

    - **STOP**：表示不接受新任务，且不再执行队列中的任务，且中断正在执行的任务。

    - **TIDYING**：所有任务已经中止，且工作线程数量为 0，最后变迁到这个状态的线程将要执行 `termainated()` 钩子方法，只会有一个线程执行这个方法。

    - **TERMINATED**：中止状态，已经执行完 `terminated()` 方法。

## 流程图

这些状态之间的流转过程如下图：

<!-- ![]() -->

1. 新建线程池的时候，它的初始状态为 **RUNNING**，这个在上面定义 ctl 的时候可以看到。

2. **RUNNING -> SHUTDOWN**，执行 `shutdown()` 方法时。

3. **RUNNING -> STOP**，执行 `shutdonwNow()` 方法时。

4. **SHUTDOWN -> STOP**，执行 `shutdonwNow()` 方法时。

5. **STOP -> TIDYING**，执行了 `shutdown()` 或 `shutdonwNow()` 后，所有任务已中止，且工作线程数量为 0 时，此时会执行 terminated() 方法。

6. **TDYING -> TERMINATE**，执行完 `terminated()` 方法后。


## 源码分析

### RUNNING

RUNNING，比较简单，创建线程池的时候就会初始化 ctl，而 ctl 初始化为 RUNNING 状态，所以线程池的初始状态就为 RUNNING。

```java
//  初始化状态为 RUNNING
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

### SHUTDOWN

执行 `shutdown()` 方法时会将状态修改为 SHUTDOWN，这里肯定会成功，因为 `advanceRunState()` 方法中是个自旋，不成功不会退出。

```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        // 修改状态为 SHUTDOWN
        advanceRunState(SHUTDOWN);
        // 标记空闲线程为中断状态
        interruptIdleWorkers();
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
}


private void advanceRunState(int targetState) {
    for (;;) {
        int c = ctl.get();
        // 如果状态大于 SHUTDOWN(0)，或者修改为 SHUTDOWN 成功了，才会 break 跳出自旋
        if (runStateAtLeast(c, targetState) ||
            ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))))
            break;
    }
}

```

### STOP

执行 `shutdownNow()` 方法时，会将线程池状态修改为 STOP 状态，同时标记所以线程为中断状态。

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        // 修改为 STOP 状态
        advanceRunState(STOP);
        // 标记所以线程为中断状态
        interruptWorkers();
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}
```

至于线程是否响应中断其实是在队列的 `take()` 或 `poll()` 方法中响应的，最后会到 AQS 中，它们检测到线程中断了会抛出一个 InterruptedException 异常，然后 `getTask()` 中捕获这个异常，并且在下一次的自旋时退出当前线程并减少工作线程的数量。

```java
private Runnable getTask() {
    
    boolean timedOut = false; // Did the last poll() time out?
    
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        
        // Check if queue empty only if necessary.
        // 如果状态为 STOP 了，这里会直接退出循环，且减少工作线程数量
        // 退出循环了也就相当于这个线程的生命周期结束了
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        
        int wc = workerCountOf(c);
        
        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {\
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 真正响应中断是在 poll() 方法或者 take() 方法中
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            // 这里捕获中断异常
            timedOut = false;
        }
    }
}
```


这里有一个问题，就是已经通过 `getTask()` 取出来且返回的任务怎么办 ？

实际上它们会正常执行完毕，可以看 `runWorker()` 这个方法。

### TIDYING

当执行 shutdown() 或 shutdownNow() 之后，如果所有任务已中止，且工作线程数量为 0，就会进入这个状态。

```java
final void tryTerminate() {
    for (;;) {
        int c = ctl.get();
        // 下面几种情况不会执行后续代码
        // 1. 运行中
        // 2. 状态的值比 TIDYING 还大，也就是 TERMINATED
        // 3. SHUTDOWN 状态且任务队列不为空
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
        // 工作线程数量不为 0，也不会执行后续代码
        if (workerCountOf(c) != 0) { // Eligible to terminate
            // 尝试中断空闲的线程
            interruptIdleWorkers(ONLY_ONE);
            return;
        }

        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            // CAS 修改状态为 TIDYING 状态
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    // 更新成功，执行 terminated 钩子方法
                    terminated();
                } finally {
                    // 强制更新状态为 TERMINATED，这里不需要 CAS
                    ctl.set(ctlOf(TERMINATED, 0));
                    termination.signalAll();
                }
                return;
            }
        } finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
    }
}
```

实际更新状态为 TIDYING 和 TERMINATED 状态的代码都在 `tryTerminate()` 方法中，并且 `tryTerminate()` 方法在很多地方都有调用，比如 `shutdown()`、`shutdownNow()`、线程退出时，所以说几乎每个线程最后消亡的时候都会调用 `tryTerminate()` 方法，但最后只会有一个线程真正执行到修改状态为 TIDYING 的地方。

修改状态为 TIDYING 后执行 `terminated()` 方法，最后修改状态为 TERMINATED，标志着线程池真正消亡了。

### TERMINATE

与 TIDYING 差不多。