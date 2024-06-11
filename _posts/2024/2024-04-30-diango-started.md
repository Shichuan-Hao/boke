---
title: Diango 入门（一）
date: 2024-04-30 09:26:57 +0800
categories: [博客]
tags: [python, diango]
---

> 随着互联网的发展，网站与移动应用程序之间的界限不再清晰，它们能够让用户以各种方式与数
据交互。幸运的是，可以使用 Django 来创建能同时作为动态网站和移动应用程序的项目。Django 
是最流行的 Python Web 框架，提供了一系列旨在帮助开发交互式网站的工具。<br/>
本文开始使用 Diango 来开发一个名为 “学习笔记”（Learning Log）的项目。这是一个在线日志
系统，可以让用户能够记录针对哪些特定主题学到了哪些知识。
{: .prompt-tip }


针对此项目，首要做的事情是为此项目指定规范，然后为使用的数据定义模型。在这里，我将使用 
Diango 的管理系统来输入一些初始数据，然后编写视图和模板，从而让 Django 能够创建网页。

Django 能够响应网页请求，还让我们能够更轻松地读写数据库、管理用户等等...最终将其部署到
活动的服务器上，让所有人能够使用它。


## 建立项目

在着手开发像 Web 应用这样的大项目，首先需要制定 **规范（spec）**，对项目的目标进行描述。
确定要达成的目标侯，就能着手找出为达成这些目标而需要完成的任务了。

本文将为 “学习笔记”项目制定规范，并进入项目开发的第一个阶段，包括搭建虚拟环境以及构建
Django 项目框架。

### 指定规范

完整的规范要详细说明项目的目标，阐述项目的功能，讨论项目的外观和用户界面。与任何良好的项
目规划书和商业计划书一样，规范应突出重点，帮助避免项目偏离轨道。当然，这里不制定完整的项
目规划，只列出一些明确的目标，以突出开发的重点。

规范如下：

> 编写一个名为“学习笔记”的 Web 应用程序，让用户能够记录感兴趣的主题，并在学习每个主题
的过程中添加日志条目。“学习笔记”的主页对这个网站进行描述，并邀请用户注册或登录。用户登
录后，可以创建新主题、添加新条目以及阅读既有的条目。

在学习新主题时，记录学到的知识可有助于建立知识体系，研究技术主题时尤其如此。优秀的应用
程序（如接下来将创建的应用程序）能让这个记录过程简单高效。

### 建立虚拟环境

使用 Django 之前，需要建立虚拟的工作环境。**虚拟环境**是系统的一个位置，我们可在其中
安装包，并将这些包与其他 Python 包隔离开来。将项目的库与其它项目分离是有益的（这也是将
最终的 “学习笔记” 部署到服务器，这也是必须的）。

为项目新建一个目录，并将其命名欸 learning_log，再在终端中切换到这个目录，并执行如下命
令创建一个虚拟环境：