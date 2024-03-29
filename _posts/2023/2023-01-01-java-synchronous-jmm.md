---
title: Java 同步之 JMM（Java Memory Model）
author: Shaun
date: 2023-01-01 15:49:00 +0800
categories: [博客]
tags: [synchronous]
math: true
mermaid: true
---

## 简介

Java 内存模型是在硬件内存模型上的更高层的抽象，它屏蔽了各种硬件和操作系统访问的差异性，保证了 Java 程序在各种平台下对内存的访问都能达到一致的效果。

## 硬件内存模型

在现代计算机的硬件体系中，CPU 的运算速度非常快，远远高于它从存储介质读取数据的速度，这里的存储介质非常多，比如磁盘、光盘、网卡、内存等，这些存储介质有一个很明显的特点：距离 CPU 越近的存储介质往往越小越贵越快，距离 CPU 越远的存储介质往往越大越便宜越慢。

因此，在程序运行的过程中，CPU 大部分时间都浪费在磁盘 IO、网络通讯、数据库访问上，如果不想让 CPU 在那里白白等待，我们就必须想办法将 CPU 的运算能力压榨出来，否则就会造成很大的浪费，而让 CPU 同时处理多项任务就是最容易想到的，也是被证明非常有效的压榨手段，这也是常说的**“并发执行”**。

但是，让 CPU 并发地执行多项任务是不容易的，因为所有运算都不可能只依靠 CPU 的计算就能完成，往往还需要跟内存进行交互，如读取运算数据、存储运算结果等。

正如上文所说，CPU 与内存的交互是缓慢的，所以这就要求我们要想办法在 CPU 和内存之间建立一种连接，使它们达到一种平衡，让运算能快速地运行，而这种连接就是常说的**“高速缓存”**。

高速换成的速度非常接近 CPU 的，但是它的引入又带来新的问题，现代的 CPU 往往是有多个核心的，每个核心都有自己的换成，而多个核心之间是不存在时间片的竞争的，它们可以并行地执行，那么，怎么保证这些缓存与主内存的数据的一致性就成了一个难题。

为解决换成一致性的问题，多个核心在访问缓存时遵循一些协议，在读写操作时根据协议来操作，这些协议有 MSI、MESI、MOSI 等，它们定义了何时应该访问缓存中的数据、何时应该让缓存失效、何时应该访问主内存中的数据等基本原则。

<!-- ![CPU与主内存的交互]() -->

随着 CPU 能力的不断提升，一层缓存就无法满足要求了，就逐渐衍生出了多级缓存。

按照数据读取顺序和 CPU 的紧密程度，CPU 的缓存可以分为一级缓存（L1）、二级缓存（L2）、三级缓存（L3），每一级缓存存储的数据都是下一级的一部分。

这三种缓存的技术难度和制作成本是相对递减的，容量也是相对递增的。

所以，在有了多级缓存后，程序的运行就变成了：

- 当 CPU 要读取一个数据的时候，先从一级缓存中查找，如果没有找到再从二级缓存中查找，如果没有找到再从三级缓存中查找，如果没有找到再从主内存中查找，然后再把找到的数据依次加载到多级缓存中，下次再使用相关的数据直接从缓存中查找即可。

加载到缓存中数据也不是说用到哪个就加载哪个，而是加载内存中连续的数据，一般来说是加载连续 64 个字节，因此，如果访问一个 long 类型的数组时，当数组中的一个值被加载到缓存中时，另外 7 个元素也会被加载到缓存中，这就是 “缓存行” 的概念。

缓存行虽然能极大地提高程序运行的效率，但是在多线程对共享变量的访问过程中又带来了新的问题，也就是非常著名的“伪共享”。

除此之外，为了使 CPU 中的运算单元能够充分地被利用，CPU 可能会对输入的代码进行**乱序执行优化**，然后在计算之后再将乱序执行的结果进行重组，保证该结果与顺序执行的结果一致，但并不保证程序中的各个语句计算的先后顺序与代码输入顺序一致，因此，如果一个计算任务依赖于两一个计算任务的结果，那么其顺序性并不能靠代码的先后顺序来保证。

与 CPU 的**乱序执行优化**，java 虚拟机的**即时编译器**也有类型的**指令重排序优化**。

为了解决上面提到的**多个缓存读写一致性**以及**乱序执行优化**的问题，这就有了**内存模型**。

**内存模型**<font color=red>定义了共享内存系统中多线程读写操作行为的规范。</font>


## Java 内存模型

Java 内存模型（Java Memory Model，JMM）是在硬件内存模型基础上更高层的抽象，它屏蔽了各种硬件和操作系统对内存访问的差异性，从而实现让 Java 程序在各种平台上都能达到一致的并发效果。

Java 内存模型定义了程序中各个变量的访问规则，即**在虚拟机中将变量存储到内存和从内存中取出**这样的底层细节。这里所说的变量包括实例字段、静态字段、但不包括局部变量和方法参数，因为它们是线程私有的，它们不会被共享，自然不存在竞争问题。

为获得更好的执行效能，Java 内存模型没有限制执行引擎使用处理器的特定寄存器或缓存来和主内存进行交互，也没有限制即时编译器调整代码的执行顺序等这类权利。

Java 内存模型规定了所有的变量都存储在主内存中，这里的主内存跟介绍硬件时所用的名字一样，两者可以类比，但此处仅仅指虚拟机中内存的一部分。

