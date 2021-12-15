[MySQL 数据表优化设计（五）：如何选择一个合适的时间类型？](https://juejin.cn/post/6969155125025718279)

> MySQL 有多种类型存储日期和时间，例如 YEAR 和 DATE。MySQL 的时间类型存储的精确度能到秒（MariaDB 可以到毫秒级）。但是，也可以通过时间计算达到毫秒级。时间类型的选择没有最佳，而是取决于业务需要如何处理时间的存储。

MySQL 提供了 DATETIME 和 TIMESTAMP 两种非常相似的类型处理日期和时间，大部分情况下两种都是 OK 的，但是有些情况二者会互有优劣。

#### DATETIME

DATETIME 的时间跨度更大，可以从1001年到9999年，精度是秒。并且存储的格式是将日期和时间打包使用 YYYYMMDDhhmmss格式的整数存储，这个时间与时区无关，需要占用8个字节的存储空间。默认，MySQL 显示 的DATETIME是有序的，明确的格式，例如2021-06-02 18:35:23。这是 ANSI 的标准日期时间格式。

#### TIMESTAMP

TIMESTAMP即时间戳，存储的是自格林威治时间（GMT）1970年1月1日零点以来的秒数。和 Unix 系统的时间戳一样。TIMESTAMP 仅需要4个字节存储，因此能够表示的时间跨度更小，从1970年到2038年。MySQL 提供了 FROM_UNIXTIME 和 UNIX_TIMESTAMP 函数来完成时间戳和日期之间的转换。

在 MySQL 4.1版本后，TIMESTAMP 显示的格式和 DATETIME 类似，但是，TIMESTAMP 的显示依赖于时区。MySQL 的服务端、操作系统以及客户端连接都有时区的设置。因此，如果时间是从多个时区存储的话，那 TIMESTAMP 和 DATETIME 的差别就会很大。TIMESTAMP 会保留使用时的时区信息，而 DATETIME 仅仅是使用文本表示时间。

TIMESTAMP 还有额外的特性。默认地，MySQL会在没有指定值的情况下使用当前时间插入到 TIMESTAMP列，而更新的时候如果没有指定值会使用当前时间更新该字段，以下面的测试表为例：

```sql
CREATE TABLE t_time_test (
    id INT PRIMARY KEY,
    t_stamp TIMESTAMP,
    t_datetime DATETIME
);
复制代码
```

可以看到MySQL 给的默认值就是当前时间戳 CURRENT_TIMESTAMP，并且有个 ON UPDATE CURRENT_TIMESTAMP表示会随之更新： ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/308ceb1a60a041cbb0a3a28412e9df67~tplv-k3u1fbpfcp-zoom-1.image)

```sql
INSERT INTO t_time_test(id, t_datetime) VALUES
	(1, NULL), 
	(2, '2021-06-02 18:48:04'), 
	(3, NULL);
复制代码
```

可以看到 t_stamp 列自动填充了当前时间。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11e09df881564996874d8a5a4b54a279~tplv-k3u1fbpfcp-zoom-1.image)

```sql
UPDATE `t_time_test` 
SET `t_datetime`='2021-06-02 19:00:00' WHERE id=1;
复制代码
```

可以看到 id 为1的一列的 t_stamp 列会自动更新为当前时间。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c320f5b17694192857e5d2c269739fa~tplv-k3u1fbpfcp-zoom-1.image) 这个特性使得我们可以物协程序维护数据更新时间字段，而交由 MySQL 完成。

#### 如何选择

从特性上看，可能会优先选择使用 TIMESTAMP 来存储时间，相比 DATETIME 来说更高效。也有些人使用整数存储 Unix 时间戳，实际上这种方式并不能获益，而且整数还需要额外进行处理，因此并不推荐这么做。但是一些情况需要注意不要使用 TIMESTAMP 存储时间：

- 生日：生日肯定会有早于1970年的，会超出 TIMESTAMP 的范围
- 有效期截止时间：TIMESTAMP 的最大时间是2038年，如果用来存类似身份证的有效期截止时间，营业执照的截止时间等就不合适。
- 业务生存时间：互联网时代讲究快，~~发展~~（死得）快。如果要成为长久存在的企业，那么你的业务时间很可能在2038年还在继续运营，毕竟现在都2021年了。如果你觉得公司业务挺不到2038年，那没关系。当然，如果幸运地挺到2038年，请务必写下一条待办事项：**到2038年1月1日前修改数据表时间戳字段类型**。

#### 如何存储毫秒级时间

通常这个时候需要使用 BIGINT 来将时间转换为整型存储，或者是使用浮点数，用分数部分表示秒精度一下的时间，这两种方式都可行。当然，这个时候需要应用支持做格式转换。

#### 结语

从安全稳妥的角度考虑，建议还是优先选择 DATETIME 类型，虽然相比 TIMESTAMP 会牺牲一点性能，但是 TIMESTAMP 的时间范围是硬伤，不要埋下一个隐患，等到真的2038年，你的公司可能是上市公司的时候，程序员可能会遭遇洪水般的 bug 冲击而不明所以，结果公司的股价迎来闪崩！然后找出来这个程序员，发现是**曾经公司的大神**，**目前的股东**，**已经实现财务自由的你**！你说尴尬不尴尬？

