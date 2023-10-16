---
title: 模拟分级队列调度的模型
author:
  name: superhsc
  link: https://github.com/happymaya
date: 2020-06-20 23:33:35 +0800
categories: [计算机基础, 操作系统]
tags: [Thread scheduling]
math: true
mermaid: true
---


用 Java 实现了一个简单的 yield 框架。 没有到协程的级别，但是也初具规模。

考虑到协程实现需要更复杂一些，所以我用 PriorityQueue 来放高优任务；然后我用 LinkedList 来作为放普通任务的队列。Java 语言中的`add`和`remove`方法刚好构成了入队和出队操作。

```java
/**
* 存放高优任务 High-priority
*/
private PriorityQueue<Task> urgents;

/**
* 存放普通任务队列
*/
private ArrayList<LinkedList<Task>> multLevelQueues;
```

而后，实现了一个`submit`方法用于提交任务，代码如下：

```java
var scheduler = new MultiLevelScheduler();

scheduler.submit((IYieldFunction yield) -> {
    System.out.println("Urgent");
}, 10);
```

普通任务，默认是 3 级队列。提交的任务优先级小于 100 的会放入紧急队列。每个任务就是一个简单的函数。构造了一个 next() 方法用于决定下一个执行的任务，代码如下：

```java
private Task next() {
    if (this.urgents.size() > 0) {
        return this.urgents.remove();
    } else {
        for (int i = 0; i < this.level; i++) {
            var queue = this.multLevelQueues.get(i);
            if (queue.size() > 0) {
                return queue.remove();
            }
        }
    }
    return null;
}
```

先判断高优队列，然后再逐级看普通队列。

执行的程序就是递归调用 runNext() 方法，代码如下：

```java
private void runNext() {
    var nextTask = this.next();
    if (nextTask == null) {
        return;
    }
    
    if (nextTask.isYield()) {
        return;
    }
    
    nextTask.run(() -> {
        // yiled 内容 .... 省略
    });
    this.runNext();
}
```

上面程序中，如果当前任务在`yield`状态，那么终止当前程序。`yield`相当于函数调用，从`yield`函数调用中返回相当于继续执行。`yield`相当于任务主动让出执行时间。使用`yield`模式不需要线程切换，可以最大程度利用单核效率。

最后是`yield`的实现，nextTask.run 后面的匿名函数就是`yield`方法，它像一个调度程序一样，先简单保存当前的状态，然后将当前任务放到对应的位置（重新入队，或者移动到下一级队列）。如果当前任务是高优任务，`yield`程序会直接返回，因为高优任务没有必要`yield`，代码如下：

```java

    nextTask.run(() -> {
        if (nextTask.level == -1) {
            // high-priority forbid yield
            return;
        }
        
        nextTask.setYield(true);
        if (nextTask.level < this.level - 1) {
            multLevelQueues.get(nextTask.level + 1).add(nextTask);
            nextTask.setLevel(nextTask.level + 1);
        } else {
            multLevelQueues.get(nextTask.level).add(nextTask);
        }
        this.runNext();
    });
    this.runNext();
```

下面是完成的程序：

```java
@FunctionalInterface
public interface ITask {
    void run(IYieldFunction yieldFunction);
}

@FunctionalInterface
public interface IYieldFunction {
    void yield();
}

public class Task implements Comparable<Task>{

    int level = -1;
    ITask task;
    int priority;
    private boolean yield;

    public Task(ITask task, int priority) {
        this.task = task;
        this.priority = priority;
    }


    @Override
    public int compareTo(Task o) {
        return this.priority - o.priority;
    }

    public int getLevel() {
        return level;
    }

    public void setLevel(int level) {
        this.level = level;
    }

    public void run(IYieldFunction f) {
        this.task.run(f);
    }

    public void setYield(boolean yield) {
        this.yield = yield;
    }

    public boolean isYield() {
        return yield;
    }
}

public class MultiLevelScheduler {

    /**
     * 存放高优任务 High-priority
     */
    private PriorityQueue<Task> urgents;
    /**
     * 存放普通任务队列
     */
    private ArrayList<LinkedList<Task>> multLevelQueues;

    /**
     * Levels of Scheduler
     */
    private int level = 3;

    public MultiLevelScheduler() {
        this.init();
    }

    public MultiLevelScheduler(int level) {
        this.level = level;
        this.init();
    }

    private void init() {
        urgents = new PriorityQueue<>();
        multLevelQueues = new ArrayList<>();
        for (int i = 0; i < this.level; i++) {
            multLevelQueues.add(new LinkedList<Task>());
        }
    }

    public void submit(ITask itask, int priority) {
        var task = new Task(itask, priority);
        if (priority >= 100) {
            this.multLevelQueues.get(0).add(task);
            task.setLevel(0);
        } else {
            this.urgents.add(task);
        }
    }

    public void submit(ITask t) {
        this.submit(t, 100);
    }

    private Task next() {
        if (this.urgents.size() > 0) {
            return this.urgents.remove();
        } else {
            for (int i = 0; i < this.level; i++) {
                var queue = this.multLevelQueues.get(i);
                if (queue.size() > 0) {
                    return queue.remove();
                }
            }
        }
        return null;
    }

    private void runNext() {
        var nextTask = this.next();
        if (nextTask == null) {
            return;
        }
        if (nextTask.isYield()) {
            return;
        }
        nextTask.run(() -> {
            if (nextTask.level == -1) {
                // high-priority forbid yield
                return;
            }
            nextTask.setYield(true);
            if (nextTask.level < this.level - 1) {
                multLevelQueues.get(nextTask.level + 1).add(nextTask);
                nextTask.setLevel(nextTask.level + 1);
            } else {
                multLevelQueues.get(nextTask.level).add(nextTask);
            }
            this.runNext();
        });
        this.runNext();
    }

    public void start() throws InterruptedException {
        this.runNext();
    }

}

public class Demo {
    public static void main(String[] args) throws InterruptedException {
        var scheduler = new MultiLevelScheduler();
        scheduler.submit((IYieldFunction yield) -> {
            System.out.println("Urgent");
        }, 10);
        scheduler.submit((IYieldFunction yield) -> {
            System.out.println("Most Urgent");
        }, 0);
        scheduler.submit((IYieldFunction yield) -> {
            System.out.println("A1");
            yield.yield();
            System.out.println("A2");
        });
        scheduler.submit((IYieldFunction yield) -> {
            System.out.println("B");
        });
        scheduler.submit((IYieldFunction f) -> {
            System.out.println("C");
        });
        scheduler.start();
    }
}

// 输出结果
// Most Urgent
// Urgent
// A1
// B
// C
// A2
```

看到结果中任务 1 发生了`yield`在打印 A2 之前交出了控制权导致任务 B,C 先被执行。如果想在 yield 出上增加定时的功能，可以考虑 yield 发生后将任务移出队列，并在定时结束后重新插入回来。