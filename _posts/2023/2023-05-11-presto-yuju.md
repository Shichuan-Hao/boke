---
title: Presto 语句
date: 2023-05-11 14:31:57 +0800
categories: [文章]
tags: [Presto] 
---


整理 Preso 所支持 SQL 的细节、能力，包括：

- 一组数据定义语言（data definition language，DDL），它们主要用于创建和操作数据库对象，如 schema、表、列和视图
- 数据类型
- 操作符
- 函数
> 可以通过使用 Presto ClI 或任何使用 JDBC 或 ODBC 驱动的应用程序来使用 SQL

>      在 Presto 中执行的所有操作都依赖数据源的连接器及其对特定命令的支持。如果连接数据源的连接器不支持 DELETE 语句，执行 DELETE 就会失败。
>      此外，创建的连接通常使用特定的用户或其他授权机制，即存在特定的限制。如果用户没有读取数据以外的权限，或者甚至只能从特定的 schema 中读取数据，那么其他操作（如删除数据或写入数据）就会执行失败。


### Presto 语句
在使用 Presto 深入查询数据之前，您需要知道哪些数据可用、在什么地方、是什么类型很重要。
您可以通过** Presto 语句（Presto statement）**收集这些信息。Presto 语句可以查询系统表和源数据，以获取 catalog 和 schema 等信息。这些语句与所有 SQL 语句在相同的上下文工作。
语句中的 FROM 和 FOR 子句需要输入一个完全限定的表，catalog 或 schema，除非之前使用 USE 设置了默认值。
LIKE 子句可以用来限制结果，其使用的 schema 匹配语法类似于 SQL 中的 LIKE 命令。
下面是 Presto 可用的一些语句：

- `** show catalogs [ like pattern ]**** **` 列出可用的 catalogs 
- `** show schemas [ from|in catalog ] [ like pattern ] **`** **列出 catalog 中的 schema
- `** show tables [ from|in schema ] [ like pattern ] **` 列出 schema 中的表
- `** show functions **` 显示所有可用 SQL 函数的列表
- `** show columns from|in table 或 describe table **` 列出表中的列及其数据类型和其他属性
- `** use catalog.schema 或 use schema **` 更新会话，以使用指定的 catalog 和 schema 作为默认值。如果没有指定 catalog，则使用当前 catalog 来解析 schema
- `** show stats for table_name **` 显示某个表中的数据规模和数量等统计信息
- `** explain **`生成查询计划，并详细列出各个步骤

如下图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692770016933-e634db53-5c46-436d-aabe-2f21e4f7fd06.png#averageHue=%23060402&clientId=u8217b70f-a183-4&from=paste&height=367&id=u0096dadc&originHeight=367&originWidth=619&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23027&status=done&style=none&taskId=uab21856f-73a6-49a7-bb07-fec13281d64&title=&width=619)

explain 语句非常强大，完整语法如下：
```sql
explain [ ( option [, ... ] ) ] <query>
    option: format { text | graphviz | json }
            type { logical | distributed | io | validate }
```
您可以使用 explain 语句来显示查询计划：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692784621896-f94b552c-9f24-4b37-bf05-47f37eca8610.png#averageHue=%23060403&clientId=u8217b70f-a183-4&from=paste&height=255&id=ue7273894&originHeight=255&originWidth=1025&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24361&status=done&style=none&taskId=u4887201c-e3ab-48a9-b58d-1c2b4defa31&title=&width=1025)
这些计划信息能帮助您进行性能调优，并更好地理解 Presto 如何处理查询。
**explain 的一个非常简单的使用场景是检查查询在语法上是否正确：**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692784753594-96d318a4-3dcd-49d9-810c-deb7116acad1.png#averageHue=%23030201&clientId=u8217b70f-a183-4&from=paste&height=164&id=ued0641bc&originHeight=164&originWidth=1044&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10450&status=done&style=none&taskId=u3f45ac2b-efe0-49d1-b6aa-e45a9485d00&title=&width=1044)

