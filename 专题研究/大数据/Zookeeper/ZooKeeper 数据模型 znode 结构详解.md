# ZooKeeper 数据模型 znode 结构详解

## 数据模型

在 zookeeper 中，可以说 zookeeper 中的所有存储的数据是由 znode 组成的，节点也称为 znode，并以 key/value 形式存储数据。

整体结构类似于 linux 文件系统的模式以树形结构存储。其中根路径以 / 开头。

进入 zookeeper 安装的 bin 目录，通过sh zkCli.sh打开命令行终端，执行 "ls /" 命令显示：
```bash
$ ls /
$ ls /zookeeper
$ ls /zookeeper/quota
```


![](https://www.runoob.com/wp-content/uploads/2020/09/data-model-01.png)

我们直观的看到此时存储的数据在根目录下存在 runoob 和 zookeeper 两个节点，zookeeper 节点下存在 quota 这个节点。

![](https://www.runoob.com/wp-content/uploads/2020/09/data-model-02.png)

runoob 节点是在我们之前章节创建，并且通过 java 客户端设置值 0，现在我们在命令行终端执行 get /runoob 显示此节点的属性。

`$ get /runoob`

![](https://www.runoob.com/wp-content/uploads/2020/09/data-model-03.png)

其中第一行显示的 0 是该节点的 value 值。

## Znode 的状态属性

| 命令             | 作用                                                                                                                                                                                                   |
|:---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cZxid`          | 创建节点时的事务ID                                                                                                                                                                                     |
| `ctime`          | 创建节点时的时间                                                                                                                                                                                       |
| `mZxid`          | 最后修改节点时的事务ID                                                                                                                                                                                 |
| `mtime`          | 最后修改节点时的时间                                                                                                                                                                                   |
| `mtime`          | 最后修改节点时的时间                                                                                                                                                                                   |
| `pZxid`          | 表示该节点的子节点列表最后一次修改的事务ID，添加子节点或删除子节点就会影响子节点列表，但是修改子节点的数据内容则不影响该ID**（注意，只有子节点列表变更了才会变更pzxid，子节点内容变更不会影响pzxid）** |
| `cversion`       | 子节点版本号，子节点每次修改版本号加1                                                                                                                                                                  |
| `dataversion`    | 数据版本号，数据每次修改该版本号加1                                                                                                                                                                    |
| `aclversion`     | 权限版本号，权限每次修改该版本号加1                                                                                                                                                                    |
| `ephemeralOwner` | 创建该临时节点的会话的sessionID。（如果该节点是持久节点，那么这个属性值为0）                                                                                                                           |
| `dataLength`     | 该节点的数据长度                                                                                                                                                                                       |
| `numChildren`    | 该节点拥有子节点的数量**（只统计直接子节点的数量）**                                                                                                                                                   |


了解上面状态属性值，我们对 **/runoob** 节点做一次修改，执行命令 set /runoob 1 ，如下图所示:

`$ set /runoob 1`

![](https://www.runoob.com/wp-content/uploads/2020/09/data-model-04.png)

对比上面结果，可以看到 mZxid、mtime、dataVersion 都发生了变化。

在 **/runoob** 节点下，我们再添加一子节点，执行：

```bash
$ create -e  /runoob/child  0
$ get /runoob
```

**提示：** 更多命令使用后面章节会详解介绍。

执行完终端命令行显示:

![](https://www.runoob.com/wp-content/uploads/2020/09/data-model-05.png)

可见 **/runoob** 节点的 pZxid、cversion、numChildren 都发生了相应的改变。