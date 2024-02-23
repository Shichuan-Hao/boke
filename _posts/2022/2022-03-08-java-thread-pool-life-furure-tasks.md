---
title: java 线程池的未来任务执行流程
author: Shaun
date: 2022-01-24 11:49:00 +0800
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

1. 线程池中的未来任务是怎么执行的 ？

2. 能学到哪些比较好的设计模式 ？

3. 对未来学习别的框架有什么帮助 ？

## 例子

定义一个线程池，并使用它提交 5 个任务，这 5 个任务分别返回 0, 1, 2, 3, 4，在未来的某一个时刻，在取用它们的返回值，做一个累加操作。

```java
public class ThreadPoolTest02 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 新建一个固定 5 个线程的线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(5);

        ArrayList<Future<Integer>> futureList = new ArrayList<>();
        // 提交 5 个任务，分别返回 0，1，2，3，4
        for (int i = 0; i < 5; i++) {
            int num = 0;

            // 任务执行的结果用 Future 包装
            Future<Integer> future = threadPool.submit(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println("return: " + num);
                // 返回值
                return num;
            });

            // 将 future 添加到 list 中
            futureList.add(future);
        }

        // 任务全部提交完再从 future 中 get 返回值，并做累加
        int sum = 0;
        for (Future<Integer> future : futureList) {
            sum += future.get();
        }
        System.out.println("sum= " + sum);
    }
}
```

这里需要思考两个问题：

**1. 如果这里使用普通任务，要怎么写 ？时间大概是多少 ？**

答：如果使用普通任务，那么就要将累加操作放到任务里面，而且并不是怎么好写（final 的问题），总时间大概是 1 秒多一点。但是，这样有一个缺点，就是累加操作跟任务本身的内容耦合到一起了，后面如果改成累乘，还要修改任务的内容。

**2. 如果这里把 future.get() 放到 for 循环里面，时间大概是多少 ？**
这个问题先不回答，先看源码分析。

## submit() 方法

`submit()` 方法，它是提交有返回值任务的一种方式，内部使用未来任务（Future Task）包装，再交给 `execute()` 去执行，最后返回未来任务本身。

```java
public <T> Future<T> submit(Callable<T> task) {
    // 非空检测
    if (task == null) throw new NullPointerException();
    // 包装成 FutureTask
    RunnableFuture<T> ftask = newTaskFor(task);
    // 交给 execute() 方法去执行
    execute(ftask);
    // 返回 futureTask
    return ftask;
}

protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
    return new FutureTask<T>(callable);
}
```

这里的设计很巧妙，实际上这两个方法都是在 AbstractExecutorService 这个抽象类中完成的，这是**模板方法**的一种运用。

下面是 FutureTask 的继承体系：

![FutureTask 的继承体系](https://shichuan-hao.github.io/images/static/futureTask-inheritance-system.png){: .light .w-75 .shadow .rounded-10 h='368' }


FutureTask 实现了 RunnbaleFuture 接口，而 RunnableFuture 接口组合了 Runnable 接口和 Future 接口的能力，而 Future 接口提供了 get 任务返回值的能力。

** Q: submit() 方法返回的为什么是 Future 接口而不是 RunnbaleFuture 接口或者 FutureTask 类呢 ？**

答：这是因为 submit() 返回的结果，对外部调用者只想暴露其 get() 的能力（Future）接口，而不想暴露其 run() 的能力（Runnable 接口）。

## FutureTask 类的 run() 方法

在[Java 线程池普通任务执行流程](./2022-03-15-java-thread-pool-life-comman-tasks.md)一文中，我们知道了 `execute()` 方法最后调用的是 task 的 `run()` 方法，上面传进入的任务，最后被包装成了 FutureTask，也就是说 `execute()` 方法最后会调用 FutureTask 的 `run()` 方法，所以直接看这个方法了：

```java
public void run() {
    // 状态不为 NEW，或者修改为当前线程来运行这个任务失败，则直接返回
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        // 真正的任务
        Callable<V> c = callable;
        // state 必须为 NEW 时才运行
        if (c != null && state == NEW) {
            // 运行的结果
            V result;
            boolean ran;
            try {
                // 任务执行的地方
                result = c.call();
                // 已执行完毕
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                // 处理异常
                setException(ex);
            }
            if (ran)
                // 处理结果
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        // 置空 runner
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        // 处理中断
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

可以看到代码比较简单，先做状态的检测，再执行任务，最后处理结果或异常。

执行任务这里没有啥问题，可以看看处理结果或异常的代码。

```java
protected void setException(Throwable t) {
    // 将状态从 NEW 设置为 COMPLETING
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        // 返回值置为传进来的异常（outcome 为调用 get() 方法时返回的）
        outcome = t;
        // 最终的状态设置为 EXCEPTIONAL
        UNSAFE.putOrderedInt(this, stateOffset, EXCEPTIONAL); // final state
        // 调用完成方法
        finishCompletion();
    }
}