除了主内存，每条线程还有自己的工作内存，此次可以与 CPU 的高速缓存进行类比。工作内存中保存着该线程使用到变量的主内存副本的拷贝，线程对变量的操作都必须在工作内存中进行，包括读取和赋值等，而不能直接读写主内存中的变量，不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递必须通过主内存来完成。

线程、工作内存、主内存三者的关系如下图所示：

<!-- ![线程、工作内存、主内存三者的关系]() -->

注意，这里所得主内存、工作内存跟 Java 虚拟机内存区域划分中得堆、栈是不同层次得内存划分。如果两者一定要勉强对应起来，主内存主要对应于堆中对象得实例部分，而工作内存主要对应虚拟机栈中得部分区域。

从更低层次来说，主内存主要对应于硬件内存部分，工作内存主要对应于 CPU 的高速缓存和寄存器部分，但也不是绝对的，主内存也可能存在于高速缓存和寄存器中，工作内存也可能存在于硬件内存中。

<!-- ![JVM 虚拟机内存区域划分与 Java 内存模型的对应]() -->

## 内存间的交互操作

关于主内存和工作内存之间具体的交互协议的，Java 内存模型定义了 8 种具体的操作来完成：

1. `lock`，锁定，作用于主内存的变量，它把主内存中的变量标识为一条线程独占状态。

2. `unlock`，解锁，作用于主内存的变量，它把锁定的变量释放出来，释放出来的变量才可以被其他线程锁定。

3. `read`，读取，作用于主内存的变量，它把一个变量从主内存传输到工作内存中，以便后续 load 操作使用。

4. `load`， 载入，作用于工作内存的变量，它把 read 操作从主内存得到的变量放入工作内存的变量副本中。

5. `use`，使用，作用于工作内存的变量，它把工作内存中的一个变量传递给执行引擎，每当虚拟机遇到一个给变量赋值的字节码指令时使用这个操作。

6. `assign`，赋值，作用于工作内存的变量，它把一个从执行引擎接收到的变量赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时使用这个操作。

7. `store`，存储，作用于工作内存的变量，它把工作内存中一个变量的值传递到主内存中，以便后续的 write 操作使用。

8. `write`，写入，作用于主内存的变量，它把 store 操作从工作内存得到的变量的值放入到主内存的变量中。

如果要把一个变量从主内存复制到工作内存，那么就要按顺序地执行 `read` 和 `load` 操作。同样地，如果要把一个变量从工作内存同步回主内存，就要按顺序地执行 `store` 和 `write` 操作。注意，这里只说明了要按顺序，并没有说一定要连续，也就是说可以在 `read` 与 `load` 之间、`store` 与 `write` 之间插入其它操作。比如，对主内存中的变量 `a` 和 `b` 的访问，可以按照以下顺序执行：

- `read a -> read b -> load b -> load a`。

此外，Java 内存模型还定义了执行上诉 8 种操作的基本原则：

1. 不允许 `read` 和 `load`、`store` 和 `write` 操作之一单独出现，即不允许出现从主内存读取了而工作内存不接受，或者从工作内存回写了但主内存不接受的情况出现。

2. 不允许一个线程丢弃它最近的 `assign` 操作，即变量在工作内存变化了必须把该变化同步回主内存。

3. 不允许一个线程没有原因地（即未发生过 `assign` 操作）把一个变量从工作内存同步回主内存。

4. 一个新的变量必须在主内存中诞生，不允许工作内存中直接使用一个未被初始化（`load` 或 `assgin`）过的变量，换句话说就是对一个变量的 `use` 和 `store` 操作之前必须执行过 `load` 和 `assign` 操作。

5. 一个变量同一时刻只允许一条线程对其进行 `lock` 操作，但 `lock` 操作可以被同一个线程执行多次，多次执行 `lock` 后，只有执行相同次数的 `unlock` 操作，变量才能被解锁。

6. 如果对一个变量执行 `lock` 操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行 load 或 assign 操作初始化变量的值。

7. 如果一个变量没有被 `lock` 操作锁定，就不允许对其执行 `unlock` 操作，也不允许 `unlock` 一个其它线程锁定的变量。

8. 对一个变量执行 `unlock` 操作之前，必须先把此变量同步回主内存中，即执行 `store` 和 `write` 操作。

> **注意**<br/>这里的 `lock` 和 `unlock` 是实现 `synchronized` 的基础，Java 没有把 `lock` 和 `unlock` 操作直接放开给用户使用，但提供了两个更高层次的指令来隐式地使用这两个操作，即 `mointerenter` 和 `mointerexit`。
{: .prompt-info }

## 原子性、可见性、有序性

Java 内存模型是为了解决多线程环境下共享变量的一致性问题。

一致性主要包含三大特性：
- 原子性。
- 可见性。
- 有序性。

### 原子性

**原子性**指的是一段操作一旦开始就会一直运行到底，中间不会被其它线程打断，这段操作可以是一个操作，也可以是多个操作。

由 Java 内存模型来直接保证的原子性操作包括 `read`、`load`、`user`、`assign`、`store`、`write` 这两个操作，我们可以大致认为基本类型变量的读写是具备原子性的。

如果应用需要一个更大范围的原子性，Java 内存模型还提高了 lock 和 unlock 这两个操作来满足这种需求，尽管不能直接使用这两个操作，但我们可以使用它们更具体的实现 synchronized 来实现。

因此，synchronized 快之间的操作也是原子性的。

### 可见性

可见性指的是当一个线程修改了共享变量的值，其它线程能立即感知到这种变化。

Java 内存模型是通过在变更修改后同步回主内存

### 有序性

## 先行发生原则（Happens-Before）

## 总监