### Presto 系统表
Presto 系统表不需要配置 catalog 文件，所有的 schema 和表都在 system catalog 中自动提供。
可以用上一小节提到的语句来查询 schema 和表，来了解更多关于 Presto 运行实例的信息。
> Presto Web UI 是一个基于网页的用户界面，它显示了系统表的相关信息，具体以后再研究

系统表包含的 schema：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692785036582-627e9567-f9cd-442b-b433-88622be11f5c.png#averageHue=%23050403&clientId=u8217b70f-a183-4&from=paste&height=188&id=u9243f6b5&originHeight=188&originWidth=602&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10793&status=done&style=none&taskId=u7c16a00f-150a-4d7d-af16-a39d94af368&title=&width=602)
当需要进行查询调优，表 `system.runtime.queries` 和 `system.runtime.tasks` 是最有用的：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692785099362-61aa827a-6968-4769-9cf1-0a709bacdc51.png#averageHue=%23060402&clientId=u8217b70f-a183-4&from=paste&height=311&id=u47493911&originHeight=311&originWidth=660&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19754&status=done&style=none&taskId=uaecbff2d-7f52-4f36-a11b-716d0ea3087&title=&width=660)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692837703105-52e7c3de-9fe7-4eeb-b0de-0925094848e0.png#averageHue=%23060403&clientId=u8217b70f-a183-4&from=paste&height=481&id=uc36590e0&originHeight=481&originWidth=635&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31214&status=done&style=none&taskId=ud8134f78-2ae8-4a43-a045-4320dd735fe&title=&width=635)

- `system.runtime.queries`表提供了 Presto 中当前和历史查询的信息
- `system.runtime.tasks`表提供了 Presto 中任务的底层细节

这些与 Presto Web UI 中 Query Detail 页面（查询详情页）的信息输出类似。
下面是几个查询系统表的有用例子：

- `**select * from system.runtime.nodes;**`  列出 Presto 集群中的节点
- `**select * from system.runtime.queries where state='FAILED';**` 显示所有失败的查询
- `**select * from system.runtime.queries where state='RUNNING';**` 显示所有运行中查询
- `**call system.runtime.kill_query(query_id => 'queryId', message => 'Killed';)**` 终结运行中的查询

除了 Presto 运行时、集群以及工作节点等信息外，Presto 连接器还能够暴露其连接的数据源中的系统数据。【**** 开始 **** 此部分需要再深入研究】例如，可以再 datalake catalog 中配置使用 Hive 连接器，它可以自动地将 Hive 的系统表暴漏出来：`show tables from datalake.system`，结果中包含了诸如已用分区等方面的信息。【此部分需要再深入研究 **** 结束 ****】

### catalog
Presto catalog 代表了在 catalog 属性文件中使用连接器配置的数据源。
一个 catalog 包含一个或多个 schema ，而 schema 提供了表的集合。
例如，您可以配置个 PostgreSQL catalog 来访问 PostgreSQL 上的关系数据库；也可以个配置一个 JMX catalog，通过 JMX 连接器对 JMX 信息进行访问；还可以通过使用 Hive 连接器的 catalog 连接到 HDFS 对象存储数据源。当在 Presto 中运行一个 SQL 语句时，您是在一个或多个 catalog 上运行它。

多个 catalog 可以使用同一个连接器。例如，您可以创建两个独立的 catalog 来暴露两个运行在同一服务器 PostgreSQL 数据库。

当在 Presto 中寻找一个表时，完全限定表名总是以 catalog 为根。比如，完全限定表名 hive.test_data.test 指的是 hive catalog、test_data schema 中的 test 表。

可以通过访问系统数据来查看 Presto 服务器中的可用 catalog 列表：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692842569327-b33b39d9-918d-488f-9675-14c058869922.png#averageHue=%23040302&clientId=ub2947df9-4ce3-4&from=paste&height=291&id=uc393883b&originHeight=291&originWidth=561&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14480&status=done&style=none&taskId=u769e8e02-ab1d-4e24-9999-a10d7c2f97f&title=&width=561)
catalog、schema 的表信息不由 Presto 存储，Presto 也没有自己的 catalog。连接器负责向 Presto 提供这些信息。
通常情况下，这是通过从底层数据库查询 catalog 或通过连接器的其他配置来完成，连接器处理这些请求，在收到请求后返回相关信息。

