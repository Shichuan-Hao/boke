---
title: Java 异常
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2018-06-02 22:33:00 +0800
categories: [Java, 异常]
tags:  [exception]
math: true
mermaid: true
---

对于任何一门编程语言来说，异常总是不可避免的。

出现异常原因可能是多样的，例如：用户非法输入导致数组越界问题、网络中断造的下载超时问题、以及代码对业务逻辑判断失误等等情况，所以，正是由于这些不可控因素存在，不能对异常视而不见。

可以通过异常处理机制让程序有更好的容错性和兼容性，当程序出现异常时，主动或被动的抛出 Exception（或子类）对象通知系统，从而将业务功能实现代码和错误处理代码分离。那么，想要知道在 Java 程序中应该如何处理异常，首先就要搞清楚 Java 语言中定义了哪些异常。

## 异常分类

广义上说，Java 异常被分为两大类：Checked 异常和 Runtime 异常（运行时异常），更细一点：

- 所有的 RuntimeException 类及其子类的实例被称为 Runtime 异常
- 不是 RuntimeException 类及其子类的异常实例则被称为 Checked 异常

好的，既然这里提到了 Java 中的核心异常类 RuntimeException，就来看一张图，这张图是 Java 异常类组织结构。同时，为了更好区分异常类别（上文所说的两类异常），用红色和绿色区分。

