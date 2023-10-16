---
title: Presto 二开笔记
date: 2023-05-15 14:31:57 +0800
categories: [文章]
tags: [Presto] 
---


[https://github.com/happymaya/presto/tree/master](https://github.com/happymaya/presto/tree/master)，基于最新 tag 0.281
# Presto
Presto 是一个用于大数据的分布式 SQL 查询引擎。有关部署说明和最终用户文档，请参阅[用户手册](https://prestodb.io/docs/current/)。
# 环境要求

- Mac OS X or Linux（本次是在 Ubuntu 22.04.02 lts desktop）
- Java 8 Update 151 or higher (8u151+), 64-bit。支持 Oracle JDK and OpenJDK
- Maven 3.3.9+（构建项目）
- Python 2.4+ (用于执行启动脚本)
# 构建 Presto
Presto 是一个标准的 Maven 项目。只需从项目根目录运行以下命令：
```shell
./mvnw clean install
```
在首次次构建时，Maven 将从互联网上下载所有依赖项并将它们缓存在本地存储库 (~/.m2/repository) 中，这可能会花费大量时间。后续构建会更快。
Presto有一套全面的单元测试，运行起来可能需要几分钟时间。你可以在构建时禁用这些测试：
```shell
./mvnw clean install -DskipTests
```
第一次 ./mvnw clean install   报错，信息如下：![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685931891924-758734d5-03f5-4923-bb51-6fab8fa53c06.png#averageHue=%23373e44&clientId=u67585dca-d6c8-4&from=paste&height=409&id=ub580d612&originHeight=409&originWidth=913&originalType=binary&ratio=1&rotation=0&showTitle=false&size=85249&status=done&style=none&taskId=u00a4d47d-5035-4231-881c-0803d8c3162&title=&width=913)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685932012034-c5e2a2eb-f450-492d-8bb0-58494556f350.png#averageHue=%232b3239&clientId=u67585dca-d6c8-4&from=paste&height=701&id=ub481f249&originHeight=701&originWidth=1000&originalType=binary&ratio=1&rotation=0&showTitle=false&size=117815&status=done&style=none&taskId=u74cc7c26-fcd8-4a01-a7f7-9bba28201e9&title=&width=1000)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685932031487-004c0999-cf6a-4b71-a3bd-d9ccd46e455f.png#averageHue=%232c333b&clientId=u67585dca-d6c8-4&from=paste&height=748&id=u8525a295&originHeight=748&originWidth=636&originalType=binary&ratio=1&rotation=0&showTitle=false&size=94759&status=done&style=none&taskId=u411ba7c2-c43e-44d5-b1e2-d92ca2cadb1&title=&width=636)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685932273554-ecdd9c8a-1f15-4f12-874a-7791aa19d4a7.png#averageHue=%232d343c&clientId=u67585dca-d6c8-4&from=paste&height=637&id=ua3a972a3&originHeight=637&originWidth=577&originalType=binary&ratio=1&rotation=0&showTitle=false&size=83009&status=done&style=none&taskId=u54e4fda3-72f7-4075-ab63-11827382026&title=&width=577)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685932305930-ad6cd2fe-4c71-4099-a79a-2cc347c4e18e.png#averageHue=%23272e36&clientId=u67585dca-d6c8-4&from=paste&height=708&id=uc4e81e4d&originHeight=708&originWidth=899&originalType=binary&ratio=1&rotation=0&showTitle=false&size=109047&status=done&style=none&taskId=uc0a2437b-beb9-4138-8ac5-c9168220c5e&title=&width=899)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685932348848-1c7a2d03-eb89-4515-84f4-0eb7dce5992d.png#averageHue=%232e353b&clientId=u67585dca-d6c8-4&from=paste&height=172&id=ud9b0ecfa&originHeight=172&originWidth=770&originalType=binary&ratio=1&rotation=0&showTitle=false&size=22299&status=done&style=none&taskId=uddbc089e-9cec-484c-8394-a275397c9c8&title=&width=770)

解决方案，试一试这个 
[解决：target\surefire-reports for the individual test results_东天里的冬天的博客-CSDN博客](https://blog.csdn.net/gwd1154978352/article/details/78815844)
第二次 ./mvnw clean install -DskipTests报错，信息如下：
解决方案，参考：
[https://github.com/trinodb/trino/issues/5764](https://github.com/trinodb/trino/issues/5764)
使用上午链接中的方案，构建成功，如下
![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685934946563-b91111ef-0923-4da8-bdc5-17e655cd19df.png#averageHue=%233f464e&clientId=u67585dca-d6c8-4&from=paste&height=737&id=uc5d1d48a&originHeight=737&originWidth=1160&originalType=binary&ratio=1&rotation=0&showTitle=false&size=183850&status=done&style=none&taskId=ue82637ce-b0e9-4448-a7bf-a189d0df411&title=&width=1160)

如果不想跳过，先去单独构建 persto-docs
# Presto native 和 Velox
[Presto native ](https://github.com/prestodb/presto/tree/master/presto-native-execution)是 Presto worker 的 C++ 重写。[Presto native ](https://github.com/prestodb/presto/tree/master/presto-native-execution)使用[ Velox](https://github.com/facebookincubator/velox) 作为其主要引擎来运行presto工作负载。
[Velox ](https://github.com/facebookincubator/velox)是一个 C++ 数据库库，它提供可重用、可扩展和高性能的数据处理组件。
查阅 [构建说明](https://github.com/prestodb/presto/tree/master/presto-native-execution#building)，就能开始。

# 在 IDE 中运行 Presto
## 概述
首次构建 Presto 后，可以将项目加载到 IDE 中并运行服务器。Presto 官方推荐使用 IntelliJ IDEA。因为 Presto 是一个标准的 Maven 项目，可以使用根 pom.xml 文件将其导入到您的 IDE 中。在 IntelliJ 中，从“快速启动”框中选择“打开项目”或从“文件”菜单中选择“打开”并选择根 pom.xml 文件。

在 IntelliJ 中打开项目后，仔细检查是否为项目正确配置了 Java SDK：

- 打开文件菜单，选择项目结构
- 在 SDK 部分，确保选择了1.8 JDK（如果没有，则创建一个）
- 在项目部分，确保项目语言级别设置为 8.0，因为 Presto 使用了多个 Java 8 语言功能

Presto 附带示例配置，应该开箱即用以进行开发。使用以下选项创建运行配置：

- 主类: `com.facebook.presto.server.PrestoServer`
- 虚拟机配置（VM Options）: `-ea -XX:+UseG1GC -XX:G1HeapRegionSize=32M -XX:+UseGCOverheadLimit -XX:+ExplicitGCInvokesConcurrent -Xmx2G -Dconfig=etc/config.properties -Dlog.levels-file=etc/log.properties`
- 工作目录（Working directory）：`$MODULE_WORKING_DIR$` or `$MODULE_DIR$`(取决于 IntelliJ 版本)
- 模块的 classpath ：`presto-main`

工作目录应该是 presto-main 子目录。在 IntelliJ 中，使用 `$MODULE_DIR$` 可以自动完成此操作。
此外，必须使用 Hive metastore Thrift 服务的位置配置 Hive 插件。
将以下内容添加到 VM 选项列表中，将 localhost:9083 替换为正确的主机和端口（如果没有 Hive 元存储，则使用以下值）：
```properties
-Dhive.metastore.uri=thrift://localhost:9083
```
## 为 Hive或 HDFS 使用 SOCKS
如果你的 Hive 元存储或 HDFS 集群不能被本地机器直接访问，你可以使用 SSH 端口转发来访问它。设置一个动态的 SOCKS 代理，让 SSH 在本地端口 1080 上监听：
```properties
ssh -v -N -D 1080 server
```
然后将以下内容添加到 VM 选项列表中：
```properties
-Dhive.metastore.thrift.client.socks-proxy=localhost:1080
```
## 运行命令行
启动 CLI 连接到服务器并运行 SQL 查询：
```properties
presto-cli/target/presto-cli-*-executable.jar
```
运行下面查询以查看集群中的节点：
```properties
SELECT * FROM system.runtime.nodes;
```
在示例配置中，Hive 连接器（Hive connector）挂载在 Hive 目录中，因此你可以运行以下查询以显示 Hive 数据库默认表中的表：
```properties
SHOW TABLES FROM hive.default;
```

# 开发
有关做出新贡献和审查它们的指南，请参阅贡献。
## 构建文档
要了解如何构建文档，请参阅[文档自述文件](https://github.com/prestodb/presto/blob/master/presto-docs/README.md)。
构建文档，使用 mvn clean 遇到这样的报错：![image.png](https://cdn.nlark.com/yuque/0/2023/png/35987817/1685936017517-87568fca-d1b3-4c50-b923-98d796f2e453.png#averageHue=%23300a24&clientId=u67585dca-d6c8-4&from=paste&height=390&id=u4b334411&originHeight=390&originWidth=989&originalType=binary&ratio=1&rotation=0&showTitle=false&size=60051&status=done&style=none&taskId=u170fbd96-df13-407d-ab01-2f580aa59f5&title=&width=989)
## Building the Web UI
Presto Web UI 由几个 React 组件组成，并以 JSX 和 ES6 编写。这个源代码被编译并打包成与浏览器兼容的JavaScript，然后被检查到Presto源代码中（在dist文件夹中）。你必须安装 Node.js 和 Yarn 来执行这些命令。要在做完修改后更新这个文件夹，只需运行：
```properties
yarn --cwd presto-main/src/main/resources/webapp/src install
```
如果没有改变 JavaScript 的依赖关系（ 即没有改变package.json ），运行速度会更快：
```properties
yarn --cwd presto-main/src/main/resources/webapp/src run package
```
为了简化迭代，您还可以在监视模式下运行，该模式会在检测到源文件更改时自动重新编译.
## 发行说明
在创建拉取请求时，PR 描述应包括其相关的发行说明。编写发行说明时请遵循发行[说明指南](https://github.com/prestodb/presto/wiki/Release-Notes-Guidelines)。

[Presto: SQL on Everything（全文翻译）](https://www.jianshu.com/p/de0a1de9f26e)
