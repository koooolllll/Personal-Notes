# Zookeeper 权限控制 ACL


zookeeper 的 ACL（Access Control List，访问控制表）权限在生产环境是特别重要的，所以本章节特别介绍一下。

ACL 权限可以针对节点设置相关读写等权限，保障数据安全性。

permissions 可以指定不同的权限范围及角色。

### ACL 命令行

-   **getAcl 命令**：获取某个节点的 acl 权限信息。
-   **setAcl 命令**：设置某个节点的 acl 权限信息。
-   **addauth 命令**：输入认证授权信息，注册时输入明文密码，加密形式保存。

### ACL 构成

zookeeper 的 acl 通过 **[scheme:id:permissions]** 来构成权限列表。

-   1、**scheme**：代表采用的某种权限机制，包括 world、auth、digest、ip、super 几种。
-   2、**id**：代表允许访问的用户。
-   3、**permissions**：权限组合字符串，由 cdrwa 组成，其中每个字母代表支持不同权限， 创建权限 create(c)、删除权限 delete(d)、读权限 read(r)、写权限 write(w)、管理权限admin(a)。

### world 实例

查看默认节点权限，再更新节点 permissions 权限部分为 crwa，结果删除节点失败。其中 world 代表开放式权限。

$ getAcl /runoob/child
$ setAcl /runoob/child world:anyone:crwa
$ delete /runoob/child

![](https://www.runoob.com/wp-content/uploads/2020/09/zk-acl-01.png)

### auth 实例

auth 用于授予权限，注意需要先创建用户。

$ setAcl /runoob/child auth:user1:123456:cdrwa
$ addauth digest user1:123456
$ setAcl /runoob/child auth:user1:123456:cdrwa
$ getAcl /runoob/child

![](https://www.runoob.com/wp-content/uploads/2020/09/zk-acl-02.png)

### digest 实例

退出当前用户，重新连接终端，digest 可用于账号密码登录和验证。。

$ ls /runoob
$ create /runoob/child01 runoob
$ getAcl /runoob/child01
$ setAcl /runoob/child01 digest:user1:HYGa7IZRm2PUBFiFFu8xY2pPP/s=:cdra
$ getAcl /runoob/child01
$ addauth digest user1:123456
$ getAcl /runoob/child01

**提示：**加密密码是上一步创建的。

![](https://www.runoob.com/wp-content/uploads/2020/09/zk-acl-03.png)

### IP 实例

限制 IP 地址的访问权限，把权限设置给 IP 地址为 192.168.3.7 后，IP 为 192.168.3.38 已经没有访问权限。

$ create /runoob/ip 0
$ getAcl /runoob/ip
$ setAcl /runoob/ip ip:192.168.3.7:cdrwa
$ get /runoob/ip

![](https://www.runoob.com/wp-content/uploads/2020/09/zk-acl-04.png)