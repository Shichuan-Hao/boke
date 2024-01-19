---
title: 一次 SpringBoot 工程的编写和调试过程
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2018-06-09 21:33:11 +0800
categories: [常用框架, SpringBoot]
tags:  [SpringBoot Coding and debug]
math: true
mermaid: true
---

以一个 SpringBoot 工程（之所以选择 SpringBoot 工程，是因为它是当前最主流的选择）将如下方面串联起来：

- 异常的排查与处理；
- logback 日志配置；
- JVM 性能调优；
- 利用堆栈日志定位代码问题；
- AOP 切面打印请求日志；
- 全局统一异常的捕获与处理。

当然，该工程只是对一些问题、通用做法的解释说明，所以，这是一个 demo 性质的工程，而不是某个业务的解决方案实现。

## SpringBoot 工程功能设计

该工程名字叫做：`log-stack-springboot`，主要围绕 Thinker 来做一些工作。Thinker 是一个 Java POJO，它的定义如下：

工程中所有的功能都是对 Thinker 来进行处理，包含有：

- 创建 Thinker 
- 查看 Thinker 的信息
- 审核 Thinker 的头像是否符合要求
- 定时任务，生成关于 Thinker 的 Excel 报表

这些功能虽说非常的简单，但是，它对本工程的目标以及足够。此外，可以在此基础之上，对它进行二次开发，实现你想要实现的功能。

## JVM 环境配置

关于部署时配置 JVM 参数！首先，应该搞清楚需要配置什么：

- 堆空间，对年轻代和整堆空间进行配置；
- 垃圾收集器，由于还没有 GC 日志，所以，选择使用最常见的 ParNew + CMS 即可。不过，记得打开 GC 日志；
- 栈深度、元空间直接采用默认值就可以了

所以，所谓 JVM 环境配置，目前只需要去考虑堆空间。先来看一份 Java Performance 中推荐的原则：

| 空间    | 命令行选项                 | 配置                                    |
| ------- | -------------------------- | --------------------------------------- |
| Java 堆 | Java 堆                    | 3 ~ 4 倍 Full GC 后的老年代空间占用量   |
| 年轻代  | -Xmn                       | 1 ~ 1.5 倍 Full GC 后的老年代空间占用量 |
| 老年代  | = Java 堆空间 - 年轻代空间 | = Java 堆空间 - 年轻代空间              |

需要特别注意的是：JDK8 之后把永久代移除了（`-XX:PermSize` 和 `-XX:MaxPermGen`），取而代之的是元空间（Metaspace）：`-XX:MetaspaceSize`（元空间默认大小）和 `-XX:MaxMetaspaceSize`（元空间最大大小）。

另外，Sun 官方建议堆空间大小设置为机器内存空间的 1/4（还要考虑机器中其他程序的开销），年轻代的大小为整堆的 3/8 左右。Sun 并没有说明这样配置的依据，不过，应该是根据大量采样调优之后得出的一个结论。那么，工程在启动之初，不妨就遵从这一配置。最终，得到了 JVM 配置如下（我的机器是 8G 内存）：

```bash
-Xmn768M -Xms2048M -Xmx2048M
-XX:SurvivorRatio=8 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps
```

- `-Xmn768M`：年轻代大小为 768M，计算公式为 8G * (1/4) * (3/8) = 768M
- `-Xms2048M -Xmx2048M`：初始堆与最大堆大小为 2G（机器内存的 1/4），相同的设置使得 GC 之后 JVM 不去对堆做调整，提高响应
- `-XX:SurvivorRatio=8`：Eden 区与 Survivor 区的大小比值，设置为 8，则两个 Survivor 区与一个 Eden 区的比值为 2:8，一个 Survivor 区占整个年轻代的 1/10
- `-XX:+UseParNewGC -XX:+UseConcMarkSweepGC`：垃圾收集器设定，年轻代使用 ParNew，老年代使用 CMS
- `-XX:+PrintGCDetails -XX:+PrintGCTimeStamps`：打印 GC 详细信息，并带有时间信息

