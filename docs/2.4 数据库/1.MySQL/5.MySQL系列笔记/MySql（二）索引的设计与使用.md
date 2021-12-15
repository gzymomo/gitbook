[TOC]

# 一、索引概述
所有Mysql列类型都可以被索引，对相关列使用索引时提高select操作性能的最佳途径。
根据存储引擎可以定义每个表的最大索引数和最大索引长度，每种存储引擎对每个表至少支持16个索引，总索引长度至少为256字节。

MyISAM和InnoDB存储引擎的表默认创建的都是BTREE索引。

索引在创建表的时候可以同时创建，也可以随时增加新的索引。
创建新索引的语法为：
```sql
create [unique|fulltext|spatial] index index_name
[using index_type]
on tb1_name (index_col_name,...)

index_col_name:
col_name[(length)][ASC|DESC]
```
也可使用alter table的语法来增加索引。
例如：为city表创建10个字节的前缀索引：
```sql
mysql> create index cityname on city (city(10));
```
如果以city为条件进行查询，可以发现索引cityname被使用：
```sql
mysql> explain select * from city where city = 'yt' \G
```
索引删除语法为：
```sql
drop index index_nama on tb1_name
```
例如：想要删除city表上的索引cityname，可以：
```sql
mysql> drop index cityname on city;
```
# 二、设计索引的原则
 - 搜索的索引列，不一定是要选择的列。也就是说，最适合索引的列是出现在where字句中的列，或连接子局中指定的列，而不是出现在select关键字后的选择列表中的列。
 - 使用唯一索引。考虑某列中值的分布。索引的列的基数越大，索引的效果越好。例如：存放出生日期的列具有不同的值，易于区分各行。而用来记录性别的列，只含有“M”和“F”，则对此列进行索引没有多大用处。
 - 使用短索引。如果对字符串列进行索引，应该指定一个前缀长度，只要有可能就应该这样做。例如，一个char(200)列，如果在前10或20个字符内，多数值是唯一的，那么就不要对整个列进行索引。对前10个或20个字符进行索引能够节省大量索引空间，也可能会使查询更快。较小的索引涉及的磁盘IO较少，较短的值比较起来更快。更为重要的是，对于较短的键值，索引高速缓存中的块能容纳更多的键值，因此，MySQL也可以在内存中容纳更多的值。这样就增加了找到行而不用读取索引中较多块的可能性。
 - 利用最左前缀。在创建一个n列的索引时，实际是创建了mysql可利用的n个索引。多列索引可起几个索引的作用，因为可利用索引中最左边的列集来匹配行。
 - 不要过度索引。不要以为索引越多越好。每个额外的索引都要占用额外的磁盘空间，并降低写操作的性能。在修改表的内容时，索引必须进行更新，有时可能需要重构，因此，索引越多，所花的时间越长。创建多余的索引给查询优化带来了更多的工作。
 - 对于InnoDB存储引擎的表，记录默认会按照一定的顺序保存，如果有明确定义的主键，则按照主键顺序保存，如果没有主键，但是有唯一索引，那么就按照唯一索引的顺序保存。如果既没有主键又没有唯一索引，那么表中会自动生成一个内部列，按照这个列的顺序保存。按照主键或内部列进行的访问是最快的，所以InnoDB表尽量自己指定主键，当表中同时有几个列都是唯一的，都可以作为主键的时候，要选择最常作为访问条件的列作为主键，提高查询的效率。此外，InnoDB表的普通索引都会保存主键的键值，所以主键要尽可能选择较短的数据类型，可以有效地减少索引的磁盘占用，提高索引的缓存效果。

# 三、BTREE索引与HASH索引
对于HASH索引：
 - 只用于使用=或<=>操作符的等式比较。
 - 优化器不能使用HASH索引来加速ORDER BY操作。
 - MySQL不能确定在两个值之间大约有多少行。如果将一个MyISAM表改为HASH索引的Memory表，会影响一些查询的执行效率。
 - 只能使用整个关键字来搜索一行。

对于BTREE索引，当使用>、<、>=、<=、BETWEEN、！=或者<>，或者like 'pattern'（pattern不以通配符开始）操作符时，都可以使用相关列上的索引。

下列范围查询适用于BTREE索引和HASH索引：
```sql
select * from t1 where key_col = 1 or key_col in (15,18,20);
```
下列范围查询只适用于BTREE索引：
```sql
select * from t1 where key_col > 1 and key_col < 10;
select * from t1 where key_col like 'ab%' or key_col between 'lisa' and 'simon';
```
例如：创建一个和city表完全相同的memory存储引擎的表city_memory：
```sql
mysql> create table city_memory(
	city_id smallint unsigned not null auto_increment,
	city varchar(50) not null,
	country_id smallint unsigned not null,
	last_update timestamp not null default current_timestamp on update current_timestamp,
	primary key (city_id),
	key idx_fx_country_id(country_id)
)engine=memory default charset=urf-8;

mysql> insert into city_momory select * from city;
```
当对索引字段进行范围查询的时候，只有BTREE索引可通过索引访问：
```sql
mysql> explain select * from city where country_id > 1 and country_id < 10 \G
```
而HASH索引实际上是全表扫描的：
```sql
mysql> explain select * from city_memory where country_id > 1 and country_id < 10 \G
```

