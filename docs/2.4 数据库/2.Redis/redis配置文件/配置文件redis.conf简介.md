# 配置文件redis.conf简介

原文地址：https://blog.csdn.net/guorui_java/article/details/116767168



## 一、简介

我们可以在启动redis-server时指定应该加载的配置文件，方法如下：

```Lua
$ ./redis-server /path/to/redis.conf
```

接下来，我们就来讲解下redis配置文件的各个配置项的含义，注意，本文是基于redis-2.8.4版本进行讲解的。

redis官方提供的redis.conf文件，足有700+行，其中100多行为有效配置行，另外的600多行为注释说明。

在配置文件的开头部分，首先明确了一些度量单位：

```java
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
```

可以看出，redis配置中对单位的大小写不敏感，1GB、1Gb和1gB都是相同的。由此也说明，redis只支持bytes，不支持bit单位。

redis支持“主配置文件中引入外部配置文件”，很像C/C++中的include指令，比如：

```Lua
include /path/to/other.conf
```

如果你看过redis的配置文件，会发现还是很有条理的。redis配置文件被分成了几大块区域，它们分别是：

1.通用（general）
 2.快照（snapshotting）
 3.复制（replication）
 4.安全（security）
 5.限制（limits)
 6.追加模式（append only mode)
 7.LUA脚本（lua scripting)
 8.慢日志（slow log)
 9.事件通知（event notification）

下面我们就来逐一讲解。

## 二、通用

默认情况下，redis并不是以daemon形式来运行的。通过daemonize配置项可以控制redis的运行形式，如果改为yes，那么redis就会以daemon形式运行：

```Lua
daemonize no
```

当以daemon形式运行时，redis会生成一个pid文件，默认会生成在/var/run/redis.pid。当然，你可以通过pidfile来指定pid文件生成的位置，比如：

```Lua
pidfile /path/to/redis.pid
```

默认情况下，redis会响应本机所有可用网卡的连接请求。当然，redis允许你通过bind配置项来指定要绑定的IP，比如：

```Lua
bind 192.168.1.2 10.8.4.2
```

redis的默认服务端口是6379，你可以通过port配置项来修改。如果端口设置为0的话，redis便不会监听端口了。

```Lua
port 6379
```

有些同学会问“如果redis不监听端口，还怎么与外界通信呢”，其实redis还支持通过unix socket方式来接收请求。可以通过unixsocket配置项来指定unix socket文件的路径，并通过unixsocketperm来指定文件的权限。

```Lua
unixsocket /tmp/redis.sock
unixsocketperm 755
```

当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置“空闲超时时限”，0表示永不关闭。

```Lua
timeout 0
```

TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测。

```
tcp-keepalive 0
```

redis支持通过loglevel配置项设置日志等级，共分四级，即debug、verbose、notice、warning。

```
loglevel notice
```

redis也支持通过logfile配置项来设置日志文件的生成位置。如果设置为空字符串，则redis会将日志输出到标准输出。假如你在daemon情况下将日志设置为输出到标准输出，则日志会被写到/dev/null中。

```
logfile ""
```

如果希望日志打印到syslog中，也很容易，通过syslog-enabled来控制。另外，syslog-ident还可以让你指定syslog里的日志标志，比如：

```
syslog-ident redis
```

而且还支持指定syslog设备，值可以是USER或LOCAL0-LOCAL7。具体可以参考syslog服务本身的用法。

```
syslog-facility local0
```

对于redis来说，可以设置其数据库的总数量，假如你希望一个redis包含16个数据库，那么设置如下：

```
databases 16
```

这16个数据库的编号将是0到15。默认的数据库是编号为0的数据库。用户可以使用select <DBid>来选择相应的数据库。

## 三、快照

快照，主要涉及的是redis的RDB持久化相关的配置，我们来一起看一看。

我们可以用如下的指令来让数据保存到磁盘上，即控制RDB快照功能：

```
save <seconds> <changes>
```

