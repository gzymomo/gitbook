- [Redis命令总结](https://blog.51cto.com/u_15157097/2922474)



# 一、redis启动：

- **本地启动：redis-cli**
- **远程启动：redis-cli -h host -p port -a password**

**Redis 连接命令**

- AUTH password 验证密码是否正确
- ECHO message 打印字符串
- PING 查看服务是否运行
- QUIT 关闭当前连接
- SELECT index 切换到指定的数据库

# 二、redis keys命令

**1、DEL key**

DUMP key 序列化给定的key并返回序列化的值

**2、EXISTS key**

检查给定的key是否存在

**3、EXPIRE key seconds**

为key设置过期时间

**4、EXPIRE key timestamp**

用时间戳的方式给key设置过期时间

**5、PEXPIRE key milliseconds**

设置key的过期时间以毫秒计

**6、KEYS pattern**

查找所有符合给定模式的key

**7、MOVE key db**

将当前数据库的key移动到数据库db当中

**8、PERSIST key**

移除key的过期时间，key将持久保存

**9、PTTL key**

以毫秒为单位返回key的剩余过期时间

**10、TTL key**

以秒为单位，返回给定key的剩余生存时间

**11、RANDOMKEY**

从当前数据库中随机返回一个key

**12、RENAME key newkey**

修改key的名称

**13、RENAMENX key newkey**

仅当newkey不存在时，将key改名为newkey

**14、TYPE key**

返回key所存储的值的类型

# 三、reids字符串命令

**1、SET key value**

**2、GET key**

**3、GETRANGE key start end**

返回key中字符串值的子字符

**4、GETSET key value**

将给定key的值设为value，并返回key的旧值

**5、GETBIT KEY OFFSET**

对key所储存的字符串值，获取指定偏移量上的位

**6、MGET KEY1 KEY2**

获取一个或者多个给定key的值

**7、SETBIT KEY OFFSET VALUE**

对key所是存储的字符串值，设置或清除指定偏移量上的位

**8、SETEX key seconds value**

将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。

**9、SETNX key value**

只有在 key 不存在时设置 key 的值。

**10、SETRANGE key offset value**

用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。

**11、STRLEN key**

返回 key 所储存的字符串值的长度。

**12、MSET key value [key value …]**

同时设置一个或多个 key-value 对。

**13、MSETNX key value [key value …]**

同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

**14、PSETEX key milliseconds value**

这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。

**15、INCR key**

将 key 中储存的数字值增一。

**16、INCRBY key increment**

将 key 所储存的值加上给定的增量值（increment） 。

**17、INCRBYFLOAT key increment**

将 key 所储存的值加上给定的浮点增量值（increment） 。

**18、DECR key**

将 key 中储存的数字值减一。

**19、DECRBY key decrement**

key 所储存的值减去给定的减量值（decrement） 。

**20、APPEND key value**

如果 key 已经存在并且是一个字符串， APPEND 命令将 指定value 追加到改 key 原来的值（value）的末尾。

# 四、Redis hash 命令

**1、HDEL key field1 [field2]**

删除一个或多个哈希表字段

**2、HEXISTS key field**

查看哈希表 key 中，指定的字段是否存在。

**3、HGET key field**

获取存储在哈希表中指定字段的值。

**4、HGETALL key**

获取在哈希表中指定 key 的所有字段和值

**5、HINCRBY key field increment**

为哈希表 key 中的指定字段的整数值加上增量 increment 。

**6、HINCRBYFLOAT key field increment**

为哈希表 key 中的指定字段的浮点数值加上增量 increment 。

**7、HKEYS key**

获取所有哈希表中的字段

**8、HLEN key**

获取哈希表中字段的数量

**9、HMGET key field1 [field2]**

获取所有给定字段的值

**10、HMSET key field1 value1 [field2 value2 ]**

同时将多个 field-value (域-值)对设置到哈希表 key 中。

**11、HSET key field value**

将哈希表 key 中的字段 field 的值设为 value 。

**12、HSETNX key field value**

只有在字段 field 不存在时，设置哈希表字段的值。

**13、HVALS key**

获取哈希表中所有值

**14、HSCAN key cursor [MATCH pattern] [COUNT count]**

迭代哈希表中的键值对。

# 五、Redis 列表命令

**1、BLPOP key1 [key2 ] timeout**

移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

**2、BRPOP key1 [key2 ] timeout**

移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

**3、BRPOPLPUSH source destination timeout**

从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。

**4、LINDEX key index**

通过索引获取列表中的元素

**5、LINSERT key BEFORE|AFTER pivot value**

在列表的元素前或者后插入元素

**6、LLEN key**

获取列表长度

**7、LPOP key**

移出并获取列表的第一个元素

**8、LPUSH key value1 [value2]**

将一个或多个值插入到列表头部

**9、LPUSHX key value**

将一个值插入到已存在的列表头部

**10、LRANGE key start stop**

获取列表指定范围内的元素

**11、LREM key count value**

移除列表元素

**12、LSET key index value**

通过索引设置列表元素的值

**13、LTRIM key start stop**

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

**14、RPOP key**

移除并获取列表最后一个元素

**15、RPOPLPUSH source destination**

移除列表的最后一个元素，并将该元素添加到另一个列表并返回

**16、RPUSH key value1 [value2]**

在列表中添加一个或多个值

**17、RPUSHX key value**

为已存在的列表添加值

# 六、Redis 集合命令

**1、SADD key member1 [member2]**

向集合添加一个或多个成员

**2、SCARD key**

获取集合的成员数

**3、SDIFF key1 [key2]**

返回给定所有集合的差集

**4、SDIFFSTORE destination key1 [key2]**

返回给定所有集合的差集并存储在 destination 中

**5 、SINTER key1 [key2]**

返回给定所有集合的交集

**6、SINTERSTORE destination key1 [key2]**

返回给定所有集合的交集并存储在 destination 中

**7、SISMEMBER key member**

判断 member 元素是否是集合 key 的成员

**8、SMEMBERS key**

返回集合中的所有成员

**9、SMOVE source destination member**

将 member 元素从 source 集合移动到 destination 集合

**10、SPOP key**

移除并返回集合中的一个随机元素

**11、SRANDMEMBER key [count]**

返回集合中一个或多个随机数

**12、SREM key member1 [member2]**

移除集合中一个或多个成员

**13、SUNION key1 [key2]**

返回所有给定集合的并集

1**4、SUNIONSTORE destination key1 [key2]**

所有给定集合的并集存储在 destination 集合中

**15、SSCAN key cursor [MATCH pattern] [COUNT count]**

迭代集合中的元素

# 七、Redis 有序集合命令

**1、ZADD key score1 member1 [score2 member2]**

向有序集合添加一个或多个成员，或者更新已存在成员的分数

**2、ZCARD key**

获取有序集合的成员数

**3、ZCOUNT key min max**

计算在有序集合中指定区间分数的成员数

**4、ZINCRBY key increment member**

有序集合中对指定成员的分数加上增量 increment

5、ZINTERSTORE destination numkeys key [key …]

**计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中**

**6、ZLEXCOUNT key min max**

在有序集合中计算指定字典区间内成员数量

**7、ZRANGE key start stop [WITHSCORES]**

通过索引区间返回有序集合成指定区间内的成员

**8、ZRANGEBYLEX key min max [LIMIT offset count]**

通过字典区间返回有序集合的成员

**9、ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]**

