# Redis

###  

### 简介

Redis在以前的版本中是单线程的，而在6.0后对Redis的io模型做了优化，io Thread为多线程的，但是worker Thread仍然是单线程。 



### 使用场景

- 缓存
- 分布式session —— 单点登录系统 ： 单点登录中我们需要随机生成一个token作为key，将用户的信息转为json串作为value保存在redis中，同时设置一定的超时时间
- 限流 : 次数统计，用户IP作为key，可投三次票，每次incr递增，满3次则返回false ;
- 次数统计 : 
  - 点赞
    - 若不需要记录点赞用户信息，直接使用incr即可
    - sadd / srem 微博id 用户id  —— 点赞、取消点赞
    - sismember  微博id 用户id —— 是否点赞
    - smembers 微博id —— 点赞的所有用户
    - scard 微博id —— 点赞数
  - 打卡
  - 签到
- 分布式锁
- 生成全局ID



### 阿里巴巴规范

- 以业务名(或数据库名)为前缀(防止key冲突)，用冒号分隔，比如`业务名:表名:id ` ；
- 非字符串的bigkey，不要使用del删除，使用hscan、sscan、zscan方式渐进式删除，同时要注意防止bigkey过期时间自动删除问题, 例如一个200万的zset设置1小时过期，会触发del操作，造成阻塞 ;
- 实体类型, 要合理控制和使用数据结构内存编码优化配置, 例如使用压缩链表ziplist可以节省内存 ;
- O(N)命令关注N的数量，例如hgetall、lrange、smembers、zrange、sinter等并非不能使用，但是需要明确N的值，有遍历的需求可以使用hscan、sscan、zscan代替；
- 禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理 
- 使用批量操作提高效率：
  - 原生命令：例如mget、mset。
  - 非原生命令：可以使用pipeline提高效率

> redis客户端执行命令4个过程：**发送命令－〉命令排队－〉命令执行－〉返回结果**
>
> 过程称为**Round trip time**(简称RTT, 往返时间)，mget mset有效节约RTT，但大部分命令（如hgetall，并没有mhgetall）不支持批量操作，需要**消耗N次RTT** ，可以依靠pipeline解决
>
> 具体操作：https://www.jianshu.com/p/f66e9584154f



### 淘汰策略

根据自身业务类型，选好maxmemory-policy (最大内存淘汰策略)，设置好过期时间。

默认策略是volatile-lru，即超过最大内存后，**在过期键中使用lru算法进行key的剔除，保证不过期数据不被删除**，但是可能会出现OOM问题。

其他策略如下：

- allkeys-lru：根据LRU算法删除键，不管数据有没有设置超时属性，直到腾出足够空间为止。
- allkeys-random：随机删除所有键，直到腾出足够空间为止。
- volatile-random:随机删除过期键，直到腾出足够空间为止。
- volatile-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略。
- noeviction：不会剔除任何数据，拒绝所有写入操作并返回客户端错误信息"(error) OOM command not allowed when used memory"，此时Redis只响应读操作。

**备注：** > ttl [key name] 返回key剩余的过期时间

 



### 指令数据类型和主要指令

> String

**常用命令** ：set/get/decr/incr/mget等；

> Hash

**常用命令** ：hget/hset/hgetall等

> List

**常用命令** ：lpush / rpush / lpop / rpop / lrange遍历等；

> Set

**常用命令** ：sadd / spop / smembers / sunion等；

> Sorted Set

**常用命令** ：zadd / zrange / zrem / zcard等；



### 序列化

对redis的value使用以下序列化方式进行序列化操作：

1：Jdk Serialization Redis Serializer

2：Generic Jackson 2 Json Redis Serializer

3：String Redis Serializer

4：Generic FastJson Redis Serializer  （最优选）



**下面一个个简单介绍以下**

**1-JdkSerializationRedisSerializer : Bean必须实现序列化接口**

否则将爆序列化java对象时如下报错：

```
DefaultSerializer requires a Serializable payload but received an object of type [com.everestfortune.cf.bean.CaseInfoBean]
```



**2：GenericJackson2JsonRedisSerializer**

获取redis中的数据碰到下面的报错：