做好了 JVM 配置之后，自然就有了以下的工程启动命令：

```bash
java -Xmn768M -Xms2048M -Xmx2048M -XX:SurvivorRatio=8 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -jar xxx.jar
```

## 基于 logback 配置优雅的工程日志

日志是一个可用系统的核心模块，合理的日志配置会让工程日志变得优雅，即业务逻辑的查询、审计、错误排查都将会变得非常简单。SpringBoot 默认使用 logback 日志框架，因此也就遵循 SpringBoot 的建议。

### 结合业务需要去考虑日志配置

对工程进行日志配置，首要要考虑需求是什么，即日志能够带来的“收益”。例如：

- 需要使用日志做审计工作，可以把审计日志单独存储；
- 需要方便排查问题/异常，可以考虑把 info 与 error 分开等等。

显然，我的 SpringBoot 工程的日志需求（同时，也会做一些扩展性的说明），如下：

- 日志级别定义为 info（忽略大小写），即低于 info 级别的日志不会被记录到文件中；
- 日志的内容需要包含日期时间（日志文件记录日期，日志内容记录时间）、线程、日志级别、日志打印类、行数以及日志内容，这其实就是对 pattern 的要求；
- 日志文件按照时间滚动，一天生成一个日志文件（这需要看具体业务场景，如果是高并发的工程，请求量很大，日志很多，可以考虑按照小时去滚动）；
- 为了便于查看日志、分析异常，需要将 info 与 error 日志分开
  - info 存储在 logs/info 目录中，业务逻辑日志，可以用来回溯业务流程；
  - error 存储在 logs/error 目录中， 只是为了排查解决问题，让 info 日志保留时间大于 error 日志。
- 其他的一些框架日志单独配置输出级别，都只是为了调试，只在控制台输出。另外，不要输出到父级 logger
  - `org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping` 配置为 debug
  - `org.springframework.orm.jpa.JpaTransactionManager` 配置为 debug
  - `org.hibernate.type.descriptor.sql.BasicBinder` 配置为 trace

以上日志需求，相对来说，比较简单的，没有过于复杂的配置（实际上，99% 的工程日志配置都不复杂），几乎可以应用于任何一个 web 工程上。当然，可以根据需要做一些扩展，例如，把不同的业务日志输出到不同的日志文件中。

### logback.xml 配置

有了需求就好办了，只要按照需求来定义 “节点” 就好。在给出示例 `logback.xml` 之前，需再回想下`<logger>` 与 `<root>` 标签（这两个标签是 logback 配置的核心，也是理解 logback 的难点）。

`<logger>` 用来设置某一个包或者具体某一个类的日志打印级别、以及指定 `<appender>`。`<logger>` 可以包含零个或者多个 `<appender-ref>` 元素，标识这个 `appender` 将会添加到这个 `logger`。`<logger>` 仅有一个 name 属性、一个可选的 level 属性和一个可选的 additivity 属性：

- name：指定此 logger 约束的某一个包或者具体的某一个类
- level：设置打印级别，五个常用打印级别从低至高依次为 TRACE、DEBUG、INFO、WARN、ERROR，如果未设置此级别，那么当前 logger 会继承上级的级别
- additivity：是否向上级 logger 传递打印信息，默认为 true

`<root>` 也是 `<logger>` 元素，但是它是根 logger，只有一个 level 属性，因为它的 name 就是 ROOT。可以在源码 `ch.qos.logback.classic.LoggerContext` 中找到答案：

```java
public LoggerContext() {
    super();
    this.loggerCache = new ConcurrentHashMap<String, Logger>();

    this.loggerContextRemoteView = new LoggerContextVO(this);
    this.root = new Logger(Logger.ROOT_LOGGER_NAME, null, this);
    this.root.setLevel(Level.DEBUG);
    loggerCache.put(Logger.ROOT_LOGGER_NAME, root);
    initEvaluatorMap();
    size = 1;
    this.frameworkPackages = new ArrayList<String>();
}
```

再看下 `ch.qos.logback.classic.Logger` 的构造函数：

