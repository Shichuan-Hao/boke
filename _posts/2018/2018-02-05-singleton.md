---
title: 单例设计模式：有效进行程序初始化 
author:
  name: superhsc
  link: https://github.com/Shichuan-hao
date: 2018-02-05 16:45:00 +0800
categories: [设计模式, 创建型]
tags:  [design pattern]
math: true
mermaid: true
---

# 单例模式分析
在 GoF 中，单例模式最早的定义如下：
> 单例模式（Singleton）允许存在一个和仅存在一个给定类的实例。它提供一种机制让任何实体都可以访问该实例。

对应的 UML 图，如下
![单例模式的 UML](https://Shichuan-hao.github.io/images/assert/design-patterns/singleton-uml.jpg)

上图中，单例模式（Singleton）类声明了一个名为 instance 的静态对象和名为 get­Instance() 的静态方法，静态对象用来存储对象自身的属性和方法，静态方法用来返回其所属类的一个相同实例。

以单例模式经典的懒汉式初始化方式为例，其代码实现如下：
```java
package cn.happymaya.ndp.creational.singleton;

/* final 不允许被继承 */
public class LazyManSingleton {

    /* 在定义实例对象的时候直接初始化 */
    private static LazyManSingleton instance = null;

    /* 私有构造方法，不允许外部 new */
    private LazyManSingleton(){}

    private static LazyManSingleton getInstance() {
        if (null == instance) {
            instance = new LazyManSingleton();
        }
        return instance;
    }

}
```
由此，可以得出单例模式包含三个要点：
- 一个单例类只能有一个实例；
- 单例类必须自行创建这个实例；
- 单例类必须保证全局其他对象都能唯一访问到它。

显然，这三个要点是单例模式需要应对的变化，就是：
- 对象实例数量受到限制的事实；
- 对象实例的构造与销毁；
- 需要保证对象实例成为“线程安全”的某种机制。

从上面那段示例代码还可以看出，单例模式的对象职责有两个：
- 保证一个类只有一个实例；
- 为该实例提供一个全局访问节点。

单例类的默认构造函数和静态对象都是内部调用，之所以将默认构造函数设为私有，是为了防止其他对象使用单例类的 new 运算符。然后，提供一个对外的公共方法来获取唯一的对象实例。在我看来，单例模式就类似于全局变量或全局函数的角色，可以使用它来代替全局变量。

常见场景和解决方案
**单例模式更多是在程序一开始进行初始化时使用的。**

常见单例模式应用和使用的解决方案有：
- 饿汉式初始化
- 懒汉式初始化
- 同步信号
- 双重锁定
- 使用 ThreadLocal

看下 ThreadLocal 的方式，比如，下面这个 AppContext 代码示例：
```java
package cn.happymaya.ndp.creational.singleton;

import java.util.HashMap;
import java.util.Map;

public class AppContext {

    private static final ThreadLocal<AppContext> local = new ThreadLocal<>();

    private Map<String, Object> data = new HashMap<>();
    
    
    public Map<String, Object> getData() {
        return getAppContext().data;
    }
    
    // 批量存储数据
    public void setData(Map<String, Object> data) {
        getAppContext().data.putAll(data);
    }
    
    // 存数据
    public void set(String key, String value) {
        getAppContext().data.put(key, value);
    }
    // 取数据
    private void get(String key) {
        getAppContext().data.get(key);
    }
    
    // 初始化的实现方法
    private static AppContext init() {
        AppContext context = new AppContext();
        local.set(context);
        return context;
    }
    // 做延迟初始化
    public static AppContext getAppContext() {
        AppContext context = local.get();
        if (null == context) {
            context = init();
        }
        return context;
    }
    // 删除实例
    public static void  remove() {
        local.remove();
    }
}
```
上面的代码实现，实际上是懒汉式初始化的扩展，只不过用 ThreadLocal 替换静态对象来存储唯一对象实例。之所会选择 ThreadLocal，就是因为 ThreadLocal 相比传统的线程同步机制更有优势。

在传统的同步机制中，通常会通过对象的锁机制来保证同一时间只有一个线程访问单例类。这时该类是多个线程共享的，我们都知道使用同步机制时，什么时候对类进行读写、什么时候锁定和释放对象是有很烦琐要求的，这对于一般的程序员来说，设计和编写难度相对较大。

**而 ThreadLocal 则会为每一个线程提供一个独立的对象副本，**从而解决了多个线程对数据的访问冲突的问题。正因为每一个线程都拥有自己的对象副本，也就省去了线程之间的同步操作。

所以说，**现在绝大多数单例模式的实现基本上都是采用的 ThreadLocal 这一种实现方式。**

# 为什么使用单例模式

1. **系统某些资源有限。**比如:
   1. 控制某些共享资源（像数据库或文件这些）的访问权限。资源有限会带来访问冲突的问题，如果不限制实例的数量，有限的资源就会很快耗尽，同时造成大量对象处于等待资源中;
   2. 同时读写一个超大的 AI 模型文件，或使用外部进程使服务，如果不适用单例模式，随着用户进程数开启越多，系统原有进程资源就会变得越少，这不仅会导致操作系统处理速度变慢，还会造成用户进行自身处理速度缓慢;
2. **需要表示为全局唯一的对象。**比如:
   1. 系统要求提供一个唯一的序列号生成器。客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。在一个系统中要求一个类只有一个实例时才应当使用单例模式;
   2. 反过来，如果一个类可以有几个实例共存，就需要对单例模式进行改进，使之成为多例模式。

# 优缺点
优点：
1. 对有限资源的合理利用，保护有限的资源，防止资源重复竞抢；
2. 更高内聚的代码组件，能提升代码复用性；
3. 具备全局唯一访问点的权限控制，方便按照统一规则管控权限；
4. 从负载均衡角度考虑，可以轻松地将 Singleton 扩展成两个、三个或更多个实例。由于封装了基数问题，所以在适当的时候可以自由更改实例的数量。

缺点：
1. 作为全局变量使用时，引用的对象越多，代码修改影响的范围也就越大；
2. 作为全局变量，在全局变量中使用状态变量，会造成加/解锁的性能损耗；
3. 即便能扩展多实例，耦合性依然很高，因为屏蔽了不同对象之间的调用关系；
4. 不支持有参数的构造函数。

>  Spring 单例 Bean 与单例模式的区别：
>  - 单例模式，是指在一个 JVM 进程中有且仅有一个实例；\
>  - 单例 Bean，是指在一个 Spring 容器中有且仅有一个实例。
{: .prompt-tip }


# 单例设计模式的七种实现方式

## 饿汉式

```java
/* final 不允许被继承 */
public final class HungryManSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 在定义实例对象的时候直接初始化 */
    private static HungryManSingleton instance = new HungryManSingleton();

    /* 私有构造方法，不允许外部 new */
    private HungryManSingleton(){}

    private static HungryManSingleton getInstance() {
        return instance;
    }

}
```

关键点：instance 作为类变量，直接得到了初始化。

如果主动使用 Singleton 类，那么 instance 实例将会直接完成创建，包括其中的实例变量都会得到初始化，例如 1K 空间的 data 将会同时被创建。

instance 作为类变量在类初始化的过程中，会被收集进 `<clinit>()` 方法中，该方法能够百分之百保证同步，也就是说在多线程的情况下可能被实例化两次，但是 instance 被 ClassLoader 加载后可能很长一段时间才被使用，意味着 instance 实例所开辟的堆内存会驻留更久时间。

倘若类中的成员属性比较少，并且占用内存资源不多，饿汉式也未尝不可，相反，倘若类中的成员都是比较重的资源，那么这种方式就不太妥。

总之，饿汉式的单例模式可以保证多线程下的唯一实例，getInstance 方法性能也比较高，但是无法进行懒加载。

## 懒汉式

```java
/* final 不允许被继承 */
public class LazyManSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 在定义实例对象的时候直接初始化 */
    private static LazyManSingleton instance = null;

    /* 私有构造方法，不允许外部 new */
    private LazyManSingleton(){}

    private static LazyManSingleton getInstance() {
        if (null == instance) {
            instance = new LazyManSingleton();
        }
        return instance;
    }

}
```

关键点：在使用类实例的时候，再去创建（用时创建），这样可以避免类在初始化时提前创建。

Singleton 的类变量 instance = null，因此当 Singleton.class 被初始化的时候 instance 并不会被实例化；

在 getInstance 方法，会先判断 instance 实例是否被实例化，看起来没有什么问题，但是将 getInstance 方法放在多线程环境下进行分析，则会导致 instance 被实例化一次以上，不能保证单例的唯一性，如下图。

![this is images](http://processon.com/chart_image/629cf1bbe0b34d3bc3b267b0.png#crop=0&crop=0&crop=1&crop=1&id=rHAbm&originHeight=721&originWidth=1576&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

总之，懒汉式的单例模式可以保证实例的懒加载，但无法保证实例的唯一性。

## 懒汉式 + 数据同步方法

在多线程的情况下，instance 成为共享资源（数据），当多个线程对其访问使用时，需要保证数据的同步性，只要对懒汉式的代码上稍加修改，增加同步的月约束就可以，修改后的代码如下：

```java
/* final 不允许被继承 */
public class LazyManSyncSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 在定义实例对象的时候直接初始化 */
    private static LazyManSyncSingleton instance = null;

    /* 私有构造方法，不允许外部 new */
    private LazyManSyncSingleton(){}

    /* 向 getInstance 方法加入同步控制，每次只能有一个线程能够进入 */
    private static synchronized LazyManSyncSingleton getInstance() {
        if (null == instance) {
            instance = new LazyManSyncSingleton();
        }
        return instance;
    }
}
```

懒汉式 + 数据同步的方式，既满足懒加载，又能够百分之百地保证 instance 实例的唯一性，又由于 synchronized 关键字天生的排他性，导致 getInstance() 方法只能在同一时刻被一个线程所访问，性能低下。

## Double-Check

```java
/* final 不允许被继承 */
public class DoubleCheckSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 在定义实例对象的时候直接初始化 */
    private static DoubleCheckSingleton instance = null;

    Connection connection;

    Socket socket;

    /* 私有构造方法，不允许外部 new */
    private DoubleCheckSingleton() {
        this.connection = null;    // 初始化 connection
        this.socket = null;        // 初始化 socket
    }

    private static DoubleCheckSingleton getInstance() {
        // 当 instance 为 null 时，进入同步代码块
        // 同时该判断避免了每次都需要进入同步代码块，可以提高些效率
        if (null == instance) {
            // 只有一个线程能够获取 DoubleCheckSingleton.class 关联的 monitor
            synchronized (DoubleCheckSingleton.class) {
                // 判断如果 instance 为 null ，则创建
                if (null == instance) {
                    instance = new DoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

Double-Check ，提供了一种高效的数据同步策略，就是首次初始化时加锁，之后允许多个线程同时进行 getInstance 方法的调用来获得类的实例。

当两个线程发现 null=instance 成立时，只有一个线程有资格进人同步代码块，完成对 instance 的实例化；

随后的线程发现 null=instance 不成立则无须进行任何动作，以后对 getInstance 的访问就不需要数据同步的保护了。

这种方式看起来是那么的完美和巧妙，既满足了懒加载，又保证了 instance 实例的唯一性，Double-Check 的方式提供了高效的数据同步策略，可以允许多个线程同时对 getInstance 进行访问，但是这种方式在多线程的情况下有可能会引起空指针异常，原因是：在 Singleton 的构造函数中，需要分别实例化 conn 和 socket 两个资源，还有 Singleton 自身，根据 JVM 运行时指令重排序和 Happens--Before 规则，这三者之间的实例化顺序并无前后关系的约束，那么极有可能是 instance 最先被实例化，而 conn 和 socket 并未完成实例化，未完成初始化的实例调用其方法将会抛出空指针异常，如下：

![this is images](http://processon.com/chart_image/629cf8586376895abf210648.png#crop=0&crop=0&crop=1&crop=1&id=ICqNA&originHeight=1092&originWidth=1441&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## Volatile + Double-Check

Double-Check 还有可能引起类成员变量的实例化 conn 和 socket 发生在 instance 实例化之后，这一切都是由 JVM 在运行时指令重排序所导致的。

volatile 关键字则可以防止这种重排序的发生，因此稍作修改就可以满足多线程下的单例：

```java
/* final 不允许被继承 */
public class VolatileDoubleCheckSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 在定义实例对象的时候直接初始化 */
    private volatile static VolatileDoubleCheckSingleton instance = null;

    Connection connection;

    Socket socket;

    /* 私有构造方法，不允许外部 new */
    private VolatileDoubleCheckSingleton() {
        this.connection = null;    // 初始化 connection
        this.socket = null;        // 初始化 socket
    }

    private static VolatileDoubleCheckSingleton getInstance() {
        // 当 instance 为 null 时，进入同步代码块
        // 同时该判断避免了每次都需要进入同步代码块，可以提高些效率
        if (null == instance) {
            // 只有一个线程能够获取 DoubleCheckSingleton.class 关联的 monitor
            synchronized (DoubleCheckSingleton.class) {
                // 判断如果 instance 为 null ，则创建
                if (null == instance) {
                    instance = new VolatileDoubleCheckSingleton();
                }
            }
        }
        return instance;
    }
}
```

## Holder 方式

关键点： 借助了类加载的特点。

```java
/* final 不允许被继承 */
public class HolderSingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    /* 私有构造方法，不允许外部 new */
    private HolderSingleton(){}

    /* 在静态内部类中持有 Singleton 的实例，并且可能直接初始化 */
    private static class Holder {
        private static HolderSingleton instance = new HolderSingleton();
    }

    /* 调用 getInstance 方法，事实上是获得 Holder 的 instance 静态属性 */
    public static HolderSingleton getInstance() {
        return Holder.instance;
    }
}
```

在 HolderSingleton 类中，没有 instance 的静态成员，而是将其放到了静态内部类 Holder 之中，因此在HolderSingleton 类的初始化过程中并不会创建 Singleton 的实例。

Holder 类中定义了 HolderSingleton 的静态变量，并且直接进行了实例化，当 Holder 被主动引用的时候则会创建 HolderSingleton 的实例，Singleton 实例的创建过程在 Java 程序编译时期收集至`<clinit>O`方法中，该方法又是同步方法，因此可以保证内存的可见性、JVM 指令的顺序性和原子性。

Holder方式的单例设计是最好的设计之一，也是目前使用比较广的设计之一。

## 枚举方式

使用枚举的方式实现单例模式是《Effective Java》中力推的方式，在很多优秀的开源代码中经常可以看到使用枚举方式实现单例模式的（身影)。

枚举类型不允许被继承，同样是线程安全的且只能被实例化一次，但是枚举类型不能够懒加载，对 Singleton 主动使用，比如调用其中的静态方法则 INSTANCE 会立即得到实例化.

```java
/* 枚举类型本身是 final 的，不允许被继承 */
public enum EnumSingleton {