**索引用于快速找出在某个列中有一特定值的行。如果不使用索引，Mysql必须从第一条记录开始然后读完整个表直到找出相关的行。表越大，花费的时间越多。如果表中查询的列有一个索引，Mysql能快速到达一个位置去搜寻数据文件的中间，没有必要看所有数据。**

大多数Mysql索引（如PRIMARY KEY、UNIQUE、INDEX和FULLTEXT等）在BTREE中存储。
空间列类型的索引使用RTREE，且MEMORY表还支持HASH索引。

# 四、MySql创建索引的建议
## 4.1 过滤效率高的放前面
对于一个多列索引，它的存储顺序是先按第一列进行比较，然后是第二列，第三列...这样。查询时，如果第一列能够排除的越多，那么后面列需要判断的行数就越少，效率越高。

关于如何判断哪个列的过滤效率更高，可以通过选择性计算来决定。例如我们要在books表创建一个name列和author列的索引，可以计算这两列各自的选择性：
```sql
select count(distinct name) / count(*) as name, count(distinct author) / count(*) as author from books;
```
最后得出结果如下:

| Name  | author  |
| ------------ | ------------ |
| 0.95  | 0.9  |
显然name字段的选择性更高，那么如果把name放第一列，在name条件过滤时就可以排除更多的列，减少接下来 author的过滤。

## 4.2 使用频率高的放前面
建议比上一个建议优先级更高

例如一个商品管理页面，一般都是基于该店家的上架或已下架的商品，再添加其他的查询条件等等。由于所有的查询都需要带有shopid和status条件，此时应该优先将这两个条件作为基本前缀，这样就可以方便复用索引。

例如一个(shopid, status, createdat)的索引，当查询条件只有shopid和status时，也可以使用该索引。如果完全根据字段的过滤效率来决定索引，就需要创建很多不同的索引。

## 4.3 避免排序
索引的值都是有序排列的，在创建索引时还可以显式指定每个列的排序方式，例如
```sql
create index idx_books_author_created_at on books (author, created_at DESC);
```
此时，如果执行下面的的查询
```sql
select * from books where author = 'author' order by created_at DESC;
```
由于满足auhtor的索引的created_at列都是有序排列的，所以不需要再进行额外的排序操作。
![](https://img2020.cnblogs.com/blog/1003414/202004/1003414-20200428234319117-212065691.png)

当结果数据集很大时，应该尽可能的通过索引来避免查询的额外排序，因为当内存排序空间(sort_buffer_size)不够用时，就需要把一部分内容放到硬盘中，此时会很影响性能。

例如一个分页查询每页显示100条，按从大到小的顺序显示，当浏览到第100页时，如果查询是file sort的，数据库需要使用堆排序先计算出这个表里面前100 * 100 = 10000条最大的数据，然后取9900 - 10000之间的数据返回给客户端，在计算的过程中，这个最大堆如果放不下就需要保存到磁盘中，但是又需要频繁比较和替换。

## 4.4 减少随机IO
在之前对硬盘知识了解后可以知道，一次随机读会有10ms的寻址延迟，如果一次查询涉及达到多次的随机读，会很大程度的限制查询性能。常见的sql查询造成随机IO的包括回表和join

例如下面的查询
```sql
select * from books where author = 'author1';
```
如果author1有100本书，但是这100本书并不是连续录入的，也就是说这100本书在硬盘中的存储是分离的。那么在有二级索引(author, created_at)的情况下，MySQL先通过二级索引找到满足author1的所有books的id，然后再通过id在聚簇索引中找到具体数据。

在这一过程中，二级索引的存储可以认为是连续的，那么二级索引耗时就是10ms + 100 * 0.01 = 11ms，包含一次寻址以及接下来的顺序读。而主键索引回表造成的随机IO最差情况是10ms * 100 = 1000ms。那么一共就需要11ms + 1000ms = 1011ms

通常减少随机IO的一种方式就使用覆盖索引。例如上面的查询中，如果我们只是想要该作者的书名，可以将(author, createdat)扩展为(author, createdat,name)，然后将sql修改如下
```sql
select name from books where author = 'author1';
```
由于索引中已经有name的信息，此时就不会再次回表，查询耗时就变成了10ms + 100 * 0.01 = 11ms