---
title: Activiti 数据库设计和模型映射
date: 2023-12-27 10:30:55 UTC+8
categories: [博客]
tags: [java, Activiti]     # TAG names should always be lowercase
# author: 1
---

Activiti 支持多种主流关系数据库：
- DB2
- MySQL
- Oracle
- ...

Activiti 表命名规则如下：
- 表名由以<font color=red>下划线 _</font>连接的部分组成。
- 第一部分固定<font color=red>下划线 _</font>开头。
- 第二部分表示用途的两个字母标识，用途与服务的 API 对应。
- 第三部分表示存储的内容。
如：<font color=red>ACT_HI_PROCINST</font> 表示 Activiti 的历史流程实例表。

根据 Activiti 表的命名规则，可以将 Activiti 的数据表分为 5 大类：
- 通用数据表：存放流程或业务使用的通用资源数据，以 ACT_GE_ 为前缀，GE 表示 general.
- 流程存储表：存放流程定义文件和部署信息等，以 ACT_RE_ 为前缀，RE 表示 repository.
- 身份数据表：存放用户、组及关联关系等身份信息，以 ACT_ID_ 为前缀，ID 表示 identity.
- 运行时数据表：存放流程执行实例、任务、变量等流程运行过程中产生的数据，以 ACT_RU_ 为前缀，RU 表示 runtime.
- 历史数据表：存放历史流程实例、变量和任务等历史记录，以 ACT_HI_ 为前缀，HI 表示 history.