    INSTANCE;

    // 实例变量
    private byte[] data = new byte[1024];


    EnumSingleton(){
        System.out.println("INSTANCE will be initialized immediately...");
    }

    public static void method() {
        // 调用该方法，则会主动使用 Singleton，INSTANCE 将会被实例化
    }

    public static EnumSingleton getInstance() {
        return INSTANCE;
    }

}
```

在对它进行改造，增加懒加载的特性，类似于 Holder  的方式，改进后的代码如下：

```java
/* 枚举类型本身是 final 的，不允许被继承 */
public class EnumLazySingleton {

    /* 实例变量 */
    private byte[] data = new byte[1024];

    private EnumLazySingleton(){}

    /* 使用枚举充当 holder */
    private enum EnumHolder{
        INSTANCE;

        private EnumLazySingleton instance;

        EnumHolder() {
            this.instance = new EnumLazySingleton();
        }

        private EnumLazySingleton getEnumLazySingleton() {
            return instance;
        }
    }

    public static EnumLazySingleton getInstance() {
        return EnumHolder.INSTANCE.getEnumLazySingleton();
    }
}
```

## 总结

虽然单例设计模式简单，但在多线程的情况下，设计单例程序未必就能满足单实例、懒加载以及高性能。优先 Holder 和枚举方式来设计单例。
