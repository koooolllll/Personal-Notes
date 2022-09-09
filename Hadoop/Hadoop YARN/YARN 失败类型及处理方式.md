# [[YARN]] 失败类型及处理方式

1.  程序问题
2.  进程崩溃
3.  硬件问题

## ARN 失败类型及处理方式

### 任务失败

1.  运行时异常或者JVM退出都会报告给ApplicationMaster
2.  通过心跳来检查挂住的任务(timeout)，会检查多次（可配置）才判断该任务是否失效
3.  一个作业的任务失败率超过配置，则认为该作业失败
4.  失败的任务或作业都会有ApplicationMaster重新运行

### ApplicationMaster失败

1.  ApplicationMaster定时发送心跳信号到ResourceManager，通常一旦ApplicationMaster失败，则认为失败，但也可以通过配置多次后才失败
2.  一旦ApplicationMaster失败，ResourceManager会启动一个新的ApplicationMaster
3.  新的ApplicationMaster负责恢复之前错误的ApplicationMaster的状态(yarn.app.mapreduce.am.job.recovery.enable=true)，这一步是通过将应用运行状态保存到共享的存储上来实现的，ResourceManager不会负责任务状态的保存和恢复
4.  Client也会定时向ApplicationMaster查询进度和状态，一旦发现其失败，则向ResouceManager询问新的ApplicationMaster

### NodeManager失败

1.  NodeManager定时发送心跳到ResourceManager，如果超过一段时间没有收到心跳消息，ResourceManager就会将其移除
2.  任何运行在该NodeManager上的任务和ApplicationMaster都会在其他NodeManager上进行恢复
3.  如果某个NodeManager失败的次数太多，ApplicationMaster会将其加入黑名单（ResourceManager没有），任务调度时不在其上运行任务

### ResourceManager失败

1.  通过checkpoint机制，定时将其状态保存到磁盘，然后失败的时候，重新运行
2.  通过zookeeper同步状态和实现透明的HA

可以看出，**一般的错误处理都是由当前模块的父模块进行监控（心跳）和恢复。而最顶端的模块则通过定时保存、同步状态和zookeeper来ֹ实现HA**