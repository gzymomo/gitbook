[TOC]

Redis 利用了多路 I/O 复用机制，处理客户端请求时，不会阻塞主线程；Redis 单纯执行（大多数指令）一个指令不到 1 微秒，如此，单核 CPU 一秒就能处理 1 百万个指令（大概对应着几十万个请求吧），用不着实现多线程（网络才是瓶颈）。

# 1、优化网络延时
Redis 客户端和服务器的通讯一般使用 TCP 长链接。如果客户端发送请求后需要等待 Redis 返回结果再发送下一个指令，客户端和 Redis 的多个请求就构成下面的关系：
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKv2BamWIONjTIZVsmxiay3aFIP3y7s7t7YsgsopgkUnZ48W83g54IibwJklUQ8qwUXibbq6OZcEYO6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（备注：如果不是你要发送的 key 特别长，一个 TCP 包完全能放下 Redis 指令，所以只画了一个 push 包）

这样这两次请求中，客户端都需要经历一段网络传输时间。

但如果有可能，完全可以使用 multi-key 类的指令来合并请求，比如两个 GET key 可以用 MGET key1 key2 合并。这样在实际通讯中，请求数也减少了，延时自然得到好转。

如果不能用 multi-key 指令来合并，比如一个 SET，一个 GET 无法合并。怎么办？

Redis 中有至少这样两个方法能合并多个指令到一个 request 中，一个是 MULTI/EXEC，一个是 script。前者本来是构建 Redis 事务的方法，但确实可以合并多个指令为一个 request，它到通讯过程如下。至于 script，最好利用缓存脚本的 sha1 hash key 来调起脚本，这样通讯量更小。
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKv2BamWIONjTIZVsmxiay3akhzYOzuqGclvcE0mHujUBzjEnhfJbxqWibiaIZotia6Dxfuuia0MlndQzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
这样确实更能减少网络传输时间，不是么？但如此以来，就必须要求这个 transaction / script 中涉及的 key 在同一个 node 上，所以要酌情考虑。

