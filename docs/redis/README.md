## Redis

> `Reids`是一个开源的内存数据库，可以用作数据库、缓存和消息中间件
>
> 它支持多种类型的数据结构（如：`字符串(strings)`、`散列(hashes)`、`列表(lists)`、`集合(sets)`、`有序集合(sorted sets)`）与范围查询（如：`bitmaps`、`hyperloglogs`、`geospatial`）。
>
> 内置了`复制`、`LUA脚本`、`LRU驱动事件`、`事务`和不同级别的`磁盘持久化`
>
> 通过`Redis哨兵（Sentinel）`和`自动分区`提供高可用性

### 1. 安装

~~~bash
# 拉取redis镜像
docker pull redis
# 启动redis容器
docker run --name myredis -d -p 6379:6379 redis
# 执行redis-cli，方便通过命令行操作redis
docker exec -it myredis redis-cli
# 命令查询,通过命令组(cluster、connection、geo、hash、HyperLogLog、keys、list、
# pubsub、scripting、server、set、sorted_set、strigns、transactions)
help @cluster
~~~

### 2. 基本数据结构

![结构](https://gitee.com/hiloj/imgs/raw/master/test/redis.png)

> Redis有5种基本数据结构：`strings`、`lists`、`sets`、`hashes`、`sorted sets`

#### 2.1 strings

常用来缓存用户信息，缓存JSON序列化后的字符串，取信息要反序列化

> Redis的字符串是**动态字符串**，内部结构实现类似于ArrayList，**采用预分配冗余空间的方式来减少内存的频繁分配**，即实际分配空间capacity一般大于字符串实际长度len。
>
> 扩容方式：**len<1M,capacity翻倍；len>1M,每次增加1M空间**
>
> **最大长度为512M**

<img src="https://gitee.com/hiloj/imgs/raw/master/test/redis-strings.png" alt="strings" style="zoom:80%;" />

| 命令及用法                   | 作用                    |
| ---------------------------- | ----------------------- |
| `set  `<K>  <V>              | 添加                    |
| `get  `<K>                   | 查询属性值              |
| `exists `<K>                 | 查询数量                |
| `del `<K>                    | 删除                    |
| `mset `<K>  <V>  <K>  <V>... | 批量添加                |
| `mget` <K>  <K> ...          | 批量查询                |
| `expire `<K>  5              | 5s后过期                |
| `setex` <K>  5  <V>          | set + expire            |
| `setnx` <K>  <V>             | <K>不存在就创建         |
| `incr` <K>                   | <V>增加1，<V>必须是整数 |
| `incrby` <K> 5               | <V>增加5                |

#### 2.2 lists

常用来做异步队列使用。将需要延后处理的任务结构体序列化成字符串放入lists中，另一个线程从这个lists中轮询数据进行处理

> 相当于LinkedList(链表)，增删快，查找慢
>
> lists弹出最后一个元素之后，该list自动被删除，内存被回收

<img src="https://gitee.com/hiloj/imgs/raw/master/test/redis-lists.png" alt="lists" style="zoom:80%;" />

底层存储的是一个快速链表`quicklist`或压缩列表`ziplist`。

当列表元素较少，会使用一块连续的内存存储，即为`ziplist`。

当列表元素较多，改用`quicklist`(由链表和`ziplist`组成)。

| 命令                          | 作用                                |
| ----------------------------- | ----------------------------------- |
| `rpush/lpush` <K>  <V> <V>... | 右进/左进 列表数据                  |
| `rpop/lpop` <K>               | 右出/左出 列表数据                  |
| `llen` <K>                    | 返回列表长度                        |
| `lindex` <K>  <index>         | 返回指定索引位置的数据，复杂度 O(n) |
| `lrange` <K> <start> <stop>   | 返回指定范围数据，复杂度O(n)        |
| `ltrim`<K> <start> <stop>     | 移除指定范围的数据，复杂度O(n)      |

#### 2.3 hashes

> 相当于HashMap，数组＋链表
>
> 采用渐进式rehash策略，一次扩容会保留两个hash结构，然后在后续定时任务中及hash的子指令中一点点迁移到新的hash结构中，迁移结束，删除旧的hash结构，内存回收

<img src="https://gitee.com/hiloj/imgs/raw/master/test/reids-hashes.png" alt="hashes" style="zoom:80%;" />

| 命令                      | 作用          |
| ------------------------- | ------------- |
| hset <K>  <field>  <V>    | 添加/修改字典 |
| hgetall <K>               | 获取所有字典  |
| hlen  <K>                 | 返回字典长度  |
| hincrby <K>  <fidle>  <V> | 自增          |

#### 2.4 sets

用来存放中奖的用户ID

>相当于HashSet，`无序`且`唯一`，内部实现相当于一个特殊的字典，字典所有的`value`都是`NULL`
>
>集合中最后一个元素移除后，数据结构自动删除，内存被回收

| 命令                  | 作用                                         |
| --------------------- | -------------------------------------------- |
| sadd <K> <member>...  | 添加/修改集合                                |
| smembers <K>          | 获取所有集合                                 |
| smembers <K> <member> | 查询member是否存在，相当于`contains(member)` |
| scard <K>             | 获取长度，相当于`count()`                    |
| spop <K>              | 弹出一个                                     |

#### 2.5 sorted sets

用来存放需要排序的数据，如点赞数、学生成绩

> 有序列表，类似于`SortedSet`和`HashMap`的结合体，内部实现是跳表

<img src="https://gitee.com/hiloj/imgs/raw/master/test/redis-zset.png" alt="zset" style="zoom:80%;" />

| 命令                                    | 作用                                |
| --------------------------------------- | ----------------------------------- |
| zadd <K> <score> <member>               | 添加元素                            |
| zrange  <K> <start> <stop>              | 按score排序列出，参数区间为排名范围 |
| zrevrange  <K> <start> <stop>           | 按score逆序列出，参数区间为排名范围 |
| zcard <K>                               | 相当于`count()`                     |
| zscore <K> <member>                     | 获取指定`member`的`score`           |
| zrank <K> <member>                      | 获取指定`member`的排名              |
| zrangebyscore <K>  <member> <min> <max> | 根据分值区间遍历集合，inf表示`∞`    |
| zrem <K> <member>                       | 删除`member`                        |

#### 2.6 容器型数据结构的通用规则

容器型数据结构：`list`、`set`、`hash`、`zset`，它们共享下面两条通用规则：

1. 如果容器不存在，就自动创建一个，再进行操作
2. 如果容器里元素没有了，就立即删除容器，释放内存

#### 2.7 注意事项

<font color=red>Redis所有的数据结构都可以设置过期时间，过期是以**对象**为单位的，如：一个hash结构过期，整个hash对象都过期，而不是其中的一个子Key</font>

<font color=red>如果一个字符串设置了过期时间，然后通过`set`方法进行修改，则过期时间会消失</font>

### 3. 分布式锁

[参考链接](https://www.cnblogs.com/2YSP/p/10389001.html)

分布式锁本质上要实现的目标就是在Redis占一个位置，其他线程也要来占用相同的位置时，只能放弃或稍后重试

~~~bash
# 分布式锁
set k 1 ex 5 nx
~~~



![分布式锁](https://gitee.com/hiloj/imgs/raw/master/test/image-20200827110230958.png)

#### 3.1 超时问题

Redis分布式锁不能解决超时问题，如果加锁和释放锁之间逻辑处理时间超出了锁的过期时间，会出现问题

+ 线程A获取锁并执行任务，任务没有执行完成，过期时间到了

+ 线程B获取锁并执行任务，此时，线程A执行完成，释放了锁
+ 线程C在线程B执行任务中间获取了锁

为避免这个问题，有两种方案：

1. 不用于较长时间的任务
2. `set`指令的value值设置为一个随机数，释放锁时先匹配随机数是否一致，一致了再删除key，由于匹配value和删除key不是一个原子操作，Redis也没有提供类似的指令，所以要**通过Lua脚本来处理，因为Lua脚本可以保证连续多个指令的原子性操作**

~~~lua
# 匹配并删除的lua脚本
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
~~~

#### 3.2 可重入性

Redis分布式锁如果要支持可重入，需要对客户端的`set`方法进行包装，使用线程`ThreadLocal`变量存储当前持有锁的计数

#### 3.3 锁冲突处理

当客户端在请求加锁失败时，一般由3种策略来处理家锁失败：

1. 直接抛出异常，通知用户稍后重试
2. `sleep`一会儿再重试
3. 将请求转移至延时队列，过一会儿再试

### 4. 延时队列

Redis用作消息队列，没有ack保证，因此，它不适用于对消息的可靠性有很高要求的场景

#### 4.1 异步消息队列

Redis的`list`(列表)数据结构常用来作为异步消息队列使用，使用`rpush/lpush`操作入队列，使用`rpop/lpop`来出队列。

![异步消息队列](https://gitee.com/hiloj/imgs/raw/master/test/延时队列.png)

#### 4.2 队列空了

客户端通过`pop`来获取消息，如果队列空了，客户端会一直空轮询，不但会导致客户端的CPU和Redis的QPS升高，还可能导致Redis的慢查询会显著增多

**通常让客户端线程调用`sleep`来解决这个问题**

#### 4.3 队列延迟

让客户端调用`sleep`能够降低空轮询的频率，但是会导致消息的延迟增大，如果有1个消费者，那么延迟就是1s，此时，可以通过`blpop/brpop`指令替代`rpop/lpop`处理延迟问题，通过设置阻塞时间，当有数据时立即读取，无数据时，阻塞休眠。

#### 4.4 空闲链接自动断开

如果线程一直阻塞在那里，Redis的客户端连接就成了闲置连接，闲置过久，服务器一般会主动断开连接。因此，编写客户端消费者时要捕获异常，并添加重试功能

### 5. 位图

位图不是一种实际的数据类型，而是**在字符串类型上定义的一组面向位的操作**。因为字符串是二进制安全的大对象，最大长度是512MB，所以只能配置`2^32`个不同的位

位图操作分为两种：

+ 获取和设置值
+ 计算给定范围内位的数量，如：总体统计

可以使用普通的`get/set`直接获取和设置整个位图的内容，也可以使用下列位图操作来处理

| 命令                             | 作用                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `setbit `<K>  <offset>  <V>      | 设置指定偏移位的值（0或1）                                   |
| `getbit` <K>  <offset>           | 获取指定便宜位的值                                           |
| `bitcount` <K> [start end]       | 统计被设置为1的bit数                                         |
| `bitop`<operation> <dest> <K>... | 对一个或多个K进行逻辑操作（`and`、`or`、`xor`、`not`），将结果保存dest中 |
| `bitpos` <K> <V>                 | 返回字符串里面第一个被设置为1或者0的bit位                    |
| `bitfield`                       | 操作多字节位域                                               |

#### 5.1 基础使用

Redis的位数组是**自动扩展**的，如果设置了某个偏移位置超出了现有的内容范围，就会自动将位数组进行零扩充

~~~java
// 字符串转二进制
for (char c : "h".toCharArray()) {
    System.out.println(Integer.toBinaryString(c));
}
~~~

通过字符串转二进制得到结果：`01101000`

通过位操作符设置值为1的位即可，即h为1、2、4位

~~~bash
setbit k 1 1
setbit k 2 1
setbit k 4 1
# 上面三个指令等同于下面一个指令
set k h
# 获取值 h
get k
# 获取偏移4位对应的值
getbit k 4
~~~

#### 5.2 bitfield

> 将Redis字符串当做位数组，对变长位宽和任意位字节对齐的指定整型位域进行寻址
>
> 操作多字节位域，执行一系列操作并返回一个响应数组，在参数列表中每个响应数组匹配响应的操作
>
> 三个子指令：`get`、`set`、`incrby`，最多只能处理64个连续的位，超出64位需要使用多个子指令
>
> 一个溢出行为设置命令：`overflow [WRAP|SAT|FAIL]`

##### 5.2.1 存取操作

~~~bash
set w hello
# 从第一个位开始取4个位,结果是无符号数(u)
bitfield w get u4 0
# 从第一个为开始取4个位,结果是有符号位(i)
bitfield w get i4 0
# 执行多个子指令
bitfield w get u4 0 get u3 2 get i4 0 get i3 2
# 将字符e改为a
bitfield w set u8 8 97
~~~

+ 有符号数是指获取的位数组中第一个位是符号位，剩下的才是值。如果第一位是1，那就是负数。
+ 无符号数是指获取的位数组没有符号位，全部都是值
+ 有符号数最多可以获取64位，无符号数只能获取63位（Redis协议中的integer是有符号数），超出位数限制报错

##### 5.2.2 自增操作

`incrby`用来对指定范围的位进行自增操作，可能出现溢出。如果增加了正数，会出现上溢；如果增加了负数，会查询下溢。

Redis默认的处理是折返。出现溢出就将符号位丢掉。8位无符号数255加1溢出，变为0；8位有符号位127加1溢出，变为-128

~~~bash
# 从第三个位开始,对接下来的4位无符号数+1
bitfield w incrby u4 2 1
~~~

##### 5.2.3 溢出策略

~~~bash
set w hello
bitfield w overflow sat incrby u4 2 1
bitfield w overflow sat incrby u4 2 1
bitfield w overflow sat incrby u4 2 1
bitfield w overflow sat incrby u4 2 1
bitfield w overflow sat incrby u4 2 1
# 保持最大值
bitfield w overflow sat incrby u4 2 1
~~~

设置溢出策略的指令`overflow`，**溢出策略仅影响接下来的一条子命令**。

溢出策略有三种：

+ `wrap(默认)`：回环算法，适用于有符号和无符号整型两种类型
  + 无符号整型：对整型最大值进行取模操作
  + 有符号整型：上溢从最小的负数开始取数，下溢从最大的正数开始取数
+ `fail`：失败算法，检测到上溢或下溢时，不做任何操作，返回NULL
+ `sat`：饱和算法，下溢之后设为最小的整型值，上溢之后设为最大的整型值

### 6. HyperLogLog

`HyperLogLog`数据结构是Redis的高级数据结构，提供了不精确（标注差是0.81%）的去重计数方案，用来解决非精准统计的业务场景，如：页面UV（独立访问次数）统计。<font color=red>需要占用12K或更少的存储空间</font>

| 命令                      | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `pfadd` <K> <element>...  | 将除了第一个参数以外的餐宿存储到以第一个参数为变量名的HLL结构体中 |
| `pfcount` <K>             | 返回存储在HLL结构体的变量K的近似总数                         |
| `pfmerge`<K> <element>... | 将多个HLL合并成一个HLL                                       |

### 7. 布隆过滤器

布隆过滤器专门用来解决**去重**的业务场景，相比set去重，它在空间上能节省90%以上。

用来告诉你：**某样东西一定不存在或可能存在**

#### 7.1 安装

Redis官方提供的布隆过滤器到了Redis 4.0作为一个插件加载到Redis Server中，给Redis提供了强大的布隆去重功能

~~~bash
docker pull redislabs/rebloom
docker run --name bloom -p 6380:6379 -d redislabs/rebloom
docker exec -it bloom bash
redis-cli
~~~

| 命令                                         | 描述                            |
| -------------------------------------------- | ------------------------------- |
| `bf.add` <K> <V>                             | 添加元素                        |
| `bf.exists` <K> <V>                          | 查询元素是否存在                |
| `bf.madd` <K> <V>                            | 批量添加                        |
| `bf.mexists` <K> <V>                         | 批量查询元素是否存在            |
| `bf.reserve` <K> <error_rate> <initial_size> | 显式创建布隆过滤器，K存在则报错 |

+ `error_rate`：错误率，**默认0.01**，越低需要的空间越大
+ `initial_size`：预计放入元素的数量，**默认100**，实际数量超出时，误判率会上升

#### 7.2 原理

> 在Reids中是**一个大型的位数组**和**几个不一样的无偏（算的比较均匀）的hash函数**
>
> 过程：
>
> 1. 多个hash函数对key进行hash，得到一个整数索引值
> 2. 整数索引值对位数组长度进行取模运算得到一个位置
> 3. 将得到的位置处设置为1

![布隆过滤器](https://gitee.com/hiloj/imgs/raw/master/test/布隆过滤器.png)

#### 7.3 常见应用

缓存穿透

数据过滤拦截

### 8. 限流(todo)

限流是指通过一系列算法，限制每个用户调用API的频率，防止API被过度使用，比较经典的限流算法有以下四种：

+ 计数器算法
+ 滑动时间窗口算法
+ 漏桶算法
+ 令牌桶算法

#### 8.1 计数器（固定窗口）算法

> 在固定的时间窗口内，可以允许固定数量的请求进入。超过数量就拒绝或者排队，等待下一个时间段
>
> 优点：<font color=green>在新的时间窗口，可以立即处理大量的请求，无需等待</font>
>
> 缺点：<font color=red>在一个窗口临界点的前后时间突发大量请求，极端情况会出现2倍流量</font>

#### 8.2 滑动窗口算法

> 将时间周期分为n个小周期，分别记录每个小周期内访问次数，并且根据时间滑动删除过期的小周期

### 9. 范围查询

`Redis`在3.2版本之后添加了`地理空间(geospatial)`索引半径查询，可以使用它来实现附附近功能

底层使用`GeoHash`算法，距离公式是`Haversine`公式（仅适用于地球，而不是完美的球体）

最大的偏差是`0.5%`

在集群环境中单个Key对应的数据量不宜超过1M，否则会导致集群迁移出现卡顿现象，影响线上服务的正常运行，因此，Geo的数据使用单独的Redis实例部署，不使用集群环境，如果数据量过亿以上，需要对Geo数据进行拆分，这样可以显著降低单个zset集合的大小

| 命令                                                   | 描述                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `geoadd` <K> <经度> <纬度> <member>                    | 添加指定的地理空间位置（经度（±180度）、纬度（±85.05度）、位置） |
| `geodist` <K> <member1> <member2>  [m\|km\|ft\|mi]     | 计算两个元素之间的距离，单位：米、千米、英里、尺             |
| `geohash` <K> <member>                                 | 返回一个或多个位置元素的11个字符的`geohash`字符串，可以再[这里](http://geohash.org/${hash})定位 |
| `geopos` <K> <member>                                  | 返回给定位置元素的经度和纬度                                 |
| `georadius` <K> <经度> <纬度> <距离> [m\|km\|ft\|mi]   | 返回以给定位置为中心，指定范围内的所有元素                   |
| `georadiusbymember` <K> <member> <距离>[m\|km\|ft\|mi] | 返回指定元素附近的其他元素                                   |

~~~bash
# 添加地理空间位置
geoadd company 116.48105 39.996794 juejin
geoadd company 116.514203 39.905409 ireader 116.489033 40.007669 meituan

# 计算两个元素之间的距离,单位为km
geodist company juejin ireader km

# 获取元素位置(经度和纬度)
geopos company juejin

# 获取元素的hash值
geohash company juejin

# 获取以(116,39)为中心,方圆1000公里以内的元素
georadius company 116 39 1000 km

# 获取juejin 20公里以内最多3个元素,按照距离正排,不排除自身
georadiusbymember company juejin 20km count 3 asc
~~~

### 10. Key查找

从海量的key中找出满足特定前缀的key列表来

+ `keys 正则表达式`
+ `scan custor match 正则表达式 count 数量` -- `custor`游标，第一次为0，后面为前一次返回的值

#### 10.1 大key(todo)

### 11. 线程IO模型

<font color=red>Redis是单线程程序</font>，执行的快是因为它**所有的数据都在内存中，所有的运算都是内存级别的运算**

Redis采用**网络IO多路复用技术**来保证在多连接的时候系统的高吞吐量

#### 11. 非阻塞IO

调用`Socket`（套接字）的读写方法，默认它们是阻塞的。

非阻塞IO在套接字对象上提供了一个选项**non-blocking**，当选项打开，读写方法不会阻塞，读写字节的大小取决于缓冲区内部分配的数据字节数

非阻塞IO有一个问题，就是线程不知道什么时候继续读操作或者写操作，可以通过**事件轮询**来解决这个问题

<img src="https://gitee.com/hiloj/imgs/raw/master/test/事件轮询.png" style="zoom:80%;" />

最简单的事件轮询API是`select`函数，它是操作系统提供给用户程序的API。输入是读写描述符列表`read_fds & write_fds`，输出是对应的**读写事件**，同时提供`timeout`参数。如果没有任何事件来，最多等待`timeout`时间，**此时线程处于阻塞状态**。一旦期间有任何事件到来，就可以立即返回。时间过了之后还是没有任何事件到来，也会立即返回。拿到事件后，线程挨个处理相应事件。处理完之后继续轮询

~~~c
read_events, write_events = select(read_fds,write_fds,timeout)
for event in read_events:
	handle_read(event.fd)
for event in write_events:
	hangle_write(event.fd)
 handle_others()	# 处理其他事情，如定时任务等
~~~

通过`select`系统调用同时处理多个通道描述符的读写事件，将这类系统调用称为多路复用API。

`select`系统调用的性能在描述符特别多的情况下性能会很差，现代操作系统改用`epoll(linux)`和`kqueue(freebsd & macosx)`

服务器套接字`serversocket`对象的`accept`也是通过`select`系统调用的读事件来得到通知的

#### 11.2 指令队列

Redis会将每个客户端套接字都关联一个指令队列。客户端的指令通过队列来排队进行顺序处理，先到先服务

#### 11.3 响应队列

Redis同样会为每个客户端套接字关联一个响应队列。Redis服务器通过响应队列来讲指令的返回结果回复给客户端。

#### 11.4 定时任务

Redis的定时任务记录在一个称为**最小堆**的数据结构中。这个堆中，最快要执行的任务排在堆的最上方。在每个循环周期，Redis都会将最小堆里面已经到点的任务立即进行处理。处理完毕后，将最快要执行的任务还需要的时间记录下来，这个时间就是`select`系统调用的`timeout`参数

### 12. 通信协议

Redis使用的是`RESP协议（Redis Serialization Protocol）`，它是一种直观的文本协议，优势在于实现简单，解析性能极好。

| 类型       | 描述                        | 示例                                    |
| ---------- | --------------------------- | --------------------------------------- |
| 单行字符串 | 以`+`开头                   | `+hello world\r\n`                      |
| 多行字符串 | 以`$`开头，后跟字符串长度   | `$11\r\nhello world\r\n`                |
| 整数值     | 以`:`开头，后跟整数的字符串 | `:1024\r\n`                             |
| 错误信息   | 以`-`开头                   | `-WRONGTYPE Operation against a key...` |
| 数组       | 以`*`开头，后跟数组长度     | `*3\r\n:1\r\n:2\r\n:3\r\n`              |
| NULL       | 用多行字符串表示，长度为-1  | `$-1\r\n`                               |
| 空串       | 用多行字符串表示，长度为0   | `$0\r\n\r\n`                            |

`set author codehole`：`*3\r\n$3\r\nset\r\n$6\r\nauthor\r\n$8\r\ncodehole\r\n`

### 13. 持久化

Redis的持久化机制有两种

+ RDB快照
  + 一次全量备份，内存数据是二进制序列化形式，存储紧凑
+ AOF日志
  + 连续的增量备份，记录内存数据修改的指令记录文本
  + 长期运行会变得很庞大，需要定期进行AOF重写，给日志瘦身

#### 13.1 RDB快照原理

Redis使用操作系统的多进程`COW(copy on write)`机制来实现快照的持久化。

在Linux操作系统中，在持久化时会调用`glibc`的函数`fork`产生一个子进程，快照持久化完全交给子进程来处理，父进程继续处理客户端请求。子进程刚产生时，**和父进程共享内存里面的代码段和数据段**。

子进程做数据持久化，不会修改现有的内存数据结构，只是对数据结构进行遍历读取，然后序列化接到磁盘中。父进程对内存数据结构进行不间断的修改

此时，会使用操作系统的`COW`机制来进行数据段页面的分离。数据段是由很多操作系统的页面（大小为`4K`）组合而成，当父进程对其中一个页面的数据进行修改时，会将被共享的页面复制一份分离出来，然后对这个复制的页面进行修改。这时，子进程响应的页面还是进程产生时那一瞬间的数据，然后，子进程就可以安心遍历数据并序列化写磁盘了

#### 13.2 AOF原理

Redis在收到客户端修改指令后，先进行参数校验，如果没有问题，就立即将该指令文本存储到AOF日志中，即：`先存磁盘，再执行指令`

#### 13.3 AOF重写

Redis提供了`bgrewriteaof`指令用于对AOF日志进行瘦身。原理是：开辟一个子进程对内存进行遍历转换成一系列Redis的操作指令，序列化到一个新的AOF日志文件中。序列化完毕后将操作期间发生的增量AOF日志追加到这个新的AOF日志文件中，追加完毕后就立即替代旧的AOF日志文件

#### 13.4 fsync

当程序对AOF日志文件进行写操作时，实际上是将内容写到了内核为文件描述符分配的一个内存缓存中，然后内核会异步将脏数据刷回到磁盘，如果机器突然宕机，AOF日志内容可能还没来得及刷到磁盘上，可能出现日志丢失

针对这个问题，Linux的`glibc`提供了`fsync`函数可以将指定文件的内容强制从内核缓存刷到磁盘，`fsync`是一个磁盘的IO操作，为了避免频繁的`fsync`导致的Redis性能下降，生产环境的服务器中，Redis通常每隔1秒左右执行一次`fsync`

通常Redis的主节点是不会进行持久化操作，持久化操作主要在从节点运行。所以，要**做好实时监控工作，保证网络畅通或者快速修复**，避免因网络分区导致从节点连不上主节点。另外，还应该再**增加一个从节点以降低网络分区的概率**

#### 13.5 混合持久化

重启Redis时，很少用RDB快照来恢复内存状态，因为会丢失大量数据，通常使用AOF日志重放，所以启动需要花费很长时间

Redis 4.0为了解决这个问题，引入了**混合持久化** ——将rdb文件的内容和增量的AOF日志文件存在一起。此时，AOF日志不再是全量的日志，而是自持久化开始到持久化结束这段时间内发生的增量AOF日志，通常很小。

于是，在Redis重启时，先加载rdb的内容，然后再重放增量AOF日志，大大提升了重启效率

### 14. 管道

> 将多条指令分批次执行改为将多条指令一次执行

#### 14.1 压力测试

Redis自带一个压力测试工具`redis-benchmark`

~~~bash
# 普通压测
redis-benchmark -t set -q
# 加入管道的压测 -P表示单个管道内并行的请求数量
redis-benchmark -t set -P 2 -q
~~~

<img src="https://gitee.com/hiloj/imgs/raw/master/test/请求交互流程.png" style="zoom:80%;" />

1. 客户端进程调用`write`将消息写到`Kernel`（操作系统内核）为套接字分配的发送缓冲区`send buffer`
2. 客户端`Kernel`将`send buffer`中的内容发送到`NIC`（网卡），网卡硬件将数据通过`Gateway Router`（网际路由）送到服务器的网卡
3. 服务器`Kernel`将`NIC`的数据放到为套接字分配的接收缓冲区`recv buffer`
4. 服务器进程调用`read`从`recv buffer`中取出消息进行处理
5. 服务器向客户端发送消息同理

> `write`操作将消息写到缓冲区就立即返回，不耗时
>
> `read`操作时，如果缓冲区是空的，就需要等待数据的到来，耗时
>
> 管道将客户端所有的`write`操作执行结束，发送数据从`send buffer`到`recv buffer`，客户端`read`只需要等待一个网络的来回开销开销，就读取到了最终的结果

### 15. 事务

#### 15.1 Redis的事务

> Redis的事务并不具有真正的原子性，事务队列中的某一个指令运行失败，后面的指令还是会继续运行
>
> Redis事务一般结合管道（pipeline）一起使用

| 命令      | 描述       |
| --------- | ---------- |
| `multi`   | 事务的开始 |
| `exec`    | 事务的执行 |
| `discard` | 事务的丢弃 |

~~~bash
# 开启事务
> multi
OK
# 指令存在放在服务器的事务队列中
> incr books
QUEUED
> incr books
QUEUED
# 丢弃事务队列需要在exec之前执行
> discard
# 执行整个事务队列
> exec
# 返回修改后的值
~~~

#### 15.2 Watch

当两个并发客户端要对Redis中的数据进行修改时，会出现并发问题，可以**通过分布式锁来避免冲突**

分布式锁是一个悲观锁，Redis还提供了一种乐观锁——`watch`

Redis禁止在`multi`和`exec`之间执行`watch`指令，必须在`multi`之前执行`watch`

1. `watch`在事务开始之前监控一个或多个**关键变量**
2. 服务器收到`exec`指令，要顺序执行缓存的事务队列时，Redis检查**关键变量**自`watch`之后，是否被修改
   + 修改，`exec`指令抛出异常或返回**null**，告知客户端事务执行失败，此时，客户端一般选择重试
   + 没有改，顺序执行事务队列

~~~java
public static int watchText(Jedis jedis, String id) {
    while(true) {
        jedis.watch(id);
        int val = Integer.parstInt(jedis.get(id));
        val *= 2;
        Transaction tx = jedis.multi();
        tx.set(id,String.valueOf(val));
        List<Object> res = tx.exec();
        // 修改成功后，中断循环
        if(res != null) {
            break;
        }
    }
    return Integer.parstInt(jedis.get(id));
}
~~~

### 16. PubSub

**Redis消息队列是不支持消息的多播机制的**

为了支持消息多播，Redis单独使用`PubSub`模块来支持消息多播

#### 16.1 消息多播

消息多播允许生产者生产一次消息，中间件负责将消息复制到多个消息队列，每个消息队列由相应的消费组进行消费。

它是分布式系统常用的一种解耦方式，用于**将多个消费组的逻辑进行拆分**。支持了消息多播，多个消费组的逻辑就可以放到不同的子系统中。

如果是普通的消息队列，就得将多个不同的消费组逻辑串接起来放在一个子系统中，进行连续消费

<img src="https://gitee.com/hiloj/imgs/raw/master/test/消息多播.png" style="zoom:80%;" />

