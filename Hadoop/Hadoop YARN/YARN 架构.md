# [[YARN]] 架构

在Hadoop 1.xx 之前，是没有yarn的，这就不得不提到之前的 MapReduce架构

## 旧的MapReduce架构

![](https://www.w3xue.com/img/up/2007-5-10/yarn-old-mapreduce.png)

-   **JobTracker:** 负责资源管理，跟踪资源消耗和可用性，作业生命周期管理（调度作业任务，跟踪进度，为任务提供容错）
-   **TaskTracker:** 加载或关闭任务，定时报告认为状态

此架构会有以下问题：

1.  JobTracker是MapReduce的集中处理点，存在单点故障
2.  JobTracker完成了太多的任务，造成了过多的资源消耗，当MapReduce job 非常多的时候，会造成很大的内存开销。这也是业界普遍总结出老Hadoop的MapReduce只能支持4000 节点主机的上限
3.  在TaskTracker端，以map/reduce task的数目作为资源的表示过于简单，没有考虑到cpu/ 内存的占用情况，如果两个大内存消耗的task被调度到了一块，很容易出现OOM
4.  在TaskTracker端，把资源强制划分为map task slot和reduce task slot, 如果当系统中只有map task或者只有reduce task的时候，会造成资源的浪费，也就集群资源利用的问题

总的来说就是**单点问题**和**资源利用率问题**

## YARN架构

![](https://www.w3xue.com/img/up/2007-5-10/yarn-architecture.png)

![](https://www.w3xue.com/img/up/2007-5-10/yarn-architecture-physical.png)

YARN就是将JobTracker的职责进行拆分，将资源管理和任务调度监控拆分成独立的进程：一个全局的资源管理和一个每个作业的管理（ApplicationMaster） ResourceManager和NodeManager提供了计算资源的分配和管理，而ApplicationMaster则完成应用程序的运行

-   **ResourceManager:** 全局资源管理和任务调度
-   **NodeManager:** 单个节点的资源管理和监控
-   **ApplicationMaster:** 单个作业的资源管理和任务监控
-   **Container:** 资源申请的单位和任务运行的容器

## 架构对比

![](https://www.w3xue.com/img/up/2007-5-10/hadoop-different.png)


- YARN架构下形成了一个通用的资源管理平台和一个通用的应用计算平台
- 避免了旧架构的单点问题和资源利用率问题
- 同时也让在其上运行的应用不再局限于MapReduce形式

---
