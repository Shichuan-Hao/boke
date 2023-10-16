---
title: 双重检查锁模式为什么必须加 volatile
author:
  name: superhsc
  link: https://github.com/happymaya
date: 2019-10-19 23:33:00 +0800
categories: [Java, 并发]
tags: [thread]
math: true
mermaid: true
---

## 双重检查锁模式的写法

单例模式有多种写法，具体可见[单例设计模式](https://maya.happymaya.cn/posts/singleton/) 和 [单例模式的七种实现方式](https://maya.happymaya.cn/posts/singleton-seven-implementation-methods/)，本文重点关注使用 volatile 强相关的双重检查锁模式的写法，代码如下所示：

```java
public class SingletonVolatile {
    private static volatile SingletonVolatile instance;

    private SingletonVolatile() {
    }

    public static SingletonVolatile getInstance() {
        if (instance == null) {
            synchronized (SingletonVolatile.class) {
                if (instance == null) {
                    instance = new SingletonVolatile();
                }
            }
        }
        return instance;
    }
}
```

方法中首先进行了一次 `if (singleton == null)` 的检查，然后是 synchronized 同步块，然后又是一次 `if (singleton == null)` 的检查，最后是 `singleton = new Singleton()` 来生成实例。

由于进行了两次 `if (singleton == null)` 检查，这就是“双重检查锁”这个名字的由来。这种写法是可以保证线程安全的，假设有两个线程同时到达 synchronized 语句块，那么实例化代码只会由其中先抢到锁的线程执行一次，而后抢到锁的线程会在第二个 if 判断中发现 singleton 不为 null，所以跳过创建实例的语句。再后面的其他线程再来调用 getInstance 方法时，只需判断第一次的 if (singleton == null) ，然后会跳过整个 if 块，直接 return 实例化后的对象。

这种写法的优点是不仅线程安全，而且延迟加载、效率也更高。

## 为什么要 double-check

先看第二次的 check，这时需要考虑这样一种情况，有两个线程同时调用 getInstance 方法，由于 singleton 是空的，因此两个线程都可以通过第一重的 if 判断；然后由于锁机制的存在，会有一个线程先进入同步语句，并进入第二重 if 判断 ，而另外的一个线程就会在外面等待。

不过，当第一个线程执行完 new Singleton() 语句后，就会退出 synchronized 保护的区域，这时如果没有第二重 `if (singleton == null)` 判断的话，那么第二个线程也会创建一个实例，此时就破坏了单例，这肯定是不行的。

而对于第一个 check 而言，如果去掉它，那么所有线程都会串行执行，效率低下，所以两个 check 都是需要保留的。

## 为什么用 volatile 关键字

在双重检查锁模式中，给 singleton 这个对象加了 volatile 关键字，那**为什么要用 volatile 呢？**主要就在于 `singleton = new Singleton()` ，它并非是一个原子操作，事实上，在 JVM 中上述语句至少做了以下这 3 件事：
![这是一张图片](https://images.happymaya.cn/assert/java/thread/java-63-2.png)

1. singleton 分配内存空间；
2. 调用 Singleton 的构造函数等，来初始化 singleton；
3. 将 singleton 对象指向分配的内存空间（执行完这步 singleton 就不是 null 了）。

这里需要留意一下 1-2-3 的顺序，因为存在指令重排序的优化，也就是说第2 步和第 3 步的顺序是不能保证的，最终的执行顺序，可能是 1-2-3，也有可能是 1-3-2。

如果是 1-3-2，那么在第 3 步执行完以后，singleton 就不是 null 了，可是这时第 2 步并没有执行，singleton 对象未完成初始化，它的属性的值可能不是我们所预期的值。假设此时线程 2 进入 getInstance 方法，由于 singleton 已经不是 null 了，所以会通过第一重检查并直接返回，但其实这时的 singleton 并没有完成初始化，所以使用这个实例的时候会报错，详细流程如下图所示：

![这是一张图片](https://images.happymaya.cn/assert/java/thread/java-63-3.png)

线程 1 首先执行新建实例的第一步，也就是分配单例对象的内存空间，由于线程 1 被重排序，所以执行了新建实例的第三步，也就是把 singleton 指向之前分配出来的内存地址，在这第三步执行之后，singleton 对象便不再是 null。

这时线程 2 进入 getInstance 方法，判断 singleton 对象不是 null，紧接着线程 2 就返回 singleton 对象并使用，由于没有初始化，所以报错了。最后，线程 1 “姗姗来迟”，才开始执行新建实例的第二步——初始化对象，可是这时的初始化已经晚了，因为前面已经报错了。

使用了 volatile 之后，相当于是表明了该字段的更新可能是在其他线程中发生的，因此应确保在读取另一个线程写入的值时，可以顺利执行接下来所需的操作。在 JDK 5 以及后续版本所使用的 JMM 中，在使用了 volatile 后，会一定程度禁止相关语句的重排序，从而避免了上述由于重排序所导致的读取到不完整对象的问题的发生。

使用 volatile 的意义主要在于它可以防止避免拿到没完成初始化的对象，从而保证了线程安全。

参考：
- [小宝马的爸爸 - 梦想的家园《单例模式（Singleton）》](https://www.cnblogs.com/BoyXiao/archive/2010/05/07/1729376.html)
- [Jark's Blog《如何正确地写出单例模式》](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)
- [Hollis Chuang《为什么我墙裂建议大家使用枚举来实现单例》](https://www.hollischuang.com/archives/2498)
- [Hollis Chuang《深度分析Java的枚举类型—-枚举的线程安全性及序列化问题》](https://www.hollischuang.com/archives/197)