### schema
Presto 的 catalog 包含 schema。schema 包含表、视图和其他各种对象，是组织表的一种方式。
catalog 和 schema 共同定义了一组可以查询的表。

当用 Presto 访问关系数据库时，schema 转化为目标数据库中的相同概念。而其他类型的连接器可能选择以底层数据源有意义的方式将表组织成 schema。

连接器的实现决定了 schema 在 catalog 中的映射方式。例如，Hive 中的数据库在 Presto 的 Hive 连接器是以 schema 的形式暴露出来的。

通常情况下，当你配置一个 catalog 时，schema 已经存在，但是 Presto 也允许创建  schema 和其他 schema 操作。

```sql
create schema [ if not exists ] schema_name
[ with ( property_name = expression [, ...] ) ]
```
with 子句可用于将属性关联到 schema。比如，对于 Hive 连接器，创建一个 schema 实际上就是在 Hive 中创建一个数据库。有时候，需要覆盖 `hive.metastore.warehouse.dir` 指定的数据库的默认位置：
```sql
CREATE SCHEMA hive.web
WITH (location = 's3://example-org/web/')
```
获取可用的 schema 属性列表，可以参考 Presto 的最新文档，或者在 Presto 中执行以下查询：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692870957347-b94fb9da-66c2-4980-9f22-445af4ba9de6.png#averageHue=%230a0705&clientId=ua5fa0fb0-519d-4&from=paste&height=144&id=u7f6870d0&originHeight=144&originWidth=634&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12798&status=done&style=none&taskId=u6d9eb4b5-5018-4188-b0bb-95bf9a67605&title=&width=634)
更改现有 schema 的名称：`alter schema name rename to new_name`
删除一个 schema ：`drop schema [ if exists ] schema_name`
如果不想让语句在 schema 不存在的情况出错时，可以指定 if exists。在成功删除 schema 之前，您需要删除其中的表。有些数据库系统支持 cascade 关键字，表示用 drop 语句将对象（如 schema）内的所有东西都删除。但是 Presto 现阶段并不支持 cascade。

### Information Schema
Information Schema 是 SQL 标准的一部分，并在 Presto 中作为一组视图！提供了关于 catalog 中的 schema、表、列、视图和其他对象的元数据。这组视图包含在一个名为 information_schema 的 schema 中。

每个 Presto catalog 都有自己的 information_schema，像 show tables、show schema 等命令的结果与 Information schema 中检索到的信息相同。
> Information Schema 对商业智能工具等第三方工具来说必不可少，其中的许多工具会查询 information schema，以便知道有哪些对象存在

在每个连接器中，Information Schema 都有 8 个视图。对于某些不支持的功能（例如角色）的连接器，查询对应的 Information Schema 可能会产生一个不支持的错误。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692929559115-16216eb7-30bd-4250-befb-802d20cdbd18.png#averageHue=%23050302&clientId=ua5fa0fb0-519d-4&from=paste&height=249&id=ub1a234b1&originHeight=249&originWidth=614&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15172&status=done&style=none&taskId=udef5a85e-69e2-4883-89a5-4d6fc66b1ba&title=&width=614)
> ‼️ 注意，当通过 Information Schema 查询 schema 中的表时，也会返回 Information Schema 自身拥有的表。

当然，也可以使用 where 子句来查询特定表的列：`select table_catalog, table_schema, table_name, column_name from hive.information_schema.columns where table_name = 'nation';`

### 表
表是一组无序的行，它将数据组织成具有特定数据类型的命名列。与关系数据库中的表一样，Presto 的表由行、列和这些列的数据类型所组成。从源数据到表的映射是由 catalog 定义的。

连接器的实现决定了表如何映射到 schema 。例如，PostgreSQL 中的表是直接暴露在 Presto 中的，因为 PostgreSQL 原生支持 SQL 和表的概念。不过，要实现与其他系统的连接器，特别是当它们在设计上缺乏严格的表概念时，需要更多的创造力。比如，Apache Kafka 连接器将 Kafka topic 暴露为 Presto 中的表。

