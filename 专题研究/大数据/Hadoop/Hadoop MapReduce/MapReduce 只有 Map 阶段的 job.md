# MapReduce 只有 Map 阶段的 job

在 Hadoop，只有 Map 任务的作业就是 mapper 处理了所有的数据处理任务，作业的整个过程没有 reducer 参与，而 mapper 的输出结果就是作业的最终输出结果。

![只有Map的Mapreduce作业](https://www.hadoopdoc.com/media/editor/file_1570258133000_20191005144855904374.png "只有Map的Mapreduce作业")

## 仅 Map 阶段的 MapReduce 作业

![只有 map 的mapreduce作业原理](https://www.hadoopdoc.com/media/editor/file_1570258195000_20191005144957180756.png "只有 map 的mapreduce作业原理")

MapReduce 是一种软件框架，利用该框架可以快速开发出处理大数据量的应用程序。MapReduce 框架的两个重要任务是：Map 任务和 Reduce 任务。Map 阶段接收一批数据，并把这些数据转换键值对。Reduce 阶段以 map 阶段的输出结果为输入数据，并且对这些数据基于 key 做聚合。

从上图我们可以知道，在一个 MapReduce job 里面有两组并行进程，即 map 和 reduce；在 map 进程，首先输入数据会被分发到多个 map 节点 ，然后每个单词被标记并生成键值对 。

第一个 mapper 节点接收到 3 个单词分别是 lion，tiger 和 river，因此节点的输出将会是 3 个键值对，其中 key 是单词本身，value 设置为 1。其他节点的处理过程跟第一个 mapper 一致。这些键值对之后会被传递给 reducer 节点，这样就会发生 shuffle 操作，也就是说，所有具有相同 key 的键值对会被分发到相同的节点。因此，在 reduce 进程中，就是对具有相同 key 的 value 进行运算操作。

现在让我们来考虑这样一个场景，如果我们只需要执行分发的操作，而不需要聚合的操作，那么这种情况下，我们就会更倾向于使用只有 map 阶段的作业。 在仅发生 Map 阶段的 Hadoop 作业中，map 阶段利用 InputSplit 就完成了所有的任务，并不需要 reducer 的参与。这里 map 的输出数据就是作业的最终输出结果了。

## 如何避免 Reduce 阶段的产生

为了避免触发 Reduce 任务，你可以在驱动程序（driver program）通过调用下面的方法来把 reduce 任务数设置为 0。

`job.setNumreduceTasks(0)`

设置完之后，Hadoop 作业在执行的时候就只有 map 阶段，而不会发生 reduce 了。

## 优点

在 map 和 reduce 两个阶段之间，包含排序和 shuffle 阶段。排序和 shuffle 负责对 key 升序排序和基于相同 key 对 value 进行分组。这个阶段的开销是非常大的。如果 reduce 阶段不是必要的，那么我们可以通过把 reduce 任务数设置为 0 来避免 reduce 阶段的发生，而且排序和 shuffle 阶段也不会发生。同时还可以避免数据传输的网络开销。

在一般的 MapReduce 作业中，mapper 的输出在被发送给 reducer 之前会先把输出数据写到本地磁盘，但在只有 map 的作业里，map 的输出会被直接写到 HDFS，这将节省了作业运行时间和 reduce 的开销。另外 partitioner 和 combiner 这两步在只有 map 的作业里是不需要的，所以这也会让作业运行的更快。