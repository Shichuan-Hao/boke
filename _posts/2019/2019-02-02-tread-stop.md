---
title:  正确停止线程
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2019-02-02 23:33:00 +0800
categories: [Java, 并发]
tags: [java, thread]
math: true
mermaid: true
---

启动一个线程非常简单，只需要两步：
1. 在 run() 方法中定义需要执行的任务
2. 调用 Thread 类的 start() 方法

但是，正确停止线程就没那么容了。

## 原理

通常情况下，不会手动停止一个线程，而是允许线程运行到结束，然后让它自然停止。

但是依然会有许多特殊情况需要提前停止线程，比如：用户突然关闭程序、程序运行出错重启等。

在这种情况下，即将停止的线程在很多业务场景下仍然很有价值。尤其是想写一个健壮性，能够安全应对各种场景的程序时，正确停止线程就显得格外重要。不过 Java 没有提供简单易用，能够直接安全停止线程的能力。

## Java 没有安全停止线程能力的原因

对于 Java 而言，最正确停止线程的方式是使用 `interrupt`。但是 `interrupt` 仅仅起到通知被停止线程的作用。

而对于被停止的线程而言，它拥有完全的自主权，既可以选择立即停止，也可以选择一段时间后停止。

事实上，Java 希望**程序间能够相互通知、相互协作地管理线程**，因为如果不了解对方（线程）正在做的工作，贸然强制停止(线程）可能会造成一些安全的问题，为了避免造成问题就需要给对方一定的时间来整理收尾工作。

比如：有一个线程正在写入一个文件，此时收到终止信号，它就需要根据自身业务判断，是选择立即停止，还是将整个文件写入成功后停止，假如选择立即停止，就可能造成数据不完整，不管是中断命令的发起者，还是接收者都不希望数据出现问题。


## 如何用 interrupt 停止线程

```java
while (!Thread.currentThread().isInterrupted() && more work to do) {
    do more work
}
```

一旦调用某个线程的 `interrupt()` 之后，这个**线程的中断标记**就会被设置成 true，每个线程都有这样的标记位。

当线程执行时，应该定期检查这个标记位，如果标记位被设置成 true，就说明有程序想终止该线程。

上面的代码，可以看到在 while 循环体判断语句中：
1. 首先通过 `Thread.currentThread().isInterrupt() `判断线程是否被中断；
2. 随后检查是否还有工作要做

其中，&& 逻辑表示只有当两个判断条件同时满足的情况下，才会去执行下面的工作。

**通过 interrupt 正确停止线程**的栗子：

```java
public class ThreadStop implements Runnable{
    @Override
    public void run() {
        int count = 0;
        // 在每次循环开始之前，检查是否被中断了
        // 首先判断线程是否被中断，而后判断 count 值是否小于 1000
        while (!Thread.currentThread().isInterrupted() && count < 1000) {
            // 打印 0 ~ 999 的数字，每打印一个数字 count 值加 1
            System.out.println("count = " + count++);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("1. 创建线程..................");
        Thread thread = new Thread(new ThreadStop());
        System.out.println("2. 启动线程..................，" + thread.getName());
        // 启动线程
        thread.start();

        // 休眠 5 毫秒后立刻中断线程，
        // 线程会检测到中断信号，会在还没打印完 1000 个数的时候停下来
        System.out.println("3. 线程休眠 5 毫秒..................");
        Thread.sleep(5);
        System.out.println("4. 线程中断..................，" + thread.getName());
        thread.interrupt();
    }
}
```

## sleep 期间能否感受到中断

有一种特殊情况，改写上面代码，如下：
```java
public class ThreadStop1 implements Runnable {

    @Override
    public void run() {
        int count = 0;
        // 在每次循环开始之前，检查是否被中断了
        // 首先判断线程是否被中断，而后判断 count 值是否小于 1000
        try {
            while (!Thread.currentThread().isInterrupted() && count < 1000) {
                // 打印 0 ~ 999 的数字，每打印一个数字 count 值加 1
                System.out.println("count = " + count++);
                // 在线程执行任务期间有休眠需求
                // 每打印一个数字，就进入一次 sleep
                // 这里将休眠时间设置为 1000 秒钟。
                Thread.sleep(1000000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("1. 创建线程..................");
        Thread thread = new Thread(new ThreadStop1());
        System.out.println("2. 启动线程..................，" + thread.getName());
        // 启动线程
        thread.start();

        // 休眠 5 毫秒后立刻中断线程，
        // 线程会检测到中断信号，会在还没打印完 1000 个数的时候停下来
        System.out.println("3. 线程休眠 5 毫秒..................");
        Thread.sleep(5);
        System.out.println("4. 线程中断..................，" + thread.getName());
        thread.interrupt();
    }
}
```
上面的代码会发生主线程休眠 5 毫秒后，通知子线程中断，此时子线程仍在执行 sleep 语句，处于休眠中。

此时需要考虑：**在休眠中的线程是否能够感受到中断通知呢？是否需要等到休眠结束后才能中断线程呢？**如果是这样，会带来严重的问题，因为响应中断太不及时了。正因此，Java 设计者在设计之初就考虑到了这一点。

如果 `sleep()`、`wait()` 等可以让线程进入阻塞的方法使线程休眠，而处于休眠中的线程被中断，线程是可以感受到中断信号的，并且会抛出一个 `InterruptedException` 异常，同时清除中断信号，将中断标记位设置成 false。这样一来就不用担心长时间休眠中线程感受不到中断了，**因为即便线程还在休眠，仍然能够响应中断通知，并抛出异常**。
```console
1. 创建线程..................
2. 启动线程..................，Thread-0
3. 线程休眠 5 毫秒..................
4. 线程中断..................，Thread-0
count = 0
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at cn.happymaya.snailsjava.concurrent.ThreadStop1.run(ThreadStop1.java:15)
	at java.base/java.lang.Thread.run(Thread.java:833)
```

## 两种最佳处理方式

在实际开发中肯定是团队协作的，不同的人负责编写不同的方法，然后相互调用来实现整个业务的逻辑。如果你负责编写的方法需要被别人调用，同时你的方法内调用了 `sleep()` 或者 `wait()` 等能响应中断的方法时，仅仅 catch 异常是不够的。

```java
void subTask1() {
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        // 在这里不处理该异常是非常不好的
    }
}
```

可以在方法中使用 `try/catch` 或在**方法签名中声明 throws InterruptedException**。

### 方法签名抛异常，run() 强制 try/catch

如上面的代码所示，catch 语句块里代码是空的，没有进行任何处理。

假设线程执行到这个方法，并且正在 sleep，此时有线程发送 interrupt 通知试图中断线程，就会立即抛出异常，并清除中断信号。虽然抛出的异常被 catch 语句块捕捉。但是捕捉到异常的 catch 没有进行任何处理逻辑，相当于把中断信号给隐藏了，这样做是非常不合理的。

基于此，可以选择在方法签名中抛出异常，如下：

```java
/**
 * 在方法签名中抛出异常
 * @throws InterruptedException
 */
void subTask2() throws InterruptedException {
    Thread.sleep(1000);
}
```
正如代码所示，要求**每一个方法的调用方有义务**主动处理异常：调用方要不使用 try/catch 并在 catch 中正确处理异常，要不将异常声明到方法签名中。

如果每层逻辑都遵守规范，便可以将中断信号层层传递到顶层，最终让 `run()` 方法可以捕获到异常。对于 `run()` 方法而言，本身没有抛出 `checkedException` 的能力，只能通过 `try/catch` 来处理异常。层层传递异常的逻辑保障了异常不会被遗漏，至于 `run()` 方法就可以根据不同的业务逻辑来进行相应的处理。

### 再次中断

除了将异常声明到方法签名中的方式外，还可以**在 catch 语句中再次中断线程**。

如下代码所示，需要在 catch 语句块中调用 `Thread.currentThread().interrupt()` 函数。
```java
/**
 * 在 catch 里面再次中断线程
 */
void reInterrupt() {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        e.printStackTrace();
    }
}
```
如果线程在休眠期间被中断，就会自动清除中断信号。假设此时手动添加中断信号，中断信号依然可以被捕捉到。这样后续执行的方法依然可以检测到这里发生过中断，可以做出相应的处理，整个线程可以正常退出。


> 在实际开发中不能盲目吞掉中断，如果不在方法签名中声明，也不在 catch 语句块中再次恢复中断，而是在 catch 中不作处理，这种行为是“屏蔽了中断请求”。倘若盲目屏蔽了中断请求，会导致中断信号被完全忽略，最终导致线程无法正确停止。
{: .prompt-danger }



## 用 volatile 标记位的停止方法是错误的

### 错误的停止方法

几种停止线程的错误方法。比如 `stop()`，`suspend()` 和 `resume()`，这些方法已经被 Java 直接标记为 **@Deprecated**。

如果再调用这些方法，IDE 会友好地提示，不应该再使用它们了。

为什么它们不能使用，原因如下：
-  stop() 会直接把线程停止，这样就没有给线程足够的时间来处理想要在停止前保存数据的逻辑，任务戛然而止，会导致出现数据完整性等问题；
-  suspend() 和 resume() 的问题在于，如果线程调用 suspend()，它并不会释放锁，就开始进入休眠，如果此时仍持有锁，很容易导致死锁问题，因为这把锁在线程被 resume() 之前，是不会被释放的。
  假设线程 A 调用了 suspend() 方法让线程 B 挂起，线程 B 进入休眠，而线程 B 又刚好持有一把锁，此时假设线程 A 想访问线程 B 持有的锁，但由于线程 B 并没有释放锁就进入休眠了，所以对于线程 A 而言，此时拿不到锁，也会陷入阻塞，那么线程 A 和线程 B 就都无法继续向下执行。
  正是因为有这样的风险，所以 suspend() 和 resume() 组合使用的方法也被废弃了。

### volatile 修饰标记位适用的场景

什么场景下 volatile 修饰标记位可以让线程正常停止呢？ 如下代码：

```java
public class VolatileThreadStop implements Runnable{
    private volatile boolean cancled = false;

    @Override
    public void run() {
        int num = 0;
        while (!cancled && num <= 1000000) {
            if (num % 10 == 0) {
                System.out.println(num + " 是 10 的倍数。");
            }
            num++;
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        VolatileThreadStop r = new VolatileThreadStop();
        Thread thread = new Thread(r);
        thread.start();
        Thread.sleep(3000);
        r.cancled = true;
    }
}
```
上面代码中，声明了一个叫作 VolatileStopThread 的类：
1. 实现了 Runnable 接口；
2. 在 run() 中进行 while 循环，在循环体中又进行了两层判断，
   1. 先判断 canceled 变量的值，canceled 变量是一个被 volatile 修饰的初始值为 false 的布尔值，当该值变为 true 时，while 跳出循环；
   2. 然后判断 num 值是否小于 1000000（一百万）。
   在while 循环体里，只要是 10 的倍数就打印出来，然后 num++。 

3. 启动线程，3 秒后，把用 cancled（volatile 修饰的布尔值的标记位）设置成 true，这样，正在运行的线程就会在下一次 while 循环中判断出 canceled 的值已经变成 true 了，这样就不再满足 while 的判断条件，跳出整个 while 循环，线程就停止了。

这种情况是 volatile 修饰的标记位可以正常工作的情况，如果说某个方法是正确的，那么它应该不仅仅是在一种情况下适用，而在其他情况下也应该是适用的。

### volatile 修饰标记位不适用的场景

接下来就用一个**生产者/消费者模式**的案例，来说明 volatile 标记位的停止方法是不完美的。

```java
/**
 * 声明一个生产者
 */
public class Producer implements Runnable {

    // 通过 volatile 标记的初始值为 false 的布尔值 canceled 来停止线程
    public volatile boolean canceled = false;

    // storage 是生产者与消费者之间进行通信的存储器，
    BlockingQueue storage;

    @Override
    public void run() {
        int num = 0;
        try {
            // while 的判断语句是 num 是否小于 100000 及 canceled 是否被标记；
            while (num <= 100000 && !canceled) {
                // 判断 num 如果是 50 的倍数就放到 storage 仓库中
                // 当 num 大于 100000 或被通知停止时，跳出 while 循环并执行 finally 语句块，告诉大家“生产者结束运行”。
                if (num % 50 == 0) {
                    storage.put(num);
                    System.out.println(num + "是 50 的倍数，被放到仓库中了");
                }
                num++;
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("生产者结束运行");
        }
    }
}

/**
 * 声明一个消费者
 */
public class Consumer {

    // 与生产者共用同一个仓库 storage；
    BlockingQueue storage;

    public Consumer(BlockingQueue storage) {
        this.storage = storage;
    }

    // 判断是否需要继续使用更多的数字
    public boolean needMoreNum() {
        // 生产者生产了一些 50 的倍数供消费者使用
        // 消费者是否继续使用数字的判断条件时产生一个随机数
        // 并与 0.97 进行比较，大于 0.97 就不再继续使用数字
        return !(Math.random() > 0.97);
    }
}

public class VolatileThreadStopDemo {
    public static void main(String[] args) throws InterruptedException {

        // 创建【生产者/消费者】共用的仓库 storage，容量是 8
        ArrayBlockingQueue storage = new ArrayBlockingQueue(8);

        // 建立生产者
        Producer producer = new Producer(storage);
        // 将生产者放入线程
        Thread producerThread = new Thread(producer);
        // 启动线程
        producerThread.start();
        // 进行 500 毫秒的休眠，休眠时间保障生产者有足够的时间把仓库塞满
        // 仓库达到容量后就不会再继续往里塞，此时生产者会阻塞
        Thread.sleep(500);

        // 500 毫秒后消费者也被创建出来，
        Consumer consumer = new Consumer(storage);
        // 判断消费者是否需要使用更多的数字
        while (consumer.needMoreNum()) {
            System.out.println(consumer.storage.take() + "被消费了");
            // 每次消费后休眠 100 毫秒
            Thread.sleep(100);
        }
        // 当消费者不再需要数据，就会将 canceled 的标记位设置为 true
        System.out.println("消费者不需要更多数据了....");
        producer.canceled = true;
        System.out.println(producer.canceled);
    }
}
```
当消费者不需要数据时，理论上生产者会跳出 while 循环，并打印输出“生产者运行结束”。然而结果却不是想象的那样，尽管已经把 canceled 设置成 true，但生产者仍然没有停止。

这是因为在这种情况下，生产者在执行 `storage.put(num)` 时发生阻塞，在它被叫醒之前是无法进入下次循环判断 canceled 的值的，所以在这种情况下用 volatile 是没有办法让生产者停下来的，相反如果用 interrupt 语句来中断，即使生产者处于阻塞状态，仍然能够感受到中断信号，并做响应处理。

### 总结

1. 如何正确停止线程：从原理上讲应该用 interrupt 来请求中断，而不是强制停止，因为这样可以避免数据错乱，也可以让线程有时间结束收尾工作；
   
2. 如果是子方法的编写，遇到了 interruptedException，这样处理：
   1. 异常声明在方法中，以便顶层方法可以感知，捕获到异常，
   2. 在 catch 中再次声明中断，这样下次循环也可以感知中断。 
   
3. 正确停止线程，就要求停止方，被停止方，子方法的编写者相互配合，大家都按照一定的规范来编写代码，就可以正确地停止线程了。
4. 比如说已经被舍弃的 `stop()`、`suspend()` 和 `resume()`，它们由于有很大的安全风险，比如死锁风险而被舍弃，而 volatile 这种方法在某些特殊的情况下，比如线程被长时间阻塞的情况，就无法及时感受中断，所以 volatile 是不够全面的停止线程的方法。

