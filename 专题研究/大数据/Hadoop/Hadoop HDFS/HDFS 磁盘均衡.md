# [[HDFS]] 磁盘均衡

## HDFS 磁盘均衡器

[[HDFS]] 提供了一个用于 Datanode 内多磁盘之间的数据均衡工具，即 Diskbalancer （磁盘均衡器），它把数据均衡的分发到一个 Datanode 下的多个磁盘。Diskbalancer 和 Hadoop 2.0 版本以前提供的 Balancer 不同，因为 Balancer 关心的是不同 Datanode 之间的数据均衡，Datanode 内多个磁盘的数据均衡它是不起作用的。

HDFS 由于以下原因，在把数据存储到 Datanode 多个磁盘的时候，会出现磁盘之间数据不均衡的情况：

-   大量的数据写入和删除
-   磁盘更换

上面这两点可能导致数据在 Datanode 内的多个磁盘发生明显倾斜。这种情况现有的 HDFS balancer 均衡工具没办法处理，上面说了，它只关心 Datanode 之间的数据均衡，所以，Hadoop 3.0 提供了 Diskbalancer 工具，用于均衡一个Datanode 内多个磁盘之间的数据均衡。

## Diskbalancer

Hadoop HDFS balancer 工具通过创建一个计划（命令集）并在 Datanode 执行该计划来工作。这里的计划主要描述的是有多少数据需要在磁盘之间做迁移。一个计划有很多迁移步骤，比如，源磁盘，目标磁盘和需要迁移的字节数。计划可以针对某一个 Datanode 执行特定操作。默认情况下，Diskbalancer 是未启用状态，您可以在 hdfs-site.xml 配置文件把 `dfs.disk.balancer.enabled` 设置为 true 来启用它。

当我们往 HDFS 上写入新的数据块，DataNode 将会使用 volume 选择策略来为这个块选择存储的地方。目前 Hadoop 支持两种 volume 选择策略：round-robin（循环策略） 和 available space（可用空间策略），选择哪种策略我们可以通过下面的参数来设置。

`dfs.datanode.fsdataset.volume.choosing.policy`

-   循环策略：将新块均匀分布在可用磁盘上。可以把每个可用磁盘看作1~n的一个数组，根据数组下标来循环写入数据。
-   可用空间策略：它是优先将数据写入具有最大可用空间的磁盘（通过百分比计算的）。