通过分数返回有序集合指定区间内的成员

**10、ZRANK key member**

返回有序集合中指定成员的索引

**11、ZREM key member [member …]**

移除有序集合中的一个或多个成员

**12、ZREMRANGEBYLEX key min max**

移除有序集合中给定的字典区间的所有成员

**13、ZREMRANGEBYRANK key start stop**

移除有序集合中给定的排名区间的所有成员

**14、ZREMRANGEBYSCORE key min max**

移除有序集合中给定的分数区间的所有成员

**15、ZREVRANGE key start stop [WITHSCORES]**

返回有序集中指定区间内的成员，通过索引，分数从高到底

**16、ZREVRANGEBYSCORE key max min [WITHSCORES]**

返回有序集中指定分数区间内的成员，分数从高到低排序

**17、ZREVRANK key member**

返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序

**18、ZSCORE key member**

返回有序集中，成员的分数值

**19、ZUNIONSTORE destination numkeys key [key …]**

计算给定的一个或多个有序集的并集，并存储在新的 key 中

**20、ZSCAN key cursor [MATCH pattern] [COUNT count]**

迭代有序集合中的元素（包括元素成员和元素分值）

# 八、Redis 发布订阅命令

**1、PSUBSCRIBE pattern [pattern …]**

订阅一个或多个符合给定模式的频道。