protected void set(V v) {
    // 将状态从 new 设置为 COMPLETING
    if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
        outcome = v;
        // 返回值置为传进来的异常（outcome 为调用 get() 方法时返回的）
        // 最终的状态设置为 NORMAL
        UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
        // 调用完成方法
        finishCompletion();
    }
}
```

咋一看，这两个方法似乎差不多，不同的是出去的结果不一样并且状态不一样，最后都调用了

```java
private void finishCompletion() {
    // assert state > COMPLETING;
    // 如果队列不为空（这个队列实际上为调用者线程）
    for (WaitNode q; (q = waiters) != null;) {
        // 置空队列
        if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
            for (;;) {
                // 调用者线程
                Thread t = q.thread;
                if (t != null) {
                    // 如果调用者线程不为空，就唤醒它
                    q.thread = null;
                    LockSupport.unpark(t);
                }
                WaitNode next = q.next;
                if (next == null)
                    break;
                q.next = null; // unlink to help gc
                q = next;
            }
            break;
        }
    }
    // 钩子方法，子类重写
    done();
    // 置空任务
    callable = null;        // to reduce footprint
}
```

整个 run() 方法总结下来：

1. FutureTask 有一个状态 state 控制任务的运行过程：

- 正常运行结束 state 从 NEW -> COMPLETING -> NORMAL

- 异常运行结束 state 从 NEW -> COMPLETING -> EXCEPTIONAL

2. FutureTask 保存了运行任务的线程 runner，它是线程池中的某个线程。

3. 调用者线程是保存在 waiters 队列中，它是什么时候设置进去的呢 ？

4. 任务执行完毕，除了设置状态 state 变化之外，还要唤醒调用者线程。

调用者线程是什么时候保存在 FutureTask 中（waiters）的呢 ？可以通过其构造方法：
```java
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```
发现并没有相关信息，再想一下，如果调用者不调用 `get()` 方法，那么这种未来任务就跟普通任务没有什么区别了！确实哈，所以只有调用 `get()` 方法了才有必要保存调用者线程到 FutureTask 中。

## FutureTask 类中的 get() 方法

调用`get()` 方法时，如果任务未执行完毕，会阻塞直到任务结束。

```java
public V get() throws InterruptedException, ExecutionException {
    int s = state;
    // 如果状态小于等于 COMPLETING，就进入队列等待
    if (s <= COMPLETING)
        s = awaitDone(false, 0L);
    // 返回结果（异常）
    return report(s);
}
```

是不是很清楚了，如果任务状态小于等于 COMPLETING，就进入队列等待。

```java
private int awaitDone(boolean timed, long nanos)
    throws InterruptedException {
    // 这里假设不带超时
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    WaitNode q = null;
    boolean queued = false;
    for (;;) {
        // 处理中断
        if (Thread.interrupted()) {
            removeWaiter(q);
            throw new InterruptedException();
        }
        // 4. 如果状态大于 COMPLETING 了，就跳出循环并返回
        // 这里是自旋的出口
        int s = state;
        if (s > COMPLETING) {
            if (q != null)
                q.thread = null;
            return s;
        }
        // 如果状态等于 COMPLETING，说明任务快完成了，就差设置状态到 NORMAL 或 EXCEPTIONAL 和设置结果了
        // 此时就让出 CPU，优先完成任务
        else if (s == COMPLETING) // cannot time out yet
            Thread.yield();
        // 1. 如果队列为空
        else if (q == null)
            // 初始化队列（WaitNode 中记录了调用者线程）
            q = new WaitNode();
        // 2. 未进入队列
        else if (!queued)
            // 尝试入队
            queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                 q.next = waiters, q);
        // 超时处理
        else if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                removeWaiter(q);
                return state;
            }
            LockSupport.parkNanos(this, nanos);
        }
        // 3. 阻塞当前线程（调用者线程）
        else
            LockSupport.park(this);
    }
}
```

这里假设调用 get() 时任务还未执行，也就是其状态为 NEW，按照上面标示的 1、2、3、4 走一遍逻辑：

1. 第一次循环，状态为 NEW，直接到 1 处，初始化队列并把调用者线程封装在 WaitNode 中。

2. 第二次循环，状态为 NEW，队列不为空，到 2 处，让包含调用者线程的 WaitNode 入队。

3. 第三次循环，状态为 NEW，队列不为空，且入队列，到 3 处，阻塞调用者线程

4. 假设过了一会任务执行完毕了，根据 run() 方法的分析最后会 unpark 调用者线程，也就是 3 处会被唤醒。

5. 第四次循环，状态肯定大于 COMPLETING，退出循环并返回。


**Q: 为什么要在 for 循环中控制整个流程呢，把这里的每一步单独拿出来写可以吗 ？**

答：**因为每一次动作都需要重新检查状态 state 有没有变化，如果拿出去写也是可以的，只是代码会变得非常冗长**。这里只分析了 `get()` 时状态为 NEW，其它的状态也可以验证，都是可以保证正确的，甚至两个线程交叉运行（断点的技巧）。

这里返回之后，再看看怎么处理最终的结果的：
```java
private V report(int s) throws ExecutionException {
    Object x = outcome;
    // 任务正常结束
    if (s == NORMAL)
        return (V)x;
    // 被取消了
    if (s >= CANCELLED)
        throw new CancellationException();
    // 执行异常
    throw new ExecutionException((Throwable)x);
}
```

在前面 run 的时候，任务执行异常时是将异常放在 outcome 里面的，这里就用到了。

1. 如果正常执行结束，就返回任务的返回值。

2. 如果异常结束，就包装成 ExecutionException 异常抛出。

通过这种方式，线程中出现的异常也可以返回给调用者线程了，不会像执行普通任务那样调用者是不知道任务执行到底有没有成功的。

## 其它

FutureTask 除了可以获取任务的返回值以外，还能够取消任务的执行。

```java
public boolean cancel(boolean mayInterruptIfRunning) {
    if (!(state == NEW &&
          UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
              mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
        return false;
    try {    // in case call to interrupt throws exception
        if (mayInterruptIfRunning) {
            try {
                Thread t = runner;
                if (t != null)
                    t.interrupt();
            } finally { // final state
                UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
            }
        }
    } finally {
        finishCompletion();
    }
    return true;
}
```

这里取消任务是通过中断执行线程来处理的。

## 回答开篇

**Q：如果这里把 future.get() 放到 for 循环里面，时间大概是多少 ？**

答：大概会是 5 秒多一点，因为每提交一个任务，都要阻塞调用者线程直到任务执行完毕，每个任务执行都是 1s 多，所以总时间就是 5 秒多点。

## 总结

1. 未来任务是通过把普通任务包装成 FutureTask 实现的。

2. 通过 FutureTask 不仅能够获取任务执行的结果，还能感知任务执行的异常，甚至还可以取消任务。

3. AbstractExecutorService 中定义了很多模板方法，这是一种很重要的设计模式。

4. FutureTask 就是典型的异常调用的实现方式，像 Netty、Dubbo 还会见到这种设计思想。

**Q：RPC 框架中异步调用是怎么实现的 ？**

答：RPC 框架常用的调用方式有**同步调用**、**异步调用**，其实它们本质上都是异步调用，且都是用 FutureTask 的方式来实现的。

一般地，通过一个线程（我们叫做远程线程）去调用远程接口，如果是同步调用，就直接让调用者线程阻塞等待远程线程调用的结果，待结果返回了再返回；如果是异步调用，就先返回一个未来可以获取到远程结果的东西 FutureXxx，当然，如果这个 FutureXxx 在远程结果返回之前调用了 `get()` 方法一样会阻塞着调用者线程。