# Zookeeper
ZooKeeper 是 Apache 软件基金会的一个软件项目，它为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。

ZooKeeper 的架构通过冗余服务实现高可用性。

Zookeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

一个典型的分布式数据一致性的解决方案，分布式应用程序可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。

## 目录

1. [[ZooKeeper 数据模型 znode 结构详解]]
2. [[Zookeeper session 基本原理]]
3. [[Zookeeper 客户端基础命令]]
4. [[Zookeeper 四字命令]]
5. [[Zookeeper 节点特性]]
6. [[Zookeeper 权限控制 ACL]]
7. [[Zookeeper watcher 事件机制原理剖析]]
8. [[Zookeeper 数据同步流程]]
9. [[Zookeeper Leader 选举原理]]
10. [[Zookeeper 分布式锁实现原理]]

## zookeeper 数据结构

zookeeper 提供的名称空间非常类似于标准文件系统，key-value 的形式存储。名称 key 由斜线 / 分割的一系列路径元素，zookeeper 名称空间中的每个节点都是由一个路径标识。

![](https://www.runoob.com/wp-content/uploads/2020/09/zknamespace.jpg)