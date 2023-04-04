# Redis学习

## 入门介绍

**1. redis是什么？**
Redis:REmote DIctionary Server(远程字典服务器)
是完全开源免费的，用C语言编写的，遵守BSD协议，是一个高性能的(key/value)分布式内存数据库，基于内存运行并支持持久化的NoSQL数据库，是当前最热门的NoSql数据库之一,也被人们称为数据结构服务器。
Redis 与其他 key - value 缓存产品有以下三个特点：

Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用；
Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储；
Redis支持数据的备份，即master-slave模式的数据备份；

**2. redis能干什么？**

内存存储和持久化：redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务；
取最新N个数据的操作，如：可以将最新的10条评论的ID放在Redis的List集合里面；
模拟类似于HttpSession这种需要设定过期时间的功能；
发布、订阅消息系统；
定时器、计数器；

**3. 从哪些方面去学习redis？**

数据类型、基本操作和配置；
持久化和复制，RDB/AOF；
事务的控制；
复制；

## Redis的安装及启动关闭

由于企业里面做Redis开发，99%都是Linux版的运用和安装，几乎不会涉及到Windows版，所以Windows版不作为讲解;

1. Linux（CentOS 6.9）下安装redis（3.0.4）

下载获得redis-3.0.4.tar.gz后将它放入我们的Linux目录/opt；
/opt目录下，解压命令

```bash
tar -zxvf redis-3.0.4.tar.gz
```


解压完成后出现文件夹：redis-3.0.4；
进入目录:cd redis-3.0.4；
在redis-3.0.4目录下执行make命令、再执行make install命令；
运行make命令时出现的错误：

安装gcc（能上网：yum install gcc-c++）；
再次make；
jemalloc/jemalloc.h：没有那个文件或目录（运行make distclean之后再make）；
如果make完成后继续执行make install；



2. 查看默认安装目录：/usr/local/bin

redis-benchmark：服务启动起来后执行性能测试工具，可以在自己本子运行，看看自己本子性能如何；
redis-check-aof：修复有问题的aof文件；
redis-check-dump：修复有问题的dump.rdb文件；
redis-cli：客户端，操作入口；
redis-sentinel：redis集群使用；
redis-server：Redis服务器启动命令；
3. 启动

修改redis.conf文件将里面的daemonize no 改成 yes，让服务在后台启动；
将默认的redis.conf拷贝到自己定义好的一个路径下，比如/myconf/redis.conf；
进入/usr/local/bin目录下运行redis-server，运行拷贝出存放了自定义myconf文件目录下的redis.conf文件（redis-server /myconf/redis.conf）;
在/usr/local/bin目录下运行redis-cli，启动客户端（redis-cli -p 6379）；

防火墙设置

```bash
# 关闭防火墙
systemctl close firewalld

# 开放防火墙端口
firewall-cmd --zone=public --add-port=6379/tcp --permanent
```



4. 关闭

```redis
-- 单实例关闭
redis-cli 
shutdown

-- 多实例关闭，指定端口关闭
redis-cli -p 6379 shutdown
```



1. 单进程

单进程模型来处理客户端的请求。对读写等事件的响应是通过对epoll函数的包装来做到的。Redis的实际处理速度完全依靠主进程的执行效率；
epoll是Linux内核为处理大批量文件描述符而作了改进的epoll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率；
2. redis数据库的一些概念及操作

默认16个数据库，类似数组下表从零开始，初始默认使用零号库；
统一密码管理，16个库都是同样密码，要么都OK要么一个也连接不上，redis默认端口是6379；
select命令切换数据库：select 0-15；
dbsize：查看当前数据库的key的数量；
flushdb：清空当前库；
flushall；通杀全部库；
Redis数据类型



# Redis的五大数据类型

## 五大数据类型

1. string（字符串）

string是redis最基本的类型，你可以理解成与Memcached一模一样的类型，一个key对应一个value；
string类型是二进制安全的。意思是redis的string可以包含任何数据。如jpg图片或者序列化的对象 ；
string类型是Redis最基本的数据类型，一个redis中字符串value最多可以是512M；



操作

```redis

```



2. list（列表）
redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）。它的底层实际是个链表。

3. set（集合）
redis的set是string类型的无序集合。它是通过HashTable实现的。

4. hash（哈希，类似java里的Map）

redis的hash 是一个键值对集合；
redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象；
类似Java里面的Map<String,Object>；
5. zset(sorted set：有序集合)

