# Spark GraphX

在实际应用中，存在许多图计算问题，如最短路径、集群、网页排名、最小切割、连通分支等。图计算算法的性能直接关系到应用问题解决的高效性，尤其对于大型图（如社交网络和网络图）而言，更是如此。

## 图

图是由很多个节点(vertex)构成的，节点之间通过边(edge)进行连接。图在网络科学中被称为网络。实际应用中，有很多图结构数据：

-   计算机网络：由许多节点(计算机或者路由器)以及节点之间的边(网线)构成的网络；
-   城市的道路系统：由节点(路口)和边(道路)构成的图；
-   微信的社交网络：由节点（个人或公众号）和边（关注或点赞）构成的图；
-   淘宝的交易网络：由节点（个人或商品）和边（购买或收藏）构成的图；
-   网页的链接网络，由节点（网页）和边（链接）构成的图。

## 图处理技术

图处理技术包括**图数据库**、**图数据查询**、**图数据分析**和**图数据可视化**。

**图数据库**：Neo4j、Titan、OrientDB、DEX和InfiniteGraph等基于遍历算法的、实时的图数据库；  
**图数据查询**：对图数据库中的内容进行查询；  
**图数据分析**：Google Pregel、Spark GraphX、GraphLab等图计算软件。传统的数据分析方法侧重于事物本身，即实体，例如银行交易、资产注册等等。而图数据不仅关注事物，还关注事物之间的联系。例如，如果在通话记录中发现张三曾打电话给李四，就可以将张三和李四关联起来，这种关联关系提供了与两者相关的有价值的信息，这样的信息是不可能仅从两者单纯的个体数据中获取的。  
**图数据可视化**：OLTP风格的图数据库或者OLAP风格的图数据分析系统（或称为图计算软件），都可以应用图数据库可视化技术。需要注意的是，图可视化与关系数据可视化之间有很大的差异，关系数据可视化的目标是对数据取得直观的了解，而图数据可视化的目标在于对数据或算法进行调试。

通常，在进行大规模图数据分析时，都会组合使用图数据库（比如Neo4j）和图计算软件（比如Spark Gtaph X）。如果只是简单地存储实体之间的关系，那么，只要使用图数据库即可，不需要使用图计算软件。业界的一个发展趋势是，会出现可以同时处理OLAP和OLTP类型应用的图系统，也就是说，在图数据库与图计算框架之间的紧密集成，或者通过Neo4j这样的图数据库与Spark GraphX这样的图处理系统间的无缝互操作来实现，或者在将来某一天这些功能会出现在同一产品中。

## 图计算软件

目前典型的图计算软件（比如Google Pregel），都是基于BSP模型实现的分布式并行图处理系统。BSP是由哈佛大学Viliant和牛津大学Bill Mc Coll提出的并行计算模型，全称为“整体同步并行计算模型”（Bulk Synchronous Parallel Computing Model，BSP模型），又名“**大同步模型**”。创始人希望BSP模型像冯•诺依曼体系结构那样，架起计算机程序语言和体系结构间的桥梁，故又称为“桥模型”。一个BSP模型由大量通过网络相互连接的处理器组成，每个处理器都有快速的本地内存和不同的计算线程，一次BSP计算过程包括一系列全局超步（超步就是指计算中的一次迭代），每个超步主要包括3个组件。

• **局部计算**。每个参与的处理器都有自身的计算任务，它们只读取存储在本地内存中的值，不同处理器的计算任务都是异步并且独立的。  
• **通信**。处理器群相互交换数据，交换的形式是，由一方发起推送（put）和获取（get）操作。  
• **栅栏同步（Barrier Synchronization）**。当一个处理器遇到“路障”（或栅栏），会等其他所有处理器完成它们的计算步骤；每一次同步也是一个超步的完成和下一个超步的开始。图1是一个超步的垂直结构图。

![spark graphx 教程](https://www.hadoopdoc.com/media/editor/spark-graphx-tutorial.jpeg "spark graphx 教程")

谷歌公司在2003年到2004年公布了关于GFS、MapReduce和BigTable的3篇技术论文，成为后来云计算和Hadoop项目的重要基石。如今，谷歌在后Hadoop时代的新“三驾马车”——Caffeine、Dremel和Pregel，再一次影响着全球大数据技术的发展潮流。Caffeine主要为谷歌网络搜索引擎提供支持，使谷歌能够更迅速地添加新的链接（包括新闻报道以及博客文章等）到自身大规模的网站索引系统中。Dremel是一种可扩展的、交互式的实时查询系统，用于只读嵌套数据的分析。通过结合多级树状执行过程和列式数据结构，它能做到几秒内完成对万亿张表的聚合查询。系统可以扩展到成千上万的CPU上，满足谷歌上万用户操作PB级的数据，并且可以在2秒～3秒钟完成PB级别数据的查询。Pregel是一种基于BSP模型实现的并行图处理系统。为了解决大型图的分布式计算问题，Pregel搭建了一套可扩展的、有容错机制的平台，该平台提供了一套非常灵活的API，可以描述各种各样的图计算。Pregel作为分布式图计算的计算框架，主要用于图遍历、最短路径、PageRank计算等。