![this is images](https://maxpixelton.github.io/images/assert/java/log-stack/0201.jpg)

从上面的这张图可以看出，Throwable 是 Java 异常的顶级类，所有的异常都继承于这个类。Error 和 Exception 则是 “语言级别” 的两个大分类。其中：

- Error 是非程序异常，即程序不能捕获的异常，一般是编译或者系统性的错误，如 OutOfMemorry 内存溢出异常等
- Exception 是程序异常类，由程序内部产生，也就是由它派生出了 Checked 异常和 Runtime 异常

那么，对于 Exception 派生出的这两类异常，在编码时应该如何处理它们呢？

## 正确对待 Checked 异常和 Runtime 异常

之前已经说过，不能够在程序内部去处理 Error，但是对于 Exception，是需要考虑处理的。

### 正确对待 Checked 异常

只有 Java 语言提供了 Checked 异常，其他语言都没有提供 Checked 异常。Java 认为 Checked 异常都是可以被修复的异常，所以 Java 程序必须显式处理 Checked 异常。如果程序没有处理 Checked 异常，该程序在编译时就会发生错误，无法通过编译。

对于 Checked 异常，有两种处理方式：

- 如果你明确的知道应该如何处理当前代码中的异常，你应该使用 `try-catch` 方式捕获异常，并在 `catch` 中修复当前的问题
- 如果你不清楚应该如何处理当前的异常，你应该在方法声明中抛出（`throws`）该异常

### 正确对待 Runtime 异常

对于 Runtime 异常来说，编译器的要求则 “宽松的多”：编译器并不要求强制处理该类异常，虽然代码可能存在错误，但是，编译器不会去主动检查这种错误，同时，在绝大多数情况下，这种检查也是根本没有必要的。

确实，不仅仅是编译器不要求处理；实际上，在写代码时，一般也不会去处理这类问题。主要有以下两个原因：

- 主动的让异常抛出，中断程序的运行，以便在早期发现程序中的问题，此时，再去处理异常
- 很难判断程序中哪里可能出现异常，特别是有些情况下可以预料到的异常也并不能在运行时处理，例如空指针问题

所以，对于 Runtime 异常来说，不需要，也不应该在程序中主动的处理它们。不过，仍旧坚持想要在运行时捕获并处理这类异常，当然也不会有什么问题，处理方式与 Checked 异常是一样的，即 `try-catch` or `throws`。

## 核心异常类

Java 语言自从诞生至今，已经发展了 20 多年，无数个程序员踩过相同的坑、遇到过同样的问题。所以，就会有一类问题被称为 “经典问题”。把这样的 “场景” 映射到异常，就成为了核心异常类，即经常遇到或者是出现频率很高的异常。

### 核心异常类及其原因分析

之所以要知道并理解这些 “核心异常类”，主要原因有两个：

1. 它们非常经典，分析这些异常有助于帮助理解其他的异常；
2. 它们在日常的工作学习中出现的频率很高。

所以，牢记导致这些异常的原因，能够做到快速响应、快速解决问题。

### NullPointerException

毫无疑问，会经常遇到这个问题：空指针异常。Java 中没有指针的概念，但是 Java 中的引用，也是要指向一个实例对象的（通过 new 方法构造）。从这种意义上说，Java 中的引用与 C/C++ 中的指针没有本质上的区别，不同的是出于安全的目的，在 Java 中不能对引用直接操作，而在 C/C++ 中可以直接对指针进行操作。所以本质上是差不多的，因为引用没有指向具体的实例对象，当访问这个引用的方法或属性的时候就会产生这种异常。

通常出现空指针异常的原因就是访问了 null 对象的方法或属性（这同时也是排查问题的思路），如下示例代码所示：

```java
// 这里并没有对 imoocer 对象进行初始化
Imoocer imoocer = null;
imoocer.setName("老郝");    // 访问 imoocer 的 setName 方法，会抛出 NullPointerException
imoocer.age = 20;          // 访问 imoocer 的 age 属性，会抛出 NullPointerException
```

解决这个问题的方法其实也很简单，在 Java8 之前，可以使用 `if...else` 去判空；在 Java8 中，我们则可以使用 `Optional` 去操作可能为 null 的对象。

### ClassCastException

这个异常就是字面解释：类型转换出错了，它是因为 JVM 在检测到两个类型间转换不兼容时引发的运行时异常。那么，为了说明这类异常，用一个经典的示例来做解释：

```java
// Animal 表示动物，Dog 和 Cat 是 Animal 的子类
Animal a1 = new Dog();
Animal a2 = new Cat();

Dog d1 = (Dog) a1;
Dog d2 = (Dog) a2;  // 这行代码会抛出 ClassCastException
```

从上面的例子看，ClassCastException 是进行强制类型转换的时候产生的异常，强制类型转换的前提是父类引用指向的对象的类型是子类的时候才可以进行强制类型转换，如果父类引用指向的对象的类型不是子类的时候将产生 ClassCastException 异常。

那么，如何在程序中避免这样的问题呢？也就是说，当我不能确定对象的类型时，我可以怎么做呢？两个思路解决你的问题：

- 通过 `o.getClass().getName()` 得到具体的类型，再去处理；
- 通过 `if(o instanceof type)` 来判断 o 的类型是否匹配。

### IndexOutOfBoundsException

索引越界异常，当操作一个字符串或者数组的时候经常遇到的异常，它属于 RuntimeException（即不需要手工捕获）。它常见的形式有 StringIndexOutOfBoundsException和 ArrayIndexOutOfBoundsException，即字符串长度越界和数组越界，通俗的说就是写的代码以为我的字符串或数组有那么长，还对它进行了想当然的操作，但其实没那么长。

最常见的情况就是在循环时使用了 “闭区间”，如下示例所示：

```java
String name = "maya";
int[] nums = {10, 20, 25, 30};
List<Double> doubles = Arrays.asList(1.0, 2.0, 3.0);

for (int i = 0; i <= name.length(); ++i) {
    System.out.println(name.charAt(i));     // StringIndexOutOfBoundsException
}

for (int i = 0; i <= nums.length; ++i) {
    System.out.println(nums[i]);            // ArrayIndexOutOfBoundsException
}

for (int i = 0; i <= doubles.size(); ++i) {
    System.out.println(doubles.get(i));     // ArrayIndexOutOfBoundsException
}
```

这类问题的解决办法只有一个：保证访问的索引不越界！所以，在循环或者直接 index 的时候一定要清楚的知道数据的 “边界” 在哪里。

### IllegalArgumentException

直接英文直译：非法参数异常，它也是在日常的工作学习中最为常见的异常之一。参数指的就是给方法传递的参数，非法参数当然就是说你传递的参数不合法，所以，出现这个异常的原因是：你没有按照方法声明传递正确的参数值。

一个经典的例子是 JDK 中 ArrayList 的构造函数，它允许你传递一个 int 值初始化数组的大小，但是，这个值不能是负数。例如：

```java
new ArrayList<Integer>(-1);     // IllegalArgumentException

// ArrayList 源码
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

所以，这个异常也是比较好解决的，两个方向的建议：

- 在调用方法时（特别是第三方提供的），一定要读一读它的文档，搞清楚参数的取值范围
- 在给别人提供方法时，也是一样，关于参数的合法取值，给出详尽的说明

随着代码量越来越多，可能遇到的异常当然也会越来越多，不过，即使超出了我这里所讲解的范围，也不需要焦虑。异常也就是错误，它代表着程序中存在错误，且异常也会 “大概” 告诉你是什么错误，哪里出现了错误。所以，顺藤摸瓜，先要去找到错误，再去分析可能出错的原因，最后再想办法解决错误。这就是面对 Java 异常的 “套路”。

## 异常类中的重要方法

Throwable 类是 Java 语言中所有错误或异常的超类。只有当对象是此类（或其子类之一）的实例时，才能通过 Java 虚拟机或者 Java throw 语句抛出。类似地，只有此类或其子类之一才可以是 catch 子句中的参数类型。程序在抛出异常时，不论是异常提示信息还是异常栈信息都是由 Throwable 的方法实现的。所以，想要搞清楚异常信息，就一定要知道 Throwable 类中定义的相关方法。

### Throwable 类中的重要方法

到底 Throwable 中的哪些方法才是重要的呢？或者说，哪些需要去掌握的呢？答案其实很简单：日常使用到的就是重要的（不论是在工作还是学习中，能够接触到的技术太多太多，总想着把所有的技术都 “拿下” 是不可能的，最终的结果一定是泛而不精的状态。所以，学习的本质是抓住重点，有了充裕的时间之后再去点缀）。

下面，从源码的角度解释构造方法、getMessage、getLocalizedMessage、toString、printStackTrace 以及 getStackTrace 方法。

#### 构造方法

Throwable 定义了四个构造方法（public 修饰的），它们比较简单，就是用来构造 Throwable 对象的，只是允许你传递不同的参数值。我这里直接给出代码及注释信息：

```java
/** 构造一个将 null 作为其详细消息的新 throwable */
Throwable()
/** 构造带指定详细消息的新 throwable */
Throwable(String message)
/** 构造一个带指定 cause 和 (cause==null ? null : cause.toString())（它通常包含类和 cause 的详细消息）的详细消息的新 throwable */
Throwable(Throwable cause)
/** 构造一个带指定详细消息和 cause 的新 throwable */
Throwable(String message, Throwable cause)
```

#### getMessage

返回当前 Throwable 对象的详细消息字符串。这是一个非常高频率使用的方法，通常我们在 `try catch` 里面，会使用 getMessage 记录下异常的描述信息。我们可以看下这个方法的源码：

```java
public String getMessage() {
    return detailMessage;
}

public Throwable(String message) {
    fillInStackTrace();
    detailMessage = message;
}
```

很显然，getMessage 方法中返回的 detailMessage 就是在构造 Throwable 时指定的（这也是在将来我们自定义异常时的指导）。

#### getLocalizedMessage

这个方法只比 getMessage 多了 “Localized”，它翻译过来的意思是 “本地化的”。getLocalizedMessage 就是加了本地化后的信息的 Message，默认和 getMessage 是一样的，如果要加入本地化信息要重写这个方法。源码如下：

```java
public String getLocalizedMessage() {
    // 这里直接返回了 getMessage 方法，所以，默认它们是相同的
    return getMessage();
}
```

这个方法就类似于方言，如果有一些特殊的标准需要提示出来（在异常抛出之后），那么，定义的异常就需要重写这个方法。大多数时候，应该用的是 getMessage。

#### toString

JDK 源码中对它的解释是：返回此 throwable 对象的简短描述。这个方法返回的信息也是广泛的应用于日志记录中。它的源码，探究一下这个方法返回了什么。

```java
public String toString() {
    String s = getClass().getName();
    String message = getLocalizedMessage();
    return (message != null) ? (s + ": " + message) : s;
}
```

可以看到，这个方法的返回值是两项信息的拼接：异常的类名 + 异常的描述信息。例如，可能会返回 “java.lang.ArithmeticException: / by zero”。这个方法相比于 getMessage 来说，能够让我们知道异常的类型是什么。那么，当你对异常很熟悉的时候（看到异常类型，就能大概的知道是什么原因会导致现在的异常），解决异常就会非常简单了。

#### printStackTrace

这个方法会把当前 throwable 对象及其追踪输出至标准错误流，也就是通常所说的 “异常栈”。这个方法最终会调用到它的一个重载方法，我把它的核心源码贴出来：

```java
private void printStackTrace(PrintStreamOrWriter s) {
    ......
    synchronized (s.lock()) {
        // 看做是 toString 方法，打印异常类名和异常信息
        s.println(this);
        StackTraceElement[] trace = getOurStackTrace();
        // 打印异常栈的跟踪信息
        for (StackTraceElement traceElement : trace)
            s.println("\tat " + traceElement);

        ......
    }
}
```

可以看到，printStackTrace 的核心是 StackTraceElement 对象，也就是说这个对象记录了异常发生的上下文信息。StackTraceElement 数组来自于 getOurStackTrace 方法，而 getStackTrace 方法返回的是 `getOurStackTrace().clone()`，所以，接下来，直接去看 getStackTrace 方法。

#### getStackTrace

其实 getStackTrace 方法的核心是它的返回值：`StackTraceElement[]`，毕竟异常栈中打印的信息就是来自于 StackTraceElement 的。StackTraceElement 的定义如下：

```java
public final class StackTraceElement implements java.io.Serializable {
    private String declaringClass;
    private String methodName;
    private String fileName;
    private int    lineNumber;

    ......
}
```

原来异常栈信息是记录在 StackTraceElement 对象中的。针对于 Java 中的异常轨迹，每一条路径都会有一个对应的 StackTraceElement 对象。所以，异常栈最后的呈现是一个数组（可以通过在 catch 中捕获异常，并使用 getStackTrace 方法得到异常轨迹数组，并探寻其中的信息）。

总之，很多时候 “会用” 是不够的，需要知道更多细节。不一定说什么都懂，也并不现实，但是，一些基础的、重要的，一定需要牢记在心。只有这样，才能够在出错的时候快速的定位并解决问题，最终达成的效果就是提升你的个人能力。能力强，说到底就是解决问题的能力强。

## 总结

###  Java 异常类结构

- Throwable 类是所有异常和错误的基类，它直接继承于 Object 类
- Error 和 Exception 是 Throwable 的子类，它们的区别是：
  - Exception 是一种我们应该对其进行捕获或者抛出的异常
  - Error 由 Java 虚拟机抛出，是相对严重的错误，我们不应该对它进行捕获。如果出现了Error，程序会终止
- RuntimeException 是 Exception 的子类，我们可以叫它运行时异常。它的出现是因为我们程序设计不当造成的
- Exception 及其他的子类被称作 checked exception，它们必须被声明或者捕获

### 核心类与核心方法

言归正传，核心类是我们的日常工作、学习中最常遇到的异常类：

- NullPointerException：空指针异常
- ClassCastException：类型转换异常
- IndexOutOfBoundsException：索引越界异常
- ClassNotFoundException：类加载失败异常
- IllegalArgumentException：非常参数异常

那么，当程序抛出了这些异常，要能够迅速的知道是什么问题，并跟着异常栈的方向及时定位并解除问题。异常发生之后，异常信息以及异常栈都是由 Throwable 中的方法打印的。所以，还需要知道异常类的核心方法：

- 构造方法
- getMessage
- getLocalizedMessage
- toString
- printStackTrace
- getStackTrace

### 异常信息代表什么

通常情况下，异常描述会告诉你：为什么会出现异常；异常栈会告诉你：异常出现的路径是怎样的。所以，面对异常信息，我们能够：

- 根据异常描述知道是什么原因导致了当前异常的发生
- 根据异常栈定位异常发生的位置
- 结合异常描述和异常栈解决异常

下面贴出一段代码中出现异常之后，程序打印的异常信息：

```java
java.lang.ArithmeticException: / by zero
	at com.imooc.log.stack._Throwable.division(_Throwable.java:13)
	at com.imooc.log.stack._Throwable.main(_Throwable.java:28)
```

### Java 对异常的处理

在 Java 中，处理异常有两种方式：

- `try...catch...finally` 捕获异常
- throws 声明抛出异常（在当前方法中不处理，throws 异常抛给调用者处理）

这里所说的对于异常的处理，是 Java 语言的规范，即规定这样做。不仅仅是针对于 Java 语言中定义的标准异常，对于自定义的异常也是一样。但是，当自定义异常时，一定要清楚的知道有没有必要，为什么使用标准异常不好等等。