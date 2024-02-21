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

关于线程的生命周期，我们可以看下 `java.lang.Thread.State` 这个类，它是线程的内部枚举类，定义了线程的各种状态，并且注释也很清晰。

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
      * 1. Object.wait() 无超时的方法后且未被 notify() 前，如果被 notify() 了会进入 BLOCKED 状态。
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

锁分为两大类：一类是 **synchronized 锁**，一类是**基于 AQS 锁（以重入锁为例）**，即内部使用了 `LockSupport.park()` 或 `LockSupport.parkNanos()` 或 `LockSupport.parkUntil()` 几个方法的锁。

无论是 synchronized 锁还是基于 AQS 的锁，内部都是分成两个队列，一个是**同步队列（AQS 的队列）**，一个是**等待队列（Contition 的队列）**。

对于内部调用了 `object.wait()/wait(timeout)` 或者 `contition.await()/await(timeout)` 方法，线程都是先进入等待队列，被 `notify()/signal()` 或者超时后，才会进入同步队列。

> **明确声明，BLOCKED 状态只有线程处于 synchronized 的同步队列的时候才会有这个状态，其它任何情况都跟这个状态无关。**
{: .prompt-tip }


### 对于 synchronized 锁

1. 对于 synchronized，线程执行 synchronized 的时候，如果立即获得了锁（没有进入同步队列），线程处于 RUNNABLE 状态。

2. 对于 synchronized，线程执行 synchronized 的时候，如果无法获得锁（直接进入同步队列），线程处于 BLOCKED 状态。

3. 对于 synchronized 内部，调用了 `object.wait()` 之后线程处于 WAITING 状态（进入等待队列）。

4. 对于 synchronized 内部，调用了 `object.wait(timeout)` 之后线程处于 TIMED_WAITING 状态（进入等待队列）。

5. 对于 synchronized 内部，调用了 `object.wait()` 之后且被 `notify()` 了，如果线程立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

6. 对于 synchronized 内部，调用了 `object.wait(timeout)` 之后且被 `notify()` 了，如果线程立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

7. 对于 synchronized 内部，调用了 `object.wait(timeout)` 之后且超时了，如果线程这时正好立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

8. 对于 synchronized 内部，调用了 `object.wait()` 之后且被 `notify()` 了，如果线程无法获得锁（也就是进入了同步队列），线程处于 BLOCKED 状态。

9. 对于 synchronized 内部，调用了 `object.wait(timeout)` 之后且被 `notify()` 了或者超时了，如果线程无法获得锁（也就是进入了同步队列），线程处于 BLOCKED 状态。


### 对于 AQS 的锁

以重入锁为例，即内部使用了 `LockSupport.park()/parkNanos()/parkUntil()` 几个方法的锁。

1. 对于重入锁，线程执行 `lock.lock()` 的时候，如果立即获得了锁（没有进入同步队列），线程处于 RUNNABLE 状态。

2. 对于重入锁，线程执行 `lock.lock()` 的时候，如果无法获得锁（直接进入同步队列），线程处于 WAITING 状态。

3. 对于重入锁内部，调用了 `condition.await()` 之后线程处于 WAITING 状态（进入等待队列）。

4. 对于重入锁内部，调用了 `condition.await(timeout)` 之后线程处于 TIMED_WAITING 状态（进入等待队列）。

5. 对于重入锁内部，调用了 `condition.await()` 之后且被 `signal()` 了，如果线程立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

6. 对于重入锁内部，调用了 `condition.await(timeout)` 之后且被 `singal()` 了，如果线程立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

7. 对于重入锁内部，调用了 `condition.await(timeout)` 之后且超时了，如果这时线程正好立即获得了锁（也就是没有进入同步队列），线程处于 RUNNABLE 状态。

8. 对于重入锁内部，调用了 `condition.await()` 之后且被 `signal()` 了，如果线程无法获得锁（也就是进入了同步队列），线程处于 WAITING 状态。

9. 对于重入锁内部，调用了 `condition.await(timeout)` 之后且被 `singal()` 了或者超时了，如果线程无法获得锁（也就是进入了同步队列），线程处于 WAITING 状态。

10. 对于重入锁， 如果内部调用了 `condition.await()` 之后且被 `signal()` 之后依然无法获取锁的，其实经历了两次 WAITING 状态的切换，一次是在等待队列，一次是在同步队列。

11. 对于重入锁，如果内部调用了 `condition.await(timeout)` 之后且被 `signal()` 或超时了的，状态会有一个从 TIMED_WAITING 切换到 WAITING 的过程，也就是从等待队列进入到同步队列。


## 测试

如何验证上面的描述呢 ？先看下面的代码：

```java
public class ThreadLifeTest {

    public static void main(String[] args) {
        Object object = new Object();
        ReentrantLock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            synchronized (object) {
                try {
                    System.out.println("thread1 waiting");
                    // object.wait(5000);
                    System.out.println("thread1 after waiting");
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        }, "Thread1").start();

        new Thread(() -> {
            synchronized (object) {
                System.out.println("thread2 notify");
                // 打开或关闭这段注释，观察 Thread1 的状态
                object.notify();
                // notify 之后当前线程并不会释放锁，只是被 notify 的线程从等待队列进入同步队列
                // sleep 也不会释放锁
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "Thread2").start();

        new Thread(() -> {
            lock.lock();
            System.out.println("thread3 waiting");
            try {
                condition.await();
//                condition.await(200, TimeUnit.SECONDS);
                System.out.println("thread3 after waiting");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }, "Thread3").start();

        new Thread(() -> {
            lock.lock();
            System.out.println("thread4");
            // 打开或关闭这段注释，观察 Thread3 的状态
            condition.signal();
            // signal 之后当前线程并不会释放锁，只是被 signal 的线程从等待队列进入同步队列
            // sleep 也不会释放锁
            try {
                Thread.sleep(1000000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }, "Thread4").start();

    }

}
```

打开或关闭上面注释部分的代码，使用 IDEA 的 RUN 模式运行代码，然后点击左边的一个摄像头按钮（jstack），查看各线程的状态。

注意：不要使用 DEBUG 模式，DEBUG 模式全部变成 WAITING 状态了，很神奇 ！！！！