举例来说：

```java
save 900 1 //表示每15分钟且至少有1个key改变，就触发一次持久化
save 300 10 //表示每5分钟且至少有10个key改变，就触发一次持久化
save 60 10000 //表示每60秒至少有10000个key改变，就触发一次持久化
```

如果你想禁用RDB持久化的策略，只要不设置任何save指令就可以，或者给save传入一个空字符串参数也可以达到相同效果，就像这样：

```
save ""
```

如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求。这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。如果redis不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。


 如果下一次RDB持久化成功，redis会自动恢复接受写请求。

当然，如果你不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，你完全可以关闭这个功能，以便在快照写入失败时，也能确保redis继续接受新的写请求。配置项如下：

```
stop-writes-on-bgsave-error yes
```

对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。

```
rdbcompression yes
```

在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果你希望获取到最大的性能提升，可以关闭此功能。

```
rdbchecksum yes
```

我们还可以设置快照文件的名称，默认是这样配置的：

```
dbfilename dump.rdb
```

最后，你还可以设置这个快照文件存放的路径。比如默认设置就是当前文件夹：

```
dir ./
```

## 四、复制

redis提供了主从同步功能。

通过slaveof配置项可以控制某一个redis作为另一个redis的从服务器，通过指定IP和端口来定位到主redis的位置。一般情况下，我们会建议用户为从redis设置一个不同频率的快照持久化的周期，或者为从redis配置一个不同的服务端口等等。

```
slaveof <masterip> <masterport>
```

如果主redis设置了验证密码的话（使用requirepass来设置），则在从redis的配置中要使用masterauth来设置校验密码，否则的话，主redis会拒绝从redis的访问请求。

```
masterauth <master-password>
```

当从redis失去了与主redis的连接，或者主从同步正在进行中时，redis该如何处理外部发来的访问请求呢？这里，从redis可以有两种选择：


 第一种选择：如果slave-serve-stale-data设置为yes（默认），则从redis仍会继续响应客户端的读写请求。

第二种选择：如果slave-serve-stale-data设置为no，则从redis会对客户端的请求返回“SYNC with master in progress”，当然也有例外，当客户端发来INFO请求和SLAVEOF请求，从redis还是会进行处理。

你可以控制一个从redis是否可以接受写请求。将数据直接写入从redis，一般只适用于那些生命周期非常短的数据，因为在主从同步时，这些临时数据就会被清理掉。自从redis2.6版本之后，默认从redis为只读。

```
slave-read-only yes
```

只读的从redis并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用rename-command指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：

```
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
```

从redis会周期性的向redis发出PING包。你可以通过repl_ping_slave_period指令来控制其周期。默认是10秒。

```
repl-ping-slave-period 10
```

在主从同步时，可能在这些情况下会有超时发生：

1.以从redis的角度来看，当有大规模IO传输时。
 2.以从redis的角度来看，当数据传输或PING时，主redis超时
 3.以主redis的角度来看，在回复从redis的PING时，从redis超时

```
repl-timeout 60
```

我们可以控制在主从同步时是否禁用TCP_NODELAY。如果开启TCP_NODELAY，那么主redis会使用更少的TCP包和更少的带宽来向从redis传输数据。但是这可能会增加一些同步的延迟，大概会达到40毫秒左右。如果你关闭了TCP_NODELAY，那么数据同步的延迟时间会降低，但是会消耗更多的带宽。（如果你不了解TCP_NODELAY，可以到这里来科普一下）。

```
repl-disable-tcp-nodelay no
```

我们还可以设置同步队列长度。队列长度（backlog)是主redis中的一个缓冲区，在与从redis断开连接期间，主redis会用这个缓冲区来缓存应该发给从redis的数据。这样的话，当从redis重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可。

```
repl-backlog-size 1mb
```

如果主redis等了一段时间之后，还是无法连接到从redis，那么缓冲队列中的数据将被清理掉。我们可以设置主redis要等待的时间长度。如果设置为0，则表示永远不清理。默认是1个小时。

