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
+ 设置myid
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

#### 6.1 客户端连接

~~~shell
zkCli.sh -server ip
~~~

