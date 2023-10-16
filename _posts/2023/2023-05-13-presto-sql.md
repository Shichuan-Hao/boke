---
title: Presto SQL
date: 2023-05-13 14:31:57 +0800
categories: [文章]
tags: [Presto] 
---


## Presto 语句
在使用 Presto 深入查询数据之前，你要知道哪些数据可用、在什么地方、是什么类型。你可以通过 Presto 语句（Presto statement）收集这些信息。

Presto 语句可以查询系统表和源数据，以获取 catalog 和 schema 等信息。这些语句与所有 SQL 语句在相同的上下文工作。

语句中的 FROM 和 FOR 子句需要输入一个完全限定的表、catalog 或 schema，除非之前使用 USE 设置了默认值。

LIKE 子句可以用来限制结果，其使用的 schema 匹配语法类似与 SQL 中的 LIKE 命令。
`[ ]`中的命令部分是可选的。
下面是 Presto 可用的一些语句：

| 语句 | 作用 |
| --- | --- |
| SHOW CATALOGS [ LIKE _pattern_ ] | 列出可用的 catalog |
| SHOW SCHEMAS [ FROM _catalog_ ] [ LIKE _pattern_ ] | 列出 catalog 中的 schema |
| SHOW TABLES [ FROM _schema _] [ LIKE _pattern_ ] | 列出一个 schema 中的表 |
| SHOW FUNCTIONS | 显示所有可用 SQL 函数的列表 |
| SHOW COLUMNS FROM _table_ 或 DESCRIBE _table_ | 列出表中的列及其数据类型和其他属性 |
| USE catalog._schema_ 或 use _schema_ | 更新会话以使用指定的 catalog 和 schema 作为默认值。如果没有指定 catalog，则是用当前 catalog 来解析 schema |
| SHOW STATS FOR _table_name_ | 显示某个表中的数据规模和数量等统计信息 |
| EXPLAIN | 生成查询计划，并详细列出各个步骤 |

下面是常用的例子：
```shell
presto> SHOW SCHEMAS IN tpch LIKE '%3%';
 Schema  
---------
 sf300   
 sf3000  
 sf30000 
(3 rows)

Query 20230803_062629_00042_qqw2z, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 238ms, server-side: 222ms] [10 rows, 119B] [45 rows/s, 536B/s]

presto> DESCRIBE tpch.tiny.nation;
  Column   |     Type     | Extra | Comment 
-----------+--------------+-------+---------
 nationkey | bigint       |       |         
 name      | varchar(25)  |       |         
 regionkey | bigint       |       |         
 comment   | varchar(152) |       |         
(4 rows)

Query 20230803_062653_00043_qqw2z, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 240ms, server-side: 217ms] [4 rows, 260B] [18 rows/s, 1.17KB/s]
```

EXPLAIN 语句非常强大，完整语法如下；
```sql
EXPLAIN [ ( option [, ...] ) ] <query>
    options: FORMAT { TEXT | GRAPHVIZ | JSON}
             TYPE { LOGICAL | DISTRIBUTED | IO | VALIDATE }
```
可以使用 EXPLAIN 语句来显示查询计划：
```sql
presto> EXPLAIN SELECT name FROM tpch.tiny.region;
                                                                   Query Plan                                                                   
------------------------------------------------------------------------------------------------------------------------------------------------
 - Output[name] => [name:varchar(25)]                                                                                                           
         Estimates: {rows: 5 (59B), cpu: 59.00, memory: 0.00, network: 59.00}                                                                   
     - RemoteStreamingExchange[GATHER] => [name:varchar(25)]                                                                                    
             Estimates: {rows: 5 (59B), cpu: 59.00, memory: 0.00, network: 59.00}                                                               
         - TableScan[TableHandle {connectorId='tpch', connectorHandle='region:sf0.01', layout='Optional[region:sf0.01]'}] => [name:varchar(25)] 
                 Estimates: {rows: 5 (59B), cpu: 59.00, memory: 0.00, network: 0.00}                                                            
                 name := tpch:name (1:26)                                                                                                       
                                                                                                                                                
(1 row)

Query 20230803_074439_00044_qqw2z, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
[Latency: client-side: 199ms, server-side: 182ms] [0 rows, 0B] [0 rows/s, 0B/s]

```
这些计划信息能够帮助你进行性能调优，从而更好地理解 Presto 如何处理查询。
EXPLAIN 的一个非常简单的使用场景是检查查询在语法上是否争取，如下：
```sql
presto> EXPLAIN (TYPE VALIDATE) SELECT name FROM tpch.tiny.region;
 Valid 
-------
 true  
(1 row)

Query 20230803_074826_00045_qqw2z, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
[Latency: client-side: 142ms, server-side: 129ms] [0 rows, 0B] [0 rows/s, 0B/s]
```

## Presto 系统表
Presto 系统表不需要配置 catalog 文件，所有的 schema 和表都在 system catalog 中自动提供。可以用上面提到的语句来查询 schema 和表，来了解更多关于 Presto 运行实例的信息，比如可用的数据包括运行时、节点、catalog 等。这些信息可以帮助你更好地理解和使用 Presto,.
