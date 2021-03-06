[MySQL 数据表优化设计（六）：id 该如何选择数据类型？](https://juejin.cn/post/6969928505932906510)

> 为 id 列选择一个好的数据类型非常重要，id 列会经常用于做比较（例如联合查询的条件），以及用于查找其他列。而且，id 也经常用于外键。因此，id 列的数据类型不仅仅关系自身数据表，也关系到与之关联的其他数据表。因此，id 用何种数据类型就显得十分重要。

选择 id 的数据类型，不仅仅需要考虑数据存储类型，还需要了解 MySQL 对该种类型如何计算和比较。例如，MySQL 将 ENUM 和 SET 类型在内部使用整型存储，但是在字符串场景下会当做字符串进行比较。一旦选择了 id 的数据类型后，需要保证引用 id 的相关数据表的数据类型一致，而且是完全一致，这包括属性，例如长度、是否有符号！如果混用不同的数据类型可能导致性能问题，即便是没有性能问题，在进行比较时的隐式数据转换可能导致难以捉摸的错误。而如果在实际开发过程中忘记了数据类型不同这个问题，可能会突然出现意想不到的问题。

在选择长度的时候，也需要尽可能选择小的字段长度并给未来留有一定的增长空间。例如，如果是用于存放省份的话，我们只有几十个值，此时使用 TINYINT 就 INT 就更好，如果是相关的表也存有这个 id 的话，那么效率差别会很大。

下面是适用于 id 的一些典型的类型：

- 整型：整型通常来说是最佳的选择，这是因为整型的运算和比较都很快，而且还可以设置 AUTO_INCREMENT 属性自动递增。
- ENUM 和 SET：通常不会选择枚举和集合作为 id，然后对于那些包含有“类型”、“状态”、“性别”这类型的列来说是挺合适的。例如我们需要有一张表存储下拉菜单时，通常会有一个值和一个名称，这个时候值使用枚举作为主键也是可以的。
- 字符串：尽可能地避免使用字符串作为 id，一是字符串占据的空间更大，二是通常会比整型慢。选用字符串作为 id 时，还需要特别注意 MD5、SHA1和 UUID 这些函数。每个值是在很大范围的随机值，没有次序，这会导致插入和查询更慢：
  - 插入的时候，由于建立索引是随机位置（会导致分页、随机磁盘访问和聚集索引碎片），会降低插入速度。
  - 查询的时候，相邻的数据行在磁盘或内存上上可能跨度很大，也会导致速度更慢。

如果确实要使用 UUID 值，应当移除掉“-”字符，或者是使用 UNHEX 函数将其转换为16字节数字，并使用 BINARY(16)存储。然后可以使用 HEX 函数以十六进制的方式进行获取。UUID 产生的方法有很多，有些是随机分布的，有些是有序的，但是即便是有序的性能也不如整型。

#### 分布式 id

对于单体应用来说，id 使用自增或者使用程序直接产生 id 问题都不大，但是如果是对于分布式应用来说，这种情况可能会导致主键冲突错误。对于分布式 id，目前有很多算法，例如有名的雪花（snowflakce）算法，以及国内大厂的一些分布式 id 开源算法，例如：

- 百度的[UidGenerator](https://github.com/baidu/uid-generator)。
- 美团的[Leaf](https://github.com/Meituan-Dianping/Leaf) 。

分布式 id 的基本原理是使用机器码、时间戳加序列号构成。通过一定的算法实现了分布式 id 的唯一性和有序性，具体有兴趣的可以看一下雪花算法的实现。