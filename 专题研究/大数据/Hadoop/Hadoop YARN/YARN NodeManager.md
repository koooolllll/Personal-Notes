# [[YARN]] - NodeManager

Node节点下的Container管理

1.  启动时向ResourceManager注册并定时发送心跳消息，等待ResourceManager的指令
2.  监控Container的运行，维护Container的生命周期，监控Container的资源使用情况
3.  启动或停止Container，管理任务运行时的依赖包（根据ApplicationMaster的需要，启动Container之前将需要的程序及其依赖包、配置文件等拷贝到本地）

## 内部结构

![](https://www.w3xue.com/img/up/2007-5-10/yarn-node-manager.png)

-   NodeStatusUpdater: 启动向ResourceManager注册，报告该节点的可用资源情况，通信的端口和后续状态的维护
-   ContainerManager: 接收RPC请求（启动、停止），资源本地化（下载应用需要的资源到本地，根据需要共享这些资源）
>PUBLIC: /filecache
>PRIVATE: /usercache//filecache
>APPLICATION: /usercache//appcache//（在程序完成后会被删除）
-   ContainersLauncher: 加载或终止Container
-   ContainerMonitor: 监控Container的运行和资源使用情况
-   ContainerExecutor: 和底层操作系统交互，加载要运行的程序

---