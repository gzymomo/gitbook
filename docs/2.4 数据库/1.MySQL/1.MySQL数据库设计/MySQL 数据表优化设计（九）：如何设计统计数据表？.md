[MySQL 数据表优化设计（九）：如何设计统计数据表？](https://juejin.cn/post/6971763528445198372)

> 有些时候，改进数据表查询性能的最佳方式是在同一张数据表中冗余一些继承的数据。然而，有些时候需要新建完全独立的统计或缓存数据表，尤其是在需要反复查询的需求情况下。如果业务允许一些时间上的误差的话，那么这种方式会更好。

缓存型数据表通常在统计数据时会经常用到，因此也会叫统计性数据。举个例子来说，对于员工、部门数据表而言，我们可能会需要查询一个部门下有多少员工。这时候有三种方式实现：

- 在部门下增加一个员工数量的字段，每次对员工进行增、改、删操作时都需要同步更新员工数量（如果员工换部门，则需要更新多个部门的员工数量）。这种方式能够保证实时性，但是却很低效。对于如果是操作不频繁时是没问题的，假设相当频繁，就意味着每次都需要操作两张表，而且业务代码都需要做埋点处理，将统计业务和普通业务深度耦合在一起了。
- 每次查询的时候，从员工表中执行 SUM 函数，获取该部门的员工数。这种方式避免了埋点，但是每次都需要去员工数据表求和，如果员工数据量大的话会很低效。
- 新建一张统计表，每隔一定时间从员工表中汇总每个部门的人员数量。这种定时抽取数据的方式会牺牲一定的实时性，但降低了代码的耦合，由于部门不会太多，这张表的大小是可预测的，也提高了数据访问的效率。这种方式即**缓存型数据表**。

以掘金的手机端个人中心为例，为展示每个用户的关注人数、关注者和掘力值，不可能每次查询都去做一次 SUM，这意味着需要做多张表的 SUM 操作，效率会很低，而且掘力值的计算还涉及到更为复杂的计算方法（与文章的浏览量和点赞数有关）。因此，可以猜测一下大致的表设计，这样在查询用户个人主页信息的时候只需要从这一张表就可以读取到所有数据了。

```sql
CREATE t_user_summay (
  id INT PRIMARY KEY,
  user_id BIGINT(20),
  focused_user_cnt INT,
  followed_user_cnt INT,
  user_value INT,
  user_level ENUM('Lv1', 'Lv2', ..., 'Lv8'),
  created_time DATETIME,
  updated_time DATETIME,
);
复制代码
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1dcc72a1804343a294227b07424ad3e6~tplv-k3u1fbpfcp-zoom-1.image)

#### 是否需要实时更新

在实际应用过程中，统计表有两种方式，一种是实时更新，一种是周期性的重建数据。两种方式有利有弊，实时更新保证了查询数据的即时性，但是会牺牲性能，并且要求代码埋点，而且由于数据更新是没有规律的，可能产生碎片。周期性的重建数据牺牲了实时性，如果说大部分数据都不变的话会带来不必要的统计计算，但如果数据经常变动，那周期性地重建数据显然会更高效而且避免了埋点的情况。当然，避免应用程序的埋点也可以通过触发器来完成，可以参考[MySQL 高级特性（七）：触发器的正确打开方式](https://juejin.cn/post/6964737836339855374)。

#### 物化视图工具（Flexviews）

在 MySQL 中，有一个 [Flexviews](https://github.com/greenlion/swanhart-tools/tree/master/flexviews) 的开源工具用于从数据库的binlog 中提取数据完成数据统计。有点类似与视图，但与视图所不同的是，Flexviews 产生的数据表是物理表，这也是为什么称之为物化视图的原因。而且，Flexviews 还支持增量更新和全量更新。推荐使用增量更新，以避免所有行的统计数据都需要重建的情况。增量更新会检查哪些数据行数据发生了改变，再执行更新操作，相比全量更新而言性能会更高。但为了检测数据改变，需要引入一个视图记录数据行的变化日志。

#### 计数表

在实际开发中，我们经常会需要对一些操作进行计数，比如文章的阅读数、点赞数。如果将计数值放入同一张表很可能在更新的时候出现并发问题。使用独立的计数表可以避免查询缓存失效问题并使用一些更高级的技巧。例如统计文章的阅读数、点赞数的数据表：

```sql
CREATE TABLE t_article_counter (
  article_id INT PRIMARY KEY,
  read_cnt INT UNSIGNED NOT NULL,
  praise_cnt INT UNSIGNED NOT NULL
);
复制代码
```

在更新阅读数的时候，可以使用 MySQL 的内置加1操作：

```sql
UPDATE t_article_counter 
SET read_cnt = read_cnt + 1
WHERE article_id = 1;
复制代码
```

这种方式可以使得操作是单行的，对事物而言是互斥的，因此会将事务序列化处理避免并发问题。但是却会影响并发请求量。可以对文章增加多个插槽来提高并发量。

```sql
CREATE TABLE t_article_counter (
  id INT NOT NULL PRIMARY KEY,
  slot TINYINT UNSIGNED,
  article_id INT,
  read_cnt INT UNSIGNED NOT NULL,
  praise_cnt INT UNSIGNED NOT NULL,
  INDEX(article_id)
);
复制代码
```

这时可以创建100个插槽初始化数据，在更新的时候可以这样操作：

```sql
UPDATE t_article_counter
SET read_cnt = read_cnt + 1 
WHERE slot = RAND() * 100 AND article_id = 1;
复制代码
```

获取某篇文章的总阅读数时，需要使用一个 SUM 操作：

```sql
SELECT SUM(read_cnt) FROM t_article_counter
WHERE article_id = 1;
复制代码
```

这种方式实际上是空间换时间，提高了并发量。

#### 总结

本篇介绍了如何设计统计数据表，关键的核心在于业务类型。对于更新频率低、数据量小的表使用实时同步或者直接 SUM 求和问题都不大。而对于大数据表，高频率的更新的情况，则可以使用独立的统计表。同时，若存在高并发的情况，统计表中可以考虑每项主体增加多个插槽的方式提高并发量。如果是周期性地同步数据，也可以使用 Flexviews 物化视图插件实现。