```java
Logger(String name, Logger parent, LoggerContext loggerContext) {
    this.name = name;
    this.parent = parent;
    this.loggerContext = loggerContext;
}
```

第一个参数就是 Root 的 name，而这个 `Logger.ROOT_LOGGER_NAME` 的定义为 `final public String ROOT_LOGGER_NAME = "ROOT"`，由此可以看出 `<root>` 节点的 name 就是 “ROOT”。

最后，logback.xml 文件如下（文件中已经给出了详细的注释，其他的就不再赘述了）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

<!-- 控制台 appender, 几乎是默认的配置 -->
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <encoder charset="UTF-8">
        <!-- 输出的日志文本格式, 其他的 appender 与之相同 -->
        <pattern>[%X{REQUEST_ID}] %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} %L - %msg%n</pattern>
        <charset>UTF-8</charset>
    </encoder>
</appender>

<!-- info 级别的 appender -->
<appender name="happymaya_info" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!-- 日志写入的文件名, 可以是相对目录, 也可以是绝对目录, 如果上级目录不存在会自动创建 -->
    <file>./logs/info/log-stack.log</file>
    <!-- 如果是 true, 日志被追加到文件结尾; 如果是 false, 清空现存文件. 默认是true -->
    <append>true</append>
    <!-- 日志级别过滤器, 只打 INFO 级别的日志-->
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>INFO</level>
        <!-- 下面2个属性表示: 匹配 level 的接受打印, 不匹配的拒绝打印 -->
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <!-- 最常用的滚动策略, 它根据时间来制定滚动策略, 既负责滚动也负责触发滚动 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- 设置滚动文件规则, 如果直接使用 %d, 默认格式是 yyyy-MM-dd -->
        <fileNamePattern>./logs/info/log-stack.%d{yyyy-MM-dd}.log</fileNamePattern>
        <!-- 保留14天的日志 -->
        <maxHistory>14</maxHistory>
    </rollingPolicy>
    <!-- 定义日志输出格式 -->
    <encoder charset="UTF-8">
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} %L - %msg%n</pattern>
        <charset>UTF-8</charset>
    </encoder>
</appender>

<!-- error 级别的 appender -->
<appender name="happymaya_error" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>./logs/error/log-stack.log</file>
    <append>true</append>
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>./logs/error/log-stack.%d{yyyy-MM-dd}.log</fileNamePattern>
        <!-- 保留7天的日志 -->
        <maxHistory>7</maxHistory>
    </rollingPolicy>
    <!-- 定义日志输出格式 -->
    <encoder charset="UTF-8">
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} %L - %msg%n</pattern>
        <charset>UTF-8</charset>
    </encoder>
</appender>

<!-- 指定 com.happymaya.log.stack 下的日志打印级别, appender -->
    <!-- 上线之前修改成 info, 否则线上会打印切面日志 -->
<!--<logger name="com.happymaya.log.stack" level="info" additivity="false">-->
<logger name="com.happymaya.log.stack" level="debug" additivity="false">
    <appender-ref ref="stdout"/>
    <appender-ref ref="happymaya_info"/>
    <appender-ref ref="happymaya_error"/>
</logger>

<logger name="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" level="debug"
        additivity="false">
    <appender-ref ref="stdout"/>
</logger>

<logger name="org.springframework.orm.jpa.JpaTransactionManager" level="debug"
        additivity="false">
    <appender-ref ref="stdout"/>
</logger>

<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="trace"
        additivity="false">
    <appender-ref ref="stdout"/>
</logger>

<!-- 根 logger -->
<root level="info">
    <appender-ref ref="stdout"/>
    <appender-ref ref="happymaya_info"/>
    <appender-ref ref="happymaya_error"/>
</root>

</configuration>
```

## 总结

也许在工作中几乎没有写过日志配置，都是去复制、粘贴其他的工程，往往问题不大，也会让工程工作的很好。但是，搞清楚这里面的一切，再去结合自身业务完成一份属于 “自己的” 日志配置无疑是更优的。