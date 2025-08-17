# MapReduce Reducer

Reducer 获取 Mapper 的输出数据（ 键值对 ），并对这些数据逐个处理，最终把最终处理结果存储到 HDFS。通常在 Hadoop 的 Reducer 阶段，我们会做聚合计算、求和计算等操作。

## 什么是 Hadoop Reducer

Reducer 处理 mapper 的输出数据，在处理这些数据的时候，它会生成一组新的输出数据。最后把数据存储到 HDFS。

Hadoop Reducer 以 mapper 输出的中间结果（键值对）作为输入数据，并对这些数据逐一执行 reducer 函数，比如可以对这些数据做聚合、过滤、合并等操作。Reducer 首先按 key 处理键值对的值，然后生成输出数据（零或多个键值对）。key 相同的键值对数据将会进入同一个 reducer，并且 reducer 是并行执行的，因为 reducer 之间不存在依赖关系。reducer 的数量由用户自己决定，默认值是 1。

## MapReduce Reducer 阶段

通过前面章节的介绍我们已经知道，Hadoop MapReduce 的 Reducer 过程包含 3 个阶段。下面对他们进行逐个详细介绍。

### Reducer 的 Shuffle 阶段

在本阶段，来自 mapper 的已经排好序的数据将作为 Reducer 的输入数据。在 Shuffle 阶段，MapReduce 通过 HTTP 协议拉取 mapper 输出数据的相应分区数据。

### Reducer 的排序阶段

在该阶段，来自不同 mapper 的输入数据将会按 key 重新做排序。shuffle 和排序阶段是同时进行的。

### Reduce 阶段

在 shuffle 和排序完成之后，reduce 任务对键值对数据做聚合操作。之后，OutputCollector.collect() 方法把 reduce 任务的输出结果写到 HDFS。Reducer 的输出不做排序。

## Reducer 任务的数量

一个作业的 Reduce 任务数量是怎么确定的呢？以及如何修改 Reduce 数量？下面我们带着问题来详细了解一下。

我们可以通过 Job.setNumreduceTasks(int) 方法设置 reduce 的数量。一般合适的 reduce 任务数量可以通过下面公式计算：

`(0.95 或者 1.75) * ( 节点数 * 每个节点最大的容器数量)`

使用 0.95 的时候，当 map 任务完成后，reducer 会立即执行并开始传输 map 的输出数据。使用 1.75 的时候，第一批 reducer 任务将在运行速度更快的节点上执行完成，而第二批 reducer 任务的执行在负载平衡方面做得更好。