如果上面的方法我们都考虑过了，还是没有办法合并多个请求，我们还可以考虑合并多个 responses。比如把 2 个回复信息合并：
![](https://mmbiz.qpic.cn/sz_mmbiz_png/HV4yTI6PjbKv2BamWIONjTIZVsmxiay3aDsYGw8dRyBNmQcXic2Dm2P5XrfvPdmbDaPpokO5YIta104zQsGdkjHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
这样，理论上可以省去 1 次回复所用的网络传输时间。这就是 pipeline 做的事情。
不是任意多个回复信息都可以放进一个 TCP 包中，如果请求数太多，回复的数据很长（比如 get 一个长字符串），TCP 还是会分包传输，但使用 pipeline，依然可以减少传输次数。

pipeline 和上面的其他方法都不一样的是，它不具有原子性。所以在 cluster 状态下的集群上，实现 pipeline 比那些原子性的方法更有可能。

小结一下：
 1. 使用 unix 进程间通信，如果单机部署
 2. 使用 multi-key 指令合并多个指令，减少请求数，如果有可能的话
 3. 使用 transaction、script 合并 requests 以及 responses
 4. 使用 pipeline 合并 response

# 2、警惕执行时间长的操作
在大数据量的情况下，有些操作的执行时间会相对长，比如 KEYS *，LRANGE mylist 0 -1，以及其他算法复杂度为 O(n) 的指令。因为 Redis 只用一个线程来做数据查询，如果这些指令耗时很长，就会阻塞 Redis，造成大量延时。

尽管官方文档中说 KEYS * 的查询挺快的，（在普通笔记本上）扫描 1 百万个 key，只需 40 毫秒(参见：https://redis.io/commands/keys)，但几十 ms 对于一个性能要求很高的系统来说，已经不短了，更何况如果有几亿个 key（一台机器完全可能存几亿个 key，比如一个 key 100字节，1 亿个 key 只有 10GB），时间更长。
Redis 中 transaction，script，因为可以合并多个 commands 为一个具有原子性的执行过程，所以也可能占用 Redis 很长时间，需要注意。

如果你想找出生产环境使用的「慢指令」，那么可以利用 SLOWLOG GET count 来查看最近的 count 个执行时间很长的指令。至于多长算长，可以通过在 redis.conf 中设置 slowlog-log-slower-than 来定义。

除此之外，在很多地方都没有提到的一个可能的慢指令是 DEL，但 redis.conf 文件的注释[9]中倒是说了。长话短说就是 DEL 一个大的 object 时候，回收相应的内存可能会需要很长时间（甚至几秒），所以，建议用 DEL 的异步版本：UNLINK。后者会启动一个新的 thread 来删除目标 key，而不阻塞原来的线程。

更进一步，当一个 key 过期之后，Redis 一般也需要同步的把它删除。其中一种删除 keys 的方式是，每秒 10 次的检查一次有设置过期时间的 keys，这些 keys 存储在一个全局的 struct 中，可以用 server.db->expires 访问。检查的方式是：
 1. 从中随机取出 20 个 keys
 2. 把过期的删掉。
 3. 如果刚刚 20 个 keys 中，有 25% 以上（也就是 5 个以上）都是过期的，Redis 认为，过期的 keys 还挺多的，继续重复步骤 1，直到满足退出条件：某次取出的 keys 中没有那么多过去的 keys。

这里对于性能的影响是，如果真的有很多的 keys 在同一时间过期，那么 Redis 真的会一直循环执行删除，占用主线程。

对此，Redis 作者的建议是警惕 EXPIREAT 这个指令，因为它更容易产生 keys 同时过期的现象。我还见到过一些建议是给 keys 的过期时间设置一个随机波动量。最后，redis.conf 中也给出了一个方法，把 keys 的过期删除操作变为异步的，即，在 redis.conf 中设置 lazyfree-lazy-expire yes。

# 3、优化数据结构、使用正确的算法
一种数据类型（比如 string，list）进行增删改查的效率是由其底层的存储结构决定的。
除了时间性能上的考虑，有时候我们还需要节省存储空间。比如上面提到的 ziplist 结构，就比 hashtable 结构节省存储空间（Redis Essentials 的作者分别在 hashtable 和 ziplist 结构的 Hash 中插入 500 个 fields，每个 field 和 value 都是一个 15 位左右的字符串，结果是 hashtable 结构使用的空间是 ziplist 的 4 倍。）。但节省空间的数据结构，其算法的复杂度可能很高。所以，这里就需要在具体问题面前做出权衡。

在使用一种数据类型时，可以适当关注一下它底层的存储结构及其算法，避免使用复杂度太高的方法。举两个例子：
 1. ZADD 的时间复杂度是 O(log(N))，这比其他数据类型增加一个新元素的操作更复杂，所以要小心使用。
 2. 若 Hash 类型的值的 fields 数量有限，它很有可能采用 ziplist 这种结构做存储，而 ziplist 的查询效率可能没有同等字段数量的 hashtable 效率高，在必要时，可以调整 Redis 的存储结构。

# 4、考虑操作系统和硬件是否影响性能
Redis 运行的外部环境，也就是操作系统和硬件显然也会影响 Redis 的性能。
 1. CPU：Intel 多种 CPU 都比 AMD 皓龙系列好
 2. 虚拟化：实体机比虚拟机好，主要是因为部分虚拟机上，硬盘不是本地硬盘，监控软件导致 fork 指令的速度慢（持久化时会用到 fork），尤其是用 Xen 来做虚拟化时。
 3. 内存管理：在 linux 操作系统中，为了让 translation lookaside buffer，即 TLB，能够管理更多内存空间（TLB 只能缓存有限个 page），操作系统把一些 memory page 变得更大，比如 2MB 或者 1GB，而不是通常的 4096 字节，这些大的内存页叫做 huge pages。同时，为了方便程序员使用这些大的内存 page，操作系统中实现了一个 transparent huge pages（THP）机制，使得大内存页对他们来说是透明的，可以像使用正常的内存 page 一样使用他们。但这种机制并不是数据库所需要的，可能是因为 THP 会把内存空间变得紧凑而连续吧，就像mongodb 的文档[11]中明确说的，数据库需要的是稀疏的内存空间，所以请禁掉 THP 功能。Redis 也不例外，但 Redis 官方博客上给出的理由是：使用大内存 page 会使 bgsave 时，fork 的速度变慢；如果 fork 之后，这些内存 page 在原进程中被修改了，他们就需要被复制（即 copy on write），这样的复制会消耗大量的内存（毕竟，人家是 huge pages，复制一份消耗成本很大）。所以，请禁止掉操作系统中的 transparent huge pages 功能。
 4. 交换空间：当一些内存 page 被存储在交换空间文件上，而 Redis 又要请求那些数据，那么操作系统会阻塞 Redis 进程，然后把想要的 page，从交换空间中拿出来，放进内存。这其中涉及整个进程的阻塞，所以可能会造成延时问题，一个解决方法是禁止使用交换空间（Redis Essentials 中如是建议，如果内存空间不足，请用别的方法处理）。

# 5、考虑持久化带来的开销
## 5.1 RDB 全量持久化。
这种持久化方式把 Redis 中的全量数据打包成 rdb 文件放在硬盘上。但是执行 RDB 持久化过程的是原进程 fork 出来一个子进程，而 fork 这个系统调用是需要时间的，根据Redis Lab 6 年前做的实验[12]，在一台新型的 AWS EC2 m1.small^13 上，fork 一个内存占用 1GB 的 Redis 进程，需要 700+ 毫秒，而这段时间，redis 是无法处理请求的。

虽然现在的机器应该都会比那个时候好，但是 fork 的开销也应该考虑吧。为此，要使用合理的 RDB 持久化的时间间隔，不要太频繁。

## 5.2 AOF 增量持久化。

这种持久化方式会把你发到 redis server 的指令以文本的形式保存下来（格式遵循 redis protocol），这个过程中，会调用两个系统调用，一个是 write(2)，同步完成，一个是 fsync(2)，异步完成。

这两部都可能是延时问题的原因：
 1. write 可能会因为输出的 buffer 满了，或者 kernal 正在把 buffer 中的数据同步到硬盘，就被阻塞了。
 2. fsync 的作用是确保 write 写入到 aof 文件的数据落到了硬盘上，在一个 7200 转/分的硬盘上可能要延时 20 毫秒左右，消耗还是挺大的。更重要的是，在 fsync 进行的时候，write 可能会被阻塞。
其中，write 的阻塞貌似只能接受，因为没有更好的方法把数据写到一个文件中了。但对于 fsync，Redis 允许三种配置，选用哪种取决于你对备份及时性和性能的平衡：
 1. always：当把 appendfsync 设置为 always，fsync 会和客户端的指令同步执行，因此最可能造成延时问题，但备份及时性最好。
 2. everysec：每秒钟异步执行一次 fsync，此时 redis 的性能表现会更好，但是 fsync 依然可能阻塞 write，算是一个折中选择。
 3. no：redis 不会主动出发 fsync （并不是永远不 fsync，那是不太可能的），而由 kernel 决定何时 fsync。

# 6、使用分布式架构 —— 读写分离、数据分片
 1. 把慢速的指令发到某些从库中执行
 2. 把持久化功能放在一个很少使用的从库上
 3. 把某些大 list 分片
其中前两条都是根据 Redis 单线程的特性，用其他进程（甚至机器）做性能补充的方法。

# 7、reids 内存分析及使用优化
redis内存使用情况：info memory
![](https://img2020.cnblogs.com/blog/603942/202005/603942-20200515222706054-324713725.png)

## 7.1 内存使用
redis的内存使用分布：自身内存，键值对象占用、缓冲区内存占用及内存碎片占用。
redis 空进程自身消耗非常的少，可以忽略不计，优化内存可以不考虑此处的因素。

### 7.1.1 对象内存
对象内存，也即真实存储的数据所占用的内存。
redis k-v结构存储，对象占用可以简单的理解为 k-size + v-size。
redis的键统一都为字符串类型，值包含多种类型：string、list、hash、set、zset五种基本类型及基于string的Bitmaps和HyperLogLog类型等。

在实际的应用中，一定要做好kv的构建形式及内存使用预期。

### 7.1.2 缓冲内存
缓冲内存包括三部分：客户端缓存、复制积压缓存及AOF缓冲区。

1）**客户端缓存**：接入redis服务器的TCP连接输入输出缓冲内存占用，TCP输入缓冲占用是不受控制的，最大允许空间为1G。输出缓冲占用可以通过client-output-buffer-limit参数配置。
redis 客户端主要分为从客户端、订阅客户端和普通客户端。

**从客户端连接占用**：也就是我们所说的slave，主节点会为每一个从节点建立一条连接用于命令复制，缓冲配置为：client-output-buffer-limit slave 256mb 64mb 60。
主从之间的间络延迟及挂载的从节点数量是影响内存占用的主要因素。因此在涉及需要异地部署主从时要特别注意，另外，也要避免主节点上挂载过多的从节点（<=2）；

**订阅客户端内存占用**：发布订阅功能连接客户端使用单独的缓冲区，默认配置：client-output-buffer-limit pubsub 32mb 8mb 60。
当消费慢于生产时会造成缓冲区积压，因此需要特别注意消费者角色配比及生产、消费速度的监控。

**普通客户端内存占用**：除了上述之外的其它客户端，如我们通常的应用连接，默认配置：client-output-buffer-limit normal 1000。
可以看到，普通客户端没有配置缓冲区限制，通常一般的客户端内存消耗也可以忽略不计。

但是当redis服务器响应较慢时，容易造成大量的慢连接，主要表现为连接数的突增，如果不能及时处理，此时会严重影响redis服务节点的服务及恢复。

关于此，在实际应用中需要注意几点：

- maxclients最大连接数配置必不可少。
- 合理预估单次操作数据量（写或读）及网络时延ttl。
- 禁止线上大吞吐量命令操作，如keys等。

高并发应用情景下，redis内存使用需要有实时的监控预警机制，

2）**复制积压缓冲区**
v2.8之后提供的一个可重用的固定大小缓冲区，用以实现向从节点的部分复制功能，避免全量复制。配置单数：repl-backlog-size，默认1M。单个主节点配置一个复制积压缓冲区。

3）**AOF缓冲区**
AOF重写期间增量的写入命令保存，此部分缓存占用大小取决于AOF重写时间及增量。

## 7.2 redis子进程内存消耗
子进程即redis执行持久化（RDB/AOF）时fork的子任务进程。

1、**关于linux系统的写时复制机制**：
父子进程会共享相同的物理内存页，父进程处理写请求时会对需要修改的页复制一份副本进行修改，子进程读取的内存则为fork时的父进程内存快照，因此，子进程的内存消耗由期间的写操作增量决定。

2、**关于linux的透明大页机制THP**（Transparent Huge Page）：
THP机制会降低fork子进程的速度；写时复制内存页由4KB增大至2M。高并发情境下，写时复制内存占用消耗影响会很大，因此需要选择性关闭。

3、**关于linux配置**：
一般需要配置linux系统 vm.overcommit_memory=1，以允许系统可以分配所有的物理内存。防止fork任务因内存而失败。

## 7.3 redis内存管理
redis的内存管理主要分为两方面：内存上限控制及内存回收管理。

### 7.3.1 内存上限：maxmemory
目的：缓存应用内存回收机制触发 + 防止物理内存用尽（redis 默认无限使用服务器内存） + 服务节点内存隔离（单服务器上部署多个redis服务节点）
在进行内存分配及限制时要充分考虑内存碎片占用影响。
动态调整，扩展redis服务节点可用内存：config set maxmemory {}。

### 7.3.2 内存回收
回收时机：键过期、内存占用达到上限

1）过期键删除：
redis 键过期时间保存在内部的过期字典中，redis采用惰性删除机制+定时任务删除机制。
惰性删除：即读时删除，读取带有超时属性的键时，如果键已过期，则删除然后返回空值。这种方式存在问题是，触发时机，加入过期键长时间未被读取，那么它将会一直存在内存中，造成内存泄漏。

定时任务删除：redis内部维护了一个定时任务（默认每秒10次，可配置），通过自适应法进行删除。
删除逻辑如下：
![](https://img2020.cnblogs.com/blog/603942/202005/603942-20200516011200807-1237380618.png)

需要说明的一点是，快慢模式执行的删除逻辑相同，这是超时时间不同。

2）内存溢出控制

当内存达到maxmemory，会触发内存回收策略，具体策略依据maxmemory-policy来执行。

- noevication：默认不回收，达到内存上限，则不再接受写操作，并返回错误。
- volatile-lru：根据LRU算法删除设置了过期时间的键，如果没有则不执行回收。
- allkeys-lru：根据LRU算法删除键，针对所有键。
- allkeys-random：随机删除键。
- volatitle-random：速记删除设置了过期时间的键。
- volatilte-ttl：根据键ttl，删除最近过期的键，同样如果没有设置过期的键，则不执行删除。

动态配置：config set maxmemory-policy {}
在设置了maxmemory情况下，每次的redis操作都会检查执行内存回收，因此对于线上环境，要确保所这只的maxmemory>used_memory。

另外，可以通过动态配置maxmemory来主动触发内存回收。