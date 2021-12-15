[TOC]

博客园：以终为始：[关于 InnoDB 锁的超全总结](https://www.cnblogs.com/michael9/p/12443975.html)
# 1、InnoDB锁
## 1.1 Shared and Exclusive Locks（读-写）
共享锁和排他锁，即常说的读锁和写锁。二者之间互斥。**共享锁和排他锁是标准的实现行基本的锁。**举例来说，单给select语句应用`lock in share mode`或者`for update`，更新某条记录时，加的都是行级别的锁。

与行级别的共享锁和排他锁类似的，还有表级别的共享锁和排他锁。如 LOCK TABLES ... WRITE/READ 等命令，实现的就是表级锁。

## 1.2 Intention Locks（意向锁）
Intention Locks - 意向锁，就是表级锁。和行级锁一样，意向锁分为 intention shared lock (IS) 和 intention exclusive lock (IX) . 但有趣的是，IS 和 IX 之间并不互斥，也就是说可以同时给不同的事务加上 IS 和 IX。

意向锁的目的就是表明有事务正在或者将要锁住某个表中的行。我们使用 select * from t where id=0 for update; 将 id=0 这行加上了写锁。假设同时，一个新的事务想要发起 LOCK TABLES ... WRITE 锁表的操作，这时如果没有意向锁的话，就需要去一行行检测是否所在表中的某行是否存在写锁，从而引发冲突，效率太低。相反有意向锁的话，在发起 lock in share mode 或者 for update 前就会自动加上意向锁，这样检测起来就方便多了。

## 1.3 Record Locks（行锁）
InnoDB 中，表都以索引的形式存在，每一个索引对应一颗 B+ 树，这里的行锁锁的就是 B+ 中的索引记录。之前提到的共享锁和排他锁，就是将锁加在这里。

## 1.4 Gap Locks（间隙锁）
间隙锁锁住的是索引记录间的空隙，是为了解决幻读问题被引入的。有一点需要注意，间隙锁和间隙锁本身之间并不冲突，仅仅和插入这个操作发生冲突。

## 1.5 Next-Key lock（行锁和间隙所的并集）
在 RR 级别下，InnoDB 使用 next-key 锁进行树搜索和索引扫描。
**加锁的基本单位是 next-key lock.**

# 2、加锁规则
规则包括：两个“原则”、两个“优化”和一个“bug”。
 1. 原则1：加锁的基本单位是 next-key lock。next-key lock 是前开后闭区间。
 2. 原则2：查找过程中访问到的对象才会加锁。
 3. 优化1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
 4. 优化2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。（从等值查询的值开始，向右遍历到第一个不满足等值条件记录结束，然后将不满足条件记录的 next-key 退化为间隙锁。）
 5. 一个BUG：唯一索引上的范围查询会访问到不满足条件的第一个值为止。

等值查询和遍历有什么关系？
在分析加锁行为时，一定要从索引的数据结构开始。通过树搜索的方式定位索引记录时，用的是"等值查询"，而遍历对应的是在记录上向前或向后扫描的过程。

# 3、应用场景
表结构如下：
```SQL
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB;

insert into t values(0,0,0),(5,5,5),
(10,10,10),(15,15,15),(20,20,20),(25,25,25);
```
## 3.1 主键索引等值间歇锁
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| update t set d=d+1 where id=7;  |   |   |
|   | insert into t values(8,8,8);  |   |
|   | 被阻塞  | update t set d=d+1 where id=10;  |
|   |   |  查询正常 |
其中 id 列为主键索引，并且 id=7 的行并不存在。
对于 Session A 来说：
 1. 根据原则1，加锁的单位是 next-key, 因为 id=7 在 5 - 10 间，next-key 默认是左开右闭。所以范围是 (5,10].
 2. 根据优化2，但因为 id=7 是等值查询，到 id=10 结束。next-key 退化成间隙锁 (5,10).
对于 Session B 来说：
 - 插入操作与间隙锁冲突，所以失败。
对于 Session C 来说：
 1. 根据原则1，next-key 加锁 (5,10].
 2. 根据优化1：给唯一索引加锁时，退化成行锁。范围变为：id=10 的行锁
 3. Session C 和 Session A (5,10) 并不起冲突，所以成功。
这里可以看出，**行锁和间隙锁都是有 next-key 锁满足一定后条件后转换的，加锁的默认单位是 next-key.**

## 3.2 非唯一索引等值锁
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| select id from t where c=5 lock in share mode;  |   |   |
|   | update t set d=d+1 where id=5;  |   |
|   | 查询正常  | insert into t values(7,7,7);  |
|   |   |  被阻塞 |
关注几点：c为非唯一索引，查询的字段仅有 id，`lock in share mode` 给满足条件的行加上读锁。
Session A：
 1. c=5，等值查询且值存在。先加 next-key 锁，范围为 (0,5].
 2. 由于 c 是普通索引，因此仅访问 c=5 这一条记录不会停止，会继续向右遍历，到 10 结束。根据原则2，这时会给 id=10 加 next-key (5,10].
 3. 但 id=10 同时满足优化2，退化成间隙锁 (5,10).
 4. 根据原则2，该查询使用覆盖索引，可以直接得到 id 的值，主键索引未被访问到，不加锁。
Session B：
 1. 根据原则1 和优化1，给 id=10 的主键索引加行锁，并不冲突，修改成功。
Session C：
 1. 由于 Session A 已经对索引 c 中 (5,10) 的间隙加锁，与插入 c=7 冲突， 所以被阻塞。
可以看出，**加锁其实是在索引上，并且只加在访问到的记录上，如果想要在 lock in share mode 下避免数据被更新，需要引入覆盖索引不能包含的字段。**

假设将 Session A 的语句改成 `select id from t where c=5 for update`;, for update 表示可能当前事务要更新数据，所以也会给满足的条件的主键索引加锁。这时 Session B 就会被阻塞了。

## 3.3 非唯一索引等值锁-锁主键
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| select id from t where c=5 lock in share mode;  |   |
|   | insert into t values(9,10,7);  |
|   | 被阻塞  |
Session A 的加锁范围不变，给索引 C 加了 (0,5] 和 (5,10) 的行锁。需要知道的是，非唯一索引形成的 key，需要包含主键 id，用于保证唯一性。

首先主键索引是另外一颗B+树确实没有被锁，但这里由于 C 是非唯一索引，形成的B+树需要将主键索引的 Id 包含进来，并在按照先 c 字段，后 id 字段进行排序。这样，在给 c 字段加行锁时，对应的 id 也同时加了行锁。

**也就说，对于非唯一索引，考虑加锁范围时要考虑到主键 Id 的情况。**

## 3.4 主键索引范围锁
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| select * from t where id>=10 and id <11 for update;  |   |   |
|   | insert into t values(8,8,8);  |   |
|   | 正常  |  |
|   | insert into t values(13,13,13);  |  |
|   | 被阻塞  |  |
|   |   | update t set d=d+1 where id=15;   |
|   |   |  被阻塞  |

Session A：
 1. 先找到 id=10 行，属于等值查询。根据原则1，优化1，范围是 id=10 这行。
 2. 向后遍历，属于范围查询，找到 id=15 这行，根据原则2，范围是 (10,15]
Session B：
 1. 插入 (8,8,8) 可以，(13,13,13) 就不行了。
Session C：
 1. id=15 同样在锁的范围内，所以也被阻塞。

## 3.5 非唯一索引范围锁
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| select * from t where c&=10 and c <11 for update;  |   |   |
|   | insert into t values(8,8,8);  |   |
|   | 被阻塞  |  |
|   | insert into t values(13,13,13);  |  |
|   | 被阻塞  |  |
|   |   | update t set d=d+1 where id=15;   |
|   |   |  被阻塞  |
Session A：
 1. 由于 c 是非唯一索引，索引对于 c=10 等值查询来说，根据原则1，加锁范围为 (5,10].
 2. 向右遍历，范围查询，加锁范围为 (10,15].
Session B:
 1. (8,8,8) 和 (13,13,13) 都冲突，所以被阻塞。
Session C:
 1. c=5 也被锁住，也会被阻塞。

## 3.6 唯一索引范围锁bug
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| select * from t where id&10 and id <=15 for update;  |   |   |
|   | update t set d=d+1 where id=20;  |   |
|   | 被阻塞  |  |
|   |   | insert into t values(16,16,16);   |
|   |   |  被阻塞  |
Session A：
 1. 由于开始找大于 10 的过程中是第一次是范围查询，所以没有优化原则。加 (10,15].
 2. 有一个特殊的地方，理论上说由于 id 是唯一主键，找到 id=15 就应该停下来了，但实际没有。根据 bug 原则，会继续扫描第一个不满足的值为止，接着找到 id=20，因为是范围查找，没有优化原则，继续加锁 (15,20].
`这个 bug 在 8.0.18 后已经修复了`
对于 Session B 和 Session C 均和加锁的范围冲突。

## 3.7 非唯一索引delete
|  Session A | Session B  | Session C  |
| ------------ | ------------ | ------------ |
| begin;  |   |   |
| delete from t where c=10;  |   |   |
|   | insert into t values(12,12,12)  |   |
|   | 被阻塞  |  |
|   |   | update t set d=d+1 where c=15;   |
|   |   |  成功  |
delete 后加 where 语句和 select * for update 语句的加锁逻辑类似。
Session A:
 1. 根据原则1，加 (5,10] 的 next-key.
 2. 向右遍历，根据优化2，加 (10,15) 的间歇锁。
## 3.8 非唯一索引limit
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| delete from t where c=10 limit 1;  |   |
|   |insert into t values(12,12,12)  |
|   | 正常  |
虽然 c=10 只有一条记录，但和场景7 的加锁范围不同。
Session A:
 1. 根据原则1，加 (5,10] 的 next-key.
 2. 因为加了 limit 1，所以找到一条就可以了，不需要继续遍历，也就是说不在加锁。
所以对于 session B 来说，就不在阻塞。
## 3.9 next-key引发死锁
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| select id from t where c=10 lock in share mode;  |   |
|   |update t set d=d+1 where c=10;  |
|   | 被阻塞  |
| insert into t values(8,8,8)  |   |
|   | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction  |
 1. Session A 第一句 加 (5,10] next-key. 和 (10,15) 的 间隙锁。
 2. Session B，和 Session A 想要的加锁范围相同，先加 (5,10] next-key 发现被阻塞，后面 (10,15) 没有被加上，暂时等待。
 3. Session A，加入 (8,8,8) 和 Session B 的加锁范围 (5,10] 冲突，被阻塞。形成 A,B 相互等待的情况。引发死锁检测，释放 Session B.
> 假如把 insert into t values(8,8,8) 改成 insert into t values(11,11,11) 是可以的，因为 Session B 的间歇锁 (10,15) 没有被加上。

分析下死锁：
```SQL
mysql> show engine innodb status\G;
------------------------
LATEST DETECTED DEADLOCK
------------------------
2020-03-08 17:04:10 0x7f9be0057700
*** (1) TRANSACTION:
TRANSACTION 836108, ACTIVE 16 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 1653, OS thread handle 140307320846080, query id 1564409 localhost cisco updating
update t set d=d+1 where c=10
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 251 page no 4 n bits 80 index c of table `my_test`.`t` trx id 836108 lock_mode X waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 4; hex 8000000a; asc     ;;

*** (2) TRANSACTION:
TRANSACTION 836109, ACTIVE 22 sec inserting
mysql tables in use 1, locked 1
5 lock struct(s), heap size 1136, 3 row lock(s), undo log entries 1
MySQL thread id 1655, OS thread handle 140307455112960, query id 1564410 localhost cisco update
insert into t values(8,8,8)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 251 page no 4 n bits 80 index c of table `my_test`.`t` trx id 836109 lock mode S
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 4; hex 8000000a; asc     ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 251 page no 4 n bits 80 index c of table `my_test`.`t` trx id 836109 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 4; hex 8000000a; asc     ;;

*** WE ROLL BACK TRANSACTION (1)
```

 1. (1) TRANSACTION 表明发生死锁的第一个事务信息。
 2. (2) TRANSACTION 表明发生死锁的第二个事务信息。
 3. WE ROLL BACK TRANSACTION (1) 表明死锁的处理方案。
针对 (1) TRANSACTION:
 1. (1) WAITING FOR THIS LOCK TO BE GRANTED 表示 update t set d=d+1 where c=10 要申请写锁,并处于锁等待的情况。
 2. 申请的对象是 n_fields 2，hex 8000000a; 和 hex 8000000a;, 也就是 id=10 和 c=10 的记录。
针对 (1) TRANSACTION:
 1. HOLDS THE LOCK(S): 表示当前事务2持有的锁是 :hex 8000000a; 和 hex 8000000a;.
 2. WAITING FOR THIS LOCK TO BE GRANTED: 表示对于 insert into t values(8,8,8) 进行所等待。
 3. lock_mode X locks gap before rec insert intention waiting: 表明在插入意向锁时，等待一个间隙锁( gap before rec)。
所以最后选择，回滚事务 (1)。

## 3.10 非唯一索引order by
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| select * from t where c&=15 and c<=20 order by c desc lock in share mode;  |   |
|   | insert into t values(6,6,6);  |
|   | 被阻塞  |
在分析具体的加锁过程时，先要分析语句的执行顺序。如 Session A 中使用了 ordery by c desc 按照降序排列的语句，这就意味着需要在索引树 C 上，找到第一个 20 的值，然后向左遍历。并且由于 C 是非唯一索引 20 的值应该是记录中最右边的值。

Session A 的加锁过程：
 1. 在找到第一个 c=20 的值后，加 next-key (15,20].
 2. 但不会停下，因为无法确定当前 c=20 是最右面的值，继续遍历到 c=25，发现不满足，根据优化2，加 (20,25) 的间隙锁。
 3. 然后从最左面的 c=20 向左遍历，找到 c=15，加锁 next-key (10,15].
 4. 和之前是一样，无法确定 c=15 是最左面的值，继续遍历到 c=10，根据优化2，加(5,10)的间隙锁 。
 5. 最后由于是 select * 对应主键索引 id=10,15,20 加行锁。

## 3.11 INSERT INTO .... SELECT ...
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| insert into t values(1,1,1);  |   |
|   | insert into t (id,c,d) select 1,1,1 from t where id=1;  |
|   | 被阻塞  |
为了保证数据的一致性，对于 INSERT INTO .... SELECT ... 中 select 部分会加 next-key 的读锁。

对于 Session A，在插入数据后，有了 id=1 的行锁。而 Session B 中的 select 虽然是一致性读，但会加上 id=1 的读锁。与 Session A 冲突，所以被阻塞。

## 3.12 不等号条件里的等值查询
```SQL
begin;
select * from t where id>9 and id<12 order by id desc for update;
```
 1. 这里由于是 order by 语句，优化器会先找到第一个比 12 小的值。在索引树搜索过程后，其实要找到 id=12 的值，但没有找到，向右遍历找到 id=15，所以加锁 (10,15].
 2. 但由于第一次查找是等值查找（在索引树上搜索），根据优化2，变为间隙锁 (10,15).
 3. 然后向左遍历，变为范围查询，找到 id=5 这行，加 (0,5] 的 next-key.

## 3.13 等值查询in
```SQL
begin;
select id from t where c in(5,20,10) lock in share mode;

mysql> explain select id from t where c in (5,20,10) lock in share mode\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: t
   partitions: NULL
         type: range
possible_keys: c
          key: c
      key_len: 5
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)

ERROR:
No query specified
```
rows=3 并且使用索引 c，说明三个值都是通过 B+ 树搜索定位的。
 1. 先查找 c=5，锁住 (0,5]. 由于 c 不是唯一索引，向右遍历到 c=10，开始是等值查找，加 (5,10).
 2. 查找 c=15，锁住 (10,15], 再加 (15,20).
 3. 最后查找 c=20，锁住 (15,20]. 再加 (20,25).
可见，**在 MySQL 中，锁是一个个逐步加的。**

假设还有一个这样的语句：
`select id from t where c in(5,20,10) order by c desc for update;`
由于是 order by c desc，虽然这里的加锁范围没有变，但是加锁的顺序发生了改变，会按照 c=20，c=10，c=5. 的顺序加锁。虽然说间隙锁本身并不冲突，但记录锁却会。这样如果是两个语句并发的情况，就可能发生死锁，第一个语句拥有了 c5 的行锁，请求c=10 的行锁。当第二个语句，拥有了 c=10 的行锁，请求 c=5 的行锁。

## 3.14 GAP动态锁
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| select * from t where id>10 and id<=15 for update;  |   |
|   | delete from t where id=10;  |
|   | 成功  |
|   | insert into t values(10,10,10);    |
|   |  被阻塞   |
这里 insert 被阻塞，就是因为间隙锁是个动态的概念，Session B 在删除 id=10 的记录后，Session A 持有的间隙变大。

对于 Session A 原来持有，(10,15] 和 (15,20] 的 next-key 锁。 Session B 删除 id=10 的记录后，(10,15] 变成了 (5,15] 的间隙。所以之后就插入不回去了。
## 3.15 update GAP动态锁
| Session A  | Session B  |
| ------------ | ------------ |
| begin;  |   |
| select * from t where id> 5 lock in share mode;  |   |
|   | update t set c=1 where c = 5; |
|   | 成功  |
|   | update t set c=5 where c = 1;  |
|   |  被阻塞   |
Session A 加锁：(5,10], (10,15], (15,20], (20,25], (25,supermum].
> c>5 第一个找到的是 c=10，并且是范围查找，没有优化原则。

Session B 的 update 可以拆成两步：
 1. 插入 (c=1,id=5).
 2. 删除 (c=5,id=5).
> 或者理解成，加(0.5] next-key 和 (5,10) 的间隙锁，但间隙锁不冲突。

修改后 Session A 的锁变为;
(c=1, 10], (10,15], (15,20], (20,25], (25,supermum].
接下来：update t set c=1 where c = 1
 1. 插入 (c=5,id=5).
 2. 删除 (c=1,id=5).
第一步插入意向锁和间隙锁冲突。