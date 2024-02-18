---
title: 动手实现线程池
author: Shaun
date: 2022-01-21 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

## 问题

1. 实现线程池需要考虑哪些因素 ？

2. 如何对线程进行测试 ？

3. 如何实现支持带返回值的任务的线程池 ？

3. 如果任务执行过程中抛出异常了应该怎么处理 ？

## 简介

线程池是 Java 并发编程种经常使用到的技术，那线程池是如何实现的呢 ？

## 属性分析

线程池，顾名思义它首先是一个 “池”，这个池里放的是线程，线程是用来执行任务的。

**首先**，线程池中的线程应该是有类别的，有的是**核心线程**，有的是**非核心线程**，所以需要两个变量标识<font color=red>核心线程数量 `coreSize`</font>和<font color=red>最大线程数量 `maxSize`</font>。

> **为什么要区分是否为核心线程呢 ？这是为了控制系统中线程的数量 ！**。<br/>
当线程池中线程数未达到核心线程数 coreSize 时，来一个任务加一个线程是可以的，也可以提高任务执行效率。<br/>
当线程池中线程数达到核心线程数后，需要控制下线程的数量，来任务了先进队列，如果任务执行足够快，这些核心线程很快就能把队列中的任务执行完毕，完全没有新增线程的必要。<br/>
当队列中任务也满了，这时候光靠核心线程数就无法及时处理任务了，所以这时候就需要增加新的线程了，但是线程也不能无限制地增加，所以需要控制其最大线程数量 maxSize。<br/>
{: .prompt-info }

**其次**，我们就需要一个任务队列来存放任务，这个队列必须是线程安全的，我们一般使用 `BlockingQueue` 阻塞队列来充当，当然使用 `ConcurrentLinkedQueue` 也是可以的（注意：`ConcurrentLinkedQueue` 不是阻塞队列，不能运用在 JDK 的线程池中）

**最后**，当任务越来越多而线程处理不及时，迟早会达到一种状态，队列满了，线程数也达到最大线程数了，这是应该怎么办 ？ 这时候就需要走**拒绝策略**了，也就是这些无法及时处理的任务怎么办的一种策略。常用的策略有<font color=red>丢弃当前任务</font>、<font color=red>丢弃最老的任务</font>、<font color=red>调用者自己处理</font> 以及 、<font color=red>抛出异常等</font>

总之，根据上面描述，定义一个线程池一共需要下面四个变量：

- 核心线程数 coreSize。
- 最大线程数 maxSize。
- 阻塞队列 BlockingQueue。
- 拒绝策略 RejectPolicy。

此外，还需要给线程池一个名称，因而再加一个变量：线程池的名称 name。

所以，线程池的属性以及构造方法大概如下：
```java
public class MyThreadPoolExecutor implements Executor {

    /**
     * 线程池的名称
     */
    private String name;
    /**
     * 核心线程数
     */
    private int coreSize;
    /**
     * 最大线程数
     */
    private int maxSize;
    /**
     * 任务队列
     */
    private BlockingQueue<Runnable> taskQueue;
    /**
     * 拒绝策略
     */
    private RejectPolicy rejectPolicy;

    public MyThreadPoolExecutor(
            String name, int coreSize, int maxSize,
            BlockingQueue<Runnable> taskQueue, RejectPolicy rejectPolicy) {
        this.name = name;
        this.coreSize = coreSize;
        this.maxSize = maxSize;
        this.taskQueue = taskQueue;
        this.rejectPolicy = rejectPolicy;
    }
}
```

## 任务流向分析

根据上文的属性分析，基本上已经得到了任务流向的完整逻辑：

**首先，** 如果运行的线程数小于核心线程数，直接创建一个新的核心线程来运行新的任务。

**其次，** 如果运行的线程数达到了核心线程数，就把新任务放入队列。

**然后，** 如果队列满了，就创建新的非核心线程来运行新的任务。

**最后，** 如果非核心线程数也达到最大了，就执行拒绝策略。

