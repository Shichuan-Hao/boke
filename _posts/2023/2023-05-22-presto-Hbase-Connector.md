---
title: Presto Hbase 连接器
date: 2023-05-22 14:31:57 +0800
categories: [文章]
tags: [Presto, Hbase] 
---


[GitHub - analysys/presto-hbase-connector: presto hbase connector 组件基于Presto Connector接口规范实现，用来给Presto增加查询HBase的功能。相比其他开源版本的HBase Connector，我们的性能要快10到100倍以上。](https://github.com/analysys/presto-hbase-connector)

该组件基于 Presto Connector 接口规范实现，用于为 Presto 添加查询 HBase 的能力。
我们的性能比其他开源版本的 HBase Connector 快 10 到 100 倍。

## 性能比较
| 数据大小 |  事件表包含500万条记录和90个字段 |
| --- | --- |
| 节点 | 3 |
| 硬件 | 16个逻辑核心 64GB内存（Presto和HBase分别为16GB内存） 4T*2硬盘 |

[https://github.com/analysys/public-docs/blob/master/](https://github.com/analysys/public-docs/blob/master/)附件一(表结构信息).sql

## 功能点对比
## Environment

1. Mac OS X or Linux
2. 8u161+, 64-bit.
3. Maven 3.3.9+
4. PrestoSql 315+