**2、PUBSUB subcommand [argument [argument …]**

查看订阅与发布系统状态。

**3、PUBLISH channel message**

将信息发送到指定的频道。

**4、PUNSUBSCRIBE [pattern [pattern …]]**

退订所有给定模式的频道。

**5、SUBSCRIBE channel [channel …]**

订阅给定的一个或多个频道的信息。

**6、UNSUBSCRIBE [channel [channel …]]**

指退订给定的频道。

**示例：**

```html
redis 127.0.0.1:6379> SUBSCRIBE redisChat
Reading messages... (press Ctrl-C to quit)
"subscribe"
"redisChat"
(integer) 1
1.2.3.4.5.
```

现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。 redis  127.0.0.1:6379> PUBLISH redisChat “Redis is a great caching  technique”

(integer) 1

```html
订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"

1.2.3.4.5.
```

# 九、Redis 事务命令

**1、DISCARD**

取消事务，放弃执行事务块内的所有命令。

**2、EXEC**

执行所有事务块内的命令。

**3、MULTI**

标记一个事务块的开始。

**4、UNWATCH**

取消 WATCH 命令对所有 key 的监视。

**5、WATCH key [key …]**

监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

# 十、Redis 脚本命令

**1、EVAL script numkeys key [key …] arg [arg …]**

执行 Lua 脚本。

**2、EVALSHA sha1 numkeys key [key …] arg [arg …**.]

执行 Lua 脚本。

**3、SCRIPT EXISTS script [script …]**

查看指定的脚本是否已经被保存在缓存当中。

**4、SCRIPT FLUSH**

从脚本缓存中移除所有脚本。

**5、SCRIPT KILL**

杀死当前正在运行的 Lua 脚本。

**6、SCRIPT LOAD script**

将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。

# 十一、Redis 服务器命令

**1、BGREWRITEAOF**

异步执行一个 AOF（AppendOnly File） 文件重写操作

**2、BGSAVE**

在后台异步保存当前数据库的数据到磁盘

**3、CLIENT KILL [ip:port] [ID client-id]**

关闭客户端连接

**4、CLIENT LIST**

获取连接到服务器的客户端连接列表

**5、CLIENT GETNAME**

获取连接的名称

**6、CLIENT PAUSE timeout**

在指定时间内终止运行来自客户端的命令

**7、CLIENT SETNAME connection-name**

设置当前连接的名称

**8、CLUSTER SLOTS**

获取集群节点的映射数组

**9、COMMAND**

获取 Redis 命令详情数组

**10、COMMAND COUNT**

获取 Redis 命令总数

**11、COMMAND GETKEYS**

获取给定命令的所有键

**12、TIME 返**

回当前服务器时间

**13、COMMAND INFO command-name [command-name …]**

获取指定 Redis 命令描述的数组

**14、CONFIG GET parameter**

获取指定配置参数的值

15、CONFIG REWRITE

对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写

**16、CONFIG SET**

arameter value 修改 redis 配置参数，无需重启

**17、CONFIG RESETSTAT**

重置 INFO 命令中的某些统计数据

**18、DBSIZE**

返回当前数据库的 key 的数量

**19、DEBUG OBJECT key**

获取 key 的调试信息

**20、DEBUG SEGFAULT**

让 Redis 服务崩溃

**21、、FLUSHALL**

删除所有数据库的所有key

**22、FLUSHDB**

删除当前数据库的所有key

**23 、INFO [section]**

获取 Redis 服务器的各种信息和统计数值

**24、LASTSAVE**

返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示

**25、MONITOR**

实时打印出 Redis 服务器接收到的命令，调试用

**26、ROLE**

返回主从实例所属的角色

**27、SAVE**

同步保存数据到硬盘

**28、SHUTDOWN [NOSAVE] [SAVE]**

异步保存数据到硬盘，并关闭服务器

**29、SLAVEOF host port**

将当前服务器转变为指定服务器的从属服务器(slave server)

**30、SLOWLOG subcommand [argument]**

管理 redis 的慢日志

**31、SYNC**

用于复制功能(replication)的内部命令