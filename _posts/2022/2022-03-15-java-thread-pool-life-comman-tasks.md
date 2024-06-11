---
title: java 线程池的普通任务执行流程
author: Shaun
date: 2022-01-23 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

> 基于 Java 8 版本
{: .prompt-tip }

> 本文基于 ThreadPoolExector 类
{: .prompt-tip }

## 问题

1. 线程池中的普通任务是怎么执行的 ？

2. 任务又是在哪里被执行的 ?

3. 线程池中有哪些主要的方法 ？

4. 如何使用 Debug 模式一步一步调试线程池 ？

## 使用案例

首先创建一个线程池，它的核心数量 5 ，最大数量为 10，空闲时间为 1 秒，队列长度为 5，拒绝策略打印一句化。

如果使用它运行 20 个任务，会是什么结果呢 ？

```java
public class ThreadPoolTest01 {
    public static void main(String[] args) {
        // 新建一个线程池
        // 核心数量为 5，最大数量为 10，空闲时间为 1 秒，队列长度为 5，拒绝策略打印一句话
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(5, 10, 1,
                TimeUnit.SECONDS, new ArrayBlockingQueue<>(5), Executors.defaultThreadFactory(), new RejectedExecutionHandler() {
            @Override
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                System.out.println(currentThreadName() + "， discard task...");
            }
        });

        // 提交 20 个任务，注意观察 num
        for (int i = 0; i < 20; i++) {
            int num = i;
            threadPool.execute(() -> {
                System.out.println(currentThreadName() + ", " + num + " running, " + System.currentTimeMillis());
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            });
        }

    }

    private static String currentThreadName() {
        return Thread.currentThread().getName();
    }
}
```

<!-- 构造方法的 7 个参数可以参考[这里](./2022-01-28-java-thread-pool-construction-method.md)。 -->

运行结果如下：
```
pool-1-thread-3, 2 running, 1708586297268
pool-1-thread-4, 3 running, 1708586297268
main， discard task...
pool-1-thread-1, 0 running, 1708586297268
pool-1-thread-2, 1 running, 1708586297268
pool-1-thread-5, 4 running, 1708586297268
main， discard task...
pool-1-thread-7, 11 running, 1708586297268
pool-1-thread-8, 12 running, 1708586297268
pool-1-thread-6, 10 running, 1708586297268
pool-1-thread-9, 13 running, 1708586297268
main， discard task...
pool-1-thread-10, 14 running, 1708586297268
main， discard task...
main， discard task...
pool-1-thread-2, 5 running, 1708586299273
pool-1-thread-6, 7 running, 1708586299273
pool-1-thread-5, 6 running, 1708586299273
pool-1-thread-9, 8 running, 1708586299274
pool-1-thread-3, 9 running, 1708586299275
```

注意观察 num 的值的打印信息，不是按顺序打印的，可以一步一步 debug 看看：

## execute() 方法

`execute()` 方法 是线程池提交任务的方法之一，也是最核心的方法。

```java
// 提交任务，任务并非立即执行，所以翻译成执行任务似乎不太合适
public void execute(Runnable command) {
    // 任务不能为空
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    // 控制变量（高 3 位存储状态，低 29 位存储工作线程的数量）
    int c = ctl.get();
    // 1. 如果工作线程数量小于核心数量
    if (workerCountOf(c) < corePoolSize) {
        // 就添加一个工作线程（核心）
        if (addWorker(command, true))
            return;
        // 重新获取下控制变量
        c = ctl.get();
    }
    // 2. 如果达到了核心数量且线程池是运行状态，任务入队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 再次检查线程池状态，如果不是运行状态，就移除任务并执行拒绝策略
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 容错检查工作线程数量是否为 0，如果为 0 就创建一个
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 3. 任务入队列失败，尝试创建非核心工作线程
    else if (!addWorker(command, false))
        // 非核心工作线程创建失败，执行拒绝策略
        reject(command);
}
```


提交任务的过程大致是：

1. 工作线程数量小于核心数量，创建核心线程。

2. 达到核心数量，进入任务队列。

3. 任务队列满了，创建非核心线程。

4. 达到最大数量，执行拒绝策略。

其实，就是三道坎 —— 核心数量、任务队列、最大数量！！！这样就比较好记了。

流程图大致如下：

<!-- ![]() -->

## addWorker() 方法

这个方法主要用来创建一个工作线程，并启动值，其中会做线程池状态、工作线程数量等各种检测。

