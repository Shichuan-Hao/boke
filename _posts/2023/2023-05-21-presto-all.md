---
title: Presto CLI
date: 2023-05-21 14:31:57 +0800
categories: [文章]
tags: [Presto] 
---


## Presto CLI
Presto CLI 提供了一个基于终端的交互式 shell，可以通过它执行查询，与 Presto 服务端交互，来查看元数据。
与 Presto 服务端一样，Presto CLI 也是通过 Maven 中央仓库分发二进制包。该 CLI 应用程序以可执行的 JAR 包的形式提供，可以像普通i UNIX 可执行程序一样使用它。

Presto CLI 可用版本列表：  [https://repo.maven.apache.org/maven2/io/prestosql/presto-cli/](https://repo.maven.apache.org/maven2/io/prestosql/presto-cli/)

找到与自身使用的 Presto 服务端版本一致的 CLI 版本，从目录中下载对应 *-executable.jar 文件，然后将其重命名，如下：
```shell
$ wget -O presto \
https://repo.maven.apache.org/maven2/\
io/prestosql/presto-cli/330/presto-cli-330-executable.jar
```
要确保文件具有可执行权限。为了方便使用，可以将其放置在 PATH 中，如下：
```shell
$ chmod +x presto
$ mv presto &#x7e;/bin
$ export PATH=~/bin/:$PATH
```
而后，可以运行如下命令，确认其版本:
```shell
$ presto --version
Presto CLI 0.280-a95c1b4
```
默认情况下，CLI 会连接到运行在 `http://localhost:8080`上的 Presto 服务端。
如果 Presto 服务端在本地运行（出于测试或开发目的），或者通过 SSH 连接到服务器，又或者使用 exec 命令连接到安装了 CLI 的 Docker 容器中，则可以直接执行以下命令：
```shell
$ presto
presto>
```
如果 Presto 服务端运行在另一台服务器上，则需要指定 URL，如下：
```shell
$ presto --server https://presto.example.com:8080
presto>
```

prosto> 命令行提示符表明你在使用交互式控制台访问 Presto 服务端。键入 help 命令来获取可用命令的列表：
```shell
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
```
CLI 中的大部分命令，尤其是所有 SQL 语句，需要以分号结尾。在 Presto 上使用 SQL 的更多信息，参见《》。
### 常用命令
`SHOW CATALOGS` 查看哪些数据源配置为 catalog
```shell
presto> SHOW CATALOGS;
 Catalog  
----------
 mysqlhsc 
 system   
(2 rows)

Query 20230803_020312_00047_yr9tw, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
[Latency: client-side: 104ms, server-side: 98ms] [0 rows, 0B] [0 rows/s, 0B/s]

presto> SHOW CATALOGS LIKE 'm%';
 Catalog  
----------
 mysqlhsc 
(1 row)

Query 20230803_020445_00048_yr9tw, FINISHED, 1 node
Splits: 35 total, 35 done (100.00%)
[Latency: client-side: 141ms, server-side: 131ms] [0 rows, 0B] [0 rows/s, 0B/s]

presto> SHOW CATALOGS LIKE '%m';
 Catalog 
---------
 system  
(1 row)

Query 20230803_020502_00049_yr9tw, FINISHED, 1 node
Splits: 35 total, 35 done (100.00%)
[Latency: client-side: 156ms, server-side: 146ms] [0 rows, 0B] [0 rows/s, 0B/s]

presto> SHOW CATALOGS LIKE '%m%';
 Catalog  
----------
 mysqlhsc 
 system   
(2 rows)

Query 20230803_020507_00050_yr9tw, FINISHED, 1 node
Splits: 35 total, 35 done (100.00%)
[Latency: client-side: 167ms, server-side: 152ms] [0 rows, 0B] [0 rows/s, 0B/s]

```
可以看到内部元数据 catalog —— system，以及本例中看到的 mysqlhsc catalogs;

`SHOW SCHEMAS FROM mysqlhsc`查看可用 shcema，`SHOW TABLES FROM mysqlhsc`查看 shcema 中的表。每次执行 Presto 查询时，查询处理过程的统计信息都会和查询结果一起返回，下面会省略，如下：
```shell
presto> SHOW SCHEMAS FROM mysqlhsc  LIKE '%a%';
       Schema       
--------------------
 asz                
 aw_presto          
 data-center-etl    
 information_schema 
 performance_schema 
 sakila             
 taier              
(7 rows)

presto> SHOW SCHEMAS FROM mysqlhsc LIKE 'a%';
  Schema   
-----------
 asz       
 aw_presto 
(2 rows)

presto> SHOW SCHEMAS FROM mysqlhsc LIKE '%a';
       Schema       
--------------------
 information_schema 
 performance_schema 
 sakila             
(3 rows)

presto> SHOW SCHEMAS FROM mysqlhsc;
       Schema       
--------------------
 asz                
 aw_presto          
 data-center-etl    
 information_schema 
 performance_schema 
 sakila             
 sys                
 taier              
 world              
(9 rows)
```

现在就查询实际数据：
此外还可以选择一个 schema 作为当前 schema，之后就可以从查询中省略限定符了：