redis的zset 和 set 一样也是string类型元素的集合,且不允许重复的成员；
不同的是每个元素都会关联一个double类型的分数；
redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复；
redis常见数据类型操作命令参考网址
http://redisdoc.com/

## Redis 键(key)

```redis
keys *：查看所有key；
exists key的名字：判断某个key是否存在；
move key dbID（0-15）： 当前库就没有了，被移除了；
expire key 秒钟： 为给定的key设置过期时间；
ttl key： 查看还有多少秒过期，-1表示永不过期，-2表示已过期；
type key： 查看你的key是什么类型；
```



## 常用命令介绍

### Redis 字符串(String)

```redis
set/get/del/append/strlen；
Incr/decr/incrby/decrby：一定要是数字才能进行加减；
getrange/setrange：

getrange：获取指定区间范围内的值，类似between and的关系从零到负一表示全部；
setrange：设置指定区间范围内的值，格式是setrange key 位置值 具体值；
setex(set with expire) 键 秒值 值/setnx(set if not exist) 键

setex：设置带过期时间的key，动态设置 : setex 键 秒值 真实值
setnx：只有在 key 不存在时设置 key 的值：setnx 键 值
mset/mget/msetnx

mset：同时设置一个或多个 key-value 对。
mget：获取所有(一个或多个)给定 key 的值。
msetnx：同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在（如果存在key，则都不会操作，因为msetnx是原子性型操作）。
getset：将给定 key 的值设为 value ，并返回 key 的旧值(old value)。简单一句话，先get然后立即set。
```



### Redis 列表(List）

```redis
lpush/rpush/lrange；

-- 移除列表key的头/尾元素
lpop/rpop

-- 按照索引下标获得元素(从上到下)
lindex key index

-- 返回列表 key 的长度
llen key

-- 根据参数 count 的值，移除列表中与参数 value 相等的元素
lrem key count value

count > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 count 。
count < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 count 的绝对值。
count = 0 : 移除表中所有与 value 相等的值。

-- 截取指定范围的值后再赋值给key
ltrim key start stop

rpoplpush，移除列表的最后一个元素，并将该元素添加到另一个列表头部并返回（格式：rpoplpush source—key destination—key）；
lset，将列表 key 下标为 index 的元素的值设置为 value（格式：lset key index value）；
linsert，（格式：linsert key before|after pivot value）将值 value 插入到列表 key 当中，位于值 pivot 之前或之后；
```

当 pivot 不存在于列表 key 时，不执行任何操作。
当 key 不存在时， key 被视为空列表，不执行任何操作。
如果 key 不是列表类型，返回一个错误。
性能总结
它是一个字符串链表，left、right都可以插入添加；

如果键不存在，创建新的链表；
如果键已存在，新增内容；
如果值全移除，对应的键也就消失了。
链表的操作无论是头和尾效率都极高，但假如是对中间元素进行操作，效率就很惨淡了。



### Redis 集合(Set) 

```redis
sadd/smembers/sismember，格式：

sadd key member [member ...]
smembers key
sismember key member
scard：获取集合里面的元素个数（格式：scard key）；
srem：删除集合中元素（格式：srem key member [member ...]）；
srandmember，（格式：srandmember key [count]）（不会修改set集合）

如果命令执行时，只提供了 key 参数，那么返回集合中的一个随机元素；
如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合；
如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值；
spop，移除并返回集合中的一个随机元素（格式：spop key）；
smove，（格式：smove source destination member）将 member 元素从 source 集合移动到 destination 集合；
数学集合类

差集：sdiff（格式：sdiff key [key ...]）
交集：sinter（格式：sinter key [key ...]）
并集：sunion（格式：sunion key [key ...]）
```



### redis 哈希(Hash) 

```redis
-- 将哈希表 key 中的域 field 的值设为 value 
hset key field value

-- 返回哈希表 key 中给定域 field 的值
hget key field

-- 同时将多个 field-value对设置到哈希表 key 中
hmset key field value [field value ...]

-- 返回哈希表 key 中，一个或多个给定域的值
hmget key field [field ...]

-- 返回哈希表 key 中，所有的域和值
hgetall key

-- 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略
hdel key field [field ...]

-- 返回哈希表 key 中域的数量
hlen key

-- 查看哈希表 key 中，给定域 field 是否存在
hexists key field

-- 返回哈希表 key 中的所有域
hkeys key

-- 返回哈希表 key 中所有域的值
hvals key

-- 自增
hincrby/hincrbyfloat

-- 为哈希表 key 中的域 field 的值加上增量 increment
hincrby key field increment

-- 为哈希表 key 中的域 field 加上浮点数增量 increment 
hincrbyfloat key field increment

-- 将哈希表 key 中的 field 值设为 value ，当且仅当域 field 不存在
hsetnx key field value
```



