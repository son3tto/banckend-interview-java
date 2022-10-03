# Database

## JDBC principle

JDBC(Java Database Connectivity)即JAVA数据库连接，是一种数据库连接规范。符合JDBC规范的、用来访问数据库的API称之为驱动。

![](https://pic4.zhimg.com/80/v2-4ff6bf526433472db64fd3859e63f7bf_1440w.jpg)

在上面那张图里面，基本上也交代出了使用一个数据库的一般步骤。

（1）注册一个驱动

（2）使用驱动和数据库连接

（3）使用连接对象获取操作数据库的执行对象

![img](/Users/jingwenzhu/Desktop/workspace/banckend-interview-java/Database.assets/v2-317844ff24702ae0c0bdd24aaa1512b0_1440w.jpeg)

![img](/Users/jingwenzhu/Desktop/workspace/banckend-interview-java/Database.assets/v2-6a6eddd2acf4f82ae4aed6ae6f39ee7e_1440w.jpeg)

完整的JDBC案例会存在以下4个类：

**（1）DriverManager：该类管理数据库驱动程序。**

**（2）Connection：管理数据库建立的连接。**

**（3）Statement：负责将要执行的sql体局提交到数据库。**

**（4）ResultSet：执行sql查询语句返回的结果集。**

## [Redis](https://try.redis.io)

- 存储位置在内存，因此读写速度非常快。

- 所以经常被用来：缓存。同时还被用来：分布式锁、消息队列
- 提供多种数据类型（支持不同的业务场景）
  - k/v, list, set, set, hash etc.
- transaction
- 持久化
  - 可以将内存中的数据serialise到磁盘中，重启的时候可以再次加载使用
  - 灾难恢复机制
- 集群

### 数据处理流程

用户请求 =〉 检查缓存中是否存在对应数据 =〉 没有，则检查数据库中是否存在对应数据 =〉 还没有，返回空数据。

### 为什么要用缓存，为什么用redis做缓存

**高性能**：假如用户第一次访问数据库中的某些数据的话**，需要从硬盘中读取，速度比较慢。但是如果用户访问的数据<u>是高频且不会经常改变</u>的数据的话，就可以把这些数据存在缓存中。

**高并发：**一般像 MySQL 这类的数据库的 QPS 大概都在 1w 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（就单机 redis 的情况，redis 集群的话会更高）。

> QPS（Query Per Second）：服务器每秒可以执行的查询次数；

由此可见，直接操作缓存能够承受的数据库请求数量是远远大于直接访问数据库的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。进而，我们也就提高了系统整体的并发。

### Redis data type table

| type                              | function                                                     | desciption                                                   |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| String                            | 二进制安全的数据结构。存储任何类型数据。如：字符串、整数、浮点数、图片（base64编码/解码/图片路径）、序列化后的对象。 | 使用场景：1. 存储常规数据（缓存session、tooken、图片地址、序列化后的对象）使用`SET`和`GET`；2.需要<u>计数</u>的场景（用户单位时间的请求数量限流、页面单位时间内的访问数）使用`set`,`get`,`incr`,`decr`. 3. 分布式锁（`SETNX` k v）。 |
|                                   | SET key value                                                |                                                              |
|                                   | SETNX key value                                              | 只有在k不存在的时候，设置k的值                               |
|                                   | MSET k1 v1 k2 v2 k3 v3...                                    | 设置一个或多个指定k的值                                      |
|                                   | MGET k1 k2 k3 ...                                            |                                                              |
|                                   | STRLEN k                                                     |                                                              |
|                                   | INCR key                                                     |                                                              |
|                                   | DECR key                                                     | 设置将k中存储的数字-1                                        |
|                                   | EXISTS key                                                   |                                                              |
|                                   | DEL key                                                      |                                                              |
|                                   | EXPIRE key seconds                                           |                                                              |
|                                   | TTL k                                                        |                                                              |
|                                   |                                                              |                                                              |
| LIST                              | 双向链表，支持反向查找遍历。                                 | 场景：1. 信息流展示（最新文章、最新动态）的命令`LPUSH`, `LRANGE`；2. 消息队列（有缺陷） |
|                                   | RPUSH k v1 v2 ...                                            | 在k列表的尾部（右边）添加一个或多个元素                      |
|                                   | LPUSH k v1 v2 ...                                            | 在k列表的头部（左边）添加一个或多个元素                      |
|                                   | LSET k index v                                               | 将k列表index位置的值设置为v                                  |
|                                   | LPOP k                                                       | 移除并获取指定列表的最左边元素                               |
|                                   | RPOP k                                                       |                                                              |
|                                   | LLEN k                                                       | 获取列表元素数量                                             |
|                                   | LRANGE k start end                                           | 获取start和end之间的元素(可用于分页查询)                     |
|                                   |                                                              |                                                              |
| HASH                              | Hash 是一个 String 类型的 field-value（键值对） 的映射表（字典），特别适合用于存储对象，后续操作的时候，你可以直接修改这个对象中的某些字段的值。 | 场景：用户信息、商品信息、文章信息、购物车信息               |
|                                   | 备注：类似于 JDK1.8 前的 `HashMap`，内部实现也差不多(数组 + 链表)。不过，Redis 的 Hash 做了更多优化（**字典**）。 | 相关命令：`hset`,`hmset`,`hget`,`hmget`.                     |
|                                   | HSET k field v                                               | 设置k表中指定字段的值                                        |
|                                   | HSETNX k f v                                                 | 只有指定字段不存在时，设置                                   |
|                                   | HMSET k f1 v1 f2 v2 ...                                      | 同时将1个或多个f-v对指定到k表中                              |
|                                   | HGET k f                                                     | 获取k表中f字段的值                                           |
|                                   | HMGET k f1 f2 ...                                            | 同时获取1个或多个f-v对的值                                   |
|                                   | HGETALL k                                                    | 获取k表所有f-v对的值                                         |
|                                   | HEXISTS k f                                                  | 查看k表中指定的f字段是否存在                                 |
|                                   | HDEL k f1 f2 ...                                             | 删除1个或多个f字段                                           |
|                                   | HLEN key                                                     | 获取k表中字段的数量                                          |
|                                   | HINCRBY k f increment                                        | 对k表的f字段进行运算操作(+)                                  |
|                                   |                                                              |                                                              |
| SET                               | 无序集合，类似hashset。能够实现交集、并集、差集操作。所有元素唯一（所以指令是add，而非set）。 | 应用场景：网站UV统计、文章点赞、动态点赞`scard`. 抽奖系统`spop`,`srandmember`. |
|                                   | SADD k member1 member2 ...                                   | 向k集合添加成员元素                                          |
|                                   | SMEMBERS k                                                   | 获取所有成员                                                 |
|                                   | SCARD k                                                      | 获取k集合的元素数量                                          |
|                                   | SISMEMBER k m                                                | 判断m是否在集合中                                            |
|                                   | SINTER k1 k2                                                 | k1 k2交集 (共同好友)                                         |
|                                   | SINTERSTORE dest k1 k2 ...                                   | 存储交集                                                     |
|                                   | SUNION k1 k2 ...                                             | 并集                                                         |
|                                   | SUNIONSTORE dest k1 k2 ...                                   | 存储并集                                                     |
|                                   | SDIFF k1 k2 ...                                              | 差集 （音乐推荐）                                            |
|                                   | SDIFFSTORE k1 k2 ...                                         | 存储差集                                                     |
|                                   | SPOP k count                                                 | 随机获取并移除k集合中的count个元素                           |
|                                   | SRANDMEMBER k count                                          | 随机获取k集合中的count个元素                                 |
| Sorted Set                        | 比起set增加权重参数score，能够按照score进行排序。            | 举例 ：各种排行榜比如直播间送礼物的排行榜、朋友圈的微信步数排行榜、王者荣耀中的段位排行榜、话题热度排行榜等等。 相关命令 ：`ZRANGE` (从小到大排序) 、 `ZREVRANGE` （从大到小排序）、`ZREVRANK` (指定元素排名)。 |
|                                   | ZADD k score1 member1 score2 member2 ...                     |                                                              |
|                                   | ZCARD k                                                      | 查看set成员数量                                              |
|                                   | ZINTERSTORE destination numkeys key1 key2 ...                |                                                              |
|                                   | ZUNIONSTORE destination numkeys key1 key2 ...                |                                                              |
|                                   | ZDIFF dest destination numkeys key1 key2 ...                 |                                                              |
|                                   | ZRANGE k start end                                           | 获取指定有序集合 start 和 end 之间的元素（score 从低到高）   |
|                                   | ZREVERANGE k start end                                       | 获取指定有序集合 start 和 end 之间的元素（score 从高到底）   |
|                                   | ZREVRANK k m                                                 | 获取指定有序集合中指定元素的排名(score 从大到小排序)         |
| BITMAP                            | 存储的是连续的二进制数字（0 和 1），通过 Bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 Bitmap 本身会极大的节省储存空间。 | 0 1即可表示的场景如举例 ：用户签到情况、活跃用户情况、用户行为统计（比如是否点赞过某个视频）。 相关命令 ：`SETBIT`、`GETBIT`、`BITCOUNT`、`BITOP`。 |
|                                   | 你可以将 Bitmap 看作是一个存储二进制数字（0 和 1）的数组，数组中每个元素的下标叫做 offset（偏移量）。 |                                                              |
| SETBIT k offset v                 |                                                              |                                                              |
| GETBIT k offset                   |                                                              |                                                              |
| BITCOUNT k start end              |                                                              |                                                              |
| BITOP operation destkey k1 k2 ... |                                                              |                                                              |
|                                   |                                                              |                                                              |
| HyperLogLog                       |                                                              |                                                              |
|                                   |                                                              |                                                              |
|                                   |                                                              |                                                              |
|                                   |                                                              |                                                              |
|                                   |                                                              |                                                              |

### String和Hash对比	

|      | String                                       | Hash                                                         |
| ---- | -------------------------------------------- | ------------------------------------------------------------ |
| 存储 | 存储序列化后的对象数据，即存储的是整个对象。 | 根据对象的每个字段进行单独的存储，也可以修改或添加部分字段。 |
| 内存 | 节省内存                                     | 需要string差不多两倍的内存                                   |

#### [使用 Set 实现抽奖系统需要用到什么命令？](https://interview.javaguide.cn/#/./docs/d-2-redis?id=使用-set-实现抽奖系统需要用到什么命令？)

- `SPOP key count` ： 随机移除并获取指定集合中一个或多个元素，适合不允许重复中奖的场景。
- `SRANDMEMBER key count` : 随机获取指定集合中指定数量的元素，适合允许重复中奖的场景。

