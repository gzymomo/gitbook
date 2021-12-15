## MySQL主从搭建教程

GTID可以在整个复制生命周期中唯一标识一个操作。它的出现为主从复制切换提供了极大的便利，我们熟知的MGR就基于GTID。

## GTID的基本概念

### GTID的作用

GTID的全称为Global Transaction Identifier，是MySQL  的一个强大的特性。MySQL会为每一个DML/DDL操作都增加一个唯一标记，叫作GTID。这个标记在整个复制环境中都是唯一的。主从环境中主库的DUMP线程可以直接通过GTID定位到需要发送的binary log位置，而不再需要指定binary log的文件名和位置，因而切换极为方便。

### GTID的基本表示

为了严谨，笔者尽量使用源码的术语解释，后面也会沿用这些术语。

- GTID：单个GTID，比如24985463-a536-11e8-a30c-5254008138e4:5。对应源码中的类结构Gtid。注意源码中用sid代表GTID前面的server_uuid，gno则用来表示GTID后面的序号。
- gno：单个GTID后面的序号，比如上面的GTID的gno就是5。这个gno实际上是从一个全局计数器next_free_gno中获取的。
- GTID  SET：一个GTID的集合，可以包含多个server_uuid，比如常见的gtid_executed变量，gtid_purged变量就是一个GTID SET。类似的，24985463-a536-11e8-a30c-5254008138e4:1- 5:7-10就是一个GTID  SET，对应源码中的类结构Gtid_set，其中还包含一个sid_map，用于表示多个server_uuid。
- GTID SET Interval：代表GTID SET中的一个区间，GTID  SET中的某个server_uuid可能包含多个区间，例如，1-5:7-10中就有2个GTID SET  Interval，分别是1-5和7-10，对应源码中的结构体Gtid_set::Interval。

### server_uuid的生成

在GTID中包含了一个server_uuid。server_uuid实际上是一个32字节+1（/0）字节的字符串。MySQL启动时会调用init_server_auto_options函数读取auto.cnf文件。如果auto.cnf文件丢失，则会调用generate_server_uuid函数生成一个新的server_uuid，但是需要注意，这样GTID必然会发生改变。

在generate_server_uuid函数中可以看到，server_uuid至少和下面3部分有关。

（1）数据库的启动时间。（2）线程的LWP ID，其中，LWP是轻量级进程（Light-Weight Process）的简称，我们在5.1节会进行描述。（3）一个随机的内存地址。

下面是部分代码：

```sql
  const time_t save_server_start_time= server_start_time; //获取MySQL启动时间
  server_start_time+= ((ulonglong)current_pid << 48) + current_pid;//加入Lwp号运算
  thd->status_var.bytes_sent= (ulonglong)thd;//这是一个内存指针，即线程结构体的内存地址
  lex_start(thd);
  func_uuid= new (thd->mem_root) Item_func_uuid();
  func_uuid->fixed= 1;
  func_uuid->val_str(&uuid); //这个函数是具体的运算过程
```

server_uuid的内部表示是binary_log::Uuid，核心是一个16字节的内存空间，在GTID相关的Event中会包含这个信息，2.3节会进行详细解析。

## GTID的生成

在发起commit命令后，当order commit执行到FLUSH阶段，需要生成GTID  Event时，会获取GTID。MySQL内部维护了一个全局的GTID计数器next_free_gno，用于生成gno。可以参考Gtid_state::get_automatic_gno函数，部分代码如下。

```sql
1、定义：
Gtid next_candidate = {sidno,sidno == get_server_sidno() ? next_free_gno:1};
2、赋值：
while (true)
  {
    const Gtid_set::Interval *iv= ivit.get(); 
//定义Interval指针，通过迭代器ivit来迭代每个Interval
    rpl_gno next_interval_start= iv != NULL ? iv->start : MAX_GNO; 
//一般情况下不会为NULL，因此 next_interval_start 等于第一个interval
//的start，当然如果为NULL，则说明Interval->next = NULL，表示
//没有区间了，那么这个时候取next_interval_start为MAX_GNO，
//此时条件next_candidate.gno < next_interval_start必然成立                                              
    while (next_candidate.gno < next_interval_start &&
           DBUG_EVALUATE_IF("simulate_gno_exhausted", false, true)) 
    {
      if (owned_gtids.get_owner(next_candidate) == 0)
        DBUG_RETURN(next_candidate.gno);
     //返回gno，那么GTID就生成了
      next_candidate.gno++;//如果本GTID已经被其他线程占用
//则next_candidate.gno自增后继续判断
    }
  ......
  }
```

