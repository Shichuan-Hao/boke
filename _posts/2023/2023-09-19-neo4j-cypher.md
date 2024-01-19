---
title: Neo4j Cypher 语句
date: 2023-09-19 14:31:57 +0800
categories: [博客]
tags: [Neo4j, Cypher] 
---


[原文地址](https://neo4j.com/docs/getting-started/appendix/tutorials/guide-cypher-basics/)
本教程解释了 Neo4j 的查询语言 Cypher® 的基本概念，包括如何创建和查询图形。完成本教程后，您应该能够阅读和理解 Cypher 查询。
### Pop culture connections
电影图（Movie Graph）是一个迷你图应用程序，其中包含通过他们合作的电影相关的演员和导演。

如果您按照本教程运行查询和 Cypher code 来创建数据，将会很有帮助。

本教程将向您展示如何：

1. 创建（Create）：将电影数据插入图表中。
2. 查找（Finde）：检索单个电影和演员。
3. 查询（Query）：在图中查找模式。
4. 解决（Solve）：回答一些有关图表的问题。
### 创建电影图 Create the Movie Graph

1. 创建并启动新的 Neo4j 数据库。
   1. 在 https://sandbox.neo4j.com 创建一个空白沙箱或...
   2. 在 Neo4j Desktop 中创建一个新数据库：
      1. 创建一个新项目。
      2. 将数据库添加到项目中。
      3. 启动数据库。
2. 打开 Neo4j 浏览器。
3. 设置浏览器设置以允许多语句：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694226290174-5197a8b0-e4b5-42fa-b4e4-db5e440ed030.png#averageHue=%23373941&clientId=u334fc670-00e0-4&from=paste&id=ub9416213&originHeight=1196&originWidth=706&originalType=url&ratio=1&rotation=0&showTitle=false&size=114808&status=done&style=none&taskId=u1b482450-dc03-42ee-a963-fafd6fb8a92&title=)

4. 在查询窗格中输入 `:guide movie-graph` 并单击右侧的“播放”按钮。查询窗格下方会打开一个新窗口，其中包含浏览器指南。
5. 转至浏览器指南的第 2 页。
6. 单击 Cypher code，将其带入查询窗格，然后单击“播放”按钮。

这是加载电影图表后您应该在 Neo4j 浏览器中看到的内容：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694226567840-a0524975-228a-4c48-a8a2-a5a9005fa034.png#averageHue=%23bfc6ca&clientId=u334fc670-00e0-4&from=paste&id=u23b1bf7f&originHeight=1066&originWidth=2384&originalType=url&ratio=1&rotation=0&showTitle=false&size=377212&status=done&style=none&taskId=u6b67fcfd-f707-4ed8-b414-ddb3ffe37b0&title=)
这是一些返回数据的图表视图。

如果您想查看返回数据的表格视图，请单击左侧的表格图标：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694226679234-22c2976c-fece-42fd-93bc-264bd30e7b8e.png#averageHue=%235d5d5c&clientId=u334fc670-00e0-4&from=paste&id=ub38f202f&originHeight=1066&originWidth=2384&originalType=url&ratio=1&rotation=0&showTitle=false&size=263069&status=done&style=none&taskId=u0b25fcb8-1915-4ad5-bb5f-ee0db80e4cf&title=)
您如何查看结果也取决于返回的数据。如果查询返回节点，则您可以以图形形式查看数据。如果查询返回属性值，则只能以表格形式查看数据。

如果你需要帮助：
```cypher
:help cypher
```
> 当您在查询窗格中运行 Cypher 代码时，它始终会在查询窗格下方创建一个包含结果的新窗格。

### 查找演员和电影
接下来，您将了解用于查找单个节点的查询。

1. 查看每个查询示例
2. 使用播放按钮运行查询
3. 注意语法模式
4. 尝试寻找其他电影或演员

 
如果您需要语法帮助：`:help MATCH, :help WHERE, and :help RETURN`

#### 找到那个叫“汤姆·汉克斯（Tom Hanks）”的人……
将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (tom:Person)
WHERE tom.name = "Tom Hanks"
RETURN tom
```
这是一些返回数据的图表视图。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694228835255-d12fa5f4-e0b1-406e-becc-8bea6e663593.png#averageHue=%23d6f2f8&clientId=u334fc670-00e0-4&from=paste&height=542&id=ucb52d628&originHeight=542&originWidth=1816&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34612&status=done&style=none&taskId=ua51085ec-0b55-4926-9f76-07fa09c571c&title=&width=1816)
您还可以使用表视图（table view）查看节点的属性：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694228868999-35637761-7308-4933-b918-aeae7c3b1c88.png#averageHue=%23e6caa1&clientId=u334fc670-00e0-4&from=paste&height=463&id=uacf39a76&originHeight=463&originWidth=1827&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32845&status=done&style=none&taskId=u90ed7ef1-0ac9-4058-8ed6-7b7eed66599&title=&width=1827)
#### 查找名为“Cloud Atlas”的电影...
在这里，我们以不同的方式过滤查询，在节点规范中指定值，而不是使用 WHERE 子句。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (cloudAtlas:Movie {title: "Cloud Atlas"})
RETURN cloudAtlas
```
 这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694229118103-6df63b6b-ce06-4fdb-aaac-02c24ac8c865.png#averageHue=%23e0f9fa&clientId=u334fc670-00e0-4&from=paste&height=542&id=u6e319871&originHeight=542&originWidth=1834&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38144&status=done&style=none&taskId=uee272774-0219-4898-a29f-e93c20dc896&title=&width=1834)
 
这是表视图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694229187831-ee82c8e2-8541-411a-871d-6b542f3421f0.png#averageHue=%23e4c89e&clientId=u334fc670-00e0-4&from=paste&height=490&id=u8da37032&originHeight=490&originWidth=841&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35739&status=done&style=none&taskId=ud30ef620-660f-401d-8243-4d2b152b550&title=&width=841)

#### Find 10 people…
接下来我们要查找图中 10 个人的姓名。此代码查找图中的所有 Person 节点，但仅返回其中 10 个的 name 属性值。

 
将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (people:Person)
RETURN people.name LIMIT 10
```
 
这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694229455836-4221eecb-08c3-4a3e-958d-5431104b7028.png#averageHue=%23f6fafd&clientId=u334fc670-00e0-4&from=paste&height=592&id=u92a4b754&originHeight=592&originWidth=1531&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31037&status=done&style=none&taskId=u1bdf2497-6708-4a10-9eea-604b5841dee&title=&width=1531)
 对于此查询，将返回属性值，并且您只能以表格形式查看结果。

#### 查找 20 世纪 90 年代上映的电影...
 
这是一个查询，我们在其中指定用于选择要检索的 Movie 节点的值范围。然后我们返回这些 Movie 节点的标题。
 
将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (nineties:Movie)
WHERE nineties.released > 1990 AND nineties.released < 2000
RETURN nineties.title
```
 
这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694230098334-bcae52a9-67ea-48e7-a8fe-d02cb96b0871.png#averageHue=%23f7fafe&clientId=u334fc670-00e0-4&from=paste&height=576&id=u0a17511f&originHeight=576&originWidth=1542&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33115&status=done&style=none&taskId=u260cad14-52c0-44ef-84a6-28ce4b09940&title=&width=1542)
###  查找图表中的模式
到目前为止，您已经在图中查询了节点。接下来，您将获得检索相关节点的经验。

您将执行 Cypher 代码来查找图中的模式：

1. 演员是在电影中表演的人。
2. 导演是导演电影的人。
3. 还存在哪些其他关系？
####  列出所有汤姆·汉克斯（Tom Hanks）的电影……
 
在下面的查询中，我们希望返回演员 Tom Hanks 的 Person 节点，并且还希望返回与 Tom Hanks 具有 ACTED_IN 关系的所有 Movie 节点。即汤姆·汉克斯出演的所有电影。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (tom:Person {name: "Tom Hanks"})-[:ACTED_IN]->(tomHanksMovies)
RETURN tom,tomHanksMovies
```
 
这是该查询的结果：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231457992-e37cedba-92bc-4328-87f2-7ef39cd73765.png#averageHue=%23e5f4f4&clientId=u334fc670-00e0-4&from=paste&height=1007&id=ub4b3ac14&originHeight=1007&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=165505&status=done&style=none&taskId=u152608a5-f1d0-49bc-9889-fcd7edb94a8&title=&width=1920)
请注意，这里我们还看到 Tom Hanks 节点和 Movie 节点之间的有向关系。这是因为我们在 Neo4j 浏览器中有一个设置，其中结果节点将被连接：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231582876-440b4b32-0599-4b7c-88bf-a8724e56e506.png#averageHue=%23c9e3e6&clientId=u334fc670-00e0-4&from=paste&height=919&id=u587e741b&originHeight=919&originWidth=1917&originalType=binary&ratio=1&rotation=0&showTitle=false&size=186564&status=done&style=none&taskId=ufb2f8c64-52b8-4da2-835a-767a0cb8cb4&title=&width=1917)
这是表视图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231623462-f2f51f4e-b05c-419e-a79f-7e9137d54342.png#averageHue=%23dbbc91&clientId=u334fc670-00e0-4&from=paste&height=1007&id=uf4c95345&originHeight=1007&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98647&status=done&style=none&taskId=u0ecf0d7f-7e68-4288-9f5e-93ea67c96c6&title=&width=1920)
#### 《Colud Atlas》是谁导演的？
这是一个查询，我们希望返回与 Cloud Atlas Movie 节点有 DIRECTED 关系的节点。它将返回电影导演的姓名。

将此代码复制并粘贴到查询窗格中并执行它：

```cypher
MATCH (cloudAtlas:Movie {title: "Cloud Atlas"})<-[:DIRECTED]-(directors)
RETURN directors.name
```
这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231743661-33110370-d24c-4393-bceb-75c0b60b4130.png#averageHue=%23f7fafe&clientId=u334fc670-00e0-4&from=paste&height=369&id=u7493827a&originHeight=369&originWidth=1536&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28302&status=done&style=none&taskId=u545ac1dd-f287-4240-b556-c0be44693b0&title=&width=1536)
####  汤姆·汉克斯的合作演员…
接下来，我们要查找汤姆·汉克斯 (Tom Hanks) 出演的所有电影，并为检索到的每部电影找到出演该电影的人。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (tom:Person {name:"Tom Hanks"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(coActors)
RETURN tom, m, coActors
```
这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231927665-5b32d15e-4fff-44a2-8259-9a74b45f981e.png#averageHue=%23c7e4ed&clientId=u334fc670-00e0-4&from=paste&height=1007&id=ub4ce8698&originHeight=1007&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=232656&status=done&style=none&taskId=u8cb6d975-3876-4568-86f4-af242ae253e&title=&width=1920)
 
这是表视图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694231936365-c320d11c-efd7-4b77-b609-ebed3de17ff0.png#averageHue=%23d9b886&clientId=u334fc670-00e0-4&from=paste&height=1007&id=u64bec7c3&originHeight=1007&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=115260&status=done&style=none&taskId=ua7045583-130f-4b9a-a6e0-8b73f1a280f&title=&width=1920)
#### 人们与“Cloud Atlas”有何关系……
这是一个查询，我们希望在其中返回有关 Cloud Atlas 影片之间的关系的信息。我们找到相关节点，然后返回人员姓名、关系类型以及该关系的属性。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (people:Person)-[relatedTo]-(:Movie {title: "Cloud Atlas"})
RETURN people.name, type(relatedTo), relatedTo
```

这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694242774774-eac1dc39-f149-4488-b076-0d617abe70c8.png#averageHue=%23f7fbfe&clientId=u46fd5cc3-7581-4&from=paste&height=375&id=ub9dc4151&originHeight=375&originWidth=1942&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28055&status=done&style=none&taskId=ufd9a6e6b-4ffc-424e-960b-ef6fd350310&title=&width=1942)
### 回答一些关于图表的问题
你听说过经典的《Six Degrees of Kevin Bacon》吗？也就是说，找到图中距离 Kevin Bacon 最多 6 hops 的所有人员。这只是一个称为“Bacon Path”的最短路径查询。要执行此类查询，您需要指定：

- 可变长度模式（Variable length patterns）: [variable length relationships](https://neo4j.com/docs/cypher-manual/current/syntax/patterns/#cypher-pattern-varlength)
- 内置 `shortestPath()` 算法（Built-in `shortestPath()`algorithm） : [shortestPath](https://neo4j.com/docs/cypher-manual/current/execution-plans/shortestpath-planning/)

#### 距离凯文·培根（Kevin Bacon）最多 3 hops 的电影和演员
在我们的第一个查询中，我们希望找到图表中距 Kevin Bacon 最多 3 跳的所有电影和/或人物。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (bacon:Person {name:"Kevin Bacon"})-[*1..3]-(hollywood)
RETURN DISTINCT bacon, hollywood
```
这是该查询的结果：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1694242758245-30b8d042-aa96-48e2-b3a8-826c7922842d.png#averageHue=%23cee8ec&clientId=u46fd5cc3-7581-4&from=paste&height=1007&id=ua32cff15&originHeight=1007&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=213642&status=done&style=none&taskId=u3b48a299-c205-4051-8530-105a9e6be96&title=&width=1920)
#### 寻找梅格·瑞恩（Meg Ryan）和凯文·培根（Kevin Bacon） 的最短路径

图中 Kevin Bacon 和 Meg Ryan 之间的最短路径是什么？在此 Cypher 中，我们返回包含节点和关系的路径。

将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH p=shortestPath(
  (bacon:Person {name:"Kevin Bacon"})-[*]-(meg:Person {name:"Meg Ryan"})
)
RETURN p
```
 
在执行查询之前，您将看到一条警告，指出“*”关系可能需要很长时间才能执行。我们的电影图很小，因此您可以忽略此警告。

 
这是该查询的结果：

###  Clean up
完成实验后，您可以删除电影数据集。

1. 如果节点存在关系，则无法删除它们。
2. 一起删除节点和关系。
:::danger
 💥
这将删除图中的所有节点和关系！
:::
 
将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (n)
DETACH DELETE n
```
这是该查询的结果：

请注意，虽然左侧面板中的数据库信息在图中没有显示节点或关系，但属性键名称仍然保留。

#### 验证电影图表数据是否已消失
如果执行此查询来检索图中的所有节点并返回计数，您应该会看到返回值 0。

 
将此代码复制并粘贴到查询窗格中并执行它：
```cypher
MATCH (n)
RETURN count(*)
```
这是该查询的结果：


恭喜！您已经学习了如何使用 Cypher 查询 Neo4j 数据库。