```
2019-04-26 11:26:41.510 ERROR 11656 --- [nio-9076-exec-7] c.e.cf.controller.ApplyController
```

: redis获取数据失败，mes=Could not read JSON: Cannot construct instance of `java.time.LocalDate`

```
(no Creators, like default construct, exist):
 cannot deserialize from Object value (no delegate- or property-based Creator)
```

原因：LocalDate这是java8新增的类，GenericJackson2JsonRedisSerializer序列化方式无法识别



**3：StringRedisSerializer**

不能序列化Bean，只能序列化字符串类型的数据，如果value都是字符串类型，可以用该方式序列化



**4： GenericFastJsonRedisSerializer**

目前没有发现问题，很好用



### 高并发高可用递进策略

|         策略          | 缺陷                                                         |
| :-------------------: | :----------------------------------------------------------- |
|       单体架构        | 内存型, 断电记录全部丢失                                     |
|        持久化         | 缓存多为读取服务, QPS高时, 单体架构可能会崩溃                |
|       主从架构        | 主服务器负责 写入 / 读取 ; 从服务器负责读 ; 主服务器实时同步到从服务器 --> 写入过多, 主服务器可能宕机 --> 需手动将从服务器升级为主 |
|       哨兵机制        | 单个哨兵 : 由于网络延迟, 误判主服务器已下线 --> 哨兵集群 : 投票机制 |
|           -           | 一主多从只能提高读性能, 若写入也频繁, 单个主从架构不足以支撑 |
|       分片集群        | 多个主从架构合成一个集群, 数据按分片规则存在不同的节点 ; **每个节点各自存储一部分数据，所有节点数据之和 才是全量数据** ; 制定一个路由规则，对于不同的 key，把它路由到固定一个实例上进行读写 |
| 客户端分片/服务端分片 | 客户端分片 : key 的路由规则 由 客户端维护, 为避免将路由规则耦合到业务代码中, 将路由规则封装成一个模块SDK **><** 服务端分片 : 路由规则不放在客户端来做，而是在客户端和服务端之间增加一个「中间代理层」，数据的路由规则由它维护 ---> 无需与redis集群交互, 仅与proxy交互即可 |

![64011](E:\doc_repo\002-应用\images\64011.png)



客户端分片 :

![6401](E:\doc_repo\002-应用\images\6401.png)



服务端分片 :

![64022](E:\doc_repo\002-应用\images\64022.png)





### 持久化 : RDB  / AOF

RDB （redis database）/ AOF (append-only file)

Redis本身是一种内存性数据库，读写快优势所在，但为了防止系统宕机等意外情况，它默认会有自己的数据持久化策略 —— RDB和AOF



**RDB :** 

- **默认开启**的一种数据持久化策略 ;
- 根据配置，定期地把数据**快照存储**到一个dump.rdb文件;
- redis客户端操作时，可以直接输入 save / bgsave 进行同步、异步操作 ;
- 打开redis.conf文件，可以改变属性:
  - 生成快照文件阈值，比如save 60 1000 (60秒内有1000个key以上发生变化, 则生成快照文件)
  - 快照文件是否压缩 ; 
  - 异步持久化存储 报错，是否继续写入数据等 ;



**AOF**:

- 需要在配置文件中开启AOF持久化方式 ;
- 以**追加日志**的形式, 在一个文件中**不断追加redis的一系列操作指令，因此文件会越来越大** ;
- 追加频率分为三种 : no \ always \ everysec ;
- 存在 rewrite 复写的过程 : 
  - 因为redis的key经常会出现过期失效的操作，比如内存中仅有10w条有效数据，AOF文件中则有100w条的操作记录，Redis会**定期针对 这些有效数据 重新构建AOF文件去覆盖旧文件**
  - 也可手动执行rewrite命令, 分别是rewriteaof和bgrewriteaof
- AOF的日志文件的记录可读性非常的高，即使某一时刻有人执行`flushall`清空了所有数据，只需要拿到aof的日志文件，然后把最后一条的flushall给删除掉，就可以恢复数据



**异同** :

