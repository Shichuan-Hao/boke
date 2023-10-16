---
title: Presto Kafka 连接器
date: 2023-06-11 14:31:57 +0800
categories: [文章]
tags: [Presto] 
---


### 概述
此连接器允许使用 Apache Kafka topics 作为 Presto 中的表。每条消息在 Presto 中显示为一行。
topics 可以是实时的：行将在数据到达时出现，并在消息删除时消失。如果在单个查询中多次访问同一个表（例如，执行自连接），这可能会导致奇怪的行为。
支持 Apache Kafka 2.3.1+

### 配置
要配置 Kafka 连接器，请创建一个 `etc/catalog/kafka.properties`包含以下内容的目录属性文件，并根据需要替换属性：
```properties
connector.name=kafka
kafka.table-names=table1,table2
kafka.nodes=host1:port,host2:port
```
#### 多个 Kafka 集群
您可以根据需要拥有任意数量的目录，因此，如果您有其他 Kafka 集群，只需添加另一个etc/catalog 具有不同名称的属性文件（确保其以 结尾 .properties）。例如，如果您将属性文件命名为 sales.properties，Presto 将创建一个 sales 使用配置的连接器命名的目录。

## 配置属性
以下配置属性可用：

| 配置项 | 描述 |
| --- | --- |
| kafka.table-names | 目录提供的所有表格的列表 |
| kafka.default-schema | 表的默认模式名称 |
| kafka.nodes | Kafka 集群中的节点列表 |
| kafka.connect-timeout | 连接 Kafka 集群超时 |
| kafka.max-poll-records | 每次轮询的最大记录数 |
| kafka.max-partition-fetch-bytes | 每次轮询来自一个分区的最大字节数 |
| kafka.table-description-dir | 包含主题描述文件的目录 |
| kafka.hide-internal-columns | 控制内部列是否是表架构的一部分 |

