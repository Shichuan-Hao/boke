---
title: GeoTools Quickstart
date: 2023-11-21 09:47:12 UTC+8
categories: [Java, Library]
tags: [java, geo tools, open source java library, opsj]     # TAG names should always be lowercase
# author: 1
---

欢迎来到您的第一个 GeoTools 项目！我们将使用 GeoTools 设置一个项目，以便在屏幕上快速显示 shapefile。

本教程适用于：
1. [Eclipse Quickstart](https://docs.geotools.org/latest/userguide/tutorial/quickstart/eclipse.html)
2. [Netbeans Quickstart](https://docs.geotools.org/latest/userguide/tutorial/quickstart/netbeans.html)
3. [IntelliJ Quickstart](https://docs.geotools.org/latest/userguide/tutorial/quickstart/intellij.html)
4. [Maven Quickstart](https://docs.geotools.org/latest/userguide/tutorial/quickstart/maven.html)
5. [Java Modules](https://docs.geotools.org/latest/userguide/tutorial/quickstart/modules.html)


## 说明和勘误表
- 如果您在 OS X 下运行，可能会遇到文件选择器对话框从未出现且应用程序挂起的问题。这是 OS X Swing 原生外观的一个已知问题。作为一种变通方法，你可以使用跨平台外观；或者在文件中使用以下静态块：
```java
static {
  // Set System L&F
  try {
      UIManager.setLookAndFeel(
          UIManager.getCrossPlatformLookAndFeelClassName());
  } catch (Exception e) {
      e.printStackTrace();
      System.exit(1);
  }
}
```
或使用虚拟机参数运行应用程序 `Dswing.defaultlaf=javax.swing.plaf.metal.MetalLookAndFeel`

- 在撰写本文时，如果您尝试第二次更改投影，CRSLab 应用程序可能会失败。目前还没有解决方法或修复方案。

## IntelliJ Quickstart

### 前提条件

本指南将帮助您设置 IntelliJ IDE 以使用 GeoTools，并跟进 GeoTools 教程的其他部分。
- 您已安装了最新的 JDK（本文撰写时为 8）。如果没有，Eclipse 快速入门将提供如何安装的说明。
- 您已安装 IntelliJ。本文以 IntelliJ CE 2016 为目标；不过，至少早在 13 之前的版本也可以正常运行。Ultimate 版本也可以正常运行。IntelliJ 可从 JetBrains 下载，一般在普通操作系统上开箱即用。

### 创建一个新的项目
首先，我们将使用 Maven 快速启动原型创建一个新项目。

未完..待续