在 SQL 查询中，使用完全限定名（catalog-name.schema-name.table-name）来访问表。
```sql
CREATE TABLE [ IF NOT EXISTS ]
table_name (
 { column_name data_type [ COMMENT comment ]
 [ WITH ( property_name = expression [, ...] ) ]
 | LIKE existing_table_name [ { INCLUDING | EXCLUDING } PROPERTIES ] }
 [, ...]
)
[ COMMENT table_comment ]
[ WITH ( property_name = expression [, ...] ) ]
```
在 Presto 中，可选的 WITH 子句有一个重要的用途：

- 其他的系统（如 Hive）扩展了 SQL 语言，使用户可以使用那些无法用标准 SQL 表达的逻辑或数据。遵循相同的方法违反了 Presto 尽可能地接近 SQL 标准的理念，也导致难以管理多种不同的连接器。因此 Presto 选择使用 with 语句指定表和列的属性，而不是扩展 SQL 语言。

表创建之后，就可以使用标准 SQL 中的 insert into 语句。

比如：
```sql
INSERT INTO iris (
 sepal_length_cm,
 sepal_width_cm,
 petal_length_cm,
 petal_width_cm,
 species )
VALUES
 ( ... )
```
如果数据时通过单独的查询获得的，可以在 insert 中使用 select。假如把数据从 memory catalog 中复制到 PrestgreSQL 中已有一张表里：
```sql
INSERT INTO postgresql.flowers.iris SELECT * FROM memory.default.iris;
```
select 语句可以包括条件语句以及任何其他支持的语句。

#### 表和列的属性
通过 Hive 连接器来创建一个表，以了解 with 子句的用法。
![Hive 连接器支持的表属性](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692931755131-ee131369-a31b-4cd2-8636-d6c812b60d78.png#averageHue=%23f0eed8&clientId=ua5fa0fb0-519d-4&from=paste&height=84&id=u9c79ac3a&originHeight=84&originWidth=640&originalType=binary&ratio=1&rotation=0&showTitle=true&size=23482&status=done&style=none&taskId=u7e68c617-53cf-490f-908a-14de2f14d1a&title=Hive%20%E8%BF%9E%E6%8E%A5%E5%99%A8%E6%94%AF%E6%8C%81%E7%9A%84%E8%A1%A8%E5%B1%9E%E6%80%A7&width=640 "Hive 连接器支持的表属性")
Hive 的语法：
```sql
CREATE EXTERNAL TABLE page_views(
 view_time INT,
 user_id BIGINT,
 page_url STRING,
 view_date DATE,
 country STRING)
 STORED AS ORC
 LOCATION 's3://example-org/web/page_views/';
```
与 Presto 中的 SQL 相比较：
```sql
CREATE TABLE hive.web.page_views(
 view_time timestamp,
 user_id BIGINT,
 page_url VARCHAR,
 view_date DATE,
 country VARCHAR
)
WITH (
 format = 'ORC',
 external_location = 's3://example-org/web/page_views'
);
```
可以看到，Hive DDL 扩展了 SQL 标准，而 Presto 使用属性达成同样的目的，因此符合 
SQL 标准。 
你可以查询 Presto 的系统元数据来列出可用的表属性： `SELECT * FROM system.metadata.table_properties; `
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692932112217-8a9836f1-6f54-4bd3-8c42-de662bc7bf8f.png#averageHue=%23080504&clientId=ua5fa0fb0-519d-4&from=paste&height=382&id=u7f6fc2e8&originHeight=382&originWidth=1292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50177&status=done&style=none&taskId=u13f7e938-3a09-487b-a7f1-b18cc6c385e&title=&width=1292)
要列出可用的列属性，可以运行下面的查询： `SELECT * FROM system.metadata.column_properties;`
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1692932161881-054c2c91-80ab-464a-8513-e1960082944b.png#averageHue=%23090604&clientId=ua5fa0fb0-519d-4&from=paste&height=131&id=u34e90259&originHeight=131&originWidth=597&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9427&status=done&style=none&taskId=u1cb3b3e8-f839-42ec-8a70-a0635731924&title=&width=597)
#### 复制现有的表
可以将现有的表作为模板来新建一个表。LIKE 子句会创建一个与现有表具有相同列定义的