```
repl-backlog-ttl 3600
```

我们可以给众多的从redis设置优先级，在主redis持续工作不正常的情况，优先级高的从redis将会升级为主redis。而编号越小，优先级越高。比如一个主redis有三个从redis，优先级编号分别为10、100、25，那么编号为10的从redis将会被首先选中升级为主redis。当优先级被设置为0时，这个从redis将永远也不会被选中。默认的优先级为100。

```
slave-priority 100
```

假如主redis发现有超过M个从redis的连接延时大于N秒，那么主redis就停止接受外来的写请求。这是因为从redis一般会每秒钟都向主redis发出PING，而主redis会记录每一个从redis最近一次发来PING的时间点，所以主redis能够了解每一个从redis的运行情况。

1. `min-slaves-to-write 3`
2. `min-slaves-max-lag 10`

上面这个例子表示，假如有大于等于3个从redis的连接延迟大于10秒，那么主redis就不再接受外部的写请求。上述两个配置中有一个被置为0，则这个特性将被关闭。默认情况下min-slaves-to-write为0，而min-slaves-max-lag为10。

## 五、安全

我们可以要求redis客户端在向redis-server发送请求之前，先进行密码验证。当你的redis-server处于一个不太可信的网络环境中时，相信你会用上这个功能。由于redis性能非常高，所以每秒钟可以完成多达15万次的密码尝试，所以你最好设置一个足够复杂的密码，否则很容易被黑客破解。

```
requirepass zhimakaimen
```

 这里我们通过requirepass将密码设置成“芝麻开门”。

redis允许我们对redis指令进行更名，比如将一些比较危险的命令改个名字，避免被误执行。比如可以把CONFIG命令改成一个很复杂的名字，这样可以避免外部的调用，同时还可以满足内部调用的需要：

```
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c89
```

我们甚至可以禁用掉CONFIG命令，那就是把CONFIG的名字改成一个空字符串：

```
rename-command CONFIG ""
```

但需要注意的是，如果你使用AOF方式进行数据持久化，或者需要与从redis进行通信，那么更改指令的名字可能会引起一些问题。

## 六、限制

我们可以设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。

如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。

```
maxclients 10000
```

我们甚至可以设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。

如果redis无法根据移除规则来移除内存中的数据，或者我们设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。

```
maxmemory <bytes>
```

需要注意的一点是，如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

对于内存移除规则来说，redis提供了多达6种的移除规则。他们是：

1.volatile-lru：使用LRU算法移除过期集合中的key
 2.allkeys-lru：使用LRU算法移除key
 3.volatile-random：在过期集合中移除随机的key
 4.allkeys-random：移除随机的key
 5.volatile-ttl：移除那些TTL值最小的key，即那些最近才过期的key。
 6.noeviction：不进行移除。针对写操作，只是返回错误信息。

无论使用上述哪一种移除规则，如果没有合适的key可以移除的话，redis都会针对写请求返回错误信息。

```
maxmemory-policy volatile-lru
```

LRU算法和最小TTL算法都并非是精确的算法，而是估算值。所以你可以设置样本的大小。假如redis默认会检查三个key并选择其中LRU的那个，那么你可以改变这个key样本的数量。

```
maxmemory-samples 3
```

最后，我们补充一个信息，那就是到目前版本（2.8.4）为止，redis支持的写指令包括了如下这些：

```java
set setnx setex append
incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
getset mset msetnx exec sort
```

## 七、追加模式

默认情况下，redis会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的，但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。

如博文上半部分中介绍的，追加文件（Append Only File）是一种更好的保持数据一致性的方式。即使当服务器断电时，也仅会有1秒钟的写请求丢失，当redis进程出现问题且操作系统运行正常时，甚至只会丢失一条写请求。

