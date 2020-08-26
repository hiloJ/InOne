## Zookeeper

### 1. 概述

​		  Zookeeper是一个**分布式协调服务**的开源框架。主要用来**解决分布式集群中应用系统的一致性问题**，如：怎样避免同时操作统一数据造成脏读的问题

​		  Zookeeper本质上是一个**分布式的小文件存储系统**。提供基于类似于文件系统的目录树方式的数据存储，并且可以对树中的节点进行有效管理，从而用来维护和监控存储的数据的状态的变化。通过监控这些数据状态的变化，可以达到基于数据的集群管理。如：统一命名服务、分布式配置管理、分布式消息队列、分布式锁、分布式协调等功能

### 2. 特性

+ `全局数据一致性`

  > 集群中每个服务器保存一份相同的数据副本

+ 可靠性

  > 消息被其中一台服务器接口，那么将被所有的服务器接受

+ 顺序性

  + 全局有序

    > 在一台服务器上，消息a在消息b之前发布，则在所有服务器上，消息a都在消息b之前发布

  + 偏序

    > 消息b在消息a之后被同一发送者发布，消息a必排在消息b之前

+ 数据更新原子性

  > 一次数据更新要么成功(**半数以上**节点成功)，要么失败

+ 实时性

  > ZK保证客户端将在一个时间间隔范围内获得服务器的更新信息或失效信息

### 3. 集群角色

