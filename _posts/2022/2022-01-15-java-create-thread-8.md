---
title: Java 中创建线程的 8 种方式
author: Shaun
date: 2022-01-15 11:49:00 +0800
categories: [博客]
tags: [java thread]
math: true
mermaid: true
---

## 问题

1. 创建线程有几种方式 ？

2. 它们分别有什么运用场景 ？

## 简介

创建线程，是多线程编程中的基操，整理一下，大概有 8 种创建线程的方式。

### 继承 Thread 类并重写 `run()` 方法

```java
public class CreatingThread01 extends Thread {

    @Override
    public void run() {
        System.out.println(getName() + " is running...");
    }

    public static void main(String[] args) {
        new CreatingThread01().start();
        new CreatingThread01().start();
        new CreatingThread01().start();
        new CreatingThread01().start();
    }
}
```

### 实现 Runnable 接口

```java
public class CreatingThread02 implements Runnable{

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " is running...");
    }

    public static void main(String[] args) {
        new Thread(new CreatingThread02()).start();
        new Thread(new CreatingThread02()).start();
        new Thread(new CreatingThread02()).start();
        new Thread(new CreatingThread02()).start();
    }
}
```

### 匿名内部类

```java
public class CreatingThread03 {
    public static void main(String[] args) {
        // Thread 匿名类，重写 Thread 的 run() 方法
        new Thread() {
            @Override
            public void run() {
                System.out.println(getName() + " is running...");
            }
        }.start();
        
        // Runnable 匿名类，实现其 run() 方法
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " is running...");
            }
        }).start();
    }
}
```

使用匿名类有 3 种方式：

1. 重写 Thread 的 run() 方法
2. 传入 Runnable 的匿名类，
3. 使用 lambda 方式，现在一般使用第三种（java8+），简单快捷。

### 实现 Callable 接口

```java
public class CreatingThread04
        implements Callable<Long> {

    @Override
    public Long call() throws Exception {
        Thread.sleep(2000);
        System.out.println(Thread.currentThread().getId());
        return Thread.currentThread().getId();
    }

    public static void main(String[] args)
            throws ExecutionException, InterruptedException {
        FutureTask<Long> task = new FutureTask<>(new CreatingThread04());
        new Thread(task).start();
        System.out.println("等待完成任务...");
        Long result = task.get();
        System.out.println("任务结果：" + result);
    }
}
```

实现 Callable 接口，可以获取线程执行的结果，FutureTask 实际上实现了 Runnable 接口。

### 定时器（java.util.Timer）

```java
public class CreatingThread05 {
    public static void main(String[] args) {
        Timer timer = new Timer();
        // 每隔 1s 执行一次
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " is running");
            }
        }, 0, 1000);
    }
}
```

使用定时器 `java.util.Timer` 可以快速地实现定时任务，`TimerTask` 实际上实现了 `Runnable` 接口。

### 线程池

```java
public class CreatingThread06 {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 100; i++) {
            threadPool.execute(() -> System.out.println(Thread.currentThread().getName()));
        }
    }
}
```

### 并行计算（Java8+）

```java
public class CreatingThread07 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
        // 串行，打印结果为 12345
        list.forEach(System.out::print);
        System.out.println();
        // 并行，打印结果随机，比如 35214
        list.parallelStream().forEach(System.out::print);
    }
}
```

使用**并行计算**的方式，可以提供程序运行的效率，多线程并行执行。

### Spring 异步方法

首先，Spring Boot 启动类加上 `@EnableAsync` 注解（`@EnableAsync` 是 Spring 支持的，这里方便举例使用 Spring Boot）。

```java
@EnableAsync
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

其次，方法上加上 `@Async` 注解。

```java
@Service
public class CreatingThread08Service {
    @Async
    public void call(){
        System.out.println(Thread.currentThread().getName() + " is running");
    }
}
```

然后，测试用例直接跟使用一般的 Service 方法一模一样。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Appendable.class)
public class CreatingThread08Test {

    @Autowired
    private CreatingThread08Service creatingThread08Service;

    @Test
    public void test() {
        creatingThread08Service.call();
        creatingThread08Service.call();
        creatingThread08Service.call();
        creatingThread08Service.call();
    }

}
```

运行结果如下：

```
task-1 is running
task-2 is running
task-3 is running
task-4 is running
```

可以看到每次执行方法时使用的线程都不一样。

使用 Spring 异步方法的方式，可以说是相当地方便，适用于前后逻辑不相关联的适合用异步调用的一些方法，比如发送短信的功能。

### 小结

1. 继承 Thread 类并重写 `run()` 方法

2. 实现 Runnable 接口

3. 匿名内部类

4. 实现 Callable 接口

5. 定时器（java.util.Timer）

6. 线程池

7. 并行计算（Java8+）

8. Spring 异步方法

## 总结

上面整理的 8 种创建线程的方式，本质就两种：
1. 继承 Thread 类并重写 run() 方法。
2. 实现 Runnable 接口的 run() 方法。
它们之间到底有什么联系 ？

看下面代码，同时继承了 Thread 并实现 Runnable 接口，应该输出什么 ？

```java
public class CreatingThread09 {
    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println("Runnable: " + Thread.currentThread().getName());
        }) {
            @Override
            public void run() {
                System.out.println("Thread: " + getName());
            }
        }.start();
    }
}
```

这里，需要有必要看下 Thread 类的源码：

```java
public class Thread implements Runnable {

    /* Thread 维护了一个 Runnbale 的实例 */
    /* What will be run（将运行什么）.*/
    private Runnable target;

    /**
     * Initializes a Thread.
     *
     * @param g 线程组
     * @param target run() 方法被调用的对象
     * @param name 新线程的名称
     * @param stackSize 新线程所需的堆栈大小，或为零表示忽略此参数。

     * @param acc 要继承的 AccessControlContext，如果为 null，则为 AccessController.getContext()
     * @param inheritThreadLocals 如果为 true，则从构造线程继承可继承线程局部变量的初始值
     */
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        // ...
        // 构造方法传进来的 Runnable 会赋值给 target
        this.target = target;
        // ...
 
    }

    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }


    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }

    @Override
    public void run() {
        // Thread 默认的 run() 方法，如果 target 不为空，就会执行 target 的 run() 方法
        if (target != null) {
            target.run();
        }
    }
}
```

上面的例子同时继承 Thread 并实现了 Runnable 接口，根据源码，实际上相当于重写了 Thread 的 run() 方法，在，在 Thread 的 run() 方法时实际上跟 target 都没有关系了。

因此，上面的例子输出结果为 `Thread: Thread - 0`，只是输出了重写 Thread 的 run() 方法中的内容。