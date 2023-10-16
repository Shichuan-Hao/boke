---
title: Presto 的使用
date: 2023-05-10 14:31:57 +0800
categories: [文章]
tags: [rvm, ruby, ubuntu, openssl] 
---


### Presto CLI
Presto CLI 提供了一个基于终端的交互式 shell，可以通过它运行查询，并与 Presto 服务端交互来查看数据。
正如 Presto 服务端一样，Presto CLI 也是通过 Maven 中央仓库分发二进制包。这个 CLI 应用程序以可执行 JAR 包的形式提供，可以像普通 UNIX 可执行程序一样使用它。
Presto CLI 可用版本的列表：[https://repo.maven.apache.org/maven2/io/prestosql/presto-cli/](https://repo.maven.apache.org/maven2/io/prestosql/presto-cli/)
找到与您使用的 Presto 服务端版本相一致的 CLI 版本，从版本目录中下载对应的 *-executable.jar 文件，然后将其重命名为 presto。具体步骤如下。
```shell
# 使用 wget 下载 Presto CLI
$ wget -O presto \
https://repo.maven.apache.org/maven2/\
io/prestosql/presto-cli/330/presto-cli-330-executable.jar

# 确保文件具有可执行权限(此处《Presto 权威指南》书上错了，)
$ chmod +x presto
$ mkdir ~/bin/
$ mv presto ~/bin
$ export PATH=~/bin/:$PATH

# 运行 Presto CLI 并确认其版本
$ presto --version
# 查看所有可用的选项和命令的文档
$ presto --help
```

在开始用 CLI 之前，需要决定与哪个 Presto 服务端交互。在默认情况下，CLI 连接到运行在 `http://localhost:8080` 上的 Presto 服务端。如果 Presto 服务端在本地运行（出于测试或开发目的）或者通过 SSH 连接到了服务器，又或者你使用 exec 命令连接到安装了 CLI 的 Docker 容器，则可以直接执行以下命令：
```shell
$ presto
presto >
```
如果 Presto 服务端运行在另一台服务器上，则需要手动指定 URL：
```shell
$ presto --server https://presto.example.com:8080
presto>
```
`presto>`命令行提示符表明你在使用交互式控制台访问 Presto 服务端。键入 help 命令来获取可用命令的列表：
```shell
[root@localhost ~]# ./presto --server 192.168.8.54:18080
presto> help

Supported commands:
QUIT
EXPLAIN [ ( option [, ...] ) ] <query>
    options: FORMAT { TEXT | GRAPHVIZ }
             TYPE { LOGICAL | DISTRIBUTED }
DESCRIBE <table>
SHOW COLUMNS FROM <table>
SHOW FUNCTIONS
SHOW CATALOGS [LIKE <pattern>]
SHOW SCHEMAS [FROM <catalog>] [LIKE <pattern>]
SHOW TABLES [FROM <schema>] [LIKE <pattern>]
USE [<catalog>.]<schema>

presto> 
```
CLI 中的大多数命令，尤其是所有 SQL 语句，需要以分号结尾。
#### 分页
默认情况下，查询的结果将使用精心配置好的 less 程序分页，可以通过将环境变量 PRESTO_PAGER 设置为另一个程序名字（如 more）来覆盖这一行为，或将其置为空来彻底禁用分页。
#### 命令历史
Presto CLI 会保留之前运行过的命令的历史，查看这些历史命令，通过以下方式：

1. 使用上下箭头滚动浏览
2. 是哦那个 Ctrl-S 组合键和 Ctrl-R 组合键查询历史

要再次执行一条查询，只需按回车键即可。
默认情况下，Presto 的命令历史保存在 `~/.presto_history` 文件中，可以使用 PRESTO_HISTORY_FILE 环境变量来修改默认值。
#### 额外诊断
Presto CLI 提供了 --debug 选项来打开运行查询时的调试信息输出：
```shell
[root@localhost ~]# ./presto --debug --server 192.168.8.54:18080
presto> show catalogs;
             Catalog              
----------------------------------
 116                              
 12306                            
 123456                           
 2b0fd9430faf45d494d43ac40fddc560 
 936bb56                          
 936bb8                           
 es116                            
 es26                             
 fr1gr                            
 gp                               
 myssql8581                       
 sqlserver58                      
 system                           
(13 rows)

Query 20230821_054431_00000_4wf7b, FINISHED, 1 node
http://192.168.8.54:18080/ui/query.html?20230821_054431_00000_4wf7b
Splits: 19 total, 19 done (100.00%)
CPU Time: 0.0s total,     0 rows/s,     0B/s, 50% active
Per Node: 0.0 parallelism,     0 rows/s,     0B/s
Parallelism: 0.0
Peak User Memory: 0B
Peak Total Memory: 0B
Peak Task Total Memory: 0B
S0-driverCountPerTask: sum=1 count=1 min=1 max=1
S0-taskBlockedTimeNanos: sum=34ms count=1 min=34ms max=34ms
S0-taskElapsedTimeNanos: sum=52ms count=1 min=52ms max=52ms
S0-taskQueuedTimeNanos: sum=10ms count=1 min=10ms max=10ms
S0-taskScheduledTimeNanos: sum=1ms count=1 min=1ms max=1ms
S1-driverCountPerTask: sum=17 count=1 min=17 max=17
S1-taskBlockedTimeNanos: sum=382ms count=1 min=382ms max=382ms
S1-taskElapsedTimeNanos: sum=47ms count=1 min=47ms max=47ms
S1-taskQueuedTimeNanos: sum=2ms count=1 min=2ms max=2ms
S1-taskScheduledTimeNanos: sum=5ms count=1 min=5ms max=5ms
S2-driverCountPerTask: sum=1 count=1 min=1 max=1
S2-taskElapsedTimeNanos: sum=17ms count=1 min=17ms max=17ms
S2-taskQueuedTimeNanos: sum=5ms count=1 min=5ms max=5ms
S2-taskScheduledTimeNanos: sum=0ms count=1 min=0ms max=0ms
fragmentPlanTimeNanos: sum=5ms count=1 min=5ms max=5ms
logicalPlannerTimeNanos: sum=3ms count=1 min=3ms max=3ms
optimizerTimeNanos: sum=31ms count=1 min=31ms max=31ms
[Latency: client-side: 167ms, server-side: 125ms] [0 rows, 0B] [0 rows/s, 0B/s]

presto> 
```
#### 执行查询
可以直接使用 presto 命令执行查询，并在查询执行完毕时退出 Presto CLI。这在需要编写执行多条件查询的脚本，或使用其他系统实现更复杂的自动化工作流时很有用。以这种方式执行命令时，它会返回来自 Presto 的查询结果。

如果要使用 Presto CLI 直接执行查询，需要使用 --execute 选项。注意，在查询语句中，需要使用完全限定符来引用一个表（例如：catalog.schema.table）
同样，可以使用 --catalog 和 --schema 选项来避免使用完全限定符：
```shell
$ Presto --catalog tpch --schema sf1 \
  --execute 'select nationkey, name, regionkey from nation limit 5'
```
借助分号分隔不同的查询语句，可以一次执行多个查询
Presto CLI 也支持读入并执行文件中的命令和 SQL 查询，如 nations.sql 文件包含以下内容：
```sql
USE tpch.sf1;
SELECT name FROM nation;
```
可以在 CLI 中使用 -f 选项执行该文件中的命令，Presto CLI 会将结果输出到命令行中，然后退出：
```shell
$ presto -f nations.sql
```
#### 输出格式
Presto CLI 提供了 --output-format 选型来控制如何在非交互模式下显示输出，可用的选项有：

- ALIGNED
- VERTICAL
- CSV
- TSV
- CSV_HEADER
- TSV_HEADER
- NULL

默认值是 CSV
#### 忽略错误
Presto CLI 提供了 --ignore-error 选项，使用它可以忽略执行文件中查询时遇到的任何错误。默认行为是在遇到第一个错误时终止执行脚本。

### Presto JDBC 驱动
任何 Java 应用程序都可以通过 Java 数据库连接（JDBC）驱动连接到 Presto 通过 JDBC 驱动，所有这些应用程序都可以使用 Presto。Presto 的 JDBC 驱动允许你连接到 Presto 并使用 SQL 语句与 Presto 交互。（Presto 的 JDBC 驱动是 Type 4 驱 动，这仅仅意味着它直接与 Presto 原生接口通信。）
其他步骤省略....

### Presto 与 ODBC
略

### 客户端库
除了由 Presto 团队直接维护的 Presto CLI 和 JDBC 驱动，Presto 社区的很多成员也为 Presto 
创建了很多客户端库。 
你可以找到为 Python、C、Go、Node.js、R、Ruby 等语言开发的库。Presto 网站（参见 
1.4.1 节）维护了一个客户端列表。 
这些库使你可以将 Presto 与这些语言生态系统中的应用程序集成起来，包括你自己编写的 
应用程序。 

### Presto Web UI
Presto 服务端提供了一个 Web 界面，通常叫作 Presto Web UI。如图 3-3 所示，Presto Web 
UI 提供了 Presto 服务端及其查询处理的详细信息。 
Presto Web UI 可以借助与 Presto 服务端相同的 HTTP 端口号，使用相同的地址访问。默认 
端口是 8080，则一个可能的访问地址是 http://presto.example.com:8080。因此，在本地的安 
装环境下，可以在 http://localhost:8080 处访问 Web UI。 
主仪表盘显示了 Presto 利用率的细节和查询列表。你还可以通过 UI 获得更进一步的信息。 
这些信息对操作 Presto 和管理运行中的查询具有极大的价值。 
使用 Web UI 有助于监控 Presto 和性能调优（后续单独介绍）。新手主要可以用它来确 
认服务端是否正常启动和运行，并正在处理查询。
### 使用 Presto 执行 SQL
Presto 是一个兼容 ANSI SQL 的查询引擎，可以让你使用相同的 SQL 语句、函数和运算符 
来查询和操作所有可以连接到的数据源。 

Presto 致力于成为一个与现有 SQL 标准兼容的系统，其主要设计原则之一是既不发明一套 
新的类 SQL 查询语言，也不偏离 SQL 标准太远。它所有的新功能和语言特性都试图与标 
准兼容。 

只有在标准没有定义某一等价的功能时，Presto 才会考虑扩展 SQL 特性。即使如此， 
Presto 在设计新的语言特性时也会非常小心。标准 SQL 和其他 SQL 实现里的相似特性会 
被考虑在内，以判断其是否有可能在未来标准化。
> 在少数 SQL 标准没有定义等价功能的情况下，Presto 会扩展标准，一个突出 
> 的例子是 lambda 表达式


Presto 没有将自己绑定在某一个特定的 SQL 标准版本上，相反，它将 SQL 标准看作活动 
的文档，并且非常重视最新版本的标准。另外，Presto 还未完全实现 SQL 标准所定义的所 
有必要特性。一条规定是，如果现存的某个功能被发现与标准不兼容，就会被弃用并很快 
替换为兼容标准的实现。

如前所述，你可以使用 Presto CLI 以及数据库管理工具（与 JDBC 或 ODBC 连接）来查 
询 Presto。 

#### 概念

- 连接器（数据源）：使 Presto 适配一个数据源。每一个 catalog 对应于一个特定的连接器。
- catalog：定义连接到一个数据源的细节。它包含了 schema 并配置了一个连接器来使用。
- schema：组织表的一种方式。catalog 和 schema 一起定义了一个集合的表，这些表可以查询
- table（表）：表是无序的行的集合。这些行内容被组织成带有数据类型的有名称的列。

### Presto  使用 SQL

#### catalogs 相关
```shell
presto> show catalogs;
 Catalog  
----------
 58       
 mysql116 
 system   
 tpch     
(4 rows)

Query 20230821_063439_00001_kt26c, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 0:02, server-side: 0:02] [0 rows, 0B] [0 rows/s, 0B/s]
```
#### schemas 相关
```shell
presto> show schemas from tpch;
       Schema       
--------------------
 information_schema 
 sf1                
 sf100              
 sf1000             
 sf10000            
 sf100000           
 sf300              
 sf3000             
 sf30000            
 tiny               
(10 rows)

Query 20230821_063904_00003_kt26c, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 325ms, server-side: 310ms] [10 rows, 119B] [32 rows/s, 383B/s]
```
#### tables 相关
```shell
presto> show tables from tpch.sf1;
  Table   
----------
 customer 
 lineitem 
 nation   
 orders   
 part     
 partsupp 
 region   
 supplier 
(8 rows)

Query 20230821_063641_00002_kt26c, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 0:01, server-side: 0:01] [8 rows, 158B] [6 rows/s, 128B/s]
```
```shell
presto> describe tpch.sf1.region;
  Column   |     Type     | Extra | Comment 
-----------+--------------+-------+---------
 regionkey | bigint       |       |         
 name      | varchar(25)  |       |         
 comment   | varchar(152) |       |         
(3 rows)

Query 20230821_064002_00004_kt26c, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 352ms, server-side: 332ms] [3 rows, 193B] [9 rows/s, 581B/s]

presto> desc tpch.sf1.region;
  Column   |     Type     | Extra | Comment 
-----------+--------------+-------+---------
 regionkey | bigint       |       |         
 name      | varchar(25)  |       |         
 comment   | varchar(152) |       |         
(3 rows)

Query 20230821_064124_00005_kt26c, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 410ms, server-side: 396ms] [3 rows, 193B] [7 rows/s, 487B/s]
```
#### 查询语句
```shell
presto> select name from tpch.sf1.region;
    name     
-------------
 AFRICA      
 AMERICA     
 ASIA        
 EUROPE      
 MIDDLE EAST 
(5 rows)

Query 20230821_064340_00006_kt26c, FINISHED, 1 node
Splits: 20 total, 20 done (100.00%)
[Latency: client-side: 0:02, server-side: 0:02] [5 rows, 0B] [2 rows/s, 0B/s]
```
```shell
presto> select name from tpch.sf1.region where name like 'A%' order by name desc;
  name   
---------
 ASIA    
 AMERICA 
 AFRICA  
(3 rows)

Query 20230821_064457_00007_kt26c, FINISHED, 1 node
Splits: 22 total, 22 done (100.00%)
[Latency: client-side: 369ms, server-side: 341ms] [5 rows, 0B] [14 rows/s, 0B/s]
```
```shell
presto> select nation.name as nation, region.name as region from tpch.sf1.region, tpch.sf1.nation where region.regionkey = nation.regionkey and region.name like 'AFRICA';
   nation   | region 
------------+--------
 ALGERIA    | AFRICA 
 ETHIOPIA   | AFRICA 
 KENYA      | AFRICA 
 MOROCCO    | AFRICA 
 MOZAMBIQUE | AFRICA 
(5 rows)

Query 20230821_064803_00008_kt26c, FINISHED, 1 node
Splits: 56 total, 56 done (100.00%)
[Latency: client-side: 0:01, server-side: 0:01] [30 rows, 0B] [40 rows/s, 0B/s]
```
Presto 支持 `||`作为字符串连接运算符，也可以使用算术运算符，如 + 和 -，将上一个查询改为使用 JOIN ，并将返回的字符串结果连接一个字段：
```shell
presto> select nation.name || '/' || region.name as localtion from tpch.sf1.region join tpch.sf1.nation on region.regionkey = nation.regionkey and region.name like 'AFRICA';
     localtion     
-------------------
 ALGERIA/AFRICA    
 ETHIOPIA/AFRICA   
 KENYA/AFRICA      
 MOROCCO/AFRICA    
 MOZAMBIQUE/AFRICA 
(5 rows)

Query 20230821_065523_00009_kt26c, FINISHED, 1 node
Splits: 56 total, 56 done (100.00%)
[Latency: client-side: 489ms, server-side: 459ms] [30 rows, 0B] [65 rows/s, 0B/s]
```
#### 支持的函数
Presto 还支持各种各样的函数，它们复杂程度各不相同。在 Presto 中， 可以使用 SHOW FUNCTIONS 语句显示这些函数的列表
```shell
presto> select round(avg(totalprice)) as average_price from tpch.sf1.orders;
 average_price 
---------------
      151220.0 
(1 row)

Query 20230821_065930_00012_kt26c, FINISHED, 1 node
Splits: 37 total, 37 done (100.00%)
[Latency: client-side: 0:05, server-side: 0:05] [1.5M rows, 0B] [279K rows/s, 0B/s]
```

