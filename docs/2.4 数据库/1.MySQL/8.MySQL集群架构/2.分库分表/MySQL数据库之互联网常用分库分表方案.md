- [MySQL性能优化：互联网公司常用分库分表方案汇总](https://ibyte.blog.csdn.net/article/details/109032553)



## 一、数据库瓶颈

不管是IO瓶颈，还是CPU瓶颈，最终都会导致数据库的活跃连接数增加，进而逼近甚至达到数据库可承载活跃连接数的阈值。在业务Service来看就是，可用数据库连接少甚至无连接可用。接下来就可以想象了吧（并发量、吞吐量、崩溃）。

### 1、IO瓶颈

第一种：磁盘读IO瓶颈，热点数据太多，数据库缓存放不下，每次查询时会产生大量的IO，降低查询速度 -> 分库和垂直分表。

第二种：网络IO瓶颈，请求的数据太多，网络带宽不够 -> 分库。

### 2、CPU瓶颈

第一种：SQL问题，如SQL中包含[join](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247488088&idx=2&sn=501652bb26f69a11f6aca0953cbfdc50&chksm=ebd62d74dca1a4624c476ceaf04327d826c916f550a5660a287dd175034d2edb9c21b4cb4b02&scene=21#wechat_redirect)，group by，order by，非索引字段条件查询等，增加CPU运算的操作 -> SQL优化，建立合适的索引，在业务Service层进行业务计算。

第二种：单表数据量太大，查询时扫描的行太多，SQL效率低，CPU率先出现瓶颈 -> 水平分表。

## 二、分库分表

### 1、水平分库

![img](https://img-blog.csdnimg.cn/img_convert/464d8d60e09c739033e774f01bfbd956.png)

**概念：**以字段为依据，按照一定策略（hash、range等），将一个库中的数据拆分到多个库中。

**结果：**

- 每个库的结构都一样；
- 每个库的数据都不一样，没有交集；
- 所有库的并集是全量数据；

**场景：**系统绝对并发量上来了，分表难以根本上解决问题，并且还没有明显的业务归属来垂直分库。

**分析：**库多了，io和cpu的压力自然可以成倍缓解。

### 2、水平分表

![img](https://img-blog.csdnimg.cn/img_convert/3d6fd52915339637a0ecad7a8865ba81.png)

**概念：**以字段为依据，按照一定策略（hash、range等），将一个表中的数据拆分到多个表中。

**结果：**

- 每个表的结构都一样；
- 每个表的数据都不一样，没有交集；
- 所有表的并集是全量数据；

**场景：**系统绝对并发量并没有上来，只是单表的数据量太多，影响了SQL效率，加重了CPU负担，以至于成为瓶颈。推荐：[一次SQL查询优化原理分析](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247491313&idx=2&sn=01d82309c459c7385a2ccf0018bb0d8a&chksm=ebd621dddca1a8cb16f33497c45aeb44e95ca0d5289afee263e047a868cb67f34ada644418b0&scene=21#wechat_redirect)

**分析：**表的数据量少了，单次SQL执行效率高，自然减轻了CPU的负担。

### 3、垂直分库

![img](https://img-blog.csdnimg.cn/img_convert/f2c1ad20396d12817d269bb003759cdd.png)

**概念：**以表为依据，按照业务归属不同，将不同的表拆分到不同的库中。

**结果：**

- 每个库的结构都不一样；
- 每个库的数据也不一样，没有交集；
- 所有库的并集是全量数据；

**场景：**系统绝对并发量上来了，并且可以抽象出单独的业务模块。

**分析：**到这一步，基本上就可以服务化了。例如，随着业务的发展一些公用的配置表、字典表等越来越多，这时可以将这些表拆到单独的库中，甚至可以服务化。再有，随着业务的发展孵化出了一套业务模式，这时可以将相关的表拆到单独的库中，甚至可以服务化。

### 4、垂直分表

![img](https://img-blog.csdnimg.cn/img_convert/1989daf97cc5322d08f07e96d5cbb4ce.png)

**概念：**以字段为依据，按照字段的活跃性，将表中字段拆到不同的表（主表和扩展表）中。

**结果：**

- 每个表的结构都不一样；
- 每个表的数据也不一样，一般来说，每个表的字段至少有一列交集，一般是主键，用于关联数据；
- 所有表的并集是全量数据；

**场景：**系统绝对并发量并没有上来，表的记录并不多，但是字段多，并且热点数据和非热点数据在一起，单行数据所需的存储空间较大。以至于数据库缓存的数据行减少，查询时会去读磁盘数据产生大量的随机读IO，产生IO瓶颈。

**分析：**可以用列表页和详情页来帮助理解。垂直分表的拆分原则是将热点数据（可能会冗余经常一起查询的数据）放在一起作为主表，非热点数据放在一起作为扩展表。这样更多的热点数据就能被缓存下来，进而减少了随机读IO。拆了之后，要想获得全部数据就需要关联两个表来取数据。

但记住，千万别用join，因为join不仅会增加CPU负担并且会讲两个表耦合在一起（必须在一个数据库实例上）。关联数据，应该在业务Service层做文章，分别获取主表和扩展表数据然后用关联字段关联得到全部数据。

## 三、分库分表工具

- sharding-sphere：jar，前身是sharding-jdbc；
- TDDL：jar，Taobao Distribute Data Layer；
- Mycat：中间件。

> 注：工具的利弊，请自行调研，官网和社区优先。

## 四、分库分表步骤

根据容量（当前容量和增长量）评估分库或分表个数 -> 选key（均匀）-> 分表规则（hash或range等）-> 执行（一般双写）-> 扩容问题（尽量减少数据的移动）。

扩展：[MySQL：分库分表与分区的区别和思考](http://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247491289&idx=1&sn=0dc45c799ca4c14782455d9d2f161674&chksm=ebd621f5dca1a8e3f2cc4d782b012bd84e61f310959fb3de9f48ef3fa7c54861a66389e9e9ff&scene=21#wechat_redirect)

## 五、分库分表问题

### 1、非partition key的查询问题

基于水平分库分表，拆分策略为常用的hash法。

**端上除了partition key只有一个非partition key作为条件查询**

映射法

![img](https://img-blog.csdnimg.cn/img_convert/855eb4d5dcba919896be6dda569b1019.png)

基因法

![img](https://img-blog.csdnimg.cn/img_convert/305e17b306067836e3314e3585d7b59a.png)

> 注：写入时，基因法生成user_id，如图。关于xbit基因，例如要分8张表，23=8，故x取3，即3bit基因。根据user_id查询时可直接取模路由到对应的分库或分表。
>
>  
>
> 根据user_name查询时，先通过user_name_code生成函数生成user_name_code再对其取模路由到对应的分库或分表。id生成常用snowflake算法。

**端上除了partition key不止一个非partition key作为条件查询**

映射法

![img](https://img-blog.csdnimg.cn/img_convert/5b0d208e00f852cfbbc7b0112ad41743.png)

冗余法

![img](https://img-blog.csdnimg.cn/img_convert/2a9ac7357daf5c8cbf5a2abaf2d99fb6.png)

> 注：按照order_id或buyer_id查询时路由到db_o_buyer库中，按照seller_id查询时路由到db_o_seller库中。感觉有点本末倒置！有其他好的办法吗？改变技术栈呢？

**后台除了partition key还有各种非partition key组合条件查询**

NoSQL法

![img](https://img-blog.csdnimg.cn/img_convert/24d72c19d30ae9a7f0f563f3944593cd.png)

冗余法

![img](https://img-blog.csdnimg.cn/img_convert/c9c158b3e7a4281b4b988bdc4c5a1857.png)

### 2、非partition key跨库跨表分页查询问题

基于水平分库分表，拆分策略为常用的hash法。

> 注：用NoSQL法解决（ES等）。

### 3、扩容问题

基于水平分库分表，拆分策略为常用的hash法。

**水平扩容库（升级从库法）**

![img](https://img-blog.csdnimg.cn/img_convert/2df68bd3c989894e4fb13db508d69f52.png)

> 注：扩容是成倍的。

**水平扩容表（双写迁移法）**

![img](https://img-blog.csdnimg.cn/img_convert/fb9eba2e8a4e5fbb107d20ae616e0c95.png)

- 第一步：（同步双写）修改应用配置和代码，加上双写，部署；
- 第二步：（同步双写）将老库中的老数据复制到新库中；
- 第三步：（同步双写）以老库为准校对新库中的老数据；
- 第四步：（同步双写）修改应用配置和代码，去掉双写，部署；

> 注：双写是通用方案。

## 六、分库分表总结

- 分库分表，首先得知道瓶颈在哪里，然后才能合理地拆分（分库还是分表？水平还是垂直？分几个？）。且不可为了分库分表而拆分。
- 选key很重要，既要考虑到拆分均匀，也要考虑到非partition key的查询。
- 只要能满足需求，拆分规则越简单越好。

## 七、分库分表示例

> 示例GitHub地址：https://github.com/littlecharacter4s/study-sharding