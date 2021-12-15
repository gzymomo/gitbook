- [MySQL 中存储时间的最佳实践](https://www.cnblogs.com/upyun/p/14957721.html)



平时开发中经常需要记录时间，比如用于记录某条记录的创建时间以及修改时间。在数据库中存储时间的方式有很多种，比如 MySQL  本身就提供了日期类型，比如 DATETIME，TIMESTAMEP 等，我们也可以直接存储时间戳为 INT  类型，也有人直接将时间存储为字符串类型。

那么到底哪种存储时间的方式更好呢？

## 不要使用字符串存储时间类型

这是初学者很容易犯的错误，容易直接将字段设置为 VARCHAR 类型，存储"2021-01-01 00:00:00"这样的字符串。当然这样做的优点是比较简单，上手快。

但是极力不推荐这样做，因为这样做有两个比较大的问题：

- 字符串占用的空间大
- 这样存储的字段比较效率太低，只能逐个字符比较，无法使用 MySQL 提供的日期API

## MySQL 中的日期类型

MySQL 数据库中常见的日期类型有 YEAR、DATE、TIME、DATETIME、TIMESTAMEP。因为一般都需要将日期精确到秒，其中比较合适的有DATETIME，TIMESTAMEP。

**DATETIME**

DATETIME 在数据库中存储的形式为：YYYY-MM-DD HH:MM:SS，固定占用 8 个字节。

从 MySQL 5.6 版本开始，DATETIME 类型支持毫秒，DATETIME(N) 中的 N 表示毫秒的精度。例如，DATETIME(6) 表示可以存储 6 位的毫秒值。

**TIMESTAMEP**

TIMESTAMP 实际存储的内容为‘1970-01-01 00:00:00’到现在的毫秒数。在 MySQL 中，由于类型 TIMESTAMP 占用 4 个字节，因此其存储的时间上限只能到‘2038-01-19 03:14:07’。

从 MySQL 5.6 版本开始，类型 TIMESTAMP 也能支持毫秒。与 DATETIME 不同的是，若带有毫秒时，类型 TIMESTAMP 占用 7 个字节，而 DATETIME 无论是否存储毫秒信息，都占用 8 个字节。

类型 TIMESTAMP 最大的优点是可以带有时区属性，因为它本质上是从毫秒转化而来。如果你的业务需要对应不同的国家时区，那么类型  TIMESTAMP 是一种不错的选择。比如新闻类的业务，通常用户想知道这篇新闻发布时对应的自己国家时间，那么 TIMESTAMP  是一种选择。Timestamp  类型字段的值会随着服务器时区的变化而变化，自动换算成相应的时间，说简单点就是在不同时区，查询到同一个条记录此字段的值会不一样。

**TIMESTAMP 的性能问题**

TIMESTAMP 还存在潜在的性能问题。

虽然从毫秒数转换到类型 TIMESTAMP 本身需要的 CPU  指令并不多，这并不会带来直接的性能问题。但是如果使用默认的操作系统时区，则每次通过时区计算时间时，要调用操作系统底层系统函数  __tz_convert()，而这个函数需要额外的加锁操作，以确保这时操作系统时区没有修改。所以，当大规模并发访问时，由于热点资源竞争，会产生两个问题：

- 性能不如 DATETIME：DATETIME 不存在时区转化问题。
- 性能抖动：海量并发时，存在性能抖动问题。

为了优化 TIMESTAMP 的使用，建议使用显式的时区，而不是操作系统时区。比如在配置文件中显示地设置时区，而不要使用系统时区：

```
[mysqld]

time_zone = "+08:00"
```

简单总结一下这两种数据类型的优缺点：

- DATETIME 没有存储的时间上限，而TIMESTAMP存储的时间上限只能到‘2038-01-19 03:14:07’
- DATETIME 不带时区属性，需要前端或者服务端处理，但是仅从数据库保存数据和读取数据而言，性能更好
- TIMESTAMP 带有时区属性，但是每次需要通过时区计算时间，并发访问时会有性能问题
- 存储 DATETIME 比 TIMESTAMEP 多占用一部分空间

**数值型时间戳（INT）**

很多时候，我们也会使用 int 或者 bigint 类型的数值也就是时间戳来表示时间。

这种存储方式的具有 Timestamp 类型的所具有一些优点，并且使用它的进行日期排序以及对比等操作的效率会更高，跨系统也很方便，毕竟只是存放的数值。缺点也很明显，就是数据的可读性太差了，你无法直观的看到具体时间。

如果需要查看某个时间段内的数据

```
select * from t where created_at > UNIX_TIMESTAMP('2021-01-01 00:00:00');
```

## DATETIME vs TIMESTAMP vs INT，怎么选？

每种方式都有各自的优势，下面再对这三种方式做一个简单的对比：

![img](https://upload-images.jianshu.io/upload_images/80097-fa7b6aad30fc2bec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

TIMESTAMP 与 INT 本质一样，但是相比而言虽然 INT 对开发友好，但是对 DBA 以及数据分析人员不友好，可读性差。所以《高性能 MySQL 》的作者推荐  TIMESTAMP 的原因就是它的数值表示时间更加直观。下面是原文：

![img](https://upload-images.jianshu.io/upload_images/80097-ccce6ad907608e38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至于时区问题，可以由前端或者服务这里做一次转化，不一定非要在数据库中解决。

## 总结

本文比较了几种最常使用的存储时间的方式，我最推荐的还是 DATETIME。理由如下：

- TIMESTAMP 比数值型时间戳可读性更好
- DATETIME 的存储上限为 9999-12-31 23:59:59，如果使用 TIMESTAMP，则 2038 年需要考虑解决方案
- DATETIME 由于不需要时区转换，所以性能比 TIMESTAMP 好
- 如果需要将时间存储到毫秒，TIMESTAMP 要 7 个字节，和 DATETIME 8 字节差不太多

