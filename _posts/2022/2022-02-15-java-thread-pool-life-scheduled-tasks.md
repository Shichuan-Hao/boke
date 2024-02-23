---
title: java 线程池的定时执行流程
author: Shaun
date: 2022-01-24 11:49:00 +0800
categories: [博客]
tags: [java thread pool]
math: true
mermaid: true
---

> 基于 Java 8 版本
{: .prompt-tip }

> 线程池源码指的是 ScheduldeThreadPoolExecutor 类
{: .prompt-tip }

## 简介

**定时任务**是我们经常会用到的一种任务，它表示在未来某个时刻执行，或者未来按照某种规则重复执行的任务。

## 问题

1. 如何保证任务是在未来某个时刻才执行 ？

2. 如何保证任务按照某种规则重复执行 ？

## 例子

创建一个定时线程池，用来它跑四种不同的定时任务：

```java
public 
```