![HDFS磁盘均衡](https://www.hadoopdoc.com/media/editor/file_1570195566000_20191004212607902952.png "HDFS磁盘均衡原理")


### 基于轮询的策略

```java
public class RoundRobinVolumeChoosingPolicy<V extends FsVolumeSpi>
    implements VolumeChoosingPolicy<V> {
  public static final Log LOG = LogFactory.getLog(RoundRobinVolumeChoosingPolicy.class);

  // 当前轮询的磁盘位置
  private int curVolume = 0;
 
  @Override
  public synchronized V chooseVolume(final List<V> volumes, long blockSize) throws IOException {

	// 如果可用磁盘数量小于1 抛出异常
	if(volumes.size() < 1) {
		throw new DiskOutOfSpaceException("No more available volumes");
	}
    
    // since volumes could've been removed because of the failure
    // make sure we are not out of bounds
    if(curVolume >= volumes.size()) {
      curVolume = 0;
    }
    
    int startVolume = curVolume;
    long maxAvailable = 0;
    
    while (true) {
      final V volume = volumes.get(curVolume);
      curVolume = (curVolume + 1) % volumes.size();
      long availableVolumeSize = volume.getAvailable();
      if (availableVolumeSize > blockSize) {
        return volume;
      }
      
      if (availableVolumeSize > maxAvailable) {
        maxAvailable = availableVolumeSize;
      }
      
      if (curVolume == startVolume) {
        throw new DiskOutOfSpaceException("Out of space: "
            + "The volume with the most available space (=" + maxAvailable
            + " B) is less than the block size (=" + blockSize + " B).");
      }
    }
  }
}
```

基于轮询的策略**可以保证每个卷的写入次数平衡**，但无法保证写入数据量平衡。例如，在一次写过程中，在卷A上写入了1M的块，但在卷B上写入了128M的块，A与B之间的数据量就不平衡了。久而久之，不平衡的现象就会越发严重。

### 基于可用空间的策略


这个策略比轮询更加聪明一些。它根据一个可用空间的阈值，将卷分为可用空间多的卷和可用空间少的卷两类。然后，会根据一个比较高的概率选择可用空间多的卷。不管选择了哪一类，最终都会采用轮询策略来写入这一类卷。可用空间阈值和选择卷的概率都是可以通过参数设定的。


```java
public class AvailableSpaceVolumeChoosingPolicy<V extends FsVolumeSpi>
    implements VolumeChoosingPolicy<V>, Configurable {
  
  private static final Log LOG = LogFactory.getLog(AvailableSpaceVolumeChoosingPolicy.class);
  
  private final Random random;
  
  private long balancedSpaceThreshold = DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_THRESHOLD_DEFAULT;
  private float balancedPreferencePercent = DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_DEFAULT;
 
  AvailableSpaceVolumeChoosingPolicy(Random random) {
    this.random = random;
  }
 
  public AvailableSpaceVolumeChoosingPolicy() {
    this(new Random());
  }
 
  @Override
  public synchronized void setConf(Configuration conf) {
    balancedSpaceThreshold = conf.getLong(
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_THRESHOLD_KEY,
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_THRESHOLD_DEFAULT);
    balancedPreferencePercent = conf.getFloat(
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_KEY,
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_DEFAULT);
    
    LOG.info("Available space volume choosing policy initialized: " +
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_THRESHOLD_KEY +
        " = " + balancedSpaceThreshold + ", " +
        DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_KEY +
        " = " + balancedPreferencePercent);
 
    if (balancedPreferencePercent > 1.0) {
      LOG.warn("The value of " + DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_KEY +
               " is greater than 1.0 but should be in the range 0.0 - 1.0");
    }
 
    if (balancedPreferencePercent < 0.5) {
      LOG.warn("The value of " + DFS_DATANODE_AVAILABLE_SPACE_VOLUME_CHOOSING_POLICY_BALANCED_SPACE_PREFERENCE_FRACTION_KEY +
               " is less than 0.5 so volumes with less available disk space will receive more block allocations");
    }
  }
  
  @Override
  public synchronized Configuration getConf() {
    // Nothing to do. Only added to fulfill the Configurable contract.
    return null;
  }
  // 已平衡的卷的轮询策略
  private final VolumeChoosingPolicy<V> roundRobinPolicyBalanced =
      new RoundRobinVolumeChoosingPolicy<V>();
  // 可用空间多的卷的轮询策略
  private final VolumeChoosingPolicy<V> roundRobinPolicyHighAvailable =
      new RoundRobinVolumeChoosingPolicy<V>();
  // 可用空间少的卷的轮询策略
  private final VolumeChoosingPolicy<V> roundRobinPolicyLowAvailable =
      new RoundRobinVolumeChoosingPolicy<V>();
 
  @Override
  public synchronized V chooseVolume(List<V> volumes,
      long replicaSize) throws IOException {
    if (volumes.size() < 1) {
      throw new DiskOutOfSpaceException("No more available volumes");
    }
    
    AvailableSpaceVolumeList volumesWithSpaces =
        new AvailableSpaceVolumeList(volumes);
    // 如果卷都在平衡阈值之内，直接轮询
    if (volumesWithSpaces.areAllVolumesWithinFreeSpaceThreshold()) {
      // If they're actually not too far out of whack, fall back on pure round
      // robin.
      V volume = roundRobinPolicyBalanced.chooseVolume(volumes, replicaSize);
      if (LOG.isDebugEnabled()) {
        LOG.debug("All volumes are within the configured free space balance " +
            "threshold. Selecting " + volume + " for write of block size " +
            replicaSize);
      }
      return volume;
    } else {
      V volume = null;
      // If none of the volumes with low free space have enough space for the
      // replica, always try to choose a volume with a lot of free space.
      long mostAvailableAmongLowVolumes = volumesWithSpaces
          .getMostAvailableSpaceAmongVolumesWithLowAvailableSpace();
      // 分别获取可用空间多和少的卷列表
      List<V> highAvailableVolumes = extractVolumesFromPairs(
          volumesWithSpaces.getVolumesWithHighAvailableSpace());
      List<V> lowAvailableVolumes = extractVolumesFromPairs(
          volumesWithSpaces.getVolumesWithLowAvailableSpace());
      
      float preferencePercentScaler =
          (highAvailableVolumes.size() * balancedPreferencePercent) +
          (lowAvailableVolumes.size() * (1 - balancedPreferencePercent));
      // 计算平衡比值，balancedPreferencePercent越大，可用空间多的卷所占比重会变大
      float scaledPreferencePercent =
          (highAvailableVolumes.size() * balancedPreferencePercent) /
          preferencePercentScaler;
      // 如果可用空间少的卷不足以放得下副本，或者随机出来的概率比上面的比例小，就轮询可用空间多的卷
      if (mostAvailableAmongLowVolumes < replicaSize ||
          random.nextFloat() < scaledPreferencePercent) {
        volume = roundRobinPolicyHighAvailable.chooseVolume(
            highAvailableVolumes, replicaSize);
        if (LOG.isDebugEnabled()) {
          LOG.debug("Volumes are imbalanced. Selecting " + volume +
              " from high available space volumes for write of block size "
              + replicaSize);
        }
      } else {
        volume = roundRobinPolicyLowAvailable.chooseVolume(
            lowAvailableVolumes, replicaSize);
        if (LOG.isDebugEnabled()) {
          LOG.debug("Volumes are imbalanced. Selecting " + volume +
              " from low available space volumes for write of block size "
              + replicaSize);
        }
      }
      return volume;
    }
  }
}
```

*这个策略可以在一定程度上削弱不平衡的现象，但仍然无法完全消除其影响。*
并且卷的可用空间只是诸多因素中的一个，仍然不够全面，磁盘I/O等指标也是比较重要的。但不管如何，它已经比纯轮询策略好得太多了。

默认情况下，DataNode 是使用基于 round-robin 策略来写入新的数据块。然而在一个长时间运行的集群中，由于 HDFS 中的大规模文件删除或者通过往 DataNode 中添加新的磁盘，仍然会导致同一个 DataNode 中的不同磁盘存储的数据很不均衡。即使你使用的是基于可用空间的策略，卷（volume）不平衡仍可导致较低效率的磁盘I/O。比如所有新增的数据块都会往新增的磁盘上写，在此期间，其他的磁盘会处于空闲状态，这样新的磁盘将会是整个系统的瓶颈。

Apache Hadoop 社区之前开发了几个离线脚本来解决磁盘不均衡的问题。然而，这些脚本都是在HDFS代码库之外，在执行这些脚本往不同磁盘之间移动数据的时候，需要要求 DataNode 处于关闭状态。结果，HDFS-1312 还引入了一个在线磁盘均衡器，旨在根据各种指标重新平衡正在运行 DataNode 上的磁盘数据。和现有的 HDFS 均衡器类似，HDFS 磁盘均衡器在 DataNode 中以线程的形式运行，并在相同存储类型的卷（volumes）之间移动数据。注意，本文介绍的HDFS 磁盘均衡器是在同一个 DataNode 中的不同磁盘之间移动数据，而之前的 HDFS 均衡器是在不同的 DataNode 之间移动数据。

## Diskbalancer 的使用

我们通过一个例子逐步探讨如何使用 Diskbalancer。

首先，确保所有 DataNode 上的 `dfs.disk.balancer.enabled` 参数设置成 `true`。本例子中，我们的 DataNode 已经挂载了一个磁盘（/mnt/disk1），现在我们往这个DataNode 上挂载新的磁盘（/mnt/disk2），我们使用 df命令来显示磁盘的使用率：

```shell
# df -h

/var/disk1      5.8G  3.6G  1.9G  66% /mnt/disk1
/var/disk2      5.8G   13M  5.5G   1% /mnt/disk2
```


从上面的输出可以看出，两个磁盘的使用率很不均衡，所以我们来将这两个磁盘的数据均衡一下。典型的磁盘平衡器任务涉及三个步骤（通过 HDFS 的 diskbalancer 命令）：`plan`, `execute` 和 `query`。

第一步，HDFS 客户端从 NameNode 上读取指定 DataNode 的必要信息以生成执行计划：


```bash
# hdfs diskbalancer -plan lei-dn-3.example.org
16/08/19 18:04:01 INFO planner.GreedyPlanner: Starting plan for Node : lei-dn-3.example.org:20001
16/08/19 18:04:01 INFO planner.GreedyPlanner: Disk Volume set 03922eb1-63af-4a16-bafe-fde772aee2fa Type : DISK plan completed.
16/08/19 18:04:01 INFO planner.GreedyPlanner: Compute Plan for Node : lei-dn-3.example.org:20001 took 5 ms
16/08/19 18:04:01 INFO command.Command: Writing plan to : /system/diskbalancer/2016-Aug-19-18-04-01
```



从上面的输出可以看出，HDFS 磁盘平衡器通过使用 DataNode 报告给 NameNode 的磁盘使用信息，并结合计划程序来计算指定 DataNode 上数据移动计划的步骤，每个步骤指定要移动数据的源卷和目标卷，以及预计移动的数据量。

截止到撰写本文的时候，HDFS仅仅支持 GreedyPlanner，其不断地将数据从最常用的设备移动到最少使用的设备，直到所有数据均匀地分布在所有设备上。用户还可以在使用 plan 命令的时候指定空间利用阀值，也就是说，如果空间利用率的差异低于此阀值，planner 则认为此磁盘已经达到了平衡。当然，我们还可以通过使用 —bandwidth 参数来限制磁盘数据移动时的I/O。

磁盘平衡执行计划生成的文件内容格式是 Json 的，并且存储在 HDFS 之上。在默认情况下，这些文件是存储在 /system/diskbalancer 目录下面：


```bash
$ hdfs dfs -ls /system/diskbalancer/2016-Aug-19-18-04-01
Found 2 items
-rw-r--r--   3 hdfs supergroup       1955 2016-08-19 18:04 /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.before.json
-rw-r--r--   3 hdfs supergroup        908 2016-08-19 18:04 /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.plan.json
```


可以通过下面的命令在 DataNode 上执行这个生成的计划：

```bash
$ hdfs diskbalancer -execute /system/diskbalancer/2016-Aug-17-17-03-56/172.26.10.16.plan.json
16/08/17 17:22:08 INFO command.Command: Executing "execute plan" command
```

这个命令将 JSON 里面的计划提交给 DataNode，而 DataNode 会启动一个名为 BlockMover 的线程中执行这个计划。我们可以使用 query 命令来查询 DataNode 上diskbalancer 任务的状态：

```bash
# hdfs diskbalancer -query lei-dn-3:20001
16/08/19 21:08:04 INFO command.Command: Executing "query plan" command.
Plan File: /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.plan.json
Plan ID: ff735b410579b2bbe15352a14bf001396f22344f7ed5fe24481ac133ce6de65fe5d721e223b08a861245be033a82469d2ce943aac84d9a111b542e6c63b40e75
Result: PLAN_DONE
```

上面结果输出的 PLAN_DONE 表示 disk-balancing task 已经执行完成。为了验证磁盘平衡器的有效性，我们可以使用 df -h 命令来查看各个磁盘的空间使用率：

```bash
# df -h
Filesystem      Size  Used Avail Use% Mounted on
….
/var/disk1      5.8G  2.1G  3.5G  37% /mnt/disk1
/var/disk2      5.8G  1.6G  4.0G  29% /mnt/disk2
```

上面的结果证明，磁盘平衡器成功地将 /var/disk1 和 /var/disk2 空间使用率的差异降低到 10% 以下，说明任务完成！