![创建线程任务流向分析](https://shichuan-hao.github.io/images/static/create-thread-task-flow.jpg)

大致代理逻辑如下：

```java
@Override
public void execute(Runnable task) {
    // 正在运行的线程数
    int count = runningCount.get();
    
    // 如果正在运行的线程数小于核心线程数，直接加一个线程
    if (count < coreSize) {
        // 注意，这里不一定添加成功，addWorker() 方法里面还要判断一次是不是确实小
        if (addWorker(task, true)) {
            return;
        }
        // 如果添加核心线程失败，就进入下面的逻辑
    }
    
    // 如果达到了核心线程数，就先尝试让任务入队
    // 这里之所以使用 offer()，是因为如果队列满了 offer() 就会立即返回 false
    if (taskQueue.offer(task)) {
        // do nothing，为了逻辑清晰这里留个空 if
    } else {
        // 如果入队失败，说明队列满了，就添加一个非核心线程
        if (!addWorker(task, false)) {
            // 如果添加非核心线程数失败了，就执行拒绝策略
            rejectPolicy.reject(task, this);
        }
    }
}
```

## 创建线程逻辑分析

**首先，** 创建线程的依据是正在运行的线程数量有没有达到核心线程数或者最大线程数，所以还需要一个变量 `runningCount` 用来记录正在运行的线程数。

**其次，** 这个变量 `runningCount` 需要在并发环境下加加减减，所以这里需要使用到 Unsafe 的 CAS 指令来控制其值的修改，用了 CAS 就要给这个变量加上 volatile 修饰，为了方便这里直接使用了 AtomicInteger 来作为这个变量的类型。

**然后，** 因为是并发环境中，所以需要判断 `runningCount < coreSize`(或 `maxSize`) (条件一) 的同时修改 `runningCount` CAS 加一（条件二）成功了才表示可以增加一个线程，如果条件一失败就表示不能再增加线程了直接返回 false，如果条件二失败则表示其他线程先修改了 `runningCount` 的值，就重试。

**最后，** 创建一个线程并运行新任务，并且不断从队列中拿任务来运行。

![创建线程逻辑分析](https://shichuan-hao.github.io/images/static/create-thread-logic.jpg)

代码逻辑如下：
```java
private boolean addWorker(Runnable newTask, boolean core) {
    // 自旋判断是不是真的可以创建一个线程
    for (; ; ) {
        // 正在运行的线程数
        int count = runningCount.get();
        // 是核心线程还是非核心线程
        int max = core ? coreSize : maxSize;
        // 不满足创建线程的条件，直接返回 false
        if (count >= max) {
            return false;
        }
        // 修改 runningCount 成功，可以创建线程
        if (runningCount.compareAndSet(count, count + 1)) {
            // 线程的名字
            String threadName = (core ? "core_" : "") + name + sequence.incrementAndGet();
            // 创建线程并启动
            new Thread(() -> {
                System.out.println("thread name: " + Thread.currentThread().getName());
                // 运行的任务
                Runnable task = newTask;
                // 不断从任务队列中取任务执行，如果取出来的任务为 null，就跳出循环，线程也就结束了
                if (task != null || (task = getTask()) != null) {
                    try {
                        // 执行任务
                        task.run();
                    } finally {
                        // 任务执行完成，就置为空
                        task = null;
                    }
                }
            }, threadName).start();
            break;
        }
    }
    return true;
}
```

## 取任务逻辑分析

从队列中取任务应该使用 take() 方法，这个方法会一直阻塞直到任务或者中断，如果中断了就返回 null，这样当前线程也就可以安静地结束了，另外还要注意中断了记得把 `runningCount` 减一。

```java
private Runnable getTask() {
    try {
        // task() 方法会一直阻塞，直到取到任务为止
        return taskQueue.take();
    } catch (InterruptedException e) {
        // 线程中断了，返回 null，可以结束当前线程
        // 当前线程都要结束了，理应要把 runningCount 的数量减一
        runningCount.decrementAndGet();
        return null;
    }
}
```

好了，目前自己实现的线程池就写完了，测试如下。

## 测试逻辑分析

写的线程池的构造方法如下：
```java
public class MyThreadPoolExecutorTest {
    public static void main(String[] args) {
        MyThreadPoolExecutor threadPool = new MyThreadPoolExecutor("test",
                5,
                10,
                new ArrayBlockingQueue<>(15),
                new DiscardRejectPolicy());
        AtomicInteger num = new AtomicInteger(0);

        for (int i = 0; i < 100; i++) {
            threadPool.execute(() -> {
                try {
                    Thread.sleep(1000);
                    System.out.println("running: " + System.currentTimeMillis() + ": " + num.getAndIncrement());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
```

- name：这个随便传

- coreSize：假设 5

- maxSize：假设 10

- taskQueue：任务队列，这里设置的是有边界的，即用的最简单的 ArrayBlockingQueue，容量设置为 15，即最多可以存储 15 条任务。

- rejectPolicy，拒绝策略，假设使用丢弃当前任务的策略，如下：

```java
public class DiscardRejectPolicy implements RejectPolicy {
    @Override
    public void reject(Runnable task, MyThreadPoolExecutor myThreadPoolExecutor) {
        // do nothing
        System.out.println("discard ont task...");
    }
}
```

这样一个线程池就创建完成了，下面就是执行任务，这里假设通过 for 循环连续不断地添加 100 个任务：

```java
public MyThreadPoolExecutor(
    String name, int coreSize, int maxSize,
    BlockingQueue<Runnable> taskQueue, RejectPolicy rejectPolicy) {
        this.name = name;
        this.coreSize = coreSize;
        this.maxSize = maxSize;
        this.taskQueue = taskQueue;
        this.rejectPolicy = rejectPolicy;
}
```

这段测试程序分析如下：

1. 先连续创建了 5 个核心线程，并执行了新任务。

2. 后面的 15 个任务进了队列。

3. 队列满了，又连续创建了 5 个线程，并执行了新任务。

4. 后面的任务就没得执行了，全部走了丢弃策略。

5. 所以真正执行成功的任务应该是 5 + 15 + 5 = 25 条任务。

运行如下：

```
thread name: core_test1
thread name: core_test2
thread name: core_test3
discard ont task...
thread name: core_test4
discard ont task...
thread name: core_test5
discard ont task...
thread name: test6

discard ont task...
thread name: test7
discard ont task...
thread name: test8
discard ont task...
thread name: test9
thread name: test10
running: 1708267081549: 2
running: 1708267081549: 5
running: 1708267081549: 4
running: 1708267081549: 0
running: 1708267081549: 1
running: 1708267081549: 3
running: 1708267081565: 6
running: 1708267081565: 9
running: 1708267081565: 8
running: 1708267081565: 7
```

可以看到，创建了 5 个核心线程，5 个非核心线程，成功执行了 25 条任务，完成没问题。

## 小结

1. 动手实现一个线程池包含 4 个因素：**核心线程数**、**最大线程数**、**任务队列**、**拒绝策略**。

2. 创建线程的时候要时刻警惕并发的陷阱。

> ** JDK 自带的线程池还有两个参数：keepAliveTime、unit，它们是干什么的呢 ？**<br/>
答：它们是用来控制如何销毁非核心线程的，当然也可以销毁核心线程。
{: .prompt-info }

虽然上面实现了一个线程池，但是它只**具有执行无返回值任务的能力**，缺点也很明显：

1. 无法处理有返回值的任务。

2. 无法处理任务执行的异常（线程中的异常不会抛出到线程外）。

现在实现的线程池的基础上可不可以实现下面两项能力。

## 有返回值和无返回值的任务到底有何不同 ？

答案明显，就是一个有返回值，一个无返回值，用伪代码来表示就是下面这样：
```java
// 无返回值
threadPool.execute(() -> {
    System.out.println(1);
});

// 有返回值，分两步走
// 1. 提交任务到线程池中
threadPool.execute(() -> {
    System.out.println(1);
    return 1;
});

// 2. 等待任务的结果返回
Object value = result.get()
```

无返回值的任务提交了就完事了，主线程并不 Care 它到底有没有执行完，也不 Care 它是不是抛出异常，主线程 Just 提交线程到线程池，其余什么都不管。

有返回值的任务就不一样了，主线程首先要提交任务到线程池中，它需要使用到任务执行的结果，所以它必须等待任务执行完毕后才能拿到任务执行的结果。

那么，为什么不直接在 `execute` 的时候就等待任务执行完毕呢 ？ 这样的话就跟串行没啥区别了，还不如直接在主线程执行任务呢，还少了线程切换的资源消耗。

之所以要分成两步，是因为主线程并不一定需要立即获取返回值，在需要用到返回值的时候才取 get，这样就可以在提交任务和获取返回值之间干些其他的事情，提高效率。

所以， 提交任务的时候不需要阻塞，get 返回值的时候才可能需要阻塞，如果 get 的时候任务已经执行完毕了，这时候也不需要阻塞，如果 get 的时候任务还未执行完毕，那就要阻塞等待任务执行完毕才能获取到返回值。

## 实现分析

**首先，** 无返回值的任务我们直接使用 Runnable 函数式接口，有返回值的任务有没有现成的接口呢 ？ 还真有，那就是 Callable 接口，它有个返回值。

```java
@FunctiuonalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

**其次，** 提交任务的时候需要有个返回值，它是在将来用来获取任务执行结果的，实际上它也是新任务的一种能力，可以使用它对任务进行包装，使其具有返回值的能力。

```java
public interface Future<T> {
    T get();
}
```

**再次，** 我们还需要给现有的线程池增加一种新的能力，根据单一职责原则，我们定义一个新的接口来承载这种能力。

```java
public interface FutureExecutor extends Executor {
    <T> Future<T> submit(Callable<T> command);
}
```

**然后，** 我们还需要一种新的任务，它既具有旧任务的执行能力（`run()` 方法），又具有新任务的返回值能力（`get()` 方法），所以我们造一个 “将来的任务” 对提交的任务进行包装，使其具有返回值的能力。

```java
public class FutureTask<T> implements Runnable, Future<T {
    /**
    * 真正的任务
    */
    public Callable<T> task;

    public FutureTask(Callable<T> task) {
        this.task = task;
    }

    @Override
    public void run() {
        // 具体实现...
    }
}
```

**最后，** 我们只要对原有的线程池进行扩展，将提交的任务包装成 “将来获取返回值的任务”，还是使用原来的方法去执行，然后返回这个将来的任务即可。

根本开闭原则，原来的代码不做任何修改，扩展新的子类来实现新的能力：

```java
public class MyThreadPoolFutureExecutor
        extends MyThreadPoolExecutor
        implements FutureExecutor, Executor {


    public MyThreadPoolFutureExecutor(
            String name,
            int coreSize,
            int maxSize,
            BlockingQueue<Runnable> taskQueue,
            RejectPolicy rejectPolicy) {
        super(name, coreSize, maxSize, taskQueue, rejectPolicy);
    }

    @Override
    public <T> Future<T> submit(Callable<T> task) {
        // 包装成将来获取返回值的任务
        FutureTask<T> futureTask = new FutureTask<>(task);
        // 还是使用原来的执行能力
        execute(futureTask);
        // 返回将来的任务，只需要返回其 get 返回值的能力即可
        // 所以这里返回的是 Future 而不是 FutureTask 类型
        return futureTask;
    }
}
```

到现在，整体的逻辑就已经比较清晰地实现完了，还剩下最关键的部分，这个 “将来的任务” 的两个能力要如何实现。

## 将来的任务

将来的任务，具有两个能力：

1. 执行真正任务的能力。

2. 将来获取返回值的能力。

```java
public class FutureTask<T>
        implements Runnable, Future<T> {

    /**
     * 真正的任务
     */
    private Callable<T> task;

    public FutureTask(Callable<T> task) {
        this.task = task;
    }

    @Override
    public void run() {
        // 具体实现...
    }

    @Override
    public T get() {
        // 具体实现...
        return null;
    }
}
```

**首先，** 首页明确一件事，任务的执行是线程池中，获取返回值是在主线程中，它们是在两个线程中执行的，而且谁先谁后我们无法确定。

**其次，** 如果 `run()` 在 `get()` 之前执行，我们需要告诉 `get()` 任务已经执行完毕了，所以需要一个状态来通知这个事，还需要一个变量来承载任务执行的返回值。

```java
public static final int NEW = 0;

public static final int FINISHED = 1;

public static final int EXPECTED = 2;

/**
* 任务执行的状态，0:未开始 1:正常完成 2:异常完成
*/
private AtomicInteger state = new AtomicInteger(NEW);

/**
* 任务执行的结果
* 如果执行正在，返回结果为 1
* 如果执行异常，返回结果为 Exception
*/
private Object result;
```

**再次，** 如果 `get()` 在 `run()` 之前执行，就需要阻塞等待 `run()` 执行完毕才能拿到返回值，所以需要保存调用者（主线程），`get()` 的时候 park 阻塞住，`run()` 完成了 unpark 唤醒它来拿返回值。

```java
/**
 * 调用者线程
 * 也可以使用 volatile + Unsafe 实现 CAS 操作
 */
private AtomicReference<Thread> caller = new AtomicReference<>();
```

**然后，** 我们先来看看 run() 方法的逻辑，它其实就是先执行真正的任务，然后修改状态为完成，并保存任务的返回值，如果保存了主线程，还要唤醒它。

```java
@Override
public void run() {
    // 如果状态是 NEW，说明执行过了，直接返回
    if (state.get() != NEW) {
        return;
    }
    try {
        // 执行任务
        T r = task.call();
        // CAS 更新 state 的值为 FINISHED
        // 如果更新成功，就把 r 赋值给 result
        // 如果更新失败，说明 state 的值不为 NEW 了，也就是任务已经执行过了
        if (state.compareAndSet(NEW, FINISHED)) {
            this.result = r;
            // finish() 必须放在 state 里面
            finish();
        }
    } catch (Exception e) {
        // 如果 CAS 更新 state 的值为 EXPECTED 成功，就把 e 赋值给 result
        // 如果 CAS 更新失败，说明 state 的值不为 NEW 了，也就是任务已经执行过了
        if (state.compareAndSet(NEW, EXPECTED)) {
            this.result = e;
            // finish() 必须放在修改 state 里面
            finish();
        }
    }
}

private void finish() {
    // 检查调用者是否为空，如果不为空，唤醒它
    // 调用者在调用 get() 方法的进入阻塞状态
    for (Thread c; (c = caller.get()) != null; ) {
        if (caller.compareAndSet(c, null)) {
            LockSupport.unpark(c);
        }
    }
}
```

**最后，** 再看看 `get()` 方法，如果任务还未执行，就阻塞等待任务的执行；如果任务已经执行完毕了，直接拿返回值即可；但是，还有一种情况，`get()` 方法执行的过程中 `run()` 方法也在执行，所以 `get()` 方法中的每一步都要检查状态的值有没有变化。

```java
@Override
public T get() {
    int s = state.get();
    // 如果任务还未执行完成，判断当前线程是否要进入阻塞状态
    if (s == NEW) {
        // 标识调用者线程是否被标记过
        boolean marked = false;
        for(;;) {
            // 重新获取 state 的值
            s = state.get();
            // 如果 state 大于 NEW 说明完成了，跳出循环
            if (s > NEW) {
                break;
                // 此处必须把 caller 的 CAS 更新和 park() 方法分成两步处理，
                // 不能把 park() 放在 CAS 里面
            } else if (!marked) {
                // 尝试更新调用者线程
                // 试想断点停在此处
                // 此时 state 为 NEW，让 run() 方法执行到底，它不会执行 finish() 中的 unpark() 方法
                // 这时打开断点，这里会更新 caller 成功，但是循环从头再执行一遍发现 state 已经遍了
                // 直接在上面的 if(s>NEW) 处跳出循环了，因为 finish() 修改 state 内部
                marked = caller.compareAndSet(null, Thread.currentThread());
            } else {
                // 调用者线程更新之后 park 当前线程
                // 试想断点停在此处
                // 此时 state 为 NEW，让 run() 方法执行到底，因为上面的 caller 已经设置值了
                // 所以会执行 finish() 方法中的 unpark() 方法
                // 这时再打开断点，这里不会 park() 值
                // 见 unpark() 方法的注释，上面写得很清楚：
                // 如果线程执行力 park() 方法，就执行 unpark() 方法会唤醒那个线程
                // 如果线程先执行 unpark() 方法，线程就下一次执行 park() 方法将不会阻塞
                LockSupport.park();
            }
        }
    }
    if (s == FINISHED) {
        return (T) result;
    }
    throw new RuntimeException((Throwable) result);
}
```

在上面的实现中，如果任务执行的过程抛出异常了，也是通过 result 返回主线程，这样主线程就拿到了这个异常，它就可以做相应的处理了。

嗯嗯，完整的实现到此结束。

## 测试用例

测试代码如下：

```java
public class MyThreadPoolFutureExecutorTest {
    public static void main(String[] args) {
        FutureExecutor threadPool = new MyThreadPoolFutureExecutor("test",
                2,
                4,
                new ArrayBlockingQueue<>(6),
                new DiscardRejectPolicy());

        ArrayList<Future<Integer>> list = new ArrayList<>();

        for (int i = 0; i < 100; i++) {
           int num = i;
            Future<Integer> future = threadPool.submit(() -> {
                Thread.sleep(1000);
                System.out.println("running: " + num);
                return num;
            });
            list.add(future);
        }

        for (Future<Integer> future : list) {
            System.out.println("ran: " + future.get() );
        }
    }
}
```

运行结果如下：

```
thread name: core_test1
thread name: core_test2
thread name: test3
thread name: test4
discard ont task...
...省略被拒绝的任务
running: 0
running: 1
ran: 0
ran: 1
running: 9
running: 8
```

## 小结

1. 有返回值的任务是通过包装成将来的任务来实现，这个任务既具有基本的执行能力，有具有将来获取返回值的能力。

2. 任务执行的异常跟任务正常的返回值是通过同一个返回值返回到主线程的，主线程根据状态判断是异常还是正常值。

3. 我们的实现中运用了单一职责原则、开闭原则等设计原则，对原有代码没有造成任何的入侵。