### redis 有序集合Zset(sorted set) 

```redis
-- 将多个 member 元素及其 score 值加入到有序集 key 当中
zadd key score member [[score member] [score member] ...]

-- 返回有序集 key 中，指定区间内的成员，其中成员的位置按 score 值递增(从小到大)来排列
zrange key start stop [WITHSCORES]

-- 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员
zrangebyscore key min max [WITHSCORES] [LIMIT offset count]

-- 移除有序集 key 中的一个或多个成员，不存在的成员将被忽略
zrem key member [member ...]

-- 返回有序集 key 的基数
zcard key

-- 返回有序集 key 中， score 值在 min 和 max 之间(默认包括 score 值等于 min 或 max )的成员的数量
zcount key min max

-- 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大)顺序排列，排名以 0 为底
zrank key member

-- 返回有序集 key 中，成员 member 的 score 值；
zscore key member

-- 返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递减(从大到小)排序，排名以 0 为底，也就是说， score 值最大的成员排名为 0 
zrevrank key member

-- 返回有序集 key 中，指定区间内的成员，其中成员的位置按 score 值递减(从大到小)来排列
zrevrange key start stop [WITHSCORES]

-- 返回有序集 key 中， score 值介于 max 和 min 之间(默认包括等于 max 或 min )的所有的成员。有序集成员按 score 值递减(从大到小)的次序排列
zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]
```



# 解析配置文件（redis.conf）



### INCLUDES（包含） ###
和我们的Struts2配置文件类似，可以通过includes包含，redis.conf可以作为总闸，包含其他；

### GENERAL（通用） ###
daemonize no
Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程；

启用守护进程后，Redis会把pid写到一个pidfile中，在/var/run/redis.pid；
pidfile /var/run/redis.pid
当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定；

port 6379
指定Redis监听端口，默认端口为6379；
如果指定0端口，表示Redis不监听TCP连接；



tcp-backlog 511
设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。在高并发环境下你需要一个高backlog值来避免慢客户端连接问题（注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值），所以需要增大somaxconn和tcp_max_syn_backlog两个值来达到想要的效果；

bind 127.0.0.1
绑定的主机地址；
你可以绑定单一接口，如果没有绑定，所有接口都会监听到来的连接；

timeout 0
当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能；

tcp-keepalive 0
TCP连接保活策略；
单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60 ；

loglevel notice
指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning；

logfile ""
指定了记录日志的文件，空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null；

syslog-enabled no
是否把日志输出到syslog中；

syslog-ident redis
指定syslog里的日志标志

syslog-facility local0
指定syslog设备，值可以是USER或LOCAL0-LOCAL7；

databases 16
设置数据库的数量，默认数据库为0；

### SNAPSHOTTING（快照） ###
save
指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合；
save 900 1：900秒（15分钟）内有1个更改
save 300 10：300秒（5分钟）内有10个更改
save 60 10000：60秒（1分钟）内有10000个更改
stop-writes-on-bgsave-error yes
后台存储错误停止写；
rdbcompression yes
指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大；
rdbchecksum yes
在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能；
dbfilename dump.rdb
指定本地数据库文件名，默认值为dump.rdb；
dir ./
指定本地数据库存放目录（rdb、aof文件也会写在这个目录）；
### REPLICATION（复制） ###
详细请看下文Redis的复制（Master | Slave）；

### SECURITY（安全） ###
requirepass foobared
设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过auth password命令提供密码，默认关闭；

命令行操作

```redis
-- 获取密码
config get requirepass

-- 设置密码
config set requirepass ""
```



### LIMITS（极限） ###
maxclients 10000
设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应；
maxmemory <bytes>
设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。如果redis无法根据移除规则来移除内存中的数据，或者设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素
maxmemory-policy noeviction
数据淘汰策略，Reids 具体有 6 种淘汰策略：
（1）volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；
（2）allkeys-lru：使用LRU算法移除key；
（3）volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键；
（4）allkeys-random：移除随机的key；
（5）volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key；
（6）noeviction：不进行移除。针对写操作，只是返回错误信息；
maxmemory-samples 5
设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个；
### APPEND ONLY MODE（追加） ###
appendonly no
指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失；因为redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no；
appendfilename "appendonly.aof"
指定更新日志文件名，默认为appendonly.aof；
appendfsync everysec
always：同步持久化，每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好；
everysec：出厂默认推荐，异步操作，每秒记录 如果一秒内宕机，有数据丢失；
no：让操作系统来决定何时同步，不能给服务器性能带来多大的提升，而且也会增加系统奔溃时数据丢失的数量；
no-appendfsync-on-rewrite no
重写时是否可以运用Appendfsync，用默认no即可，保证数据安全性；
auto-aof-rewrite-percentage 100
重写指定百分比，为0会禁用AOF自动重写特性；
auto-aof-rewrite-min-size 64mb
设置重写的基准值；