- **RDB每次快照会生成不同的dump.rdb文件，AOF则如名字一样，采取追加的方式存储 redis操作指令到AOF文件中 ;**
- 恢复数据方式不同：rdb恢复数据直接把文件加载到内存，而aof是重新去执行一遍日志记录的指令；
- RDB备份的时间间隔比较长，**几分钟备份一次，如果期间宕机，丢失数据量比较大** >< AOF备份一般设置为1秒钟，所以数据丢失的可能性不大 ; 
- 两者对于生产环境下都是必须的，**RDB用于恢复阶段状态的数据 >< AOF保证数据的完整性**



### 主从架构 是什么？

- 主从架构是redis应对高并发的策略，因为**单机redis能够承载的并发量有限，因此可以建立redis集群，采取主从架构的方式来分解压力，实现高并发下的高可用** ;
- 主从架构本身是一种读写分离的策略 : master主节点负责写，并将数据同步到从节点salve，从节点负责读操作 ;
- 默认新启动的三个redis服务角色都为master，通过设置，如`slaveof 172.17.0.2 6379 `配置主从关系 ;
- 如何保证redis集群主从之间数据一致性？
  - Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份 ;
  - Redis增量复制是指Slave初始化后，开始**正常工作时主服务器发生的写操作同步到从服务器**的过程 ;

![640 (2)](E:\doc_repo\002-应用\images\640 (2).png)





### 哨兵机制是什么？

哨兵（Sentinel）是Redis的主从架构模式下，实现高可用性（high availability）的一种机制。
由一个或多个Sentinel实例（instance）组成的Sentinel系统（system）可以**监视任意多个主服务器，以及这些主服务器属下的所有从服务器**，**并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器**，然后由新的主服务器代替已下线的主服务器继续处理命令请求。



### 哨兵机制的原理

1)：每个Sentinel以**每秒钟一次的频率**向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令。

2)：如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值 (这个配置项指定了需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的。 单位是毫秒，默认为30秒)， 则这个实例会被 Sentinel 标记为主观下线。

3)：如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。

4)：当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线 。

5)：当主服务器宕机，哨兵可以监控到服务宕机，在从服务器中选举产生一个新的主服务器。 



**如何实现哨兵机制？**

- 创建sentinel配置文件，内部进行个性化配置

  ```
  cat <<EOF > /etc/redis/sentinel.conf 
  sentinel monitor redis6379 172.17.0.2 6379 1
  EOF
  
  //对于sentinel.conf文件中的内容，还可以基于实际需求，进行增强配置
  
  sentinel monitor redis6379 172.17.0.2 6379 1 
  daemonize yes #后台运行
  logfile "/var/log/sentinel_log.log" #运行日志
  sentinel down-after-milliseconds redis6379 30000 #默认30秒
  ```

  

### 缓存和数据库的一致性

**逻辑1 :**

**查询操作:**

- 每次查询都优先从redis进行查询，若无数据，则前往数据库查询（@Cacheable注解）;

- 返回数据库数据的同时，查询结果存入redis ;

  

**更新删除操作:**

- 针对插入、更新，**直接操作数据库**，每次均会同步到redis，注意是删除缓存（@CachePut注解）

  - 为什么同步更新缓存时是删除 而不是 更新缓存 ？ 

    - 比如新线程最后更新了数据，但在它之前更新的旧线程因为网络问题滞后了，最后缓存更新过去地就是 旧的数据

  - 更新数据时异步缓存删除，此时用户读取到另一台服务器旧数据怎么办？

    - 对于访问量不大的情况下，这类脏读发生的概率不大 ；
    - 优化：更新与删除操作，每次先删除缓存再前往操作数据库；

  - 更新数据时 缓存删除 失败怎么办？

    - **捕捉 删除失败异常，放入消息队列中进行 删除重试** —— ack回执方式保证删除成功

    - 利用数据库binlog（存储了数据库更新日志），利用阿里的canal实时订阅数据库操作的binlog，同步到消息队列中，之后利用ack机制确保 完成删除缓存

      

**逻辑2 :**

- 对于变化不大的数据，比如首页课程信息等，可以采用定期更新缓存的方式 ;
- 为了避免更新缓存操作同时生效，还可**随机设置key的过期时间** ;

**前提 :**

- 在启动类上添加@EnableCaching注解

**补充 :** memcache也可用于缓存



