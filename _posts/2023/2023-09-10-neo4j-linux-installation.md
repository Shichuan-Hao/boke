---
title: Neo4j 的安装
date: 2023-09-10 14:31:57 +0800
categories: [博客]
tags: [neo4j, graph database] 
---


[原文地址](https://neo4j.com/docs/operations-manual/current/installation/linux/)

您可以使用 Debian 或 RPM 软件包或从 Tar 存档在 Linux 上安装 Neo4j。
本节描述以下内容：

- 在 Debian 和基于 Debian 的发行版上安装 Neo4j
   - 安装
   - 文件位置
   - 操作
- 使用 Neo4j RPM 包部署 Neo4j
   -  在 Red Hat、CentOS、Fedora 或 Amazon Linux 上安装
   -  在 SUSE 上安装
   -  离线安装
- 从 tarball 在 Linux 上安装 Neo4j
   -  Unix 控制台应用程序
   -  Linux 服务
   -  设置打开文件的数量
- 将 Neo4j 安装为系统服务
   -  **配置**
   - 控制服务
   - 日志
### 在 Debian 和基于 Debian 的发行版 (.deb)
### Red Hat、CentOS、Fedora 和 Amazon Linux 发行版 (.rpm)
您可以使用 Neo4j RPM 软件包在 Red Hat、CentOS、Fedora 或 Amazon Linux 发行版上部署 Neo4j。
#### Java 先决条件
Neo4j 5 需要 Java 17 运行时。
##### OpenJDK Java 17
我们支持的大多数 Linux 发行版默认都提供 OpenJDK Java 17。因此，如果您使用 OpenJDK Java，则不需要额外的设置，安装 Neo4j 时包管理器将安装正确的 Java 依赖项。
##### Oracle Java 17
我们支持的大多数 Linux 发行版默认都提供 OpenJDK Java 17。因此，如果您使用 OpenJDK Java，则不需要额外的设置，安装 Neo4j 时包管理器将安装正确的 Java 依赖项。

我们为 Oracle Java 17 提供了一个适配器，必须在 Neo4j 之前安装。该适配器不包含任何代码，但会阻止包管理器将 OpenJDK 17 作为依赖项安装，尽管已安装 Oracle Java 17。

1. 从[ Oracle ](https://www.oracle.com/java/technologies/downloads/)网站下载并安装 Oracle Java 17 JDK：
2. 安装适配器：
```shell
sudo yum install https://dist.neo4j.org/neo4j-java17-adapter.noarch.rpm
```
适配器包的 SHA-256 可以根据 [https://dist.neo4j.org/neo4j-java17-adapter.noarch.rpm.sha256](https://dist.neo4j.org/neo4j-java17-adapter.noarch.rpm.sha256) 进行验证。
##### Zulu JDK 17 或 Corretto 17
如果您希望使用非默认 JDK，则必须在开始 Neo4j 安装之前安装它。否则，您的包管理器将为您的操作系统安装默认的 java 发行版，通常是 OpenJDK。

安装说明可以在制造商的网站上找到：

- [Zulu JDK](https://www.azul.com/downloads/?package=jdk) 🚀
- [Amazon Corretto JDK](https://aws.amazon.com/corretto) 🚀
#### 在 Red Hat、CentOS 或 Amazon Linux 上安装
#### 设置存储库
要使用 Neo4j 通用版本的存储库，请以 root 身份运行以下命令来添加存储库：
```shell
rpm --import https://debian.neo4j.com/neotechnology.gpg.key
cat << EOF >  /etc/yum.repos.d/neo4j.repo
[neo4j]
name=Neo4j RPM Repository
baseurl=https://yum.neo4j.com/stable/5
enabled=1
gpgcheck=1
EOF
```
#### 安装 Neo4j
根据您使用的版本，使用以下命令以 root 身份安装 Neo4j。

- 社区版 Community Edition
```shell
yum install neo4j-5.11.0
```

- 企业版 Enterprise Edition

从 Neo4j 5.4 开始，您需要在运行 Neo4j 企业版之前接受商业或评估许可协议。以下示例使用交互式提示：
```shell
yum install neo4j-enterprise-5.11.0
```
在允许完成交互式安装之前，您必须选择[商业许可证（commercial license）](https://neo4j.com/terms/licensing/)或[评估许可证（evaluation license）](https://neo4j.com/terms/enterprise_us/)。对于非交互式安装，您可以将 `NEO4J_ACCEPT_LICENSE_AGREEMENT` 设置为 `yes`（对于商业许可证）或 `eval`（对于评估许可证），如下例所示：
```shell
yum install neo4j-enterprise-5.11.0
```
```shell
NEO4J_ACCEPT_LICENSE_AGREEMENT=yes
yum install neo4j-enterprise-5.11.0
```
Neo4j 默认支持安全增强型 Linux (SELinux)。
#### Install on SUSE
对于基于 SUSE 的发行版，步骤如下：

1. 使用以下命令作为 root 添加存储库：
```shell
zypper addrepo --refresh https://yum.neo4j.com/stable/5 neo4j-repository
```

2. 根据您使用的版本，使用以下命令以 root 身份安装 Neo4j：
   1. 社区版
```shell
zypper install neo4j-5.11.0
```

   2. 企业版
```shell
zypper install neo4j-enterprise-5.11.0
```
从 Neo4j 5.4 开始，您需要在运行 Neo4j 企业版之前接受商业或评估许可协议。以下示例使用交互式提示：
```shell
zypper install neo4j-enterprise-5.11.0
```
在允许完成交互式安装之前，您必须选择[商业许可证（commercial license）](https://neo4j.com/terms/licensing/)或[评估许可证（evaluation license）](https://neo4j.com/terms/enterprise_us/)。对于非交互式安装，您可以将 `NEO4J_ACCEPT_LICENSE_AGREEMENT` 设置为 `yes`（对于商业许可证）或 `eval`（对于评估许可证），如下例所示：
```shell
NEO4J_ACCEPT_LICENSE_AGREEMENT=yes zypper install neo4j-enterprise-5.11.0
```
#### 离线安装
如果您无法访问 `https://yum.neo4j.com/stable/5` 使用 RPM 安装 Neo4j（可能是由于防火墙），您将需要通过具有相关访问权限的备用计算机获取 Neo4j，然后移动 RPM 手动打包。
:::info
**💥 注意**
使用这种方法将意味着离线机器将不会收到通常使用 yum 安装 Neo4j 时自动下载安装的依赖项：

- Neo4j Cypher Shell
- Java

有关支持的 Java 版本的信息，请参阅系统要求。
:::
#####  使用 RPM 安装程序安装 Neo4j

1. 运行以下命令获取所需的 Neo4j RPM 包：
   1. 企业版：
```shell
curl -O https://dist.neo4j.org/rpm/neo4j-enterprise-5.11.0-1.noarch.rpm
```

   2. 教育版：
```shell
curl -O https://dist.neo4j.org/rpm/neo4j-5.11.0-1.noarch.rpm
```

2. 手动将下载的 RPM 包移动到离线机器上。在安装 Neo4j 之前，您必须手动安装所需的 Java 17 包。
3. 根据您使用的版本，使用以下命令以 root 身份安装 Neo4j：
   1. 社区版
```shell
rpm --install neo4j-5.11.0-1.noarch.rpm
```

   2. 企业版

从 Neo4j 5.4 开始，您需要在运行 Neo4j 企业版之前接受商业或评估许可协议。以下示例使用交互式提示：
```shell
rpm --install neo4j-enterprise-5.11.0
```
在允许完成交互式安装之前，您必须选择[商业许可证（commercial license）](https://neo4j.com/terms/licensing/)或[评估许可证（evaluation license）](https://neo4j.com/terms/enterprise_us/)。对于非交互式安装，您可以将 `NEO4J_ACCEPT_LICENSE_AGREEMENT` 设置为 `yes`（对于商业许可证）或 `eval`（对于评估许可证），如下例所示：
```shell
NEO4J_ACCEPT_LICENSE_AGREEMENT=yes rpm --install neo4j-enterprise-5.11.0-1.noarch.rpm
```
##### 使用 RPM 安装程序安装 Cypher Shell

1. [从 Neo4j 下载中心](https://neo4j.com/download-center/#cyphershell)下载了 Cypher Shell RPM 安装程序。
2.  通过以 root 用户身份运行以下命令来安装 Cypher Shell：

**从 4.4.0 或更高版本离线升级**
在开始之前，您需要预安装 Java 17 并设置为默认 Java 版本。如果使用 Oracle Java 17，则存在与 Oracle Java 先决条件相同的依赖性问题。
由于 Neo4j 和 Cypher Shell 之间的严格依赖关系，两个软件包必须同时升级。在离线计算机上以 root 身份运行以下命令，以同时安装 Neo4j Cypher Shell 和 Neo4j：
```shell
rpm -U <Cypher Shell RPM file name> <Neo4j RPM file name>
```
 这必须是一个命令，并且 Neo4j Cypher Shell 必须是命令中的第一个包。
####  系统启动时自动启动服务
要使 Neo4j 在系统启动时自动启动，请运行以下命令：
```shell
systemctl enable neo4j
```
:::info
** **💡**注意**
第一次启动数据库之前，建议使用 neo4j-admin 的 set-initial-password 命令定义本机用户 neo4j 的密码。

如果未使用此方法显式设置密码，则会将其设置为默认密码 neo4j。在这种情况下，首次登录时系统将提示您更改默认密码。

详细信息，请参阅设置初始密码。
:::
 
有关操作 Neo4j 系统服务的更多信息，请参阅 Neo4j 系统服务。
### Linux 可执行文件 (.tar)
在通过 tarball 在 Linux 上安装 Neo4j 并将其作为控制台应用程序或服务运行之前，请检查[系统要求](https://neo4j.com/docs/operations-manual/current/installation/requirements/)以查看您的设置是否合适。
####  从 tarball 安装 Neo4j

1.  如果尚未安装，请获取[ OpenJDK 17 ](https://openjdk.org)或[ Oracle Java 17](https://www.oracle.com/java/technologies/downloads/)。
2. 从[ Neo4j ](https://neo4j.com/download-center/#community)下载中心下载最新的 Neo4j tarball 并解压它：`tar zxf neo4j-enterprise-5.11.0-unix.tar.gz`
3. 将提取的文件移至服务器的 `/opt` 目录，并创建指向它的软连接：
```shell
mv neo4j-enterprise-5.11.0 /opt/
ln -s /opt/neo4j-enterprise-5.11.0 /opt/neo4j
```

4. 创建 neo4j 用户和组：
```shell
groupadd neo4j
useradd -g neo4j neo4j -s /bin/bash
```

5. 使用以下选项之一为目录提供正确的所有权：
   1. Ubuntu `chown -R neo4j:adm /opt/neo4j-enterprise-5.11.0`
   2. ReaHat `chown -R neo4j /opt/neo4j-enterprise-5.11.0`
6. 从 Neo4j 5.4 开始，您需要在运行 Neo4j 企业版之前接受商业或评估许可协议。如果您使用的是社区版，则可以跳过此步骤。
   1. 用以下选项之一接受商业许可协议。有关可用协议的详细信息，请参阅[ Neo4j 许可页面](https://neo4j.com/terms/licensing/)。
      1. 设置环境变量 `$NEO4J_ACCEPT_LICENSE_AGREEMENT=yes`。
      2. 运行 `$NEO4J_HOME /bin/neo4j-admin` 服务器许可证 `--accept-commercial`
   2.  使用以下选项之一接受[ Neo4j 软件的 Neo4j 评估协议](https://neo4j.com/terms/enterprise_us/)。
      1. 设置环境变量 `NEO4J_ACCEPT_LICENSE_AGREEMENT=eval`。
      2. 运行 `$NEO4J_HOME /bin/neo4j-admin` 服务器许可证 `--accept-evaluation`。
7. 启动 Neo4j：
   1. 要将 Neo4j 作为控制台应用程序运行，请使用：`$NEO4J_HOME/bin/neo4j console`
   2. 要在后台进程中运行 Neo4j，请使用：`$NEO4J_HOME/bin/neo4j start`
8. 在 Web 浏览器中打开 `http://localhost:7474`。
9. 使用用户名 neo4j 和默认密码 neo4j 进行连接。然后系统将提示您更改密码。
10.  通过在控制台中键入 Ctrl-C 来停止服务器。
####  配置 Neo4j 在系统启动时自动启动
 您可以创建 Neo4j 服务并将其配置为在系统启动时自动启动。

1. 创建文件 `/lib/systemd/system/neo4j.service` 并包含以下内容：
```shell
[Unit]
Description=Neo4j Graph Database
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/opt/neo4j/bin/neo4j console
Restart=on-abnormal
User=neo4j
Group=neo4j
Environment="NEO4J_CONF=/opt/neo4j/conf" "NEO4J_HOME=/opt/neo4j"
LimitNOFILE=60000
TimeoutSec=120

[Install]
WantedBy=multi-user.target
```

2. 重新加载 systemctl 以获取新的服务文件并将 Neo4j 配置为在启动时启动：`systemctl enable neo4j`
3. 启动 Neo4j：`systemctl start neo4j`
4. 检查新创建的服务的状态：`systemctl status neo4j`
5. 重新启动系统（如果需要）以验证 Neo4j 是否在启动时重新启动：`reboot`

 有关操作 Neo4j 系统服务的更多信息，请参阅下文中的 Neo4j 系统服务。
#### 设置打开文件的数量
 Linux 平台对每个用户和会话同时打开的文件数量施加上限。要检查当前会话的限制，请运行命令 `ulimit -n`。默认值为 1024。

但是，如果您遇到“打开文件过多”或“无法 `stat()` 目录”的异常，**则必须将限制增加到 40000 或更多**，具体取决于您的使用模式。当使用许多索引或者服务器安装看到太多打开的网络连接或套接字时尤其如此。

一个快速的解决方案是命令 `ulimit -n the-new-limit` ，但它只会为 root 用户设置新的限制，并且只会影响当前会话。如果您想在系统范围内设置该值，请按照适用于您的平台的说明进行操作。

 以下步骤将 Ubuntu 16.04 LTS、Debian 8、CentOS 7 或更高版本下用户 neo4j 的打开文件描述符限制设置为 60000。

**将 Neo4j 作为服务运行**

1. 使用 root 权限打开 neo4j.service 文件
```shell
sudo systemctl edit neo4j.service
```

2.  将以下内容附加到在配置 Neo4j 中创建的 [Service] 部分[上面]，使其在系统启动时自动启动：
```shell
[Service]
...
LimitNOFILE=60000
```
#### 作为交互式用户运行 Neo4j（例如，出于测试目的）

1. 在文本编辑器中使用 root 权限打开 user.conf 文件。此示例使用 Vim：`sudo vi /etc/systemd/user.conf`
2. 取消注释并定义 DefaultLimitNOFILE 的值，该值可在 `[Manager]` 部分：
```shell
[Manager]
...
DefaultLimitNOFILE=60000
```

3. 打开 `/etc/security/limits.conf` 文件：`sudo vi /etc/security/limits.conf`。
4. 定义以下值：
```shell
neo4j	soft	nofile	60000
neo4j	hard	nofile	60000
```

5. 重新加载系统设置：`sudo systemctl daemon-reload`
6. 重新启动您的机器。
###  Neo4j 系统服务
 本节介绍了 Neo4j 系统服务的配置和操作。它假设您的系统具有 systemd，这是大多数 Linux 发行版的情况。
:::info
**💡 设置打开文件的数量。**
有关如何设置用户可以打开的并发文件数的说明，请参阅[设置打开文件数](https://neo4j.com/docs/operations-manual/current/installation/linux/tarball/#linux-open-files)。
:::
#### 配置
配置存储在 `/etc/neo4j/neo4j.conf`中。请参阅[默认文件位置](https://neo4j.com/docs/operations-manual/current/configuration/file-locations/)以获取各种包的文件所在位置的完整目录。
#### 控制服务
系统服务由 `systemctl`命令控制。它接受许多命令：`systemctl {start|stop|restart} neo4j`
服务定制可以放置在服务覆盖文件中。要编辑您的特定选项，请执行以下命令，这将打开相应文件的编辑器：`systemctl edit neo4j`
然后将所有自定义设置放在 [Service] 部分下。以下示例列出了某些用户可能有兴趣更改的默认值：
```
[Service]
# The user and group which the service runs as.
User=neo4j
Group=neo4j
# If it takes longer than this then the shutdown is considered to have failed.
# This may need to be increased if the system serves long-running transactions.
TimeoutSec=120
```
您可以通过以下方式打印有效服务，包括可能的覆盖：`systemctl cat neo4j`。
如果更改任何设置，请记住重新启动 neo4j：`systemctl restart neo4j`。
## Log
neo4j 日志被写入 `journald`，可以使用 `journalctl` 命令查看：
```
journalctl -e -u neo4j
```
 
一段时间后，`journald` 会自动轮换日志，并且默认情况下，它通常不会在重新启动后持续存在。请参阅 `man journald.conf` 了解更多详细信息。