# 事务操作

Redis事务是一个单独的隔离操作，事务中的所有命令都会被序列化，按顺序执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要作用：串联多个命令，防止别的命令插队。



## Multi、Exec、discard

从输入命令multi开始，输入的命令会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前命令队列中的命令依次执行。

组队过程中可以通过discard放弃组队。



- 组队过程某个命令出错，整个事务都不会执行

- 执行阶段某个命令出错，只有错误的命令不会执行



## 事务锁

> 悲观锁

每次对数据进行操作的时候，都会该数据进行上锁，直到解锁之后，别人才能对数据进行操作



> 乐观锁

数据上增加版本号，每次修改数据时，检查数据的版本号是否跟取出数据版本号一致。

- 若一致，说明数据没被改动过，则修改成功
- 若不一致，说明数据已经被改动，则修改失败

乐观锁适用于多读的应用类型，可以提高吞吐量，Redis就是使用这种check-and-set机制实现事务的



## Watch key

在执行multi之前，先执行watch key1 [key2]操作，可以监视一个或多个key，如果在事务执行（exec）之前key被其他命令锁改动，那么事务将被打断

客户端执行过程

```redis
-- 监视版本号
watch version

-- 开启事务
multi
incr version
...

-- 在执行前，会检查版本号是否一致
exec
```



事务三特性

- 单独的隔离操作
- 没有隔离级别的概念
- 不保证原子性



# Redis持久化

## RDB（Redis DataBase）

> ### save持久化

① 在/usr/local/bin中存在dump.rdb文件

② 修改rdb文件：打开/opt/redis/redis.conf配置文件

```
-- 若60秒内进行了5次及以上操作，则写入rdb文件中进行持久化保存
save 60 5  
```

③ 恢复rdb文件
将备份的rdb文件放在我们的redis启动目录

Redis启动的时候会自动检查dump.rdb文件并恢复其中的数据！

③ 优缺点：
优点：

- 适合大规模的数据恢复
- 节省磁盘空间
- 恢复速度快

- 1、适合大规模的数据恢复！
- 2、对数据的完整性要求不高！

缺点：
1、若redis意外宕机了，最后一次修改的数据就没有了（对数据一致性要求不高使用）
2、fork进程的时候，会占用一定的内容空间！

④ 总结：
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。
这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。我们默认的就是RDB，一般情况下不需要修改这个配置！

在生产环境我们会将这个文件进行备份！



## AOF（Append Only File）

### 概念

AOF以日志的形式来记录每个写操作，将Redis执行的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件

Redis重启的话就会根据日志文件内容将指令从前到后执行依次，从而完成数据恢复工作



### 启动

Redis默认使用的是RDB模式，需要手动开启AOF模式

进入redis.conf文件，进行一下操作

```
-- 默认AOF不开启，将no改为yes开启
appendonly no
```

重启redis时，在启动目录下会生成appendonly文件夹

若RDB和AOF同时开启，Redis默认恢复AOP的数据



异常修复
/usr/local/bin下通过redis-check-aof文件来进行修复

```redis
redis-check-aof --fix appendonly.aof
```



### AOF配置

> 同步频率设置

```
-- 同步写入（只要有写操作就记录），性能较差但数据完整
appendfsync always 

-- 每秒执行一次 sync，可能会丢失这1s的数据
appendfsync everysec 

-- Redis不主动进行同步，而是交给操作系统进行同步
appendfsync no 
```

> Rewrite压缩

AOF采用文件追加方式，文件会越来越大，为了避免这种情况，Redis新增重写机制：当AOF文件超过设定阈值时，Redis会启动AOF文件压缩，只保留可以恢复数据的最小指令集

```
auto-aof-rewrite-percentage 100  #写入百分比
auto-aof-rewrite-min-size 64mb  #达到64MB开始重写
```

缺点：
1、相对于数据文件来说，aof远远大于 rdb，修复的速度也比 rdb慢！
2、Aof 运行效率也要比 rdb 慢，所以我们redis默认的配置就是rdb持久化！