### Redis缓存雪崩 / 击穿

**缓存雪崩** : 大量缓存key在某一刻集中失效，外加巨大的QPS，大量请求直接冲击数据库 ;

**缓存击穿 :** 某一时刻大量请求涌来，而缓存中没有这一数据（或者因为对数据的请求QPS太大，导致缓存失效），直接冲击数据库，宕机，一重启就宕机一重启就宕机 ;

**解决办法 :** 

- 多级缓存、主从架构、哨兵机制
- 备份 + 消息队列 + 加锁 （注意不能加synchronized互斥锁 -> 读操作也加锁，疯了！）



### 数据结构-跳表(skiplist)

#### 概念

**关键** ：

- 跳表结合了链表和二分查找的思想
- 由原始链表和一些通过“跳跃”生成的链表组成
- 第0层是原始链表，越上层“跳跃”的越高，元素越少
- 上层链表是下层链表的子序列
- 查找时从顶层向下，不断缩小搜索范围

**例如：**

当我们想要查询一个数据的时候，先查上层的链表，就很容易知道数据落在**哪个范围**，然后**跳到下一个层级里进行查询。**这样逐级往下，就把搜索范围一下子缩小了一大半。 

![640 (3)](E:\doc_repo\002-应用\images\640 (3).png)



#### **B+树 vs 跳表**

> 查询操作

**B+树**是多叉树结构，每个结点都是一个16k的数据页，能存放较多索引信息，所以**扇出很高**。**三层**左右就可以存储`2kw`左右的数据。也就是说查询一次数据，如果这些数据页都在磁盘里，那么最多需要查询**三次磁盘IO**。

**跳表**是链表结构，一条数据一个结点，如果最底层要存放`2kw`数据，且每次查询都要能达到**二分查找**的效果，`2kw`大概在`2的24次方`左右，所以，跳表大概高度在**24层**左右。最坏情况下，这24层数据会分散在不同的数据页里，也即是查一次数据会经历**24次磁盘IO**。

因此存放同样量级的数据，B+树的高度比跳表的要少，如果放在mysql数据库上来说，就是**磁盘IO次数更少，因此，B+树查询更快**。

> 写入操作

而针对**写操作**，B+树需要拆分合并索引数据页，跳表则独立插入，并根据随机函数确定层数，没有旋转和维持平衡的开销，因此，**跳表的写入性能会比B+树要好。**

其实，mysql的**存储引擎是可以换的**，以前是`MyIsam`，后来才有的`Innodb`，它们底层索引用的都是**B+树**。也就是说，你完全可以造一个索引为跳表的存储引擎装到mysql里。事实上，`facebook`造了个`rocksDB`的存储引擎，里面就用了**跳表**。直接说结论，它的**写入性能**确实是比innodb要好，但**读性能**确实比innodb要差不少。



#### 为什么Redis选用跳表

redis 是纯纯的内存数据库。

进行读写数据都是操作内存，跟磁盘没啥关系，因此也**不存在磁盘IO**了，所以层高就不再是跳表的劣势了。

并且前面也提到B+树是有一系列合并拆分操作的，换成红黑树或者其他AVL树的话也是各种旋转，目的也是**为了保持树的平衡**。

而跳表插入数据时，只需要随机一下，就知道自己要不要往上加索引，根本不用考虑前后结点的感受，也就**少了旋转平衡的开销**。





### Redis事务



| 命令      | 功能描述                                                     |
| --------- | ------------------------------------------------------------ |
| MULTI     | **事务开始的命令**，执行该命令后，后面执行的对Redis数据类型的**操作命令都会顺序的放进队列中**，等待执行EXEC命令后队列中的命令才会被执行 |
| DISCARD   | **放弃执行队列中的命令**，你可以理解为Mysql的回滚操作，**并且将当前的状态从事务状态改为非事务状态**。 |
| EXEC      | 执行该命令后**表示顺序执行队列中的命令**，执行完后并将结果显示在客户端，**将当前状态从事务状态改为非事务状态**。若是执行该命令之前有key被执行WATCH命令并且又被其它客户端修改，那么就会放弃执行队列中的所有命令，在客户端显示报错信息，若是没有修改就会执行队列中的所有命令。 |
| WATCH key | 表示指定监视某个key，**该命令只能在MULTI命令之前执行**，如果监视的key被其他客户端修改，**EXEC将会放弃执行队列中的所有命令** |
| UNWATCH   | **取消监视之前通过WATCH 命令监视的key**，通过执行EXEC 、DISCARD 两个命令之前监视的key也会被取消监视 |



