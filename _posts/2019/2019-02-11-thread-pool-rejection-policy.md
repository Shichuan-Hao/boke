---
title: 线程池的 4 种拒绝策略
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2019-02-11 23:33:00 +0800
categories: [Java, 并发]
tags: [thread]
math: true
mermaid: true
---

## 拒绝时机

新建线程池时可以指定它的任务拒绝策略，例如：

```java
public class ThreadPoolRejectStrategy {
    public static void main(String[] args) {
        new ThreadPoolExecutor(
                5,
                10,
                5,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(),
                new ThreadPoolExecutor.DiscardOldestPolicy());
    }
}
```

在自定义策略来拒绝任务，需要关注拒绝任务的时机是什么呢？ 通常情况下，线程池会在以下两种情况下会拒绝新提交的任务：
1. 当调用 shutdown 等方法关闭线程池后，即便此时可能线程池内部依然有没执行完的任务正在执行，但是由于线程池已经关闭，此时如果再向线程池内提交任务，就会遭到拒绝；
1. 当线程池没有能力继续处理新提交的任务，也就是工作已经非常饱和的时候。

比如：新建一个线程池，使用容量上限为 10 的 ArrayBlockingQueue 作为任务队列，并且指定线程池的核心线程数为 5，最大线程数为 10。假设此时有 20 个耗时任务被提交，在这种情况下：
- 线程池会首先创建核心数量的线程，也就是 5个线程来执行任务；
- 然后往队列里去放任务，队列的 10 个容量被放满了之后，会继续创建新线程，直到达到最大线程数 10；
- 此时线程池中一共有 20 个任务，其中 10 个任务正在被 10 个线程执行，还有 10 个任务在任务队列中等待，而且由于线程池的最大线程数量就是 10，所以已经不能再增加更多的线程来帮忙处理任务了，这就意味着此时线程池工作饱和，这个时候再提交新任务时就会被拒绝

![由于工作饱和导致的拒绝](https://maxpixelton.github.io/images/assert/java/thread/java-thread-rejection-policy-1.png)

如上图：

1. 首先看右侧上方的队列部分，可以看到目前队列已经满了；
2. 而图中队列下方的每个线程都在工作，且线程数已经达到最大值 10；
3. 如果此时再有新的任务提交，线程池由于没有能力继续处理新提交的任务，所以就会拒绝。


## 拒绝策略

Java 在 ThreadPoolExecutor 类中为提供了 4 种默认的拒绝策略来应对不同的场景，都实现了 RejectedExecutionHandler 接口，如图所示：

![Java ThreadPoolExecutor 类中提供的 4 种默认的拒绝策略](https://maxpixelton.github.io/images/assert/java/thread/java-thread-rejection-policy-2.png)

1. **AbortPolicy**。在拒绝任务时，直接抛出 `RejecetExecutionException` 异常，从而感知到任务被拒绝，进而根据业务逻辑选择**重试**或者**放弃提交**等策略；
2. **DiscardPolicy**。如名字一样，当新任务被提交后直接被丢弃掉，也不会给任何的通知，相对而言该策略存在一定的风险，因为提交的时候根本不知道这个任务会被丢弃，所以有可能造成数据丢失；
3. **DiscardOldestPolicy**。如果线程池没被关闭，且没有能力执行，则会丢弃任务队列中的头结点，通常是存活时间最长的任务。与第二种不同之处在于它丢弃的不是最新提交的，而是队列中存活时间最长的，这样就可以腾出空间给新提交的任务，它同样也存在数据丢失风险。
4. **CallerRunsPolicy**。相对其它拒绝策略，它比较完善，当有新任务提交后，如果线程池没被关闭且没有能力执行，则把这个任务交于提交任务线程执行，也就是谁提交任务，谁就负责执行任务（提交任务的线程是不固定的，取决于具体是哪个线程执行 submit 等方法）。这样做有两点好处：
   - 新提交的任务不会被丢弃，也就不会造成数据丢失，更不会出现业务损失；
   - 由于谁提交任务谁就要负责执行任务，这样提交任务的线程就得负责执行任务，而执行任务又是比较耗时的，在这段期间，提交任务的线程被占用，也就不会再提交新的任务，减缓了任务提交的速度，相当于是一个负反馈。在此期间，线程池中的线程也可以充分利用这段时间来执行掉一部分任务，腾出一定的空间，相当于是给了线程池一定的缓冲期。
