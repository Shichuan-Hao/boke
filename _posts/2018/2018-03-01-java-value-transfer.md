---
title: Java 的值传递
author:
  name: superhsc
  link: https://github.com/Shichuan-hao
date: 2018-02-24 21:11:22 +0800
categories: [Java]
tags:  [值传递]
math: true
mermaid: true
---


## 形参 & 实参

在定义方法的时候可能会用到参数（有参数的方法），参数在程序语言中分为：
- __实参（实际参数，Arguments）__ ：用来传递给函数或方法的参数，必须有确定的值。
- __形参（形式参数，Prameters）__：用来定义函数或方法，接收实参，不需要有确定的值。

如下图：
<!-- ![形参 & 实参](https://images.happymaya.cn/static/java/picture_name.png) -->

## 值传递 & 引用传递

程序设计语言将实参传递给方法（或函数）的方式分两种：
- __值传递__：方法接收的是实参值的拷贝，会创建副本。
- __引用传递__：方法接收的直接是实参所引用的对象在堆中的地址，不会创建副本，对形参的修改影响到实参。
很多程序设计语言（比如 C++、Pascal）提供了两种参数传递的方式，不过，在 Java 中只有值传递。

## 为什么说 Java 只有值传递 ？


### 例子 1：传递基本类型参数

__代码：__
```java
public static void main(String[] args) {
  int num1 = 10;
  int num2 = 20;
  
  swap(num1, num2);
  
  System.out.println("num1 = " + num1);
  System.out.println("num2 = " + num2);

}

private static void swap(int a, int b) {
  int temp = a;
  a = b;
  b = temp;
  System.out.println("a = " + a);
  System.out.println("b = " + b);
}
```

__输出：__
```plain
a = 20
b = 10
num1 = 10
num2 = 20
```

__解析：__ 

> 在 `swap()` 方法中，`a`、`b` 的值进行交互，不会影响到 `num1`、`num2`。因为 `a`、`b` 的值是从 `num1`、`num2` 复制过来（即 `a`、`b` 是 num1、num2 的副本），所以无论副本的内容怎么修改都不会影响到原件的值。 
{: .prompt-info }


### 例子 2：传递引用类型参数 1

__代码：__
```java
public static void main(String[] args) {
  int[] arr = {1, 2, 3, 4, 5, 6};
  
  System.out.println(arr[0]);

  change(arr);

  System.out.println(arr[0]);

}

private static void change(int[] arr) {
  // 将数组的第一个元素变为 0
  arr[0] = 0;
}
```

__输出：__
```plain
1
0
```


__解析：__ 

> 看到这个案例很多人肯定认为 Java 对引用类型的参数采用的引用传递。<br/> 
实际上，并不是的，这里传递的依旧是值，只不过这个值是实参的地址！<br/>
也就是说 `change()` 方法的参数拷贝的是 `arr`（实参）的地址，因此，它和 `arr` 指向的是同一数组对象，这也就说明了为什么方法内部对形参的修改会影响到实参。<br>
{: .prompt-info }

下面 __例子 3__ 可以更强有力地反驳 Java 对引用类型的参数采用的不是引用传递！

## 例子 3：传递引用类型参数 2

__代码：__
```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    
}

public static void main(String[] args) {
  Person zhangsan = new Person();
  zhangsan.setName("张三");

  Person lisi = new Person();
  lisi.setName("李四");
  
  swap(zhangsan, lisi);
  
  System.out.println("zhangsan: " + zhangsan.getName());
  System.out.println("lisi：" + lisi.getName());
}

private static void swap(Person person1, Person person2) {
  Person temp = person1;
  person1 = person2;
  person2 = temp;
  System.out.println("person1：" + person1.getName());
  System.out.println("person2：" + person2.getName());
}
```

__输出：__
```plain
person1：李四
person2：张三
zhangsan: 张三
lisi：李四
```


__解析：__ 

> 很明显，`swap()` 方法的参数 `person1` 和 `person2` 只是拷贝的实参 `zhangsan` 和 `lisi` 的地址。因此，`person1` 和 `person2` 的互换只是拷贝的两个地址的互换罢了，并不会影响实参 `zhangsan` 和 `lishi`。 
{: .prompt-info }


## 引用传递时怎么样的 ？

综上，Java 中只有值传递，没有引用传递。但是，引用传递的真面目可以通过 C++ 代码看到：

```c++
#include <iostream>

using namespace std;

void incr(int& num) {
    cout << "incr before: " << num << endl;
    num++;
    cout << "incr after: " << num << endl;
}

int main() {
    int age = 10;
    cout << "invoke before: " << age << endl;
    incr(age);
    cout << "invoke after: " << age << endl;
    return 0;
}
```
输出的结果：
```plain
invoke before: 10
incr before: 10
incr after: 11
invoke after: 11
```

__解析：__ 

> 可以看到，在 `incr()` 函数中对形参的修改，可以影响到实参的值。需要注意的是：这里的 `incr()` 形参的数据类型用的是 `int&`(C++ 用 `&` 符号标识变量类型为 __引用__ ) 才是引用传递，如果是用 `int` 的话还是值传递。
{: .prompt-info }


## 为什么 Java 不引入引用传递 ？

> Java 实际上采用了引用传递的概念，但是在传递时传递的是引用的值，而不是引用本身。这是因为 Java 的设计目标之一是简化内存管理和提高程序的安全性。<br/><br/>
引入引用传递可能导致一些潜在的问题，例如更难管理内存、更容易出现悬垂引用（dangling references）和更难追踪和调试代码。Java 的设计者选择了通过值传递引用的方式来解决这些问题。<br/><br/>
在 Java 中，当你传递一个对象给方法时，实际上是传递了对象引用的值。这意味着方法内部可以修改对象的状态，但无法修改原始引用，也就是无法使原始引用指向不同的对象。这有助于避免一些潜在的错误和提高代码的安全性。<br/><br/>
总的来说，Java 选择了以安全和简化内存管理为优先的设计理念，采用值传递引用的方式。这并非是不支持引用传递，而是一种权衡取舍的设计决策。
{: .prompt-tip }

## 总结

Java 中将实参传递给方法（或函数）的方式是 __值传递__。
- 当参数类型是基本类型，传递的是该参数的字面量值的拷贝，会创建副本；
- 当参数类型是引用类型，传递的是实参所引用的对象在堆中地址值的拷贝，同样也会创建副本。
