# Doris安装部署
## 安装说明
本次安装doris版本为2.0.11

需要新建用户，用户名为doris，此用户拥有所有运行脚本的权限



参考链接：

> [https://blog.csdn.net/weixin_43820556/article/details/133754689](https://blog.csdn.net/weixin_43820556/article/details/133754689)
>
> [https://doris.apache.org/zh-CN/docs/2.0/install/cluster-deployment/standard-deployment](https://doris.apache.org/zh-CN/docs/2.0/install/cluster-deployment/standard-deployment)
>



## 部署架构图
| 服务器（IP） | 服务 | 启动脚本 | 关闭脚本 |
| --- | --- | --- | --- |
| 172.16.200.90 | MySqlClient | - | - |
| - | FE | sh /home/doris/software/doris/fe/bin/start_fe.sh --daemon | sh /home/doris/software/doris/fe/bin/stop_fe.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |
| 172.16.200.91 | FE | sh /home/doris/software/doris/fe/bin/start_fe.sh --daemon | sh /home/doris/software/doris/fe/bin/stop_fe.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |
| - | Doris Manager | sh /data/doris/software/doris-manager/webserver/bin/start.sh | sh /data/doris/software/doris-manager/webserver/bin/stop.sh |
| 172.16.200.92 | FE | sh /home/doris/software/doris/fe/bin/start_fe.sh --daemon | sh /home/doris/software/doris/fe/bin/stop_fe.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |
| 172.16.200.93 | BE | sh /home/doris/software/doris/be/bin/start_be.sh --daemon | sh /home/doris/software/doris/be/bin/stop_be.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |
| 172.16.200.94 | BE | sh /home/doris/software/doris/be/bin/start_be.sh --daemon | sh /home/doris/software/doris/be/bin/stop_be.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |
| 172.16.200.95 | BE | sh /home/doris/software/doris/be/bin/start_be.sh --daemon | sh /home/doris/software/doris/be/bin/stop_be.sh --daemon |
| - | Doris Agent | sh /data/doris/software/manager-agent/bin/start.sh | sh /data/doris/software/manager-agent/bin/stop.sh |


## 软件包准备
> apache-doris-2.0.11-bin-x64.tar.gz
>
> jdk-8u131-linux-x64.tar.gz
>
> mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz
>
> doris-manager-24.0.1-x64-bin.tar.gz
>



doris安装包，官网（[https://doris.apache.org/download）找到2.0.11版本](https://doris.apache.org/download）找到2.0.11版本)

![](C:\Users\vox1824\AppData\Roaming\Typora\typora-user-images\image-20240612161101476.png)

jdk，官网（[https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html）](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html）)



![](C:\Users\vox1824\AppData\Roaming\Typora\typora-user-images\image-20240612161638597.png)

mysql，官网（[https://downloads.mysql.com/archives/community/）](https://downloads.mysql.com/archives/community/）)

![](C:\Users\vox1824\AppData\Roaming\Typora\typora-user-images\image-20240612162129157.png)

doris-manager，官网（[https://docs.selectdb.com/docs/enterprise/cluster-manager-guide/deployment-guide/deployment-guide-24.x#%E9%83%A8%E7%BD%B2-doris-manager-agent）](https://docs.selectdb.com/docs/enterprise/cluster-manager-guide/deployment-guide/deployment-guide-24.x#%E9%83%A8%E7%BD%B2-doris-manager-agent）)

![](C:\Users\vox1824\AppData\Roaming\Typora\typora-user-images\image-20240612162324285.png)

## 新建用户
需要在每个机器上面新建用户

```shell
sudo groupadd -g 900 doris
sudo useradd -u 900 -g 900 doris
```

设置密码

```shell
sudo passwd doris
```





## 系统设置
### 最大打开文件句柄数
Doris 由于依赖大量文件来管理表数据，所以需要将系统对程序打开文件数的限制调高。

```sql
vi /etc/security/limits.conf 
* soft nofile 1000000
* hard nofile 1000000
```

### 关闭 Swap
```shell
sudo vi /etc/fstab
```

**永久关闭**，使用 Linux root 账户，注释掉 /etc/fstab 中的 swap 分区，然后重启即可彻底关闭 swap 分区。

```plain
# /etc/fstab
# <file system>        <dir>         <type>    <options>             <dump> <pass>
tmpfs                  /tmp          tmpfs     nodev,nosuid          0      0
/dev/sda1              /             ext4      defaults,noatime      0      1
# /dev/sda2              none          swap      defaults              0      0
/dev/sda3              /home         ext4      defaults,noatime      0      2
```

### 配置 NTP 服务
Doris 的元数据要求时间精度要小于 5000ms，所以所有集群所有机器要进行时钟同步，避免因为时钟问题引发的元数据不一致导致服务出现异常。

通常情况下，可以通过配置 NTP 服务保证各节点时钟同步。

```sql
sudo systemctl start ntpd.service
sudo systemctl enable ntpd.service
```

如果不存在ntpd.service

安装 `sudo yum install ntp`



### 修改虚拟内存区域数量
修改虚拟内存区域至少 2000000

```sql
# 每次启动前都需启动
sysctl -w vm.max_map_count=2000000
# 永久修改
sudo vi /etc/sysctl.d/99-sysctl.conf
vm.max_map_count = 2000000
```



### 配置java环境变量
这一步需要在 `doris` 用户下进行

```shell
su - doris;
vi /home/doris/.bashrc;
# 在.bashrc文件中追加以下内容
export JAVA_HOME=/home/doris/software/java
export PATH=$JAVA_HOME/bin:$PATH

# 配置生效
source /home/doris/.bashrc;
# 检查jdk
java -version;

```





### SSH配置
使得集群中任意一台机器与其他机器实现免密登录



**it-cjx 用户**

在每个机器上运行

```shell
# 服务端
sudo yum install openssh-server

# 客户端
sudo yum install openssh-clients

# 创建秘钥
ssh-keygen -t rsa
```

在各个机器上运行

```shell
# 汇集公钥
scp ~/.ssh/id_rsa.pub it-cjx@172.16.200.90:~/.ssh/fe02.id.rsa.pub
scp ~/.ssh/id_rsa.pub it-cjx@172.16.200.90:~/.ssh/fe03.id.rsa.pub
scp ~/.ssh/id_rsa.pub it-cjx@172.16.200.90:~/.ssh/be01.id.rsa.pub
scp ~/.ssh/id_rsa.pub it-cjx@172.16.200.90:~/.ssh/be02.id.rsa.pub
scp ~/.ssh/id_rsa.pub it-cjx@172.16.200.90:~/.ssh/be03.id.rsa.pub
```

172.16.200.90节点生成认证文件

```shell
# 生成秘钥认证文件
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/fe02.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/fe03.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be01.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be02.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be03.id.rsa.pub >> ~/.ssh/authorized_keys
# 向集群机器分发认证文件
scp ~/.ssh/authorized_keys it-cjx@172.16.200.91:~/.ssh/
scp ~/.ssh/authorized_keys it-cjx@172.16.200.92:~/.ssh/
scp ~/.ssh/authorized_keys it-cjx@172.16.200.93:~/.ssh/
scp ~/.ssh/authorized_keys it-cjx@172.16.200.94:~/.ssh/
scp ~/.ssh/authorized_keys it-cjx@172.16.200.95:~/.ssh/
```

在各个机器上执行

```shell
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys  
chmod 600 ~/.ssh/id_rsa
```



**doris 用户**

重复上述，需要更改 `it-cjx` 为 `doris`，**需要在**`doris`**用户下执行操作**

在各个机器上运行

```shell
# 创建秘钥
ssh-keygen -t rsa
```

汇集公钥

```shell
# 汇集公钥
scp ~/.ssh/id_rsa.pub doris@172.16.200.90:~/.ssh/fe02.id.rsa.pub
scp ~/.ssh/id_rsa.pub doris@172.16.200.90:~/.ssh/fe03.id.rsa.pub
scp ~/.ssh/id_rsa.pub doris@172.16.200.90:~/.ssh/be01.id.rsa.pub
scp ~/.ssh/id_rsa.pub doris@172.16.200.90:~/.ssh/be02.id.rsa.pub
scp ~/.ssh/id_rsa.pub doris@172.16.200.90:~/.ssh/be03.id.rsa.pub
```

172.16.200.90节点生成认证文件

```shell
# 生成秘钥认证文件
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/fe02.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/fe03.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be01.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be02.id.rsa.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/be03.id.rsa.pub >> ~/.ssh/authorized_keys
# 向集群机器分发认证文件
scp ~/.ssh/authorized_keys doris@172.16.200.91:~/.ssh/
scp ~/.ssh/authorized_keys doris@172.16.200.92:~/.ssh/
scp ~/.ssh/authorized_keys doris@172.16.200.93:~/.ssh/
scp ~/.ssh/authorized_keys doris@172.16.200.94:~/.ssh/
scp ~/.ssh/authorized_keys doris@172.16.200.95:~/.ssh/
```

在各个机器上执行

```shell
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys  
chmod 600 ~/.ssh/id_rsa
```



## 安装
安装环境

> `mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz` **只选一个节点安装即可**
>
> `apache-doris-2.0.11-bin-x64.tar.gz`
>
> `jdk-8u131-linux-x64.tar`
>

```shell
sudo mkdir -p /data/doris/software;
sudo mkdir -p /data/doris/logs;
sudo mkdir -p /data/doris/data;
sudo ln -snf /data/doris/software /home/doris/software;
sudo ln -snf /data/doris/logs /home/doris/logs;
sudo ln -snf /data/doris/data /home/doris/data;
# fe的元数据存储目录,be节点不需要
mkdir -p /data/doris/data/doris-meta;
# be的数据存储目录,fe节点不需要
mkdir -p /data/doris/data/datastorage;

# 将安装包解压至software目录并配置软链接
sudo tar -zxf /home/it-cjx/jdk-8u131-linux-x64.tar.gz -C /home/doris/software/;
sudo tar -zxf /home/it-cjx/apache-doris-2.0.11-bin-x64.tar.gz -C /home/doris/software/;
sudo ln -snf /home/doris/software/jdk1.8.0_131 /home/doris/software/java;
sudo ln -snf /home/doris/software/apache-doris-2.0.11-bin-x64 /home/doris/software/doris;
# 在某一台机器上安装mysql客户端，在此选择的是172.16.200.90节点
sudo tar -zxf /data/install_package/mysql-5.7.44-linux-glibc2.12-x86_64.tar.gz -C /home/doris/software/;
sudo ln -snf /home/doris/software/mysql-5.7.44-linux-glibc2.12-x86_64 /home/doris/software/mysql-client;

# 修改目录权限
sudo chown -R doris:doris /data/doris/;
sudo chown -R doris:doris /home/doris/;

```







## 配置Doris文件
**在**`doris`**用户下执行**

`fe.conf`

```shell
vi /home/doris/software/doris/fe/conf/fe.conf
# 添加配置（使用ip a命令可获取）
# priority_networks = 实际本机IP地址/掩码长度
priority_networks = 172.16.200.90/24
meta_dir=/data/doris/data/doris-meta

# the output dir of stderr and stdout 
LOG_DIR = /data/doris/logs


JAVA_HOME=/data/doris/software/jdk1.8.0_131/
# 修改 Xmx8192m -> Xmx30720m
JAVA_OPTS="-Djavax.security.auth.useSubjectCredsOnly=false -Xss4m -Xmx30720m -XX:+UnlockExperimentalVMOptions -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:$LOG_DIR/log/fe.gc.log.$CUR_DATE -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=50M -Dlog4j2.formatMsgNoLookups=true"
```

`be.conf`

be节点的`JAVA_OPTS`配置暂时不用修改

```shell
vi /home/doris/software/doris/be/conf/be.conf
# 添加配置（使用ip a命令可获取）
# priority_networks = 实际本机IP地址/掩码长度
priority_networks = 172.16.200.93/24
storage_root_path=/data/doris/data/datastorage

# the output dir of stderr and stdout 
sys_log_dir = /data/doris/logs

JAVA_HOME=/data/doris/software/jdk1.8.0_131/
```





## 启动
### FE
```shell
su - doris;
cd  /home/doris/software/doris/fe;
sh bin/start_fe.sh --daemon;
```

### FE follower
在第一次启动时需要添加`--helper` 参数，除了第一次之外，都正常启动

```shell
su - doris;
cd  /home/doris/software/doris/fe;
sh bin/start_fe.sh --helper masterFE的IP:edit_log_port --daemon;
```

如果没有加 `--helper masterFE的IP:edit_log_port` 则需要完全清空`doris-meta`中的数据，再重新启动此节点



### BE
```shell
su - doris;
cd  /home/doris/software/doris/be;
sh bin/start_be.sh --daemon
```





### 配置节点信息
**`doris`**用户下 MysqlClient 连接第一个启动的FE 增加其他的FE节点和BE节点，第一次启动为 无密码连接方式

```sql
mysqld -h 172.16.200.90 -P 9030 -uroot 

-- 新增 FE-FOLLOWER 节点
mysql> ALTER SYSTEM ADD FOLLOWER "fe2-ip:9010";
mysql> ALTER SYSTEM ADD FOLLOWER "fe3-ip:9010";

-- 查看 FE 节点信息
mysql> show frontends\G

-- 新增 BE 节点
mysql> ALTER SYSTEM ADD BACKEND "be1-ip:9050";
mysql> ALTER SYSTEM ADD BACKEND "be2-ip:9050";
mysql> ALTER SYSTEM ADD BACKEND "be3-ip:9050";

-- 查看 BE 节点信息
mysql> show backends\G;

```



`SET PASSWORD = PASSWORD('******');` 设置doris-root用户密码

设置密码启动时

```shell
mysql -h 172.16.200.90 -P 9030 -uroot -p
输入密码:
```



此时即可访问Doris FE 节点Web端：[http://172.16.200.90:8030/](http://172.16.200.90:8030/)  [http://172.16.200.91:8030/](http://172.16.200.91:8030/)  [http://172.16.200.92:8030/](http://172.16.200.92:8030/)



## 配置Doris Manager
> 参考链接 [https://docs.selectdb.com/docs/enterprise/cluster-manager-guide/deployment-guide/deployment-guide-24.x#%E9%83%A8%E7%BD%B2-doris-manager-agent](https://docs.selectdb.com/docs/enterprise/cluster-manager-guide/deployment-guide/deployment-guide-24.x#%E9%83%A8%E7%BD%B2-doris-manager-agent)
>



Doris Manager可以负责做到集群监控告警、定时巡检、升级、自动拉起等操作

现阶段Doris Manager被 SelectDB 分享出来免费开放使用，但是并未开放源码

**下载**

[https://selectdb.com/download/enterprise#manager](https://selectdb.com/download/enterprise#manager)



**安装包说明**

只需一个节点部署即可，这里选择172.16.200.91作为Doris Manager部署地址

**安装包名称**

doris-manager-24.0.0-x64-bin.tar.gz

**安装包解压**

```shell
$ tar -zxvf doris-manager-24.0.0-x64-bin.tar.gz
```



**安装包目录结构**

```shell
doris-manager-24.0.0-x64-bin
    webserver // Doris Manager Web 服务组件，这是网页入口服务，需要手工启动
       bin  // 启停脚本
       conf  // 配置文件
       lib  // 服务二进制
       static  // 前端静态文件
       config-tool  // Doris Manager 服务管理工具
       inspection  // 巡检脚本
    deps // Doris Manager 管控依赖组件
       alertmanager // 告警工具
       jdk // jdk依赖包
       prometheus // 监控指标存储工具
       grafana // 监控看板工具
       Doris-Dashboard.json // 默认仪表盘json文件，名称以实际为主
    agent
       manager-agent-24.0.0-x64-bin.tar.gz // Doris Manager 服务的 Agent 压缩安装包，注意，这个压缩包不能删除
```



手工部署 Doris Manager Web 服务组件

解压 Doris Manager 安装包之后得到的`doris-manager-24.0.0-x64-bin`目录

**修改安装包目录名称**

建议在启动服务之前将`doris-manager-24.0.0-x64-bin`目录更换一个名字，例如`doris-manager`

```shell
$ mv doris-manager-24.0.0-x64-bin doris-manager
```



**进入安装路径**

```shell
$ cd doris-manager
```

**启动 Web 服务**

在 webserver 目录直接运行脚本：

```shell
$ sh /data/doris/software/doris-manager/webserver/bin/start.sh
```



#### 部署 Doris Manager Agent
准备安装包

一般来说，Doris Manager Agent的安装包就在`doris-manager/agent/`目录下，如果没有，则需要下载

为其他节点分发

```shell
scp /data/doris/software/doris-manager/agent/manager-agent-24.0.1-x64-bin.tar.gz doris@172.16.200.90:/data/doris/software/manager-agent-24.0.1-x64-bin.tar.gz
scp /data/doris/software/doris-manager/agent/manager-agent-24.0.1-x64-bin.tar.gz doris@172.16.200.92:/data/doris/software/manager-agent-24.0.1-x64-bin.tar.gz
scp /data/doris/software/doris-manager/agent/manager-agent-24.0.1-x64-bin.tar.gz doris@172.16.200.93:/data/doris/software/manager-agent-24.0.1-x64-bin.tar.gz
scp /data/doris/software/doris-manager/agent/manager-agent-24.0.1-x64-bin.tar.gz doris@172.16.200.94:/data/doris/software/manager-agent-24.0.1-x64-bin.tar.gz
scp /data/doris/software/doris-manager/agent/manager-agent-24.0.1-x64-bin.tar.gz doris@172.16.200.95:/data/doris/software/manager-agent-24.0.1-x64-bin.tar.gz
```



在每个节点执行

```shell
cd /data/doris/software
tar -xvf manager-agent-24.0.1-x64-bin.tar.gz
mv manager-agent-24.0.1-x64-bin manager-agent
sh ./manager-agent/bin/start.sh
```



**访问 Web 服务**

直接通过浏览器输入 URL——[http://172.16.200.91:8004/login](http://172.16.200.91:8004/login)