我们建议大家，AOF机制和RDB机制可以同时使用，不会有任何冲突。对于如何保持数据一致性的讨论，请参见[本文](https://redis.io/topics/keyspace-events)。

```
appendonly no
```

我们还可以设置aof文件的名称：

```
appendfilename "appendonly.aof"
```

fsync()调用，用来告诉操作系统立即将缓存的指令写入磁盘。一些操作系统会“立即”进行，而另外一些操作系统则会“尽快”进行。

redis支持三种不同的模式：

1.no：不调用fsync()。而是让操作系统自行决定sync的时间。这种模式下，redis的性能会最快。
 2.always：在每次写请求后都调用fsync()。这种模式下，redis会相对较慢，但数据最安全。
 3.everysec：每秒钟调用一次fsync()。这是性能和安全的折衷。

默认情况下为everysec。有关数据一致性的揭秘。

```
appendfsync everysec
```

当fsync方式设置为always或everysec时，如果后台持久化进程需要执行一个很大的磁盘IO操作，那么redis可能会在fsync()调用时卡住。目前尚未修复这个问题，这是因为即使我们在另一个新的线程中去执行fsync()，也会阻塞住同步写调用。


  为了缓解这个问题，我们可以使用下面的配置项，这样的话，当BGSAVE或BGWRITEAOF运行时，fsync()在主进程中的调用会被阻止。这意味着当另一路进程正在对AOF文件进行重构时，redis的持久化功能就失效了，就好像我们设置了“appendsync none”一样。如果你的redis有时延问题，那么请将下面的选项设置为yes。否则请保持no，因为这是保证数据完整性的最安全的选择。

```
no-appendfsync-on-rewrite no
```

我们允许redis自动重写aof。当aof增长到一定规模时，redis会隐式调用BGREWRITEAOF来重写log文件，以缩减文件体积。


  redis是这样工作的：redis会记录上次重写时的aof大小。假如redis自启动至今还没有进行过重写，那么启动时aof文件的大小会被作为基准值。这个基准值会和当前的aof大小进行比较。如果当前aof大小超出所设置的增长比例，则会触发重写。另外，你还需要设置一个最小大小，是为了防止在aof很小时就触发重写。

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

如果设置auto-aof-rewrite-percentage为0，则会关闭此重写功能。

## 八、LUA脚本

lua脚本的最大运行时间是需要被严格限制的，要注意单位是毫秒：

```
lua-time-limit 5000
```

如果此值设置为0或负数，则既不会有报错也不会有时间限制。

## 九、慢日志

redis慢日志是指一个系统进行日志查询超过了指定的时长。这个时长不包括IO操作，比如与客户端的交互、发送响应内容等，而仅包括实际执行查询命令的时间。

针对慢日志，你可以设置两个参数，一个是执行时长，单位是微秒，另一个是慢日志的长度。当一个新的命令被写入日志时，最老的一条会从命令日志队列中被移除。

单位是微秒，即1000000表示一秒。负数则会禁用慢日志功能，而0则表示强制记录每一个命令。

```
slowlog-log-slower-than 10000
```

慢日志最大长度，可以随便填写数值，没有上限，但要注意它会消耗内存。你可以使用SLOWLOG RESET来重设这个值。

```
slowlog-max-len 128
```

## 十、事件通知

redis可以向客户端通知某些事件的发生。

## 十一、高级配置

有关哈希数据结构的一些配置项：

1. `hash-max-ziplist-entries 512`
2. `hash-max-ziplist-value 64`

有关列表数据结构的一些配置项：

1. `list-max-ziplist-entries 512`
2. `list-max-ziplist-value 64`

有关集合数据结构的配置项：

```
set-max-intset-entries 512
```

有关有序集合数据结构的配置项：

1. `zset-max-ziplist-entries 128`
2. `zset-max-ziplist-value 64`

关于是否需要再哈希的配置项：

```
activerehashing yes
```

关于客户端输出缓冲的控制项：

```
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

有关频率的配置项：

```
hz 10
```

有关重写aof的配置项：

```
aof-rewrite-incremental-fsync yes
```