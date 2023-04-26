# Hive 架构
![hive架构图](https://www.hadoopdoc.com/media/editor/file_1573354918000_20191110110159565902.png "hive架构图")

从 Hive 架构图可以知道，Hadoop 和 MapReduce 是 Hive 架构的根基。

Hive 架构包括如下组件：

-   CLI（command line interface）
-   JDBC/ODBC
-   Thrift Server
-   WEB GUI
-   Metastore
-   Driver（Complier、Optimizer 和 Executor）

这些组件可以分为两大类：

**服务端组件**

-   Driver 组件：该组件包括 Complier、Optimizer 和 Executor，它的作用是将 HiveQL 语句进行解析、编译优化，生成执行计划，然后调用底层的 MapReduce 计算框架。
    
-   Metastore 组件：元数据服务组件，这个组件存储 Hive 的元数据，hive 的元数据存储在关系数据库里，hive 支持的关系数据库有 derby、mysql。元数据对于 hive 十分重要，因此 hive 支持把 metastore 服务独立出来，安装到远程的服务器集群里，从而解耦 hive 服务和 metastore 服务，保证 hive 运行的健壮性。
    
-   Thrift 服务：thrift 是 facebook 开发的一个软件框架，它用来进行可扩展且跨语言的服务的开发，hive 集成了该服务，能让不同的编程语言调用 hive 的接口。
    

**客户端组件**

-   CLI：command line interface，命令行接口。
    
-   Thrift 客户端：上面的架构图里没有写上 Thrift 客户端，但是 hive 架构的许多客户端接口是建立在 thrift客户端之上，包括 JDBC 和 ODBC 接口。
    
-   WEB GUI：hive 客户端提供了一种通过网页的方式访问 hive 所提供的服务。这个接口对应 hive 的 hwi 组件（hive web interface），使用前要启动 hwi 服务。
    

## Hive 数据处理流程

![hive数据处理流程](https://www.hadoopdoc.com/media/editor/file_1573355104000_20191110110505194852.png)

-   UI 调用 Driver 的 execute 接口。
    
-   Driver 为查询创建会话句柄，并将查询发送给 compiler 以生成执行计划。
    
-   compiler 需要元数据信息，所以它会给 metastore 发送getMetaData 请求，然后接受从 metastore 接收 sendMetaData 请求。
    
-   compiler 利用 metastore 返回的元数据信息对查询里面的表达式进行类型检查。之后生成计划，该执行计划是阶段的有向无环图（DAG），每个阶段可能是一个 MapReduce 作业，也可能是元数据操作，也有可能是 HDFS 操作。计划中的 MapReduce 阶段包括 map 操作树和 reduce 操作树。
    
-   现在 Execution Engine 将各个阶段提交个适当的组件执行。在每个任务（mapper / reducer）中，表或者中间输出相关的反序列化器从 HDFS 读取行，并通过相关的操作树进行传递。一旦这些输出产生，将通过序列化器生成临时的 HDFS 文件（这个只发生在只有 Map 没有 reduce 的情况），生成的 HDFS 临时文件用于执行计划后续的 Map/Reduce 阶段。对于 DML 操作，临时文件最终移动到表的位置。该方案确保不出现脏数据读取（文件重命名是HDFS中的原子操作）
    
-   对于查询，临时文件的内容由 Execution Engine 直接从 HDFS 读取，作为从 Driver Fetch API 的一部分。