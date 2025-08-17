# Zookeeper 节点特性


本章节介绍一下 zookeeper 的节点特性和简单使用场景，正是由于这些节点特性的存在使 zookeeper 开发出不同的场景应用。

### 1、同一级节点 key 名称是唯一的

实例：

$ ls /
$ create /runoob 2

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-01.png)

已存在 **/runoob** 节点，再次创建会提示已经存在。

### 2、创建节点时，必须要带上全路径

实例：

$ ls /runoob
$ create /runoob/child 0
$ create /runoob/child/ch01 0

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-02.png)

### 3、session 关闭，临时节点清除

实例：

$ ls /runoob
$ create -e /runoob/echild 0

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-03.png)

同时终端二查看该节点:

$ ls /runoob

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-04.png)

ctrl+c 关闭终端一连接后，查询终端二 **/runoob/echild** 节点消失。

$ ls /runoob

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-05.png)

### 4、自动创建顺序节点

实例：

$ create -s -e /runoob 0

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-06.png)

### 5、watch 机制，监听节点变化

事件监听机制类似于观察者模式，watch 流程是客户端向服务端某个节点路径上注册一个 watcher，同时客户端也会存储特定的 watcher，当节点数据或子节点发生变化时，服务端通知客户端，客户端进行回调处理。特别注意：监听事件被单次触发后，事件就失效了。

**提示：**参考常用命令章节 get 命令监听 watch 使用，后面章节将详细介绍 watch 实现原理。

### 6、delete 命令只能一层一层删除

实例：

$ ls /
$ delete /runoob

![](https://www.runoob.com/wp-content/uploads/2020/09/node-t-07.png)

**提示**：新版本可以通过 deleteall 命令递归删除。

有了上述众多节点特性，使得 zookeeper 能开发不出不同的经典应用场景，比如：

-   1. 数据发布/订阅
-   2. 负载均衡
-   3. 分布式协调/通知
-   4. 集群管理
-   5. 集群管理
-   6. master 管理
-   7. 分布式锁
-   8. 分布式队列