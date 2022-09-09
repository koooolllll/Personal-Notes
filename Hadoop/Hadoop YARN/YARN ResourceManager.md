# [[YARN]] ResourceManager

负责全局的资源管理和任务调度，把整个集群当成计算资源池，只关注分配，不管应用，且不负责容错

### 资源管理

1. 以前资源是每个节点分成一个个的Map slot和Reduce slot，现在是一个个Container，每个Container可以根据需要运行ApplicationMaster、Map、Reduce或者任意的程序
2. 以前的资源分配是静态的，目前是动态的，资源利用率更高
3. Container是资源申请的单位，一个资源申请格式：<resource-name, priority, resource-requirement, number-of-containers>, resource-name：主机名、机架名或*（代表任意机器）, resource-requirement：目前只支持CPU和内存
4. 用户提交作业到ResourceManager，然后在某个NodeManager上分配一个Container来运行ApplicationMaster，ApplicationMaster再根据自身程序需要向ResourceManager申请资源
5. YARN有一套Container的生命周期管理机制，而ApplicationMaster和其Container之间的管理是应用程序自己定义的

### 任务调度

1. 只关注资源的使用情况，根据需求合理分配资源
2. Scheluer可以根据申请的需要，在特定的机器上申请特定的资源（ApplicationMaster负责申请资源时的数据本地化的考虑，ResourceManager将尽量满足其申请需求，在指定的机器上分配Container，从而减少数据移动）

### 内部结构

![](https://www.w3xue.com/img/up/2007-5-10/yarn-resource-manager.png)

- Client Service: 应用提交、终止、输出信息（应用、队列、集群等的状态信息）
- Adaminstration Service: 队列、节点、Client权限管理
- ApplicationMasterService: 注册、终止ApplicationMaster, 获取ApplicationMaster的资源申请或取消的请求，并将其异步地传给Scheduler, 单线程处理
- ApplicationMaster Liveliness Monitor: 接收ApplicationMaster的心跳消息，如果某个ApplicationMaster在一定时间内没有发送心跳，则被任务失效，其资源将会被回收，然后ResourceManager会重新分配一个ApplicationMaster运行该应用（默认尝试2次）
- Resource Tracker Service: 注册节点, 接收各注册节点的心跳消息
- NodeManagers Liveliness Monitor: 监控每个节点的心跳消息，如果长时间没有收到心跳消息，则认为该节点无效, 同时所有在该节点上的Container都标记成无效，也不会调度任务到该节点运行
- ApplicationManager: 管理应用程序，记录和管理已完成的应用
- ApplicationMaster Launcher: 一个应用提交后，负责与NodeManager交互，分配Container并加载ApplicationMaster，也负责终止或销毁
- YarnScheduler: 资源调度分配， 有FIFO(with Priority)，Fair，Capacity方式
- ContainerAllocationExpirer: 管理已分配但没有启用的Container，超过一定时间则将其回收

---