## GTID_EVENT和PREVIOUS_GTIDS_LOG_EVENT简介

这里先解释一下它们的作用，因为后面会用到。

### 1．GTID_EVENT

GTID_EVENT作为DML/DDL的第一个Event，用于描述这个操作的GTID是多少。在MySQL 5.7中，为了支持从库基于LOGICAL_CLOCK的并行回放，封装了last commit和seq  number两个值，可以称其为逻辑时钟。在MySQL  5.7中，即便不开启GTID也会包含一个匿名的ANONYMOUS_GTID_EVENT，但是其中不会携带GTID信息，只包含last  commit和seq number两个值。

### 2．PREVIOUS_GTIDS_LOG_EVENT

PREVIOUS_GTIDS_LOG_EVENT包含在每一个binary log的开头，用于描述直到上一个binary log所包含的全部GTID（包括已经删除的binary log）。在MySQL  5.7中，即便不开启GTID，也会包含这个PREVIOUS_GTIDS_LOG_EVENT，实际上这一点意义是非常大的。简单地说，它为快速扫描binary log获得正确的gtid_executed变量提供了基础，否则可能扫描大量的binary  log才能得到正确的gtid_executed变量（比如MySQL 5.6中关闭GTID的情况）。这一点将在1.3节详细描述。

### gtid_executed表的作用

官方文档这样描述gtid_executed表：

```sql
Beginning with MySQL 5.7.5, GTIDs are stored in a table named gtid_executed, in the  mysql database. A row in this table contains, for each GTID or set of GTIDs that it represents, the UUID of the originating server, and the starting and ending transaction IDs of the set; for a row referencing only a single GTID, these last two values are the same.
```

也就是说，gtid_executed表是GTID持久化的一个介质。实例重启后所有的内存信息都会丢失，GTID模块初始化需要读取GTID持久化介质。

可以发现，gtid_executed表是InnoDB表，建表语句如下，并且可以手动更改它，但是除非是测试，否则千万不要修改它。

```sql
Table: gtid_executed
Create Table: CREATE TABLE `gtid_executed` (
  `source_uuid` char(36) NOT NULL COMMENT 'uuid of the source where the transaction was originally executed.',
  `interval_start` bigint(20) NOT NULL COMMENT 'First number of interval.',
  `interval_end` bigint(20) NOT NULL COMMENT 'Last number of interval.',
  PRIMARY KEY (`source_uuid`,`interval_start`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 STATS_PERSISTENT=0
```

除了gtid_executed表，还有一个GTID持久化的介质，那就是binary log中的GTID_EVENT。

既然有了 binary log 中的GTID_EVENT 进行 GTID 的持久化，为什么还需要gtid_executed表呢？笔者认为，这是MySQL 5.7.5之后的一个优化，可以反过来思考，在MySQL 5.6 中，如果使用 GTID 做从库，那么从库必须开启 binary  log，并且设置参数 log_slave_ updates=ture，因为从库执行过的GTID操作都需要保留在binary  log中，所以当GTID模块初始化的时候会读取它获取正确的GTID SET。接下来，看一段MySQL 5.6官方文档对于搭建GTID从库的说明。

```sql
Step 3: Restart both servers with GTIDs enabled. To enable binary logging with globaltransaction identifiers, each server must be started with GTID mode, binary logging, slave update logging enabled, and with statements that are unsafe for GTID-based replication disabled. In addition,you should prevent unwanted or accidental updates from being performed on either server by starting both in read-only mode. This means that both servers must be started with (at least) the options shown in the following invocation of mysqld_safe:
shell> mysqld_safe --gtid_mode=ON --log-bin --log-slave-updates --enforce-gtid-consistency &
```

然而，开启binary  log的同时设置参数log_slave_updates=ture必然会造成一个问题。很多时候，从库是不需要做级联的，设置参数log_slave_updates=ture会造成额外的空间和性能开销。因此需要另外一种 GTID 持久化介质，而并不是 binary log 中的  GTID_EVENT，gtid_executed表正是这样一种GTID持久化的介质。