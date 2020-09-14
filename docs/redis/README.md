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

