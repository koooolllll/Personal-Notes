# Zookeeper 四字命令


zookeeper 支持某些特定的四字命令与其交互，用户获取 zookeeper 服务的当前状态及相关信息，用户在客户端可以通过 telenet 或者 nc（netcat） 向 zookeeper 提交相应的命令。

安装 nc 命令：

 `$ yum install nc                # centos`  
 或  
 ``$ sudo apt install netcat       # ubuntu`

四字命令格式：

 echo [command] | nc [ip] [port]

ZooKeeper 常用四字命令主要如下：

四字命令

功能描述

conf

3.3.0版本引入的。打印出服务相关配置的详细信息。

cons

3.3.0版本引入的。列出所有连接到这台服务器的客户端全部连接/会话详细信息。包括"接受/发送"的包数量、会话id、操作延迟、最后的操作执行等等信息。

crst

3.3.0版本引入的。重置所有连接的连接和会话统计信息。

dump

列出那些比较重要的会话和临时节点。这个命令只能在leader节点上有用。

envi

打印出服务环境的详细信息。

reqs

列出未经处理的请求

ruok

测试服务是否处于正确状态。如果确实如此，那么服务返回"imok"，否则不做任何相应。

stat

输出关于性能和连接的客户端的列表。

srst

重置服务器的统计。

srvr

3.3.0版本引入的。列出连接服务器的详细信息

wchs

3.3.0版本引入的。列出服务器watch的详细信息。

wchc

3.3.0版本引入的。通过session列出服务器watch的详细信息，它的输出是一个与watch相关的会话的列表。

wchp

3.3.0版本引入的。通过路径列出服务器watch的详细信息。它输出一个与session相关的路径。

mntr

3.4.0版本引入的。输出可用于检测集群健康状态的变量列表

> 参考官方链接：[https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_4lw](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_4lw)

---

## 四字命令使用

### stat 命令

stat 命令用于查看 zk 的状态信息，实例如下：

 $ echo stat | nc 192.168.3.38 2181

![img](http://gitlab.siger-data.com/yurui.gao/bigdata-arch-pic/uploads/d8da68619cbfb15923d74c6a06b68e2b/fc-01.png)

### ruok 命令

ruok 命令用于查看当前 zkserver 是否启动，若返回 imok 表示正常。实例如下：

 $ echo ruok | nc 192.168.3.38 2181

![img](http://gitlab.siger-data.com/yurui.gao/bigdata-arch-pic/uploads/485d9c5283798409ae915a2c52abdb00/fc-02.png)

### dump 命令

dump 命令用于列出未经处理的会话和临时节点。实例如下：

 $ echo dump | nc 192.168.3.38 2181

![img](http://gitlab.siger-data.com/yurui.gao/bigdata-arch-pic/uploads/d091c5f18f90f2648c21d6bfb8eb69eb/fc-03.png)

### conf 命令

conf 命令用于查看服务器配置。实例如下：

 $ echo conf | nc 192.168.3.38 2181

![img](http://gitlab.siger-data.com/yurui.gao/bigdata-arch-pic/uploads/c72499de7b21c2f3a804c9e1e14f7f88/fc-04.png)

### cons 命令

cons 命令用于展示连接到服务器的客户端信息。实例如下：

 $ echo cons | nc 192.168.3.38 2181

![img](http://gitlab.siger-data.com/yurui.gao/bigdata-arch-pic/uploads/7cde48cb74399e69d13527212090ed3b/fc-05.png)

### envi 命令

envi 命令用于查看环境变量。实例如下：

 $ echo envi | nc 192.168.3.38 2181