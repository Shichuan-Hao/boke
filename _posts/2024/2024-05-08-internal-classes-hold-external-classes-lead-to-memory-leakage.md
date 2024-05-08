---
title: Java 内部类持有外部类导致的内存泄露的解决方案
date: 2024-05-01 10:51:57 +0800
categories: [博客]
tags: [java, internal class, external class, memory leakage]
---

## 简介

### 说明

整理 Java 内部类持有外部类导致内存泄漏的原因以及其解决方案。

> 为什么内部类持有外部类会导致内存泄漏 ？<br/>
非静态内部类会持有外部类，如果有地方引用了这个非静态内部类，会导致外部类也被引用，垃圾
回收时无法回收这个内部类（即使外部类已经没有其它地方在使用了）。
{: .prompt-tip }

### 解决方案

不要让其它的地方持有这个非静态内部类的引用，直接在这个非静态内部类执行业务。

将非静态内部类改为静态内部类。内部类改为静态的之后，它所引用的对象或属性也必须是静态的，
所以静态内部类无法获得外部对象得引用，只能从 JVM 的 Method Area（方法区）获取到static
类型的引用。


## 非静态内部类持有外部类的原因

Java 语言中，非静态内部类的主要作用（原因）有两个：
- 当内部类只在外部类中使用时，匿名内部类可以让外部不知道它的存在，从而减少了代码的维护
工作。
- 当内部类持有外部类时，它就可以直接使用外部类中的变量了，这样可以很方便的完成调用。

如下代码所示：

```java
package org.example;

class Outer {

    private String outerName = "Tony";

    class Inner {
        private String name;

        public Inner() {
            this.name = outerName;
        }
    }

    Inner createInner() {
        return new Inner();
    }

}

public class Demo {
    public static void main(String[] args) {
        Outer.Inner inner = new Outer().createInner();
        System.out.println(inner);
    }
}
```

但是，静态内部类就无法持有外部类和其非静态字段了，比如下边这样就会报错：
![静态内部类持有外部类和其非静态字段报错](https://shichuan-hao.github.io/images/java/basic/static-class-hold-external-class-error.png)
_静态内部类持有外部类和其非静态字段报错_

## 实例：持有外部类

直接见代码：

```java
package org.example;

class Outer {
    class Inner {
    }

    Inner createInner() {
        return new Inner();
    }

}

public class Demo {
    public static void main(String[] args) {
        Outer.Inner inner = new Outer().createInner();
        System.out.println(inner);
    }
}
```

经过端点调试可以看到：内部类持有外部类的对象的引用，是以“this$0”这个字段来保存的。
![内部类持有外部类的对象的引用，是以“this$0”这个字段来保存的](https://shichuan-hao.github.io/images/java/basic/internal-class-hold-external-class-refernces.png)
_内部类持有外部类的对象的引用，是以“this$0”这个字段来保存的_

## 实例：不持有外部类

```java
package org.example;

class Outer {

    static class Inner {
    }

    Inner createInner() {
        return new Inner();
    }

}

public class Demo {
    public static void main(String[] args) {
        Outer.Inner inner = new Outer().createInner();
        System.out.println(inner);
    }
}
```

经过端点调试，可以发现内部类不再持有外部类了。
![内部类不再持有外部类了](https://shichuan-hao.github.io/images/java/basic/static-internal-class-not-hold-external-class-refernces.png)
_内部类不再持有外部类了_


## 实例：内存泄漏

若内部类持有外部类的引用，对内部类的使用很多时，会导致外部类数目很多。此时，就算是外部
类的数据没有被用到，外部类的数据所占空间也不会被释放。

```java
package org.example;

import java.util.ArrayList;

class Outer {

    private int[] data;
    
    public Outer(int size) {
        this.data = new int[size];
    }
    class Inner {
    }

    Inner createInner() {
        return new Inner();
    }

}

public class Demo {
    public static void main(String[] args) {
        ArrayList<Object> objects = new ArrayList<>();
        int count = 0;
        while (true) {
            objects.add(new Outer(100000).createInner());
            System.out.println(count++);
        }
    }
}
```

经过测试可以看到，运行了八千多次的时候就内存溢出了：
![内部类持有外部类的对象的引用导致的内存泄漏](https://shichuan-hao.github.io/images/java/basic/internal-class-hold-external-class-solution.png)
_内部类持有外部类的对象的引用导致的内存泄漏_

## 不会内存泄漏的方案

内部类改为静态的之后，它所引用的对象或属性也必须是静态的，所以静态内部类无法获得外部对
象的引用，只能从 JVM 的 Method Area（方法区）获取到 static 类型的引用。

```java
package org.example;

import java.util.ArrayList;

class Outer {

    private int[] data;

    public Outer(int size) {
        this.data = new int[size];
    }
    static class Inner {
    }

    Inner createInner() {
        return new Inner();
    }

}

public class Demo {
    public static void main(String[] args) {
        ArrayList<Object> objects = new ArrayList<>();
        int count = 0;
        while (true) {
            objects.add(new Outer(100000).createInner());
            System.out.println(count++);
        }
    }
}
```

经过测试可以发现，循环成千上万次都没有内存溢出。
![内部类持有外部类的对象的引用导致的内存泄漏](https://shichuan-hao.github.io/images/java/basic/internal-class-hold-external-class-solution.png)
_内部类持有外部类的对象的引用导致的内存泄漏_