![集群角色](https://gitee.com/hiloj/imgs/raw/master/test/zk集群角色.png)

`Leader`：集群工作的核心

> **事务请求（写操作）**的唯一调度和处理者，保证集群事务处理的顺序性
>
> 集群内部各个服务器的调度者
>
> `create`、`setData`、`delete`等有写操作的请求，需要统一转发给leader处理，Leader需要决定编号、执行操作，此过程称为一个事务

`Follower`

> 处理客户端**非事务（读操作）请求**，转发事务请求给Leader
>
> 参与集群Leader选举投票

针对访问量比较大的ZK集群，可以新增观察者角色

`Observer`：在不影响集群事务处理能力的前提下提升集群的非事务处理能力

> 观察集群的最新状态变化并将这些状态同步过来
>
> 非事务请求可以独立处理，事务请求会转发Leader
>
> **不参与任何形式的投票**

### 4. 单机搭建

~~~bash
docker run --privileged=true --name zk --publish 2181:2181 -d zookeeper
~~~

### 5. 集群搭建

为了保证Leader选举（基于`Paxos`算法实现）能够得到多数的支持，集群通常由`2n+1`台servers组成

搭建过程如下：

+ 配置主机名称到IP地址映射配置
+ 修改ZK配置文件
  + 使用Observer模式（非必须）
    + 添加配置`PeerType=observer`
    + 指定Observer：`server.1:localhost:2181:3181:observer`
+ 远程赋值分发安装文件
+ 设置`myid`
+ 启动集群

~~~bash
# 安装docker compose

# 创建文件并进行编辑
touch docker-compose.yml
vim docker-compose.yml

# 文件内容
version: '3' #版本号
services:
	zk01:
		image: zookeeper #使用的镜像
		restart: always #宕机后自动重启
		hostname： zk01 #承载ZK容器的主机(父容器)名  可省略
		container_name: zk01 #容器名
		privileged: true #container内的root拥有真正的root权
		ports: #主机和容器的端口映射
			- "2181":"2181"
		volumes: #容器在宿主机的挂载目录
			- /opt/zookeeper-cluster/zk01/data:/data
			- /opt/zookeeper-cluster/zk01/log:/datalog
			- /opt/zookeeper-cluster/zk01/conf:/conf
		environment: #集群环境 #zoo1为容器名，也是主机名，意思为使用容器的内网通信（1）Zookeeper3.5 中指定的 ZOO_SERVERS 参数的 IP 地址和端口号后面多加了 “;2181 ”。（2）ZOO_SERVERS 指定ip时本机的ip地址写 0.0.0.0。
			ZOO_MY_ID: 1
			ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181

~~~

### 6. shell

~~~bash
# 3.6.1版本所有命令
ZooKeeper -server host:port -client-configuration properties-file cmd args
        addWatch [-m mode] path # optional mode is one of [PERSISTENT, PERSISTENT_RECURSIVE] - default is PERSISTENT_RECURSIVE
        addauth scheme auth
        close 
        config [-c] [-w] [-s]
        connect host:port
        create [-s] [-e] [-c] [-t ttl] path [data] [acl]
        delete [-v version] path
        deleteall path [-b batch size]
        delquota [-n|-b] path
        get [-s] [-w] path
        getAcl [-s] path
        getAllChildrenNumber path
        getEphemerals path
        history 
        listquota path
        ls [-s] [-w] [-R] path
        printwatches on|off
        quit 
        reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
        redo cmdno
        removewatches path [-c|-d|-a] [-l]
        set [-s] [-v version] path data
        setAcl [-s] [-v version] [-R] path acl
        setquota -n|-b val path
        stat [-w] path
        sync path
        version
~~~

#### 6.1 创建节点

~~~shell
# -s序列化  -e 临时节点  不指定表示持久节点  acl 权限控制
create [-s] [-e] [-c] [-t ttl] path [data] [acl]
~~~

#### 6.2 读取节点

~~~bash
# 列出指定节点下的所有子节点 -s序列化 -w添加watcher -R所有文件绝对路径 根节点为/
ls [-s] [-w] [-R] path
# 获取指定节点的数据内容和属性信息 -s序列化 -w添加watcher
get [-s] [-w]  path
~~~

#### 6.3 更新节点

~~~bash
# -s更新后显示全部信息 -v指定版本，版本不对更新无效
set [-s] [-v version] path data
~~~

#### 6.4 删除节点

~~~bash
delete [-v version] path
~~~

#### 6.5 quota

对节点增加限制，**超出限制之后日志打印warn级别信息，不会报错**

~~~bash
# -n子节点的最大个数 -b数据值的最大长度 val子节点最大个数或数据值的最大长度
delquota [-n|-b] path
setquota -n|-b val path
listquota path
~~~

### 7. 数据模型

ZK的数据模型在结构上和标准文件系统非常相似，拥有一个层次的命名空间，采用**树形结构层次**。树中的每个子节点被称为`Znode`，具有以下特点：

1. 兼具文件和目录两种特点：可以像文件一样**维护数据**、元信息、`ACL(访问控制列表)`、时间戳等数据结构，又能像目录一样可以**作为路径标识**的一部分，并有用子`Znode`
2. 具有原子性操作：读/写操作都将获取/替换掉节点的所有数据。每个节点有用自己的`ACL`，限定了特定用户对目标节点的可执行操作
3. 存储数据大小有限制：通常以`KB`为大小单位，常规使用中应当`小于1MB`
4. 通过路径引用：类似于Linux中的文件路径。**路径必须是绝对的**，必须由`/`开头。路径由Unicode字符串组成，**必须是唯一的**。`/zookeeper`用来保存管理信息，如：关键配额信息

#### 7.1 数据结构图

<img src="https://gitee.com/hiloj/imgs/raw/master/test/zk数据结构图.png" alt="结构图" style="zoom: 80%;" />

图中的每个节点称为一个`Znode`。每个`Znode`有3部分组成：

1. `stat`：状态信息，描述该`Znode`的版本、权限等信息
2. `data`：与该`Znode`关联的数据
3. `children`：该`Znode`下的子节点

#### 7.2 节点类型

节点的类型在创建时就确定了，不能改变

`Znode`有两种：**临时节点**和**永久节点**

+ 临时节点：生命周期依赖于会话。会话结束，临时节点被自动删除，也可以手动删除。**临时节点不允许拥有子节点**
+ 永久节点：生命周期不依赖于会话。只能在客户端显示执行删除

`Znode`有序列化的特性。如果创建的时候指定的话，该`Znode`的名字后面会自动追加一个不断增加的序列号。序列号对于当前节点的父节点唯一，可以用来记录每个子节点创建的先后顺序。格式为`%10d`

因此，存在四种类型的`Znode`：

+ PERSISTENT：永久节点
+ EPHEMERAL：临时节点
+ PERSISTENT_SEQUENTIAL：序列化永久节点
+ EPHEMERAL_SEQUENTIAL：序列化临时节点

#### 7.3 节点属性

~~~bash
[zk: localhost:2181(CONNECTED) 56] get -s /text
111
# Znode创建的事务id
cZxid = 0x7
# 节点创建时的时间戳
ctime = Wed Aug 26 00:41:31 UTC 2020
# Znode被修改的事务id，每次Znode的变化都会产生一个唯一的事务id
mZxid = 0xb
# 节点最新一次更新发生时的时间戳
mtime = Wed Aug 26 00:50:44 UTC 2020
pZxid = 0x7
# 子节点的版本号。当子节点有变化(增加或删除)时加1
cversion = 0
# 数据版本号,每次set操作都加1
dataVersion = 3
# acl版本号
aclVersion = 0
# 如果为临时节点，值表示与该节点绑定的session id;如果不是，值为0
ephemeralOwner = 0x0
# 数据长度
dataLength = 3
# 子节点数量
numChildren = 0
~~~

### 8 Watcher

ZK提供了**分布式数据发布/订阅功能**。通过引入**Watcher机制**来实现这种分布式的通知功能。它允许客户端向服务端注册一个Watcher监听，当服务端的一些事件触发了这个Watcher，那么就向指定客户端发送一个事件通知来实现分布式的通知功能。可以概括为三个过程：

1. 客户端向服务端注册Watcher
2. 服务端事件发生触发Watcher
3. 客户端回调Watcher得到触发事件情况

触发事件种类很多，如：节点创建、删除、改变、子节点改变等。

#### 8.1 Watcher机制特点

+ 一次性触发

  > 触发监听后，一个watcher event就会被发送到设置监听的客户端，这种效果是一次性的，后续再次发生同样的事件，不会再次触发

+ 事件封装

  > ZK使用`WatcherEvent`对象来封装服务器端事件并传递
  >
  > `WatcherEvent`对象包含事件的三个基本属性：
  >
  > + 通知状态（`keeperState`）
  > + 事件类型（`EventType`）
  > + 节点路径（`path`）

+ event异步发送

  > 通知事件都是异步的

+ 先注册再触发

  > 必须客户端先去服务端注册监听，这样事件发送才会触发监听，通知给客户端

#### 8.2 通知状态和事件类型

常见的通知状态和事件类型如下表所示：

| 通知状态           | 事件类型              | 触发条件           | 说明        |
| ------------------ | --------------------- | ------------------ | ----------- |
|                    | `None`(-1)            | C/S建立连接        |             |
| `SyncConnected`(0) | `NodeCreated`(1)      | 数据节点被创建     |             |
|                    | `NodeDeleted`(2)      | 数据节点被删除     | C/S保持连接 |
|                    | `NodeDataChanged`(3)  | 数据节点数据变更   | C/S保持连接 |
|                    | `NodeChildChanged`(4) | 数据节点子节点变更 | C/S保持连接 |
| `Disconnected`(0)  | `None`(-1)            |                    | C/S断开连接 |
| `Expired`(-112)    | `None`(-1)            |                    |             |
| `AuthFailed`(4)    | `None`(-1)            |                    |             |

其中，`连接状态事件（type=Node，path=null）`不需要客户端注册

#### 8.3 设置watcher

~~~bash
get [-s] [-w] path
~~~

### 9. Java 连接

#### 9.1 pom文件

~~~xml
<dependencies>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.6.1</version>
    </dependency>
</dependencies>
~~~

#### 9.2 java调用

~~~java
public class App {
    public static void main(String[] args) {
        try {
            ZooKeeper zooKeeper = new ZooKeeper("192.168.56.101:2181", 30000, new Watcher() {
                @Override
                public void process(WatchedEvent watchedEvent) {
                    System.out.println("事件类型为：" + watchedEvent.getType());
                    System.out.println("通知状态为：" + watchedEvent.getState());
                    System.out.println("路径为：" + watchedEvent.getPath());
                }
            });
            System.out.println("节点状态：" + zooKeeper.exists("/text1",true));
            String s = zooKeeper.create("/text1", "测试数据".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
            zooKeeper.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~

### 10. 选举机制

默认的算法是`FastLeaderElection`，采用`投票数大于半数则胜出`的逻辑

#### 10.1 概念

+ 服务器ID

  > 即配置的`myid`中的值，值越大在选举算法中权重越大

+ 选举状态

  + `LOOKING`：竞选状态
  + `FOLLOWING`：随从状态，同步leader状态，参与投票
  + `OBSERVING`：观察状态，同步leader状态，不参与投票
  + `LEADING`：领导者状态

+ 数据ID

  > 服务器中存放的最新数据version
  >
  > 值越大说明数据越新，在选举算法中权重越大

+ 逻辑时钟

  > 即投票次数。从0开始递增，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其他服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断

#### 10.2 全新集群选举

假设有5台服务器，**每个服务器均没有数据**，它们的编号分别为：1，2，3，4，5，**按编号依次启动**，选举过程如下：

+ 服务器1启动，给自己投票，然后发投票信息。由于其他机器还没有启动所有它收不到反馈信息，服务器1的状态为`Looking`
+ 服务器2启动，给自己投票，与服务器1交换信息，由于服务器2的编号大，服务器2胜出，但此时投票数没有大于一半，服务器2状态为`Looking`
+ 服务器3启动，给自己投票，与服务器1、2交换信息，由于服务器3的编号大，服务器3胜出，此时投票数大于一半，服务器3成为`Leader`，服务器1、2成为`Follower`
+ 服务器4启动，给自己投票，与服务器1、2、3交换信息，由于服务器3为`Leader`，服务器4成为`Follower`
+ 服务器5启动，给自己投票，与服务器1、2、3、4交换信息，由于服务器3为`Leader`，服务器5成为`Follower`

#### 10.3 非全新集群选举

对于运行正常的ZK集群，中途有机器宕机，需要重新选举时，选举过程需要加入**数据ID、服务器ID和逻辑时钟**

选举的规则：

1. 逻辑时钟小的选举结果被忽略，重新投票
2. 统一逻辑时钟后，数据ID大的胜出
3. 数据ID相同的情况下，服务器ID大的胜出

### 11. 典型应用

#### 11.1 数据发布与订阅(配置中心)

发布者将数据发布到ZK节点上，共订阅者动态获取数据，实现配置信息的集中式管理和动态更新。

应用在启动的时候回主动来获取一次配置，同时在节点上注册一个Watcher，以后每次配置有更新的时候，都会实时通知到订阅的客户端，从而达到获取最新配置信息的目的

适合**数据量很小的场景**，这样数据更新可能会比较快

#### 11.2 命名服务(Naming Service)

在分布式系统中，通过使用命名服务，客户端应用能够根据制定名字来获取资源或服务的地址、提供者等信息。被命名的实体通常可以是集群中的机器、提供的服务地址、远程对象等等——这些我们可以统称为名字。其中较为常见的就是一些分布式服务框架中的服务地址列表。通过调用ZK提供的创建节点的API，能够很容易创建一个`全局唯一的path`，这个path可以作为一个名称。

Dubbo使用ZK来作为其命名服务

#### 11.3 分布式锁

得益于ZK保证了数据的强一致性，可以用于分布式锁。锁服务可以分为两类：保持独占和控制时序

+ 保持独占

  > 所有试图来获取这个锁的客户端，最终只有一个可以获得这把锁。通常的做法就是通过`create`节点的方式来实现。所有客户端都去创建，最终成功创建的客户端就拥有了这把锁

+ 控制时序

  > 所以视图来获取这个锁的客户端，最终都会被安排执行，只是有个全局时序。也是通过`create`节点的方式来实现，前提是节点是**预先存在**的，客户端在它下面**创建临时有序节点**。ZK的父节点维持序列，保证子节点创建的时序性，从而形成每个客户端的全局时序