---
title: Presto 安装
date: 2023-05-10 14:31:57 +0800
categories: [博客]
tags: [presto] 
---


## 使用 Docker 部署 Presto

## 使用压缩包安装 Presto

### 前提条件

#### JVM

Presto 使用 Java 编写
#### Python

Presto 的启动器脚本依赖 Python 2.7 或更改版本
### 安装

Presto 的二进制包由 Maven 中心仓库分发，服务端应用程序可以通过 `tar.gz`归档文件获得，链接是：[https://repo.maven.apache.org/maven2/io/prestosql/presto-server/](https://repo.maven.apache.org/maven2/io/prestosql/presto-server/)
详细步骤：
```bash
$ wget https://repo.maven.apache.org/maven2/\
 io/prestosql/presto-server/330/presto-server-330.tar.gz
$ tar xzvf presto-server-330.tar.gz
```
Presto 的安装目录包含以下子目录：

- **lib：**包含组成 Presto 服务端及其全部依赖关系的 Java 归档文件（JAR）
- **plugins：**包含 Presto 插件及其依赖关系，各个插件分开存放在不同的目录下。Presto 默认已经包含许多插件，我们也可以添加第三方插件。Presto 允许多种可插拔的组件，如连接器、函数和安全访问控制等与其集成。
- **bin：**包含 Presto 的启动脚本。这些脚本用于启动、停止、重启、终结 Presto 进程，以及获得运行该进程的状态。更多有关这些脚本的使用见下文。
- **etc：**配置文件所在目录。此目录由用户创建并提供 Presto 必需的配置文件。更多配置选项信息见下文。
- **var：**存放日志信息的数据目录。Presto 服务端第一次启动时会创建此目录，它默认情况下位于安装目录下。推荐将其配置为安装目录之外的位置，以便在升级过程中保留这些数据。
### 配置
启动 Presto 前，需要先提供一些配置文件：

- Presto 日志配置
- Presto 节点配置
- JVM 配置

默认情况下，配置文件应该存放在安装目录下的 etc 目录里。
除 JVM 配置之外，其它配置文件的格式遵循 Java properties 标准。也就是，在 Java properties 中，每个配置参数都是以 key=value 键值对（字符串）的格式存储为一行。
我们需要在 `$PFESTO_HOME/etc/`目录上创建这些基本的配置文件，这三个配置文件的内容是：
```properties
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8080
query.max-memory=5GB
query.max-memory-per-node=1GB
query.max-total-memory-per-node=2GB
discovery-server.enabled=true
discovery.uri=http://localhost:8080
```
```properties
node.environment=demo
```
```
-server
-Xmx4G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
-Djdk.nio.maxCachedBufferSize=2000000
-Djdk.attach.allowAttachSelf=true
```
这三个配置文件搞定后，Presto 就可以启动了，这些配置文件的更详细用法见下文。

### 添加数据源
到目前为止，虽然已经安装好了 Presto，但还不能就这样启动它。因为，你想要在 Presto 里查询一些外部数据，需要在 Presto 中添加一个数据源。

Persto 的 catalog 定义了用户可用的数据源。数据访问是由 Presto 连接器执行的，这个连接器由 connector.name 属性在 catalog 中配置。catalog 会将数据源中的所有 schema 和表暴露给 Presto。
:::info
例如：Hive 连接器将每个 Hive 数据库映射到一个 schema。如果想要使用 Hive 连接器将一 个名为 web 的 Hive 数据库（其中包含一张 clicks 表）暴露为名为 sitehive 的 catalog，那么我们需要在 catalog 文件中指定使用 Hive 连接器。我们可以使用完全限定名语法 catalog. schema.table 来访问 catalog，本例中就是 sitehive.web.clicks
:::

我们可以通过在 `$PFESTO_HOME/etc/catalog`目录下创建一个 catalog 属性文件来注册 catalog。该文件的名字就是 catalog 在 Presto 服务中的名字。
:::info
比如，我们创建了 `etc/catalog/cdh-hadoop.properties`、`etc/catalog/sales.properties`、`etc/catalog/web-traffic.properties` 和 `etc/catalog/mysql-dev.properties` 等 catalog 属性文件，则这些 catalog 在 Presto 里就是 `cdh-hadoop`、`sales`、`web-traffic` 和 `mysql-dev`。
:::

**我们可以使用 TPC-H 连接器探索 Presto。TPC-H 连接器内置于 Presto 中并提供了一系列的 schema 用于支持 TPC-H。**要配置 TPC-H 连接器，需要创建一个 catalog 属性文件`$PFESTO_HOME/etc/catalog/tpch.properties`，并将连接器配置为 tpch，如下：
```properties
connector.name=tpch
```
每个 catalog 文件都必须有 connector.name 属性。额外的属性由对应 Presto 连接器（每个都要研究）的实现决定。

## 运行 Presto
到现在为止，我们终于真正准备好启动 Presto 了。安装目录下包含了启动脚本，启动命令为：
```bash
$ bin/launcher run
```
run 命令将 Presto 启动为前台进程，Presto 的日志和其他输出将写入标准输出流（stdout）和标准错误流（stderr）。成功启动后会有一条日志：`INFO main io.prestosql.server.PrestoServer ======== SERVER STARTED`
:::info
在前台运行 Presto 有助于测试和快速检验进程能否正确启动，以及它是否使用了期望的配置。可以使用 Ctrl-C 组合键结束服务器进程。
:::
## 生产环境部署
### 配置细节
Presto 的配置由下面 个文件管理：

它们都默认存放在安装目录下的 `$PFESTO_HOME/etc/` 目录下。这些配置文件目录以及各个配置文件的默认位置，都可以使用启动器脚本的参数覆盖（具体如下文）。
### 服务端配置
Presto 服务端的配置由 `$PFESTO_HOME/etc/config.properties`文件提供。
Presto 服务端可以作为协调器或工作节点，也可以兼具两种角色。将服务端程序设定为只执行协调器的工作，并添加一系列其他服务端程序作为专用工作节点，这种配置方式可以提供最佳的性能并创建 Presto 集群。
配置文件 `$PFESTO_HOME/etc/config.properties`非常重要，因为它们决定了服务端程序是作为工作节点还是协调器，而这会影响资源的使用和配置。 
:::info
Presto 集群中的所有工作节点应该具有完全相同的设置。
:::
Presto 服务端的一些基础配置：

| **配置项** | **值** | **含义** |
| ---        | ---     | --- |
| `coordinator`                                     | true &#124; false | 设置当前实例是否为协调器，默认为 true |
| `node-scheduler.include-coordinator`              | true &#124; false | 设置是否将调度工作运行于协调器上，默认为 true。对于大集群，建议将此属性设为 false。 |
| `http-server.http.port` &#124; `http-server.https.port`  | 8080 &#124; 8443 | 指定服务端使用的 HTTP/HTTPS 连接端口。默认使用 HTTP 。 |
| `query.max-memory`                                | 5GB（示例值） | 查询可使用的最大内存 |
| `query.max-memory-per-node`                       | 1GB（示例值） | 在单个机器上可以使用的用户内存的最大值 |
| `query.max-total-memory-per-node`                 | 2GB（示例值） | 所有节点可使用的用户内存和系统内存加起来的最大值。系统内存是在执行过程中由读取器、写入器和网络缓冲等组件所使用的内存， |
| `discovery-server.enabled`                        | true &#124; false | Presto 使用发现服务来找到集群中的所有节点。每个 Presto 实例在启动时都会注册到发现服务。为了简化部署和避免运行额外的服务，Presto 协调器可以运行内置版本的发现服务。它与 Presto 共享 HTTP 服务器，因此使用相同的端口。通常此属性在协调器上设置为 true。在所有的工作节点上都必须删除此属性，从而禁用它。 |
| `discovery.uri`                                   | protocols://ip:port | 发现服务的 URI。使用 Presto 协调器内置的发现服务时，此属性的值应为 Presto 协调器 的 URI 且应指定正确的端口。这个 URI 不能以斜杠结尾。 |