```java
private boolean addWorker(Runnable firstTask, boolean core) {
    // 判断有没有资格创建新的工作线程
    // 主要是一些状态/数量的检查等待
    // 这段代码比较复杂，可以先跳过
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
        
        // Check if queue empty only if necessary（仅在必要时检查队列是否为空）.
        // 线程池状态检查
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
                firstTask == null &&
                    ! workQueue.isEmpty()))
            return false;

        // 工作线程数量检查
        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 数量加 1 并跳出循环
            if (compareAndIncrementWorkerCount(c))
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    // 如果上面的条件满足，就会把工作线程数量加 1，然后执行下面线程的动作
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 创建工作线程
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // 再次检查线程池的状态
                int rs = runStateOf(ctl.get());
                
                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    // 添加到工作线程队列
                    workers.add(w);
                    // 还在池子中的线程数量（只能在 mainLock 中使用）
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    // 标记线程添加成功
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 线程添加成功之后启动线程
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 线程启动失败，执行失败方法（线程数量减 1，执行 tryTerminate() 方法等）
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

这里其实还没有到任务执行得地方，上面可以看到线程是包含在 Worker 这个类中，那么，跟踪到这个类中看看。

## Worker 内部类

Worker 内部类可以看作是对工作线程的包装，一般情况下，常说的工作线程就是 Worker，但实际上是指**维护的 Thread 实例**。

```java
private final class Worker
    extends AbstractQueuedSynchronizer
    implements Runnable {
    // 真正工作的线程
    final Thread thread;
    // 第一个任务，从构造方法传进来
    Runnable firstTask;
    /** 完成任务数 */
    volatile long completedTasks;

    /**
     * 构造方法
     */
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        // 使用线程工厂生成一个线程
        // 注意，这里把 Worker 本身作为 Runnable 传给线程 
        this.thread = getThreadFactory().newThread(this);
    }

    // 实现 Runnable 的 run() 方法
    public void run() {
        // 调用 ThreadPoolExecutor 的 runWorkler() 方法
        runWorker(this);
    }

    // 省略锁的部分

}
```

这里能够看出来工作线程 Thread 启动的时候实际上是调用的 Worker 的 `run()` 方法，进而调用的是 ThreadPoolExecutor 的 `runWorker()` 方法。

## runWorker() 方法

`runWorker()` 方法是真正执行任务的地方。

```java
final void runWorker(Worker w) {
    // 工作线程
    Thread wt = Thread.currentThread();
    // 任务
    Runnable task = w.firstTask;
    w.firstTask = null;
    // 强制释放锁（shutdown() 里面有加锁）
    // 这里相当于无视那边的中断标记
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 取任务，如果有第一个任务，这里先执行第一个任务
        // 只要能取到任务，这就是个死循环
        // 正常来说，getTask() 返回的任务是不可能为空，因为签名的 execute() 方法是有空判断的
        // 那么，getTask() 什么时候才会返回空任务呢 ？
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // 检查线程池的状态
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 钩子方法，便于子类在任务执行前做一些处理
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 真正任务执行的地方
                    task.run();
                    // 异常处理
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 钩子方法，便于子类在任务执行后做一些处理
                    afterExecute(task, thrown);
                }
            } finally {
                // task 置为空，重新从队列中取
                task = null;
                // 完成任务数加 1
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 到这里肯定是上面的 while 循环退出了
        processWorkerExit(w, completedAbruptly);
    }
}
```

这个方法比较简单，忽略状态检测和锁的内容，如果有第一个任务，就先执行之，之后再从任务队列中取任务来执行，获取任务是通过 `getTask()` 来进行的。

## getTask()

从队列中获取任务的方法，里面包含了对线程池状态、空闲时间等的控制。

```java
private Runnable getTask() {
    // 是否超时
    boolean timedOut = false; // Did the last poll() time out?
    
    // 死循环
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 线程池状态是 SHUTDOWN 的时候会把队列中的任务执行完直到队列为空
        // 线程池状态是 STOP 时立即退出
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
        
        // 工作线程数量
        int wc = workerCountOf(c);
        
        // 是否允许超时，有两种情况：
        // 1. 是允许核心线程数超时，这种就是说所有的线程都可能超时
        // 2. 是工作线程数大于核心数量，这种肯定是允许超时的
        // 注意：非核心线程是一定允许超时的，这里的超时其实是指取任务超时
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
        
        // 超时判断（还包含一些容错判断）
        if ((wc > maximumPoolSize || (timed && timedOut))
             && (wc > 1 || workQueue.isEmpty())) {
            // 超时了，减少工作线程数量，并返回 null
            if (compareAndDecrementWorkerCount(c))
                return null;
            // 减少工作线程数量
            continue;
        }

        try {
            // 真正取任务的地方
            // 默认情况下，只有当工作线程数量大于核心线程数量时，才会调用 poll() 方法触发超时调用
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            // 取到任务了就正常返回
            if (r != null)
                return r;
            // 没有取到任务表明超时了，回到 continue 那个 if 中返回 null
            timedOut = true;
        } catch (InterruptedException retry) {
            // 捕获到了中断异常
            // 中断标记是在调用 shutDown() 或者 shutDownNow() 的时候设置进去的
            // 此时，会回到 for 循环的第一个 if 处判断状态是否要返回 null
            timedOut = false;
        }
    }
}
```

注意，这里取任务会根据工作线程的数量判断是使用 BlockingQueue 的 `poll(timeout, unit)` 方法还是  `take()` 方法。

`poll(timeout, unit)` 方法会在超时时返回 null，如果 timeout <= 0，队列为空时直接返回 null。

`take()` 方法会一直阻塞直到取到任务或抛出中断异常。

所以，如果 keepAliveTime 设置为 0，当任务队列为空时，非核心线程取不出来任务，会立即结束其生命周期。

默认情况下，是不允许核心线程超时的，但是可以通过下面这个方法设置使核心线程也可超时。

```java
public void allowCoreThreadTimeOut(boolean value) {
    if (value && keepAliveTime <= 0)
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    if (value != allowCoreThreadTimeOut) {
        allowCoreThreadTimeOut = value;
        if (value)
            interruptIdleWorkers();
    }
}
```

至此，线程池中任务的执行流程就结束了。

## 思考

**Q：对于上面案例 num 值的打印信息，为什么不是按顺序打印的 ？**

线程池的参数：核心数量 5 个，最大数量 10 个，任务队列 5 个

答：执行前 5 个任务执行时，正好还不到核心数量，所以新建核心线程并执行了他们。

执行中间 5 个任务时，已经达到核心数量，所以他们先入队列。

执行后面 5 个任务时，已经达到核心数量且队列已经满了，所以新建非核心线程并执行了他们。

在执行最后 5 个任务时，线程池已达到了满负荷状态，所以执行了拒绝策略。


**Q：核心线程和非核心线程有什么区别 ？**

答：实际上并没有什么区别，主要是根据 corePoolSize 来判断任务该去哪里，两者在执行任务的过程中并没有任何区别。有可能新建的时候是核心线程，而 keepAliveTime 时间到了结束了的也可能是刚开始创建的核心线程。


**Q：Worker 继承自 AQS 有何意义？**

Worker 内部类的定义，它继承自AQS，天生自带锁的特性，那么，它的锁是用来干什么的呢？跟任务的执行有关系吗？

答：既然是跟锁（同步）有关，说明 Worker 类跨线程使用了，此时我们查看它的 `lock()` 方法发现只在`runWorker()` 方法中使用了，但是其 `tryLock()` 却是在 `interruptIdleWorkers()` 方法中使用的。
```java
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        for (Worker w : workers) {
            Thread t = w.thread;
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock();
    }
}
```
`interruptIdleWorkers()` 方法的意思是中断空闲线程的意思，它只会中断 BlockingQueue 的 `poll()` 或 `take()` 方法，而不会中断正在执行的任务。

一般而言，`interruptIdleWorkers()` 方法的调用不是在本工作线程，而是在主线程中调用的，观察在 [线程池生命周期](./2022-03-01-java-thread-pool-life-cycle.md) 中说过的 `shutdown()` 和 `shutdownNow()` 方法：
- `shutdown()` 中也是调用了 `interruptIdleWorkers()` 方法，这里 `tryLock()` 获取到锁了再中断，如果没有获取到锁就不中断。没获取到锁只有一种情况，就是 `lock()` 所在的地方，也就是有任务正在执行。

- `shutdownNow()` 中中断线程就很暴力，并没有 `tryLock()`，而是直接中断了线程，所以调用 `shutdownNow()` 可能会中断正在执行的任务。

<font color=red>所以，Worker 继承自 AQS 实际是要使用其锁的能力，这个锁主要是用来控制 shutdown() 时不要中断正在执行任务的线程 </font>



## 总结

本文通过一个例子并结合线程池的重要方法分析了线程池中普通任务执行的流程。

1. `execute()`：提交任务的方法，根据**核心数量**、**任务队列大小**、**最大数量**、分成四种情况判断任务应该往哪里去：
    1. 工作线程数量小于核心数量，创建核心线程。

    2. 达到核心数量，进入任务队列。

    3. 任务队列满了，创建非核心线程。

    4. 达到最大数量，执行拒绝策略。

2. `addWorker()`：添加工作线程的方法，通过 Worker 内部类封装一个 Thread 实例维护工作线程的执行。

3. `runWorker()`：真正执行任务的地方，先执行第一个任务，再源源不断从任务队列中取任务来执行。

4. `getTask()`：真正从队列取任务的地方，默认情况下，根据工作线程数量与核心数量的关系判断使用队列的 `poll()` 方法还是 `take()` 方法，keepAliveTime 参数也是在这里使用的。