### Redis分布式锁

**为什么需要分布式锁 ?**

在单体的应用开发场景中，在多线程的环境下，涉及并发同步的时候，为了保证一个代码块在同一时间只能由一个线程访问，我们一般可以**使用synchronized语法和ReetrantLock去保证，这实际上是本地锁的方式**。

也就是说，在同一个JVM内部，大家往往采用synchronized或者Lock的方式来解决多线程间的安全问题。但**在分布式集群工作的开发场景中**，在JVM之间，那么就需要一种更加高级的锁机制，来处理种跨JVM进程之间的线程安全问题
![20210505213026273](E:\doc_repo\002-应用\images\20210505213026273.png)



**用途** : 实现一个简单的秒杀系统的库存扣减 

**逻辑** : 在线程A通过setnx尝试去获取到抢购对象produce对象的锁，若是获取成功旧会返回1，获取不成功，说明当前对象的锁已经被其它线程锁持有 ; 获取锁成功后并设置key的生存时间，能够有效的防止出现死锁，最后就是通过`del`来实现删除key，这样其它的线程就也可以获取到这个对象的锁。



**分布式锁指令** : 主要依靠`setnx、getset、expire、del`这四个命令来实现。

1. `setnx`：命令表示如果key不存在，就会执行set命令，若是key已经存在，不会执行任何操作 ;
2. `getset`：将key设置为给定的value值，并返回原来的旧value值，若是key不存在就会返回返回nil ; 
3. `expire`：设置key生存时间，当当前时间超出了给定的时间，就会自动删除key ;
4. `del`：删除key，它可以删除多个key，语法如下：`DEL key [key …]`，若是key不存在直接忽略 ;



```java
public void redis(Produce produce) {
        long timeout= 10000L; // 超时时间
    
    	Long result= RedisUtil.setnx(produce.getId(), 
                                     String.valueOf(System.currentTimeMillis() + timeout));
    
        if (result!= null && result.intValue() == 1) { // 返回1表示成功获取到锁
         RedisUtil.expire(produce.getId(), 10);  // 有效期为10秒，防止死锁
         //执行业务操作
         ......
         //执行完业务后，释放锁
         RedisUtil.del(produce.getId());
        } else {
           System.println.out("没有获取到锁")
        }
    }
```



**潜在问题** : 执行完setnx成功后设置生存时间不生效，此时服务器宕机，那么key就会一直存在Redis中, 一旦出现了释放锁失败，或者没有手工释放，那么这个锁永远被占用，其他线程永远也抢不到锁, 所以, 需要**保障setnx和expire两个操作的原子性**，要么全部执行，要么全部不执行，二者不能分开。 

**解决办法** : 

1. 使用set的命令时，同时设置过期时间，不再单独使用 expire命令, 如 :  set test "111" EX 100 NX

   > set 命令的完整格式： 
   >
   > set key value \[EX seconds]\[PX milliseconds] [NX|XX] 
   >
   > 
   >
   > EX seconds：设置失效时长，单位秒 
   >
   > PX milliseconds：设置失效时长，单位毫秒 
   >
   > NX：key不存在时设置value，成功返回OK，失败返回(nil) 
   >
   > XX：key存在时设置value，成功返回OK，失败返回(nil) 

2. 通过**定时任务检查是否有设置生存时间**，没有的话都会统一进行设置生存时间 ; 

3. 还有比较好的解决方案, 即在上方逻辑中添加, 没有获取到锁则再次进行key的生存时间操作 ;



最后, Redis实现分布式锁，还可以使用`Redisson`来实现, 开箱即用

**关键** : Redisson分布式锁的框架主要的学习分为下面的5个点

1. 加锁机制
2. 解锁机制
3. 生存时间延长机制
4. 可重入加锁机制
5. 锁释放机制