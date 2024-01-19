---
title: Neo4j 示例练习
date: 2023-09-12 14:31:57 +0800
categories: [博客]
tags: [Neo4j] 
---


[原文地址](https://neo4j.com/docs/getting-started/appendix/example-data/)


💡
> 您可以在此处找到 Neo4j 的可用示例数据集列表，并了解如何导入和探索它们。
{: .prompt-info }

### 数据集
对于 Neo4j 入门，使用与您的领域和用例相关的示例数据集会很有帮助。对于每个我们想要提供描述、图形模型和一些用例查询。
### 内置示例
Neo4j 浏览器附带两个内置数据库，您可以使用交互式幻灯片创建和探索它们。

“电影”示例是通过 `:guide movie-graph` 命令启动的，包含一个小电影图以及与这些电影相关的人员（如演员、导演、制片人等）。

“Northwind”示例通过 `:guide Northwind-grap`h 运行，包含一个包含产品、订单、客户、供应商和员工的传统零售系统。它引导您完成数据导入以及使用可用数据进行增量复杂查询。
#### 其他浏览器指南
您可以在自己的 Neo4j 浏览器中运行的其他示例数据集包括：

- 权力的游戏互动（Game of Thrones Interactions） — `:play got`
-  英国公司注册、财产所有权、政治捐赠（UK company registration, property ownership, political donations） — `:play ukcompanies`
- Stack Overflow 用户、标签和问答数据（Stack Overflow users, tags and Q&A data） — `:play stackoverflow`
- BBC Good Foods 食谱数据（BBC Good Foods recipe data） — `:play recipes`
- Airbnb 房源数据（Airbnb listings data） — `:play listings`
-  足球（英式足球）转会数据（Football (Soccer) transfer data） — `:play football_transfers`
### AuraDB 免费示例数据集
 在 Neo4j AuraDB 中创建新数据库时，除了默认的空数据库之外，您还可以选择起始数据集之一：

- Movies
- Graph based Recommendations
- Graphs for Cybersecurity
- StackOverflow Data

 您可以按照浏览器指南的说明探索它们，并使用建议的 Cypher® 查询测试数据。
### Neo4j 沙箱
要在无需本地安装的情况下在线设置中探索各种数据集，您可以使用[ Neo4j Sandbox](https://neo4j.com/sandbox/?ref=developer-ex-data)。

每个沙箱在创建后至少可以使用三天，并且还可以使用任何 Neo4j 驱动程序从应用程序远程访问。

除了“空白”沙箱外，所有其他沙箱都预先填充了域数据，并专注于用例特定的查询。

所有沙箱都提供对 Neo4j 浏览器、Neo4j Bloom、APOC、图形数据科学、新语义 (n10s) 和 GraphQL 集成的访问。
### Github 上的 Neo4j Graph 示例存储库
每个沙箱的数据、浏览器指南、代码示例（JavaScript、Java、Python、Go、C#）、Cypher 查询、Bloom 透视图均可在[ GitHub 存储库](https://github.com/neo4j-graph-examples)上找到。

用例范围包括：

- [movie recommendations](https://sandbox.neo4j.com/?usecase=recommendations) ([Repo](https://github.com/neo4j-graph-examples/recommendations))
- [network management](https://sandbox.neo4j.com/?usecase=network-management) ([Repo](https://github.com/neo4j-graph-examples/network-management))
- [investigative data from the ICIJ (Panama Papers)](https://sandbox.neo4j.com/?usecase=icij-paradise-papers) ([Repo](https://github.com/neo4j-graph-examples/icij-paradise-papers))
- [crime investigation](https://sandbox.neo4j.com/?usecase=pole) ([Repo](https://github.com/neo4j-graph-examples/pole))
- [social networks](https://sandbox.neo4j.com/?usecase=twitter-v2) optionally using your own Twitter account ([Repo](https://github.com/neo4j-graph-examples/twitter)).
###  Neo4j 数据集演示服务器
#### 获取信息
如果您需要探索更多图形数据库，可以访问 https://demo.neo4jlabs.com:7473 上的服务器
该服务器托管许多具有只读访问权限的数据集供公众使用。
用户名和密码与数据库名称相同。
例如，对于`recommendations` 数据库，用户名是`recommendations` ，密码也是`recommendations `。
#### 托管数据库
 
您可以通过单击链接打开以下任何数据库。不要忘记复制用户名和密码：

- [recommendations](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://recommendations@demo.neo4jlabs.com&db=recommendations)
- [movies](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://movies@demo.neo4jlabs.com&db=movies)
- [northwind](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://northwind@demo.neo4jlabs.com&db=northwind)
- [fincen](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://fincen@demo.neo4jlabs.com&db=fincen)
- [twitter](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://twitter@demo.neo4jlabs.com&db=twitter)
- [stackoverflow](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://stackoverflow@demo.neo4jlabs.com&db=stackoverflow)
- [gameofthrones](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://gameofthrones@demo.neo4jlabs.com&db=gameofthrones)
- [neoflix](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://neoflix@demo.neo4jlabs.com&db=neoflix)
- [wordnet](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://wordnet@demo.neo4jlabs.com&db=wordnet)
- [slack](https://demo.neo4jlabs.com:7473/browser/?dbms=neo4j://slack@demo.neo4jlabs.com&db=slack)
### 数据导入方式
#### 从源数据加载数据
将数据集导入 Neo4j 的最可靠方法是从原始源导入。这样您就可以独立于数据库版本，否则您可能必须升级数据库版本。这就是为什么我们为几个数据集提供原始数据（CSV、JSON、XML），并附有 Cypher 中的导入脚本。

 您可以使用命令行客户端（例如 cypher-shell）运行 Cypher 脚本。

```java
./bin/cypher-shell -u neo4j -p "password" -f import-file.cypher
```
您还可以将脚本拖放到或粘贴到 Neo4j 浏览器中（检查设置中是否启用了多语句编辑器）并从那里运行它。

可以[使用 Cypher ](https://neo4j.com/docs/getting-started/data-import/csv-import/#import-load-csv/)中的 `LOAD CSV` 子句或 `[neo4j-admin-database import](https://neo4j.com/docs/operations-manual/current/tutorial/neo4j-admin-import/)`来导入 CSV 数据，以进行大型数据集的初始批量导入。

 
要加载 JSON、XML 文件，您需要安装[ APOC Core 库](https://neo4j.com/docs/apoc/current/)，该库附带[了许多用于从其他数据库导入数据的过程](https://neo4j.com/docs/apoc/current/import/)。

 
> 💥 提示 要加载 XLS 文件，您可以使用 APOC 扩展库。注意 APOC 扩展库不受官方支持。
{: .prompt-tips }

####  使用 Neo4j 数据库的转储（dump）

其他数据集作为 NEO4J 数据存储的转储提供。请按照链接 [http://github.com/neo4j-graph-examples](http://github.com/neo4j-graph-examples) 查找许多Graph示例数据集的转储文件。

- 社区版（替换默认数据库）
   - 停止您的Neo4J服务器。
   - 然后，使用 `./bin/neo4j-admim databasle load -overWrite-Destination true-from-stdin neo4j file.dump`命令。
   - 启动 Neo4J 服务器。
- 企业版（也是 Neo4j 桌面版）
   - 使用 `./bin/neo4j-admin database load --overwrite-destination true --from-stdin dbname file.dump` 命令导入文件。
   - 使用 `CREATE DATABASE dbname` 让系统数据库知道新数据库，这也会自动启动它。
> 某些数据集的 Neo4j 版本可能比您的 Neo4j 版本旧。然后，您可能需要使用 `neo4j-admin database migrate` 命令来配置 Neo4j 来升级数据库。请注意，`neo4j-admin database migrate`命令仅在已停止的数据库上运行。详细信息请参见[操作手册→工具。](https://neo4j.com/docs/operations-manual/current/tools/neo4j-admin/migrate-database/)

#### 大量数据转储（dump）
##### Stack Overflow
这是 Stack Overflow 存档的图形导入，包含 1640 万个问题、52k 个标签和 890 万个用户（Stack Overflow Dump (6.2GB)）。该图非常大，对于全局图查询，您需要 6G 的页面缓存和 16G 的堆才能使用它。

这是一篇文章，解释了[数据模型](https://towardsdatascience.com/tagoverflow-correlating-tags-in-stackoverflow-66e2b0e1117b)以及我们对数据进行的一些探索性分析。
a  images(https://cdn.nlark.com/yuque/0/2023/svg/35987817/1694224303561-cebe07b4-0c3d-4ec4-89dc-fdcdc40e66fd.svg#clientId=u897a9e49-d074-4&from=paste&id=u8a47467a&originHeight=509&originWidth=760&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uc2e6d79f-80fa-45e9-aefd-195f53aa4c3&title=)
如上所述，该数据库在演示服务器中可用。
