# Hadoop MapReduce

## 概述


Hadoop Map/Reduce是一个使用简易的软件框架，基于它写出来的应用程序能够运行在由上千个商用机器组成的大型集群上，并以一种可靠容错的方式并行处理上T级别的数据集。

一个Map/Reduce _作业（job）_ 通常会把输入的数据集切分为若干独立的数据块，由 _map任务（task）_以完全并行的方式处理它们。框架会对map的输出先进行排序， 然后把结果输入给_reduce任务_。通常作业的输入和输出都会被存储在文件系统中。 整个框架负责任务的调度和监控，以及重新执行已经失败的任务。

通常，Map/Reduce框架和[分布式文件系统](https://hadoop.apache.org/docs/r1.0.4/cn/hdfs_design.html)是运行在一组相同的节点上的，也就是说，计算节点和存储节点通常在一起。这种配置允许框架在那些已经存好数据的节点上高效地调度任务，这可以使整个集群的网络带宽被非常高效地利用。

Map/Reduce框架由一个单独的master JobTracker 和每个集群节点一个slave TaskTracker共同组成。master负责调度构成一个作业的所有任务，这些任务分布在不同的slave上，master监控它们的执行，重新执行已经失败的任务。而slave仅负责执行由master指派的任务。

应用程序至少应该指明输入/输出的位置（路径），并通过实现合适的接口或抽象类提供map和reduce函数。再加上其他作业的参数，就构成了_作业配置（job configuration）_。然后，Hadoop的 _job client_提交作业（jar包/可执行程序等）和配置信息给JobTracker，后者负责分发这些软件和配置信息给slave、调度任务并监控它们的执行，同时提供状态和诊断信息给job-client。

虽然Hadoop框架是用JavaTM实现的，但Map/Reduce应用程序则不一定要用 Java来写 。

-   Hadoop Streaming是一种运行作业的实用工具，它允许用户创建和运行任何可执行程序 （例如：Shell工具）来做为mapper和reducer。
-   Hadoop Pipes是一个与[SWIG](http://www.swig.org/)兼容的C++ API （没有基于JNITM技术），它也可用于实现Map/Reduce应用程序。

## 了解 Hadoop MapReduce

MapReduce 是如何工作的呢？

MapReduce 会把打任务分成小任务，每个小任务可以在集群并行执行。每个小任务都会输出计算结果，这些结果数据后续被汇总并输出最终结果。

Hadoop MapReduce 具有较好的扩展性，它可以在很多机器上跑。集群里面单个机器可能无法执行大任务，但可以执行大任务分割后的小任务。这是 MapReduce 比较核心的机制。

## Apache MapReduce 术语

本节主要介绍 MapReduce 相关概念和术语。如，什么是 Map 和 Reduce，什么是 job，task，task attempt 等。

MapReduce 是 Hadoop 的数据处理组件。MapReduce 程序把输入数据转换成特定格式的输出数据。一个 MapReduce 程序主要就做下面这两步：

-   Map
-   Reduce

在 Map 和 Reduce 中间还有一个处理阶段，叫做 **Shuffle** 和 **排序**操作。

下面介绍一下 MapReduce 里面的一些关键术语。

-    MapReduce Job（作业）

一个 MapReduce Job 过程分成两个阶段：Map 阶段和 Reduce 阶段。每个阶段都用 key/value 作为输入和输出；每个阶段都需要定义函数，也就是 map 函数和 reduce 函数；可以简单认为 map 函数是对原始数据提出出有用的部分，而 reduce 函数则是对提取出来的数据进行处理。

-    MapReduce Task

MapReduce 里面的 task 可以分两种，即 Map task 和 Reduce task，即处理分片数据的 Mapper 和 Reducer 任务，这里的 Mapper 和 Reducer 的业务逻辑由开发者定义。

-   Task Attempt

Task Attempt，即任务尝试。集群的机器在任何时间都可能发生故障，比如，正在处理数据的机器挂了，MapReduce 把任务重新调度到其他机器节点。当然这里的重新调度次数并非不受限制的，它是有上限的，默认是 4 次，如果一个任务（Mapper 任务或者 Reducer 任务）失败 4 次，那么整个 Job 就被认为失败了。对于高优先级的作业或者大型作业，这个值可以调高一点。


## 目录


1. [[MapReduce 键值对]]
2. [[MapReduce 工作原理]]
3. [[MapReduce 计数器]]
4. [[MapReduce 推测执行]]
5. [[MapReduce 性能优化]]
6. [[MapReduce 只有 Map 阶段的 job]]
7. [[MapReduce 数据本地化]]