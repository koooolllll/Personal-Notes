# Hive


## 什么是 Hive？

Apache Hive 是可实现大规模分析的分布式容错数据仓库系统。该数据仓库集中存储信息，您可以轻松对此类信息进行分析，从而做出明智的数据驱动决策。Hive 让用户可以利用 SQL 读取、写入和管理 PB 级数据。

Hive 建立在 Apache [[Hadoop]] 基础之上，后者是一种开源框架，可被用于高效存储与处理大型数据集。因此，Hive 与 Hadoop 紧密集成，其设计可快速对 PB 级数据进行操作。Hive 的与众不同之处在于它可以利用 Apache Tez 或 MapReduce 通过类似于 SQL 的界面查询大型数据集。

## Hive 如何运作？



Hive 旨在让非程序员熟悉 SQL，并使用名为 HiveQL 的类似于 SQL 的界面对 PB 级数据进行操作。传统关系数据库被用于对中小型数据集进行交互式查询，它在处理大型数据集时的表现并不理想。但 Hive 使用批处理，因此它可以快速操作非常大型的分布式数据库。Hive 会将 HiveQL 查询转换成在 Apache Hadoop 的分布式作业计划框架，亦即 Yet Another Resource Negotiator (YARN) 上运行的 [[Hadoop MapReduce]] 或 Tez 作业。它会查询存储在分布式存储解决方案，如 Hadoop 分布式文件系统 (HDFS) 或 Amazon S3 当中的数据。Hive 将其数据库和表元数据存储在元数据仓中，而元数据仓是一种可实现轻松数据提取和发现的基于数据库或文件的存储。

Hive 包含 HCatalog，它是从 Hive 元数据仓读取数据的表和存储管理层，可帮助 Hive、Apache Pig 和 MapReduce 之间的无缝集成。通过元数据仓，HCatalog 允许 Pig 和 MapReduce 使用与 Hive 相同的数据结构，从而无需为每个引擎重新定义元数据仓。自定义应用程序或第三方集成可以使用 WebHCat，而 WebHcat 是 HCatalog 用来访问和重复使用 Hive 元数据仓的 RESTful API。

Hive 依赖于 Hadoop [[HDFS]] 存储数据，Hive 将 HQL 转换成 MapReduce 执行，**所以说 Hive 是基于 Hadoop 的一个数据仓库工具，实质就是一款基于 HDFS 的 MapReduce 计算框架，对存储在 HDFS 中的数据进行分析和管理**

## Hive 的优点

### 快速

Hive 所采用的数据可通过批处理快速处理 PB 级数据。

 

### 熟悉

Hive 提供非程序员可以使用的熟悉的类似于 SQL 的界面。

 

### 可扩展

Hive 可根据您的需求被轻松分发与扩展。

 

## Apache Hive vs Apache HBase

Apache HBase 是一种 NoSQL 分布式数据库，可实现对 PB 级数据的随机、严格一致的实时访问。Apache Hive 则是一种分布式数据仓库系统，具有类似于 SQL 的查询功能。

| **特性**        | **APACHE HIVE**                                              | **APACHE HBASE**                                             |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **功能**        | 类似于 SQL 的查询引擎专为大容量数据存储而设计。支持多种文件格式。 | 通过自定义查询功能提供低延迟分布式键-值存储。数据采用列式存储格式。 |
| **处理类型**    | 采用 Apache Tez 或 MapReduce 计算框架的批处理。              | 实时处理。                                                   |
| **延迟**        | 中到高，取决于计算引擎的响应能力。对于相同数据卷，分布式执行模型提供比整体式查询系统（如 RDBMS）更出色的性能。 | 低，但可能不一致。HBase 架构的结构限制可能在密集写入负载期间导致延迟激增。 |
| **Hadoop 集成** | 在 Hadoop 顶部运行，与 Apache Tez 或 MapReduce 一起使用可进行处理，与 HDFS 或 Amazon S3 一起可进行存储。 | 在 HDFS 或 Amazon S3 顶部运行。                              |
| **SQL 支持**    | 通过 HiveQL 提供类似于 SQL 的查询功能。                      | 自身不提供 SQL 支持。您可以为 SQL 功能使用 Apache Phoenix。  |
| **Schema**      | 适用于全部表的定义 Schema。                                  | 无 Schema。                                                  |
| **数据类型**    | 支持结构化和非结构化数据。为常见的 SQL 数据类型提供原生支持，如 INT、FLOAT 和 VARCHAR。 | 仅支持非结构化数据。由用户定义数据字段到 Java 支持的数据类型的映射。 |

