# Hadoop

[Hadoop官网](https://hadoop.apache.org/)

Apache™Hadoop®项目开发开源软件，用于可靠，可扩展，分布式计算。

Apache Hadoop软件库是一个框架，允许使用简单的编程模型在计算机群中分布式处理大型数据集。它旨在从单个服务器扩展到数千台机器，每台提供本地计算和存储。该库本身不是依靠硬件来提供高可用性，而是设计用于检测和处理应用程序层的故障，因此在一组计算机之上提供高度可用的服务，每个计算机都可能容易出现故障。

## 模块

该项目包括这些模块：

-   **[[Hadoop Common]]**：支持其他 Hadoop 模块的通用实用程序。
-   **Hadoop 分布式文件系统 ([[HDFS]]™)**：一种分布式文件系统，可提供对应用程序数据的高吞吐量访问。
-   **Hadoop [[YARN]]**：作业调度和集群资源管理的框架。
-   **Hadoop MapReduce**：基于 YARN 的系统，用于并行处理大型数据集

## 相关项目

Apache 其他与 Hadoop 相关的项目包括：

-   [**Ambari™**](https://ambari.apache.org/)：一个基于 Web 的工具，用于配置、管理和监控 Apache Hadoop 集群，包括对 Hadoop HDFS、Hadoop MapReduce、Hive、HCatalog、HBase、ZooKeeper、Oozie、Pig 和 Sqoop 的支持。Ambari 还提供了一个仪表板，用于查看集群健康状况，例如热图，并能够直观地查看 MapReduce、Pig 和 Hive 应用程序以及以用户友好的方式诊断其性能特征的功能。
-   [**Avro™**](https://avro.apache.org/)：数据序列化系统。
-   [**Cassandra™**](https://cassandra.apache.org/)：一个可扩展的多主数据库，没有单点故障。
-   [**Chukwa™**](https://chukwa.apache.org/)：用于管理大型分布式系统的数据收集系统。
-   [**HBase™**](https://hbase.apache.org/)：一个可扩展的分布式数据库，支持大型表的结构化数据存储。
-   [**Hive™**](https://hive.apache.org/)：提供数据汇总和即席查询的数据仓库基础设施。
-   [**Mahout™**](https://mahout.apache.org/)：可扩展的机器学习和数据挖掘库。
-   [**Ozone™**](https://ozone.apache.org/)：Hadoop 的可扩展、冗余和分布式对象存储。
-   [**Pig™**](https://pig.apache.org/)：用于并行计算的高级数据流语言和执行框架。
-   [**Spark™**](https://spark.apache.org/)：用于 Hadoop 数据的快速通用计算引擎。Spark 提供了一个简单而富有表现力的编程模型，支持广泛的应用程序，包括 ETL、机器学习、流处理和图计算。
-   [**Submarine**](https://submarine.apache.org/)：一个统一的 AI 平台，允许工程师和数据科学家在分布式集群中运行机器学习和深度学习工作负载。
-   [**Tez™**](https://tez.apache.org/)：基于 Hadoop YARN 构建的通用数据流编程框架，它提供了一个强大而灵活的引擎来执行任意 DAG 任务来处理批处理和交互式用例的数据。Tez 正在被 Hadoop 生态系统中的 Hive™、Pig™ 和其他框架以及其他商业软件（例如 ETL 工具）采用，以取代 Hadoop™ MapReduce 作为底层执行引擎。
-   [**ZooKeeper™**](https://zookeeper.apache.org/)：分布式应用程序的高性能协调服务。