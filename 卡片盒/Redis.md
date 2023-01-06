# Redis 概述
> 一种 NoSQL (Not Only Sql) 非关系型数据库
- 以 key-value 模式储存
- 不遵守 SQL 标准
- 不支持 ACID
- 远超于 SQL 的性能
- 用不着 SQL 和用了 SQL 也不行的情况, 可以考虑使用 [[NoSQL]]
- Redis 与 Memcache 的三点不同
  1. 支持持久化
  2. 支持多数据类型
  3. 使用单线程+多路 IO 复用技术

# Redis 的应用场景
1. 使用 Redis 作为缓存用于存储极少变更的数据
2. 使用 Redis 存储 session 用于保证[[SpringCloud#微服务|微服务]]架构下的 session 一致性
3. 设置手机验证码有效事件
4. 作为购物车
5. 实现[[分布式锁]]
6. 实现[[单点登录]]
7. ...

# Redis 知识点补充
1. 默认端口号 6379
2. Redis 有 16 个数据库, 默认操作 0 号库
3. Redis 是单线程的, 使用多路 IO 复用技术来保证高并发

# 安装 Redis
- 安装 C 语言编译环境
  - `yum -y install gcc`
  - 测试 gcc 版本 `gcc --version`
- 下载 `redis-6.2.1.tar.gz 放/opt 目录 `
- 解压 `tar -zxvf redis-6.2.1.tar.gz`
- Cd redis-6.2.1 执行 make && make install
- 启动 redis-server 服务

## 配置后台启动
- 备份 redis. Conf
  - 在/root 目录下创建 myredis 目录 `mkdir myredis`
  - 拷贝一份 redis. Conf 到 myredis 目录 `cp /opt/redis-6.2.1/redis.conf /root/myredis`
- 修改配置 `daemonize no` 为 yes（L247）让服务在后台启动
- 使用修改后的配置启动 Redis `redis-server /root/myredis/redis.conf`
- 连接进入 Redis  `redis-cli`
- 关闭 Redis
  - `redis-cli shutdown`
  - 多实例时关闭指定实例，指定端口关闭：`redis-cli -p 6379 shutdown`

# Redis 使用规范
在使用 Redis 时, 存储和获取数据的关键就是 key, 它有两个要求:
1. 不能重复, 不同数据的 key 必须是唯一的, 否则会产生覆盖
2. 见名知意, 遵守一定的 Redis-key 命名规范, 使数据结构更加规范与清晰, 保证数据的可读性
==在开发中最常用的 key 格式规范为 : `object:id:field` , 例如 `user:0314:info` ==

在企业开发中, 除了极少数特殊值外, Redis key 必须设置超时时间

# 操作 Redis 的关键字
- ` keys *|pattern` 查看当前库中所有 key
- `exists key [key ...]` 判断某个 key 是否存在
- `del key [key ...]` 删除 key
- `unlink key [key ...]` 非阻塞删除
- ` expire key seconds`  为给定的 key 设置 10 面过期时间
- `[[ttl]] key` 查看过期时间
- `dbsize` 查看当前库的 key 数量
- `flushdb` 清空当前库
- `flushall` 通杀全部库

# Redis 中的数据类型

## String 类型
- `set key value` 键值对
- `get key` 获取
- `append key value`  将指定的 v 追加到原值末尾
- `strlen key` 获得值的长度
- `setnx key value` 只有在 key 不存在时设置 key 的值
- `incr key` 将 key 中储存的数字值增 1, 只能对数字值操作，如果为空，新增值为 1
- `decr key`  将 k 存储的数值减一

> Redis 中的原子性, 不会被线程机制打断, 命令一旦开始, 就一直运行到结束

- `mset key value [key value ...]`  同时设置多个 kv
- `mget key [key ...]` 同时获得多个 kv
- `getrange key start end`  获得值的范围, 类似 substring
- `getset key value` 以旧换新, 设置新值得同时获得旧值

## List 类型

> 单键多值, 可以添加元素到表头 (左边) 或表尾 (右边), 他的底层是双向链表
> 每个 key 对应一个列表, 例如 id 为 7 的商品有五个库存, 则存储结构为: 7:\[7, 7, 7, 7, 7]
- `lpush|rpush key element [element ...]` 从左边/右边插入一个或多个值
- `lpop|rpop key [count]` 从左|右边弹出多个值, 值在键在, 值光键亡
- ` rpoplpush source destination` 列表右边吐出一个值，插到\<key2>列表左边
- ` lrange key start stop` 按照索引下标获得元素
- ` llen key` 获得列表长度
- `lindex key index` 根据索引下标获得元素
- `linsert key BEFORE|AFTER pivot element` 在某个值前|后插入 element
- `lrem key count element` 移除指定个数的 element

## Set 类型
> Set 可以自动去重, 他是 string 类型的无序集合, 他的底层是一个 value 为 null 的 hash 表
- `sadd key member [member ...]` 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略
- `smembers key` 取出该集合所有值
- `sismember key member` 判断是否存在
- `scard key` 查看集合内元素个数
- `srem key member [member ...]` 删除某个元素
- `spop key [count]` 随机吐出一个值
- `srandmember key [count]` 随机取出 n 个值, 不会删除
- `sinter key [key ...]` 取交集
- `sunion key [key ...]` 返回并集
- `sdiff key [key ...]` 返回两个集合的差集元素
- `smove source destination member` 移动

## Hash 类型
> 数据结构为字典 dict{}
> 一个 key 下的多个键值对集合 key (field, value), 适合存储 pojo
- `hset key field value [field value ...]` 给 k 集合中的 f 键赋值 v
- `hget key field` 从 k 中 f 取出值
- `hkeys key` 列出该 hash 集合所有的 file
- `hvals key` 获取所有 file 键的值
- `hsetnx key field value` 只有当某个 file 不存在时才设置成功

## Zset (sorted set)
> 数据结构为字典+跳表
> 不重复的有序集合, 和 set 相似
> 不同之处为有序集合的每个成员都关联了评分 (score), 按照从低到高排序
- `zadd key score member [score member ...]` 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
- `zrange key start stop [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]` 返回评分评分**从低到高**的有序集合
- `zrange key 0-1 withscores` 同时显示分数
- `zrevrange key start stop [WITHSCORES]`  从高到低排
- `zrank key member`  查看该元素从低到高排行第几
- `zrevrank key member` 查看该元素从高到低排行第几
- `zrem key member [member ...]` 移除指定元素

## Bitmaps
以字节的方式进行存储, 将数据使用二进制的形式进行存储
可用作为布隆过滤器使用

## Geospatial
存储二维坐标, 也就是地图上的经纬度

# Redis 发布订阅模式
在发布订阅模式下, 消息发布者发布的消息会被所有订阅这个频道的订阅者接收到, 订阅者之间并无竞争关系
在 Java 程序中, 使用 `RedisMessageListenerContainer` 注入订阅主题
```java
package com.atguigu.gmall.activity.redis;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.data.redis.connection.RedisConnectionFactory;  
import org.springframework.data.redis.core.StringRedisTemplate;  
import org.springframework.data.redis.listener.PatternTopic;  
import org.springframework.data.redis.listener.RedisMessageListenerContainer;  
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;  
  
@Configuration  
public class RedisChannelConfig {  
      
      
    /**  
     * 注入订阅主题  
     *  
     * @param factory         redis连接工厂  
     * @param listenerAdapter 消息监听适配器  
     * @return  
     */  
    @Bean  
    public RedisMessageListenerContainer container(RedisConnectionFactory factory,  
                                                   MessageListenerAdapter listenerAdapter) {  
          
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();  
          
        container.setConnectionFactory(factory);  
        container.addMessageListener(listenerAdapter, new PatternTopic("seckillpush"));  
          
          
        return container;  
    }  
      
    /**  
     * 监听适配器  
     *  
     * @param messageReceive  
     * @return  
     */  
    @Bean  
    public MessageListenerAdapter listenerAdapter(MessageReceive messageReceive) {  
          
        //通过反射获取方法，并执行  
        return new MessageListenerAdapter(messageReceive, "receiverMessage");  
    }  
      
      
    /**  
     * 注入操作的模板对象  
     *  
     * @param factory redis连接工厂  
     * @return  
     */  
    @Bean  
    public StringRedisTemplate template(RedisConnectionFactory factory) {  
          
        return new StringRedisTemplate(factory);  
    }  
      
}
```



# Redis 配置文件介绍

## 网络相关配置
- Includes 创建新的配置文件, 引入其他配置文件
- Bind 默认情况只能接受本机访问请求
- Protect mode 如果开启了 protected-mode，那么在没有设定 bind ip 且没有设密码的情况下，Redis 只允许接受本机的响应
- Timeout 一个空闲的客户端维持多少秒会关闭，0 表示关闭该功能。即永不关闭
- Tcp-keepalive 对访问客户端的一种心跳检测，每个 n 秒检测一次。单位为秒，如果设置为 0，则不会进行 Keepalive 检测，建议设置成 60

## GENERAL 通用
- Daemonize 是否为后台进程，设置为 yes
- Pidfile 存放 pid 文件的位置，每个实例会产生一个不同的 pid 文件
- Loglevel  指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为**notice** 四个级别根据使用阶段来选择，生产环境选择 notice 或者 warning

## SECURITY 安全
- Requirepass foobared 访问密码的查看、设置和取消

## LIMITS 限制
- Maxmemory 设置 redis 可以使用的内存量。一旦到达内存使用上限，redis 将会试图移除内部数据，移除规则可以通过 maxmemory-policy 来指定。**必须设置**，否则，将内存占满，造成服务器宕机
- Maxmemory-policy 内存淘汰策略
  - Volatile-lru：使用 LRU 算法移除 key，只对设置了过期时间的键；（最近最少使用）
  - Allkeys-lru：在所有集合 key 中，使用 LRU 算法移除 key
  - Volatile-random：在过期集合中移除随机的 key，只对设置了过期时间的键
  - Allkeys-random：在所有集合 key 中，移除随机的 key
  - Volatile-ttl：移除那些 [[TTL]] 值最小的 key，即那些最近要过期的 key
  - Noeviction：不进行移除。针对写操作，只是返回错误信息

# 在 Java 中集成 Redis
1. 引入 redis 启动器依赖 `spring-boot-starter-data-redis`
2. 在 spring 配置文件中配置 redis 连接相关信息
3. 编写配置类初始化 `redisTemplate` 并将它注入 IoC 容器对象中
4. 在需要使用的地方引入对应的依赖后直接注入即可使用

# Redis [[事务 Transaction]]
Redis 事务的主要作用就是串联多个命令, 防止别的命令插队

## Redis 事务三特性
- 单独的隔离操作: 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- 没有隔离级别的概念: 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行
- 不保证原子性: 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## Redis 中事务指令 Multi / Exec / discard
从输入 Multi 命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入 Exec 后，Redis 会将之前的命令队列中的命令依次执行

对于事务的处理
- 在组队阶段出错, 这个事务全部回滚
- 在执行阶段出错, 只有报错的命令不被执行, 其他的正常处理

## Redis 中事务锁

Redis 采用[[乐观锁]]
- Watch key 上锁
  - 在执行 multi 之前，先执行 `watch key1 [key2]`, 可以监视一个 (或多个) key ，如果在事务执行之前这个 (或这些) key 被其他命令所改动，那么事务将被打断。
- `unwatch` 解锁
  - WATCH 命令对所有 key 的监视。
     如果在执行 WATCH 命令之后，EXEC 命令或 DISCARD 命令先被执行了的话，那么就不需要再执行 UNWATCH 了。


# Redis 解决超卖问题
1. 使用乐观锁来解决超卖问题, 但是又会留下库存遗留问题 (多个进程抢同一张票==但此时还有其他余票==, 只有一个成功, 其他的全部失败后不再抢票)
2. 通过乐观锁 + lua 脚本最终解决
3. 也可以使用 [[Sentinel]] 设置访问数量限制的方式解决

# Redis 持久化策略

## RDB (Redis Database)
- 在指定的时间间隔内将内存中的数据集以快照的方式写入磁盘
- 他的缺点是==两次持久化间隔时间之间如果发生故障，会导致数据丢失==

## AOF (Append Only File)
- 以日志的形式来记录每一个写 (增/删/改) 操作 (增量保存)
- 只允许追加但不可以改写文件, redis 启动之初会读取该文加重新构建数据

- RDB 与 AOF 同时开启, redis 会通过 aof 来恢复数据, 这是因为 RDB 可以数据丢失, 而 aof 会保存每次的写操作
- 只在命令执行完毕后, 才记录日志
- 这个文件是**不能被修改**的, 一旦修改后即损坏无法修复

## 如何选择序列化方式
- 如果数据不敏感, 可以单独使用 RDB
- 不建议单独用 AOF, 因为可能会出现 Bug
- **如果只是做纯内存缓存, 可以都不用**
- 系统默认两种都不开启

# Redis 主从复制※

是指主机数据更新后根据配置和策略，自动同步到备机的 master/slaver 机制，**Master 以写为主，Slave 以读为主**

作用
- 读写分离，性能扩展
- 容灾快速恢复

## 数据一致性问题


同步频率
- Appendfsync always 始终同步，每次 Redis 的写入都会立刻记入日志
- Appendfsync everysec 每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失
- Appendfsync no redis 不主动进行同步，把同步时机交给操作系统。

## 怎么用
1. 创建三个实例
   1. 新建 redis6379. Conf 填入以下内容

      ```shell
      #继承这个配置文件并对他的部分内容进行重写覆盖
      include /root/myredis/redis.conf
      #配置进程id
      pidfile /var/run/redis_6379.pid
      #指定端口号
      port 6379
      #生成持久化文件名
      dbfilename dump6379.rdb
      ```

   2. 拷贝该配置文件 `cp redis6379 redis63**`
   3. 将配置文件内的内容修改替换 `:%s/6379/6380`
   4. 可以在配置文件中指定该从机的优先级 **replica-priority 10** 数值越低, 优先级越高, 用于选举主机时使用, 默认 100
2. 启动三个 redis 服务
3. 使用 `info replication` 命令在 redis 服务内查看当前服务的主从复制相关情况
4. 使用 `slaveof <host> <port>` 使此服务成为某实例的从服务器
   1. 主机挂掉后, 重启进来仍然是主机
   2. 从机挂掉后, 需要重新使用该命令

# 哨兵模式 Sentinel

## 介绍
哨兵模式是一种特殊的模式, 它是一个独立的进程, 能够独立运行. 哨兵通过监控的方式获取 master 的工作状态, 当 master 的状态异常时, 它自动进行故障迁移, 从而提高系统的高可用性

## 原理
1. 哨兵节点会以每秒一次的频率, 对每个 redis 节点发送 `PING` 指令, 并通过 redis 节点的回复来判断其状态
2. 当哨兵检测到 master 故障时, 会自动在 slave 节点中选择一个节点作为新的 master, 然后通过发布订阅模式通知其他 slave 修改配置文件, 切换主机

## 主要作用
- 故障自迁移: 保障系统的高可用
- 提醒: 被监控的某个 redis 节点出现问题时, Sentinel 可以通过 api 向管理员或其他应用程序发送通知


# Redis 集群搭建
1. **删除持久化数据**, 在创建集群前, 需要将先前生成的所有持久化数据删除掉
2. 制作六个 redis 实例
   1. 修改配置文件
      ```shell
      include /root/myredis/redis.conf
      pidfile /var/run/redis_6379.pid
      port 6379
      dbfilename dump6379.rdb
      #打开集群模式
      cluster-enabled yes
      #设置阶段配置文件名
      cluster-config-file nodes-6379.conf
      #设置节点失联时间(毫秒),一旦超过该时间,集群自动进行主从切换
      cluster-node-timeout 15000
      ```

   2. 将该文件拷贝 6 分, 分别将端口名修改替换为其他端口号
   3. 启动所有 redis 服务
3. 将六个节点组合为一个集群: 组合前请确保所有 redis 实例启动后，nodes-xxxx. Conf 文件都生成正常。
4. 组合集群
   ```shell
      redis-cli --cluster create --cluster-replicas 1 192.168.6.200:6379 192.168.6.200:6380 192.168.6.200:6381 192.168.6.200:6389 192.168.6.200:6390 192.168.6.200:6391
      #这里的IP要使用真实的IP,不能使用127.0.0.1
      #--cluster-replicas 1 表示采用最简单的方式配置集群，一台主机，一台从机，正好三组。
      ```

5. 登录方式
    ```shell
      redis-cli -c -p 6379
      #添加-c命令使用集群策略连接,设置数据会自动切换到相应的写主机
      ```

   ```shell
      cluster nodes #查看集群信息
      ```

 插槽
> 一个 Redis 集群包含 16384 个插槽（hash slot），数据库中的每个键都属于这 16384 个插槽的其中一个，
>
> 集群使用公式 CRC16 (key) % 16384 来计算键 key 属于哪个槽，其中 CRC16 (key) 语句用于计算键 key 的 CRC16 校验和。
>
> 集群中的每个节点负责处理一部分插槽。举个例子，如果一个集群可以有主节点，其中：
>
> * 节点 A 负责处理 0 号至 5460 号插槽。
> * 节点 B 负责处理 5461 号至 10922 号插槽。
> * 节点 C 负责处理 10923 号至 16383 号插槽。
>
> 在 redis-cli 每次录入、查询键值，redis 都会计算出该 key 应该送往的插槽，如果不是该客户端对应服务器的插槽，redis 会报错，并告知应前往的 redis 实例地址和端口。

**redis-cli 客户端提供了 –c 参数实现自动重定向。**
不在一个 slot 下的键值，是不能使用 mget, mset 等多键操作。
可以通过{}来定义组的概念，从而使 key 中{}内相同内容的键值对放到一个 slot 中去。

# Redis 应用中产生的问题

## 缓存穿透

> 当客户端**请求的的资源 key 在缓存中不存在**, 导致每次针对这个 key 的请求都无法命中, 这个请求就会透过缓存直接请求数据库, 当同时出现大量并发/恶意请求请求到这个数据库, 就会将其压垮

解决方案
1. 对不存在数据进行存储, 使用空值 null 进行存储
2. 设置访问白名单, 与运维人员沟通, ban 掉非法访问来源
3. 采用 [[day08_AOP+Redisson实现缓存数据查询和布隆过滤器#布隆过滤器]] (Bloom Filter), 来对请求的 key 做是否存在的判断过滤

## 缓存击穿

> 某一个热门数据的 Key 在某一时刻失效, 此时若有大量并发请求涌入请求这个 Key, 这些请求都会直接打到 DB 中, 大量并发瞬间将数据库压垮

解决方案
1. 预先设置热门数据, 在访问的高峰之前就把一些热门的数据提前存入到 Redis 中, 并适当的加大这些热点值的时长
2. 实时监控数据, 动态的调整热门数据 key 的过期时长
3. 使用[[锁 Lock|本地锁]]或[[分布式锁]]对访问 DB 的请求进行隔离, 保证同时间只有一个线程访问 DB, 再取出热门数据存入缓存中

## 缓存雪崩

> 与缓存击穿的区别在于, 缓存击穿指的是某个 key 过期, 而缓存雪崩指的是在同一时间内的多个 key 集中过期, 所有对这些数据的请求都会直接瞬间全部打到 DB 上, 将 DB 直接冲垮, 缓存雪崩对系统的冲击是极其可怕的

解决方案
1. 搭建多级缓存架构, [[Nginx]] 缓存 + Redis 缓存 + 其他缓存搭配使用
2. 使用锁和队列, 保证不会有大量的线程同时对后台的数据库进行读写. 不适用于高并发的情况
3. 设置过期标志更新缓存, 记录缓存是否过期, 一旦缓存过期即触发通知一个线程去数据库中查询并更新实际 key 的缓存
4. 分散缓存失效时间, 在原有的失效时间上增加一个随机值, 使多个 key 同时过期的概率降低, 避免同时集体失效的情况


# Redis 内存淘汰策略

## 简介
Redis 默认使用的内存容量是 0, 也就是说, 默认情况下 Redis 服务器不会限制使用内存的大小. 如果没有对 Redis 进行内存限制, 它会一直使用系统可用内存, 在生成环境中可能出现内存溢出导致服务器宕机问题, 所以在 Redis 中必须指定内存淘汰策略, 通过 [[Redis#LIMITS 限制]]设置内存淘汰策略;

## 工作原理
当客户端发起需要更多内存的申请后, redis 会检查当前的内存使用情况, 如果实际使用内存已经超过了 MaxMemory, redis 就会根据用户配置的内存淘汰策略选出无用的 key 
Redis 会将设置了 [[TTL]] 的 key 存放到一个独立的字典中, 以后会定期遍历一个字典来删除到期的 key; Redis 默认会对该字典进行每秒 10 次扫描 (100ms 一次); 过期扫描不会遍历整个字典, 而是从字典中随机选取 20 个 key, 删除这 20 个 key 中的过期元素, 如果删除的比例超过 1/4, 会再次进行次步骤

## 内存淘汰策略
1.  `volatile-lru` : 使用 LRU (Least Recently Used) 算法, 从==设置 [[TTL]] 的数据集==中挑选出最近最少使用的数据进行淘汰
2. `allkeys-lru` : 从所有数据集中挑选出最近最少使用的数据进行淘汰, 此淘汰策略面向所有的 key 
3. `volatile-random` : 从已经设置 [[TTL]] 的数据集中任意选择数据淘汰
4. `allkeys-random` : 在所有数据集中, 随机淘汰 key 
5. `volatile-ttl` ：移除那些 TTL 值最小的 key，即最临近过期的 key 
6. `noeviction(不驱逐)` ：不进行任何移除。在内存使用超过配置后返回错误


## 淘汰策略使用场景
- 在 Redis 中, 不同的数据访问频率差异是较大的, 某些 key 的访问频率高, 而某些 key 访问频率极低. 针对这种情况, 在 Redis 只作为缓存使用时, 可用考虑使用 `allkeys-lru(从所有key中淘汰最近最少使用)` 策略, 如果所有数据访问频率都大致相等, 则可用选用 `allkeys-random(从所有key中随机淘汰)` 策略
- 在希望一些数据能够被长期保存, 而一些数据可用被淘汰时, 可用选择 `volatile-ttl(临近过期)` 或 `volatile-random(从过期字典中随机淘汰)`
- Redis 设置过期时间 `expire` 会消耗额外的内存, 如果此时 Redis 中的数据仅作为缓存使用时, 可用使用 `allkeys-lru(所有key最近最少使用)` 策略, 自动淘汰访问量少的间, 也能够更高效的利用内存



# Redis 序列化
在没有指定 Serializer 时会默认创建一个 JdkSerializer, 它在序列化后的数据是不可读的