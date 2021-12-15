## Hudi介绍

### 概述

Apache Hudi基于Hadoop兼容的存储，提供了以下流处理原语。

- Update/Delete Record
- Change Streams

也就是，可以将HDFS和Hudi结合起来，提供对流处理的支持能力。例如：支持记录级别的更新、删除，以及获取基于HDFS之上的Change Streams。哪些数据发生了变更。

### 架构图

传统的批处理（例如：T+1），需要更长时间，才能看到数据的更新。而Hudi将流处理引入到大数据中，在更短地时间内提供新的数据，比传统批处理效率高几个数量级。

![img](https://cdn.nlark.com/yuque/0/2021/png/1311515/1625100643995-72f9d8b5-4282-4a4b-9a0f-26a80c91e9f1.png)

1. 数据库可以通过工具将数据实时同步到Kafka、或者使用Sqoop批量导出的方式导出到DFS。
2. 基于Flink、Spark或者DeltaStreamer可以将这些数据导入到基于DFS或者Cloud Storage构建Hudi Data Lake中。

1. 通过Hudi提供的Spark DataSource，可以将Kafka、DFS等未加工的表处理为增量的ETL表
2. Spark/Flink/Presto/Hive/Impala等可以直接查询Hudi中的表

## 核心概念

### Timeline

Hudi的核心是维护一个timeline，在不同时刻对某个Hudi表的操作都会记录在Timeline中，或者这样说：

- Hudi的timeline是由一个个的Hudi Instant组成。

相当于提供了对该表的一个即时视图。通过这个timeline，我们可以按照数据的到达顺序进行检索。

![img](https://cdn.nlark.com/yuque/0/2021/png/1311515/1625100680569-4a5225f4-7446-40e3-b60f-d4b3b8274be8.png)

如上图所示，Hudi Instant由以下几个组件组成：

- Instant Action
  记录在表上的一系列操作
- Instant Time
  按Action的开始时间单调递增，通常是一个时间戳，例如：20210318143839

- state
  当前instant的状态

Hudi可以保证基于Timeline的操作是具备原子性的，而且Timeline和Instant是一致的。

#### Instant Action

Instant重要的Action有以下几个：

- **COMMITS**
  表示将一批原子写入提交到一个表中。
- **CLEANS**
  后台Action，它会删除表中比较旧的、不再需要的版本。

- **DELTA_COMMIT**
  增量提交，表示将一批原子写入到MOR（Merge On Read）类型的表中，数据可以只写入到Delta Log（增量日志中）。
- **COMPACTION**
  后台Action，它用于确保一致的数据结构，例如：将基于行的Detla Log文件中的更新合并到列式文件结构。

- **ROLLBACK**
  表示COMMIT或者DELTA COMMIT不成功，并且已经回滚，删除在写入过程中产生的所有文件。
- **SAVEPOINT**
  将某些文件标记为"saved"，这样清理程序就不会清除它们。在出现故障或者进行数据恢复时，可以用于在某个时间点进行还原。

- **REPLACE**

  使用FileGroup替换另一些FileGroup，使用InsertOverwrite或者Clustering    时对应REPLACE。

#### Instant State

- **REQUESTED**
  表示已经被准备调度执行，但还未启动
- **INFLIGHT**
  表示正在执行操作

- **COMPLETED**
  表示在timeline已经完成Action

#### Hudi底层数据存储

| ![img](https://mmbiz.qpic.cn/sz_mmbiz_png/frAIQPLLOycoDia9uFYqIpcEzCiaIMZbvY6mASArWZh3iaRsThf9yG8QBcCNErhjayA5ZEsPibAFDfMmJ5meSvp9sA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) |
| ------------------------------------------------------------ |
| 在每个Hudi的表中，都有一个.hoodie文件夹，里面包含了每一个Hudi Instant操作信息。 |

可以看到，每个Instant都是一个文件，文件名以InstantTime、Action以及State组成。例如：针对 20210318222704这个Hudi Instant，我们看到了有3个相关文件。

```
[hive@ha-node1 logs]$ hdfs dfs -ls /hudi/hudi_t_user/.hoodie | grep 20210318222704
-rw-r--r--   3 hdfs hadoop       1569 2021-03-18 22:27 /hudi/hudi_t_user/.hoodie/20210318222704.commit
-rw-r--r--   3 hdfs hadoop          0 2021-03-18 22:27 /hudi/hudi_t_user/.hoodie/20210318222704.commit.requested
-rw-r--r--   3 hdfs hadoop       1701 2021-03-18 22:27 /hudi/hudi_t_user/.hoodie/20210318222704.inflight
```

文件：**20210318222704.commit.requested**

```
[hive@ha-node1 logs]$ hdfs dfs -tail /hudi/hudi_t_user/.hoodie/20210318222704.commit.requested
[hive@ha-node1 logs]$
```

该文件是一个空的文件，它表示当前阶段是请求调度执行阶段，并没有启动。

文件：**20210318222704.inflight**

```
{
  "partitionToWriteStats" : {
    "dt=2021/03/18" : [ {
      "fileId" : "",
      "path" : null,
      "prevCommit" : "null",
      "numWrites" : 0,
      "numDeletes" : 0,
      "numUpdateWrites" : 0,
      "numInserts" : 0,
      "totalWriteBytes" : 0,
      "totalWriteErrors" : 0,
      "tempPath" : null,
      "partitionPath" : null,
      "totalLogRecords" : 0,
      "totalLogFilesCompacted" : 0,
      "totalLogSizeCompacted" : 0,
      "totalUpdatedRecordsCompacted" : 0,
      "totalLogBlocks" : 0,
      "totalCorruptLogBlock" : 0,
      "totalRollbackBlocks" : 0,
      "fileSizeInBytes" : 0
    }, {
      "fileId" : "8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0",
      "path" : null,
      "prevCommit" : "20210318222701",
      "numWrites" : 0,
      "numDeletes" : 0,
      "numUpdateWrites" : 1,
      "numInserts" : 0,
      "totalWriteBytes" : 0,
      "totalWriteErrors" : 0,
      "tempPath" : null,
      "partitionPath" : null,
      "totalLogRecords" : 0,
      "totalLogFilesCompacted" : 0,
      "totalLogSizeCompacted" : 0,
      "totalUpdatedRecordsCompacted" : 0,
      "totalLogBlocks" : 0,
      "totalCorruptLogBlock" : 0,
      "totalRollbackBlocks" : 0,
      "fileSizeInBytes" : 0
    } ]
  },
  "compacted" : false,
  "extraMetadata" : { },
  "operationType" : "UPSERT",
  "totalCompactedRecordsUpdated" : 0,
  "totalCreateTime" : 0,
  "totalLogRecordsCompacted" : 0,
  "fileIdAndRelativePaths" : {
    "" : null,
    "8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0" : null
  },
  "totalUpsertTime" : 0,
  "totalLogFilesSize" : 0,
  "totalLogFilesCompacted" : 0,
  "totalRecordsDeleted" : 0,
  "totalScanTime" : 0
}
```

可以看到，在commit.inflight表示Hudi Instant正在提交运行。它记录了要写入的分析状态、执行的操作类型等。

文件：**20210318222704.commit**

```
{
  "partitionToWriteStats" : {
    "dt=2021/03/18" : [ {
      "fileId" : "8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0",
      "path" : "dt=2021/03/18/8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0_0-113-112_20210318222704.parquet",
      "prevCommit" : "20210318222701",
      "numWrites" : 6,
      "numDeletes" : 0,
      "numUpdateWrites" : 1,
      "numInserts" : 0,
      "totalWriteBytes" : 434393,
      "totalWriteErrors" : 0,
      "tempPath" : null,
      "partitionPath" : "dt=2021/03/18",
      "totalLogRecords" : 0,
      "totalLogFilesCompacted" : 0,
      "totalLogSizeCompacted" : 0,
      "totalUpdatedRecordsCompacted" : 0,
      "totalLogBlocks" : 0,
      "totalCorruptLogBlock" : 0,
      "totalRollbackBlocks" : 0,
      "fileSizeInBytes" : 434393
    } ]
  },
  "compacted" : false,
  "extraMetadata" : {
    "schema" : "{\"type\":\"record\",\"name\":\"hudi_t_user_record\",\"namespace\":\"hoodie.hudi_t_user\",\"fields\":[{\"name\":\"id\",\"type\":[\"string\",\"null\"]},{\"name\":\"name\",\"type\":[\"string\",\"null\"]},{\"name\":\"dt\",\"type\":[\"string\",\"null\"]}]}"
  },
  "operationType" : "UPSERT",
  "totalCompactedRecordsUpdated" : 0,
  "totalCreateTime" : 0,
  "totalLogRecordsCompacted" : 0,
  "fileIdAndRelativePaths" : {
    "8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0" : "dt=2021/03/18/8ba8507d-b021-4d85-b54a-5b87ed1fd90c-0_0-113-112_20210318222704.parquet"
  },
  "totalUpsertTime" : 165,
  "totalLogFilesSize" : 0,
  "totalLogFilesCompacted" : 0,
  "totalRecordsDeleted" : 0,
  "totalScanTime" : 0
}
```

commit文件表示已提交的Hudi Instant。它记录了本地提交的具体信息，例如：总共写入的字节数量、分区的路径、对应的parquet数据文件、更新写入的数据条数、以及当前提交的Hudi表schema信息、Upsert所消耗的时间等等。

#### 乱序数据处理

这是官方的一张图。展示了Hudi会以何种策略处理延迟的数据。

图中展示了，时间轴上10:00 - 10:20之间在某个Hudi表中发生的事件。可以看到每隔5分钟会执行一次Action（COMMIT、CLEANING、COMPACTION...），这些Action都会保留在Hudi的Timeline上。

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681288-b1917679-7760-48d5-8248-463ad7b3bdaf.gif)image-20210318150956941

而时间轴之上，有些数据是延迟的，例如：原本是9点到达的数据，但却延迟了1个多小时才到达。所以，数据的实际到达事件，和实际发生事件是不一样的。

Hudi是这样处理的：

- 延迟到达的数据，Upsert操作将新的数据生成到之前的时间段（文件夹）中。
- 获取自10:00以来的数据，可以将所有的新增的数据查询出来，而并不需要扫描整个大于7点timeline上的所有数据

### 文件布局

- 目录结构
  Hudi将表以DFS的目录结构组织，表可以分为若干个分区，分区就是包含数据文件的文件夹。这和Hive非常类似。
- 分区
  每个分区都由相对与basepath唯一标识。在分区内，文件以文件组方式组织，由文件ID标识。每个文件组包含了若干个分片。每个分片包含了Hudi Instant生成的基本文件（*.parquet），以及包含对文件进行插入、更新的一组日志文件（*.log）

Hudi采用MVCC设计，压缩操作会将日志和基本文件合并，形成新的分片，清理操作就是将未使用的、旧的分片删除，以回收DFS的空间。

### 索引

要提供高效的upserts操作，那就必须能够快速定位记录在文件中的位置。Hudi通过索引机制，将给定的Hoodie  key（记录的key +  分区路径）映射到一个文件ID，一旦将record的第一个版本写入到文件，这个映射关系将永远不不再改变。映射文件组包含了文件组中所有记录的ID映射。

### 表类型与查询

Hudi中表的索引、文件结构、流式原语、时间轴上的操作都是由**表类型**决定的（如何写入数据）。而**查询类型**表示了如何把数据提供给查询（如何读取数据）。

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681443-29092e38-0007-4b73-a03c-d530a52e8af5.gif)image-20210318152428277

可以看到，COW类型的表支持快照查询、以及增量查询。而MOR表支持快照查询、增量查询、读优化查询。

#### 表类型

Hudi中支持两种类型的表，一种是COW，另外一种是MOR。要区分它们很容易，COW是不带日志的、而MOR是带日志的。

- **COW（Copy-On-Write）**只使用一种文件格式存储数据，例如：parquet。写入的时候就会执行数据的合并，更新版本、重写文件。
- **MOR（Merge-On-Read）**
  结合列式（Parquet）和行式（Avro）两种格式来存储数据。更新记录到增量文件中，然后执行COMPACTION生成列式文件。

以下是这两种类型的对比：

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681771-1a35802e-3e6c-421d-ab37-d73e210c0773.gif)image-20210318153020053

可以看到：COW表的写放大问题严重，而MOR提供了低延迟、更高效地实时写入，但读取的时候需要更高的延迟。

在Hudi表目录的.hoodie文件夹中，有一个hoodie.properties，里面记录了Hudi表的属性。例如：

```
hoodie.table.name=hudi_t_user
hoodie.archivelog.folder=archived
hoodie.table.type=COPY_ON_WRITE
hoodie.table.version=1
hoodie.timeline.layout.version=1
```

可以看到，当前Hudi的表名、archivelog目录位置、表的版本、表的类型为COW表、timeline的文件结构版本为1。

#### 查询类型

- **Snapshot Queries**
  快照查询能够查询到表的最新快照数据。如果是MOR类型表，会将基本文件和增量文件合并后，再提供数据。（可实现分钟级）；对于COW类型表，它会现有的表先进行drop、再进行replace，并提供了Upsert/Delete等功能。
- **Incremental Queries**
  增量查询只能查看到写入表的新数据。这种查询能够有效地应对Change Stream，方便对接流式处理引擎。

- **Read Optimized Queries**
  读优化查询可以查询到表的最新快照数据，它仅查询最新的基本列文件，可以保证列查询性能。这种方式保证了性能，但数据可能会有延迟。

### COW类型表详解

前面说过了，COW类型表只包含列式基本文件，没有行式日志文件。每次提交都会生成新的基本列式文件。这种方式写放大影响较大，但读放大为0。这种方式，对于读取分析非常频繁的场景很重要。

```
[hive@ha-node1 logs]$ hdfs dfs -ls /hudi/hudi_t_user_cow/dt=2021/03/19
/hudi/hudi_t_user_cow/dt=2021/03/19/3abde036-8c0f-4bed-89f1-872422264a21-0_0-128-121_20210319175250.parquet
/hudi/hudi_t_user_cow/dt=2021/03/19/3abde036-8c0f-4bed-89f1-872422264a21-0_0-38-45_20210319175212.parquet
/hudi/hudi_t_user_cow/dt=2021/03/19/3abde036-8c0f-4bed-89f1-872422264a21-0_0-68-71_20210319175217.parquet
/hudi/hudi_t_user_cow/dt=2021/03/19/3abde036-8c0f-4bed-89f1-872422264a21-0_0-98-95_20210319175247.parquet
```

可以看到，在Hudi分区中，只有parquet文件。

以下是它的工作原理：

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681777-689984e9-0b90-4d41-a3a1-aea974deeb25.gif)

针对Update操作：会在对应的文件组上生成一个带有提交时间的新的slice。

针对Insert操作：会分配一个新的文件组，并生成第一个slice。

而针对该表的查询，例如：SELECT COUNT(*)，Hudi会检查时间轴上最新的提交，过滤出来每个文件组上的最新slice，查询仅仅会查询出来已经提交的数据。（标记为绿色）。

COW类型表的目的在于从根本上改变对表的管理方式。

1. 它可以实现文件级别的数据自动更新，而无需重新整个表或者分区
2. 能够实现更小消耗的增量更新，而无需扫描整个表或者分区

1. 严格控制文件大小，并保证更高的查询性能（小文件过多会严重降低查询性能）

### MOR类型表详解

MOR类型表是COW类型表更高级的实现，其实，对应到源码中，它是COW表的子类。

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| HoodieSparkCopyOnWriteTable > HoodieSparkMergeOnReadTable    |

其实它还是提供最新的基本列文件给外部查询。此外，它会将Upsert的操作存储在基于行的增量日志存储中，通过这样方式，MOR表可以用Delta Log来实现快照查询。

大家可以看一下面的测试：

```
[hive@ha-node1 logs]$ hdfs dfs -ls /hudi/hudi_t_user_mor/dt=2021/03/19
/hudi/hudi_t_user_mor/dt=2021/03/19/.9a423733-573f-4df3-9374-e3fe05cfbf0f-0_20210319173324.log.1_0-68-71
/hudi/hudi_t_user_mor/dt=2021/03/19/.9a423733-573f-4df3-9374-e3fe05cfbf0f-0_20210319173400.log.1_0-142-142
/hudi/hudi_t_user_mor/dt=2021/03/19/9a423733-573f-4df3-9374-e3fe05cfbf0f-0_0-117-116_20210319173400.parquet
/hudi/hudi_t_user_mor/dt=2021/03/19/9a423733-573f-4df3-9374-e3fe05cfbf0f-0_0-38-45_20210319173324.parquet
```

可以看到，在分区文件夹中，MOR表有parquet文件，也有log文件。**其中，每一次新增数据，会产生parquet文件，而执行更新时，会写入到log文件中。**

这种类型的表，可以智能地平衡读放大、和写放大，提供近实时的数据。针对MOR类型的表，COMPACTION过程显得很重要，它需要选择Delta Log中哪些数据要合并到基础列式文件中，并保证查询性能。

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681789-a9b9bae7-e831-46e7-8ccf-6c4a5653f83a.gif)

从上图可以看到，MOR类型的表，可以做到每1分钟提交一次，这是COW类型的表无法做到的。而每一个文件组中，都有一个增量日志文件，它包含了对表的更新记录。

MOR类型表支持两种类型的查询：

- 读优化查询
- 快照查询

具体使用哪种，取决于我们是选择追求查询性能、还是数据新鲜度。

## 流实时摄取

### Frog造数程序

#### 实体类

```
@Data
public class User {
    // 用户ID
    @JSONField(ordinal = 1)
    private Integer id;
    // 生日
    @JSONField(ordinal = 2, format="yyyy-MM-dd HH:mm:ss")
    private Date birthday;
    // 姓名
    @JSONField(ordinal = 3)
    private String name;
    // 创建日期
    @JSONField(ordinal = 4, format="yyyy-MM-dd HH:mm:ss")
    private Date createTime;
    // 职位
    @JSONField(ordinal = 5)
    private String position;
    public User() {
    }
    public User(Integer id, String name, Date birthday, String position) {
        this.id = id;
        this.birthday = birthday;
        this.name = name;
        this.position = position;
        this.createTime = new Date();
    }
    @Override
    public String toString() {
        return JSON.toJSONString(this);
    }
}
```

#### Frog造数接口

```
/**
 * 实体（数据）生成器
 */
public interface Frog<T> {
    T getOne();
}
```

#### UserFrog造数器

```
public class UserFrog implements Frog<User> {
    private Logger logger;
    private Random r;
    private List<Date> DATE_CACHE;
    private final Integer DATE_MAX_NUM = 100000;             // 日期数量
    private final Integer MAX_DELAY_MILLS = 60 * 1000;       // 最大延迟毫秒数
    private final Integer DELAY_USER_INTERVAL = 10;          // N个User中就会有一个延迟
    private final List<String> NAME_CACHE;                   // 姓名缓存
    private final List<String> POSITION_CACHE;               // 岗位缓存
    private Long genIndex = 0L;                              // 本次实例生成用户个数
    private UserFrog() {
        r = new Random();
        synchronized (UserFrog.class) {
            logger = LogManager.getLogger(UserFrog.class);
            logger.debug("从names.txt中加载姓名缓存...");
            // 加载姓名缓存
            NAME_CACHE = loadResourceFileAsList("names.txt");
            logger.debug(String.format("已加载 %d 个姓名.", NAME_CACHE.size()));
            logger.debug("从positions.txt加载岗位职位缓存...");
            // 加载职位缓存
            POSITION_CACHE = loadResourceFileAsList("positions.txt");
            logger.debug(String.format("已加载 %d 个岗位.", POSITION_CACHE.size()));
            logger.debug("自动生成日期缓存...");
            // 加载日期缓存
            initDateCache();
            logger.debug(String.format("已加载 %d 个日期", DATE_CACHE.size()));
        }
    }
    public static UserFrog build() {
        return new UserFrog();
    }
    /**
     * 加载socket_datagen目录中的资源文件到列表中
     * @param fileName
     */
    private List<String> loadResourceFileAsList(String fileName) {
        try(InputStream resourceAsStream =
                    User.class.getClassLoader().getResourceAsStream("socket_datagen/" + fileName);
            InputStreamReader inputStreamReader =
                    new InputStreamReader(resourceAsStream);
            BufferedReader br = new BufferedReader(inputStreamReader)) {
            return br.lines()
                    .collect(Collectors.toList());
        } catch (IOException e) {
            logger.fatal(e);
        }
        return ListUtils.EMPTY_LIST;
    }
    private void initDateCache() {
        // 生成出生年月缓存
        final Calendar instance = Calendar.getInstance();
        instance.set(Calendar.YEAR, 1970);
        instance.set(Calendar.MONTH, 1);
        instance.set(Calendar.DAY_OF_MONTH, 1);
        Long startTimestamp = instance.getTimeInMillis();
        instance.set(Calendar.YEAR, 2021);
        instance.set(Calendar.MONTH, 3);
        Long endTimestamp = instance.getTimeInMillis();
        DATE_CACHE = LongStream.range(0, DATE_MAX_NUM)
                .map(n -> RandomUtils.nextLong(startTimestamp, endTimestamp))
                .mapToObj(t -> new Date(t))
                .collect(Collectors.toList());
    }
    public User getOne() {
        int userId = r.nextInt(Integer.MAX_VALUE);
        Date birthday = DATE_CACHE.get(r.nextInt(DATE_CACHE.size()));
        String name = NAME_CACHE.get(r.nextInt(NAME_CACHE.size()));
        String position = POSITION_CACHE.get(r.nextInt(POSITION_CACHE.size()));
        final User user = new User(userId, name, birthday, position);
        if(genIndex % DELAY_USER_INTERVAL == 0) {
            final int delayMills = r.nextInt(MAX_DELAY_MILLS);
            logger.debug(String.format("生成延迟数据 - User ID=%d, 延迟: %.1f 秒", userId, delayMills / 1000.0));
            user.setCreateTime(new Date(new Date().getTime() - delayMills));
        }
        ++genIndex;
        return user;
    }
    public static void main(String[] args) {
        final UserFrog userBuilder = UserFrog.build();
        IntStream.range(0, 1000)
            .forEach(n -> {
                System.out.println(userBuilder.getOne().toString());
            });
    }
}
```

#### Socket造数程序

```
/**
 * Socket方式的数据生成器
 */
public class SocketProducer implements Runnable {
    private static Logger logger = LogManager.getLogger(SocketProducer.class);
    private static Short LSTN_PORT = 9999;
    private static Short PRODUCE_INTEVAL = 1;           // 生成消息的时间间隔
    private static Short MAX_CONNECTIONS = 10;          // 最大10个连接
    private static Frog userFrog = UserFrog.build();
    private ServerSocket server;
    public SocketProducer(ServerSocket srv) {
        this.server = srv;
    }
    @Override
    public void run() {
        // 数据总条数、耗时
        long totalNum = 0;
        long startMillseconds = 0;
        try {
            final Socket client = server.accept();
            String clientInfo = String.format("主机名:%s, IP:%s"
                    , client.getInetAddress().getHostName()
                    , client.getInetAddress().getHostAddress());
            // 设置线程元数据
            Thread.currentThread().setName(clientInfo);
            logger.info(String.format("客户端[%s]已经连接到服务器.", clientInfo));
            OutputStream outputStream = client.getOutputStream();
            OutputStreamWriter outputStreamWriter = new OutputStreamWriter(outputStream);
            BufferedWriter writer = new BufferedWriter(outputStreamWriter);
            startMillseconds = System.currentTimeMillis();
            // 发送消息
            while(true) {
                String msg = userFrog.getOne().toString();
                logger.debug(msg);
                writer.write(msg);
                writer.newLine();
                writer.flush();
                totalNum++;
                try {
                    TimeUnit.MILLISECONDS.sleep(PRODUCE_INTEVAL);
                } catch (InterruptedException e) {
                    logger.warn(e);
                    logger.warn(String.format("客户端[%s]断开", clientInfo));
                    break;
                }
            }
        } catch (IOException e) {
            logger.fatal(e);
        }
        if(startMillseconds == -1) {
            logger.warn("统计耗时失败！客户端连接异常断开！");
        }
        else {
            long endMillseconds = System.currentTimeMillis();
            // 耗时
            double elapsedInSeconds = (endMillseconds - startMillseconds) / 1000.0;
            double rate = totalNum / elapsedInSeconds;
            logger.info(String.format("共计生成数据：%d条, 耗时：%.1f秒, 速率：%.1f条/s"
                    , totalNum
                    , elapsedInSeconds
                    , rate));
        }
    }
    public static void main(String[] args) {
        try {
            logger.debug(String.format("Socket服务器配置:\n"
                            + "-----------------------\n"
                            + "监听端口号:%d \n"
                            + "生成消息事件间隔: %d(毫秒)\n"
                            + "最大连接数: %d \n"
                            + "-----------------------"
                                , LSTN_PORT
                                , PRODUCE_INTEVAL
                                , MAX_CONNECTIONS));
            ServerSocket serverSocket = new ServerSocket(LSTN_PORT);
            logger.info(String.format("启动服务器，监听端口: %d", LSTN_PORT));
            ExecutorService executorService = Executors.newFixedThreadPool(MAX_CONNECTIONS);
            IntStream.range(0, MAX_CONNECTIONS)
                .forEach(n -> {
                    executorService.submit(new SocketProducer(serverSocket));
                });
            while(true) {
                if(executorService.isShutdown() || executorService.isTerminated()) {
                    IntStream.range(0, MAX_CONNECTIONS)
                        .forEach(n -> {
                            executorService.submit(new SocketProducer(serverSocket));
                        });
                }
            }
        } catch (IOException e) {
            logger.fatal(e);
        }
    }
}
```

### Structured Streaming

#### 构建开发环境

导入Spark、Hudi依赖。

```
<properties>
  <spark.scope>compile</spark.scope>
  <maven.compiler.source>8</maven.compiler.source>
  <maven.compiler.target>8</maven.compiler.target>
  <scala.version>2.12</scala.version>
  <spark.version>3.1.1</spark.version>
  <commons-cli.version>1.4</commons-cli.version>
  <maven-scala-plugin.version>2.15.2</maven-scala-plugin.version>
  <maven-shade-plugin.version>2.3</maven-shade-plugin.version>
  <commons-lang3.version>3.8.1</commons-lang3.version>
  <junit.version>4.12</junit.version>
  <hudi.version>0.7.0</hudi.version>
  <build-helper-maven-plugin.version>1.8</build-helper-maven-plugin.version>
  <hive.version>2.3.8</hive.version>
  <hadoop.version>3.2.1</hadoop.version>
  <guava.version>24.0-jre</guava.version>
</properties>
<dependencies>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>${commons-lang3.version}</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
  </dependency>
  <dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>${commons-cli.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.hudi</groupId>
    <artifactId>hudi-spark-bundle_${scala.version}</artifactId>
    <version>${hudi.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-avro_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-common</artifactId>
    <version>${hive.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>${hive.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>${hadoop.version}</version>
  </dependency>
  <dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>${guava.version}</version>
  </dependency>
</dependencies>
```

#### 构建Strcutured Streaming入口

```
def sparkSession(appName: String) = {
  val conf = new SparkConf()
  conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
  conf.set("spark.sql.streaming.checkpointLocation", "/meta/streaming_checkpoint")
  SparkSession
  .builder
  .appName(appName)
  .master("local[*]")
  .config(conf)
  .getOrCreate()
}
```

#### 实体类

```
case class User(id :Integer
                , birthday : String
                , name :String
                , createTime :String
                , position :String)
object User {
    def apply(json: String) = {
        val jsonObject = JSON.parseObject(json)
        val id = jsonObject.getInteger("id")
        val birthday = jsonObject.getString("birthday");
        val name = jsonObject.getString("name")
        val createTime = jsonObject.getString("createTime");
        val position = jsonObject.getString("position")
        new User(id
            , birthday
            , name
            , createTime
            , position)
    }
}
```

#### Hudi分区提取器

```
/**
 * 从Hudi分区路径中获取分区字段
 */
public class DayPartitionValueExtractor implements PartitionValueExtractor {
    @Override
    public List<String> extractPartitionValuesInPath(String partitionPath) {
        final String[] dateField = partitionPath.split("-");
        if(dateField != null && dateField.length >= 3) {
            return Collections.singletonList(IntStream.range(0, 3)
                    .mapToObj(idx -> dateField[idx])
                    .collect(Collectors.joining("-")));
        }
        return ListUtils.EMPTY_LIST;
    }
}
```

#### Hudi数据Upsert

```
/**
 * Hudi小文件测试
 */
object SmallFilesTestApp {
    def main(args: Array[String]): Unit = {
        // 加载Spark SQL上下文
        val spark = SparkEnv.sparkSession("Streaming Hudi API MOR Test")
        // 设置日志级别
        spark.sparkContext.setLogLevel("WARN")
        import spark.implicits._
        val userStrDataFrame = spark.readStream
            .format("socket")
            .option("host", "localhost")
            .option("port", 9999)
            .option("includeTimestamp", false)
            .option("numPartitions", 5)
            .load()
        // 解析JSON串
        val userDF = userStrDataFrame.as[String]
            .map(User(_))
            // 添加分区字段
            .withColumn("dt", $"createTime".substr(0, 10))
         userDF.writeStream
             .format("console")
             .outputMode("append")
             .option("truncate", false)
             .option("numRows", 100)
             .start()
        // 写入到Hudi
        val hudiTableName = "hudi_t_user_mor";
        val hiveDatabaseName = "hudi_datalake"
        val hiveTableName = "hudi_ods_user_mor"
        userDF.writeStream
            .outputMode(OutputMode.Append())
            .format("hudi")
            .option("hoodie.table.name", hudiTableName)
            .option("hoodie.bootstrap.base.path", "/hudi")
            .option("hoodie.datasource.write.table.name", hudiTableName)
            .option("hoodie.datasource.write.operation", "upsert")
            .option("hoodie.datasource.write.table.type", "MERGE_ON_READ")
            .option("hoodie.datasource.write.precombine.field", "dt")
            .option("hoodie.datasource.write.recordkey.field", "id")
            .option("hoodie.datasource.write.partitionpath.field", "dt")
            .option("hoodie.datasource.write.hive_style_partitioning", false)
            .option("hoodie.datasource.hive_sync.enable", true)
            .option("hoodie.datasource.hive_sync.database", hiveDatabaseName)
            .option("hoodie.datasource.hive_sync.table", hiveTableName)
            .option("hoodie.datasource.hive_sync.username", "hive")
            .option("hoodie.datasource.hive_sync.password", "hive")
            .option("hoodie.datasource.hive_sync.jdbcurl", "jdbc:hive2://ha-node1:10000")
            .option("hoodie.datasource.hive_sync.partition_extractor_class", "cn.pin.streaming.tools.DayPartitionValueExtractor")
            .option("hoodie.datasource.hive_sync.partition_fields", "dt")
            .option("hoodie.datasource.hive_sync.use_jdbc", true)
            .option("hoodie.datasource.hive_sync.auto_create_database", true)
            .option("hoodie.datasource.hive_sync.skip_ro_suffix", false)
            .option("hoodie.datasource.hive_sync.support_timestamp", false)
            .option("hoodie.bootstrap.parallelism", 5)
            .option("hoodie.bulkinsert.shuffle.parallelism", 5)
            .option("hoodie.insert.shuffle.parallelism", 5)
            .option("hoodie.delete.shuffle.parallelism", 5)
            .option("hoodie.upsert.shuffle.parallelism", 5)
            .start(s"/hudi/${hudiTableName}")
            .awaitTermination()
    }
}
```

#### 运行

1. 启动HDFS集群
2. 启动Hive MetaStore和HiveServer2

1. 启动造数程序

### 湖仓一体（Hudi + Hive）

#### COW表

Structured Streaming运行时，会自动在Hive中创建外部表。

```
+-----------------------+
|       tab_name        |
+-----------------------+
| hudi_ods_user_cow     |
+-----------------------+
```

##### 查看表结构

查看cow表信息。

```
0: jdbc:hive2://ha-node1:10000> desc hudi_ods_user_cow;
+--------------------------+------------+----------+
|         col_name         | data_type  | comment  |
+--------------------------+------------+----------+
| _hoodie_commit_time      | string     |          |
| _hoodie_commit_seqno     | string     |          |
| _hoodie_record_key       | string     |          |
| _hoodie_partition_path   | string     |          |
| _hoodie_file_name        | string     |          |
| id                       | string     |          |
| name                     | string     |          |
| dt                       | string     |          |
|                          | NULL       | NULL     |
| # Partition Information  | NULL       | NULL     |
| # col_name               | data_type  | comment  |
| dt                       | string     |          |
+--------------------------+------------+----------+
12 rows selected (0.336 seconds)
```

Hive中的表中包含了id、name以及分区字段dt。除此之外，还有hudi的相关列。

```
_hoodie_commit_time   # 提交的时间（对应Hudi Instant）
_hoodie_commit_seqno   # 提交的序号
_hoodie_record_key   # 主键
_hoodie_partition_path  # Hudi分区的路径
_hoodie_file_name    # 记录对应的文件名
```

##### 查看建表信息

```
0: jdbc:hive2://ha-node1:10000> show create table hudi_ods_user_cow;
+----------------------------------------------------+
|                   createtab_stmt                   |
+----------------------------------------------------+
| CREATE EXTERNAL TABLE `hudi_ods_user_cow`(         |
|   `_hoodie_commit_time` string,                    |
|   `_hoodie_commit_seqno` string,                   |
|   `_hoodie_record_key` string,                     |
|   `_hoodie_partition_path` string,                 |
|   `_hoodie_file_name` string,                      |
|   `id` string,                                     |
|   `name` string)                                   |
| PARTITIONED BY (                                   |
|   `dt` string)                                     |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe'  |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hudi.hadoop.HoodieParquetInputFormat'  |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat' |
| LOCATION                                           |
|   'hdfs://hadoop-ha:8020/hudi/hudi_t_user_cow'     |
| TBLPROPERTIES (                                    |
|   'bucketing_version'='2',                         |
|   'last_commit_time_sync'='20210319175250',        |
|   'transient_lastDdlTime'='1616147536')            |
+----------------------------------------------------+
```

可以看到，Hive中对应的是一张外部表、使用HoodieParquetInputFormat方式来存储表的。

#### MOR表

##### 查看表结构

Structured Streaming在运行时，MOR类型表会自动创建两个表：

```
+-----------------------+
|       tab_name        |
+-----------------------+
| hudi_ods_user_mor_ro  |
| hudi_ods_user_mor_rt  |
+-----------------------+
```

分别以ro、和rt结尾。从Hive的schema来看，两个表的结构和COW表一模一样，没有任何区别。

##### 查看建表信息

但通过show create table查看建表语句，发现这两个表的INPUTFORMAT是不一样的：

ro表使用的是：

```
org.apache.hudi.hadoop.HoodieParquetInputFormat
```

这与COW表使用的是相同InputFormat。ro对应的是：Read Optimized（读优化），这种方式只会查询出来parquet数据文件中的内容。

而rt表使用是：

```
org.apache.hudi.hadoop.realtime.HoodieParquetRealtimeInputFormat
```

这种方式是能够实时读出来写入的数据，也就是Merge On Write，会将基于Parquet的基础列式文件、和基于行的Avro日志文件合并在一起呈现给用户。

例如：

查询ro表。

```
+-----+---------+-------------+
| id  |  name   |     dt      |
+-----+---------+-------------+
| 3   | book    | 2021-03-19  |
| 4   | wuwu    | 2021-03-19  |
| 5   | build5  | 2021-03-19  | # 其实数据已经被更新为build6
| 2   | spark   | 2021-03-19  |
| 6   | keep    | 2021-03-19  |
| 1   | hadoop  | 2021-03-19  |
+-----+---------+-------------+
2 rows selected (0.294 seconds)
```

这种方式的读取不会导致读放大，直接将所有parquet文件读取出来。但如果期间数据有更新，这种方式是查询不到的。

查询rt表。

```
+-----+---------+-------------+
| id  |  name   |     dt      |
+-----+---------+-------------+
| 3   | book    | 2021-03-19  |
| 4   | wuwu    | 2021-03-19  |
| 5   | build6  | 2021-03-19  |
| 2   | spark   | 2021-03-19  |
| 6   | keep    | 2021-03-19  |
| 1   | hadoop  | 2021-03-19  |
+-----+---------+-------------+
6 rows selected (0.244 seconds)
```

rt表总是能够查询出来最新的数据，但它会导致读放大。因为它会将parquet文件和log文件合并后再展示出来。虽然保证了数据的新鲜度，但性能是有所下降的。

## Hive查询

```
set hive.fetch.task.conversion=more;
```

### 表映射

Hudi整合了Hive后，会自动在Hive中创建表。

### 分区

在每个Hudi的分区目录中，都有一个.hoodie_partition_metadata文件，该文件与分区相关的元数据。

```
commitTime=20210318211853
partitionDepth=3
```

可以看到它表明该分区是在2021年3月18日21点18分提交的，并且分区的深度为3。

## Spark SQL查询

### 配置Hive

- 下载hudi-hadoop-mr-bundle-0.7.0.jar包
- 放置到/opt/hive/lib文件夹中

- 放置到 /opt/hadoop/share/hadoop/hdfs目录中，并分发。

```
scp hudi-hadoop-mr-bundle-0.7.0.jar ha-node2:/opt/hadoop/share/hadoop/hdfs; \
scp hudi-hadoop-mr-bundle-0.7.0.jar ha-node3:/opt/hadoop/share/hadoop/hdfs; \
scp hudi-hadoop-mr-bundle-0.7.0.jar ha-node4:/opt/hadoop/share/hadoop/hdfs; \
scp hudi-hadoop-mr-bundle-0.7.0.jar ha-node5:/opt/hadoop/share/hadoop/hdfs
```

在hive-site.xml中添加

```
<property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
</property>
```

重启Hadoop、重启Hive

### 配置spark sql

- 下载hudi-hadoop-mr-bundle-0.7.0.jar包
- 放置到/opt/spark/jar文件夹中

- 设置set spark.sql.hive.convertMetastoreParquet=false; 强制使用Hive SerDe来读取数据，但执行计划仍然使用的是Spark引擎。

### 查询Hudi表数据

```
select * from hudi_ods_user;
```

### Thrift Server数据无法更新同步问题

需要手动执行：

```
refresh table hudi_datalake.hudi_ods_user;
```

源码地址为：

https://github.com/apache/hudi/blob/master/hudi-hadoop-mr/src/main/java/org/apache/hudi/hadoop/HoodieParquetInputFormat.java

## 小文件测试

在本地做了一个简单的测试。

数据：

```
{"id":1011050465,"birthday":"1985-11-08 19:30:25","name":"西门璇子","createTime":"2021-03-23 18:08:28","position":"生产或工厂工程师"}
共计生成数据：474041, 耗时：1041.7秒
```

Meta文件数量：

```
190
```

数据文件数量：

```
14
```

## Strcutured Streaming MOR写入执行计划与源码

### Job Web UI

进入到Spark的Web UI中，可以看到，Structured Streaming生成了很多的Job。我们来看看这些是什么样的JOB。

![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681791-905671f6-1d68-4cc9-b9ef-c2db9e1edbd1.gif)image-20210323182655317

为了方便Job容易被观察，我为每一个Stream Query设置一个容易识别的名称。也推荐大家在生产上给每一个Query设置容易识别的名称。

```
userDF.writeStream
 .queryName("Hudi增量输出同步Hive元数据")
userDF.writeStream
 .queryName("Socket明细数据控制台输出")
```

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 带上Job名称，一眼就能识别出来哪些Job是我们的、哪些是Hudi生成的。 |

### Structured Streaming业务作业

可以看到，描述信息为：id  = 4a776304-0711-4fb7-8dbb-95195836e024 runId =  cb3da71f-c078-48dd-8b05-81060ea7c4db batch =  249的Job是一个读取Stream源的Job。它只有一个Stage，分别是加载数据源、执行Map算子、并输出。

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |

我们可以来看看它的物理执行计划：

```
== Physical Plan ==
WriteToDataSourceV2 (7)
+- * Project (6)
   +- * SerializeFromObject (5)
      +- * MapElements (4)
         +- * DeserializeToObject (3)
            +- * Project (2)
               +- MicroBatchScan (1)
# 从Socket中读取数据源（微批方式读取数据）
(1) MicroBatchScan
Output [1]: [value#0]
class org.apache.spark.sql.execution.streaming.sources.TextSocketTable$$anon$1
# 执行列选择操作
(2) Project [codegen id : 1]
Output [1]: [value#0]
Input [1]: [value#0]
# 执行反序列化（将接受到的数据调用toString，转换为String)
(3) DeserializeToObject [codegen id : 1]
Input [1]: [value#0]
Arguments: value#0.toString, obj#10: java.lang.String
# 执行Map操作，将String转换为User对象
(4) MapElements [codegen id : 1]
Input [1]: [obj#10]
Arguments: cn.pin.streaming.app.SmallFilesTestApp$$$Lambda$1147/1076250141@292ff26f, obj#11: cn.pin.streaming.entity.User
# 将对象序列化（Spark SQL自己序列化，放入内存）
(5) SerializeFromObject [codegen id : 1]
Input [1]: [obj#11]
Arguments: [knownnotnull(assertnotnull(input[0, cn.pin.streaming.entity.User, true])).id.intValue AS id#12, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, cn.pin.streaming.entity.User, true])).birthday, true, false) AS birthday#13, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, cn.pin.streaming.entity.User, true])).name, true, false) AS name#14, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, cn.pin.streaming.entity.User, true])).createTime, true, false) AS createTime#15, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input[0, cn.pin.streaming.entity.User, true])).position, true, false) AS position#16]
# 选择4个列执行查询
(6) Project [codegen id : 1]
Output [6]: [id#12, birthday#13, name#14, createTime#15, position#16, substring(createTime#15, 0, 10) AS dt#23]
Input [5]: [id#12, birthday#13, name#14, createTime#15, position#16]
# 执行写入操作
(7) WriteToDataSourceV2
Input [6]: [id#12, birthday#13, name#14, createTime#15, position#16, dt#23]
Arguments: org.apache.spark.sql.execution.streaming.sources.MicroBatchWrite@ff75118
```

这个执行计划，可以看到，针对Structured Streaming的代码，将数据读取再到打印输出。这个Job是与Hudi无关的。

对应的代码是：

```
val userStrDataFrame = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .option("includeTimestamp", false)
  .option("numPartitions", 5)
  .load()
// 解析JSON串
val userDF = userStrDataFrame.as[String]
  .map(User(_))
  // 添加分区字段
  .withColumn("dt", $"createTime".substr(0, 10))
userDF.writeStream
  .format("console")
  .outputMode("append")
  .option("truncate", false)
  .option("numRows", 100)
  .start()
```

### Hudi Load latest base files from all partitions作业

与Hudi有关的作业，都是在writeStream.format("hudi").start代码生成的。

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 所有与Hudi相关的Job都在第74行生成的Job。                     |

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 从所有的分区加载最新的Hudi基本数据文件。                     |
| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |

以下是源码，我已经加上了注释：

```
public static List<Pair<String, HoodieBaseFile>> getLatestBaseFilesForAllPartitions(final List<String> partitions,
                                                                                      final HoodieEngineContext context,
                                                                                      final HoodieTable hoodieTable) {
    context.setJobStatus(HoodieIndexUtils.class.getSimpleName(), "Load latest base files from all partitions");
    return context.flatMap(partitions, partitionPath -> {
      // 获取对应Hoodie表提交的时间线，并过滤出来所有已完成的Instant，然后取最后一个
      Option<HoodieInstant> latestCommitTime = hoodieTable.getMetaClient().getCommitsTimeline()
          .filterCompletedInstants().lastInstant();
      List<Pair<String, HoodieBaseFile>> filteredFiles = new ArrayList<>();
      if (latestCommitTime.isPresent()) {
        // 获取最近一次commit之前的所有基础文件
        filteredFiles = hoodieTable.getBaseFileOnlyView()
            .getLatestBaseFilesBeforeOrOn(partitionPath, latestCommitTime.get().getTimestamp())
            .map(f -> Pair.of(partitionPath, f))
            .collect(toList());
      }
      return filteredFiles.stream();
    }, Math.max(partitions.size(), 1));
  }
```

### Hudi Obtain key ranges for file slices (range pruning=on)作业

| ![img](https://cdn.nlark.com/yuque/0/2021/gif/1311515/1625100681829-3b465167-1c2e-4b05-9278-d1684e992e83.gif) |
| ------------------------------------------------------------ |
| 为每一个文件片，获取key的范围                                |

代码如下：

```
List<Tuple2<String, BloomIndexFileInfo>> loadInvolvedFiles(List<String> partitions, final HoodieEngineContext context,
                                                             final HoodieTable hoodieTable) {
    // 获取所有分区对应的最新基础数据文件
    List<Pair<String, String>> partitionPathFileIDList = getLatestBaseFilesForAllPartitions(partitions, context, hoodieTable).stream()
        .map(pair -> Pair.of(pair.getKey(), pair.getValue().getFileId()))
        .collect(toList());
    // 判断hoodie.bloom.index.prune.by.ranges是否开启，默认是开启的
    if (config.getBloomIndexPruneByRanges()) {
      // also obtain file ranges, if range pruning is enabled
      context.setJobStatus(this.getClass().getName(), "Obtain key ranges for file slices (range pruning=on)");
      // 对所有最新数据文件执行map操作
      return context.map(partitionPathFileIDList, pf -> {
        try {
          // 从基础文件中读取出对应的最大key、和最小的key
          HoodieRangeInfoHandle rangeInfoHandle = new HoodieRangeInfoHandle(config, hoodieTable, pf);
          String[] minMaxKeys = rangeInfoHandle.getMinMaxKeys();
          // 为每个文件创建布隆过滤器文件
          return new Tuple2<>(pf.getKey(), new BloomIndexFileInfo(pf.getValue(), minMaxKeys[0], minMaxKeys[1]));
        } catch (MetadataNotFoundException me) {
          LOG.warn("Unable to find range metadata in file :" + pf);
          return new Tuple2<>(pf.getKey(), new BloomIndexFileInfo(pf.getValue()));
        }
      }, Math.max(partitionPathFileIDList.size(), 1));
    } else {
      return partitionPathFileIDList.stream()
          .map(pf -> new Tuple2<>(pf.getKey(), new BloomIndexFileInfo(pf.getValue()))).collect(toList());
    }
  }
```

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 这项配置仅对BLOOM类型的索引有效。                            |

Hudi还告诉我们，如果key值是随机的，应该把该配置关闭。如果密钥是单调递增的，例如：时间戳，这个就比较有帮助了。借助BloomFilter，可以快速判断某个key是否在一个文件中。

### Hudi Compute all comparisons needed between records and files作业

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 统计每个BloomFilter文件进行key过滤所需的计算量               |

源代码如下：

```
/**
   * Compute the estimated number of bloom filter comparisons to be performed on each file group.
   * 计算在每个文件组需要执行BloomFilter的计算量（按照基础文件的每个HoodieKey来计算）
   * 因为每个HFile的min key和max key是不一样的，所以要评估出来针对每个BloomFilter文件对应的key有多少
   */
  private Map<String, Long> computeComparisonsPerFileGroup(final Map<String, Long> recordsPerPartition,
                                                           final Map<String, List<BloomIndexFileInfo>> partitionToFileInfo,
                                                           final JavaRDD<Tuple2<String, HoodieKey>> fileComparisonsRDD,
                                                           final HoodieEngineContext context) {
    Map<String, Long> fileToComparisons;
    if (config.getBloomIndexPruneByRanges()) {
      // we will just try exploding the input and then count to determine comparisons
      // FIX(vc): Only do sampling here and extrapolate?
      context.setJobStatus(this.getClass().getSimpleName(), "Compute all comparisons needed between records and files");
      fileToComparisons = fileComparisonsRDD.mapToPair(t -> t).countByKey();
    } else {
      fileToComparisons = new HashMap<>();
      partitionToFileInfo.forEach((key, value) -> {
        for (BloomIndexFileInfo fileInfo : value) {
          // each file needs to be compared against all the records coming into the partition
          fileToComparisons.put(fileInfo.getFileId(), recordsPerPartition.get(key));
        }
      });
    }
    return fileToComparisons;
  }
```

### Hudi Building workload profile作业

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 构建workload profile文件                                     |

其实在这个Job中，Structured   Streaming已经开始更新Instant、写入数据了，如果配置了自动提交会直接更新索引、提交数据。创建索引会参考Index相关配置，Hudi中可以使用HBase索引或者默认存储在parquet中的布隆过滤器作为索引。索引的类型有以下几种：

- BLOOM FILTER
- HBASE Index

- SIMPLE INDEX（缓存在Spark Cache中）

代码如下：

```
/**
   * 执行HoodieRecord记录写入、并更新索引，自动提交（默认）
   * @param inputRecordsRDD
   * @return
   */
  @Override
  public HoodieWriteMetadata<JavaRDD<WriteStatus>> execute(JavaRDD<HoodieRecord<T>> inputRecordsRDD) {
    HoodieWriteMetadata<JavaRDD<WriteStatus>> result = new HoodieWriteMetadata<>();
    // Cache the tagged records, so we don't end up computing both
    // TODO: Consistent contract in HoodieWriteClient regarding preppedRecord storage level handling
    if (inputRecordsRDD.getStorageLevel() == StorageLevel.NONE()) {
      inputRecordsRDD.persist(StorageLevel.MEMORY_AND_DISK_SER());
    } else {
      LOG.info("RDD PreppedRecords was persisted at: " + inputRecordsRDD.getStorageLevel());
    }
    WorkloadProfile profile = null;
    if (isWorkloadProfileNeeded()) {
      context.setJobStatus(this.getClass().getSimpleName(), "Building workload profile");
      profile = new WorkloadProfile(buildProfile(inputRecordsRDD));
      LOG.info("Workload profile :" + profile);
      // 将workload的信息写入到metadatabase中，此处写入的是inflight文件
      saveWorkloadProfileMetadataToInflight(profile, instantTime);
    }
    // handle records update with clustering
    // 小文件聚类后是否更新文件组
    JavaRDD<HoodieRecord<T>> inputRecordsRDDWithClusteringUpdate = clusteringHandleUpdate(inputRecordsRDD);
    // partition using the insert partitioner
    // 根据要更新的记录、元数据设置分区
    final Partitioner partitioner = getPartitioner(profile);
    JavaRDD<HoodieRecord<T>> partitionedRecords = partition(inputRecordsRDDWithClusteringUpdate, partitioner);
    // 生成待更新的RDD
    JavaRDD<WriteStatus> writeStatusRDD = partitionedRecords.mapPartitionsWithIndex((partition, recordItr) -> {
      if (WriteOperationType.isChangingRecords(operationType)) {
        // 执行Upsert分区操作（写入数据）
        return handleUpsertPartition(instantTime, partition, recordItr, partitioner);
      } else {
        // 执行Insert分区写入
        return handleInsertPartition(instantTime, partition, recordItr, partitioner);
      }
    }, true).flatMap(List::iterator);
    // 如果设置了hoodie.auto.commit（默认为true），会更新索引并提交
    updateIndexAndCommitIfNeeded(writeStatusRDD, result);
    return result;
  }
```

### Hudi Getting Small files from partition作业

|                    |
| ------------------ |
| 获取分区中的小文件 |
|                    |

源代码如下：

```
@Override
  protected List<SmallFile> getSmallFiles(String partitionPath) {
    // smallFiles only for partitionPath
    List<SmallFile> smallFileLocations = new ArrayList<>();
    // Init here since this class (and member variables) might not have been initialized
    HoodieTimeline commitTimeline = table.getCompletedCommitsTimeline();
    // Find out all eligible small file slices
    if (!commitTimeline.empty()) {
      HoodieInstant latestCommitTime = commitTimeline.lastInstant().get();
      // find smallest file in partition and append to it
      List<FileSlice> allSmallFileSlices = new ArrayList<>();
      // If we cannot index log files, then we choose the smallest parquet file in the partition and add inserts to
      // it. Doing this overtime for a partition, we ensure that we handle small file issues
      // 如果找不到log文件，那么会找到符合配置的所有parquet小文件
      if (!table.getIndex().canIndexLogFiles()) {
        // TODO : choose last N small files since there can be multiple small files written to a single partition
        // by different spark partitions in a single batch
        Option<FileSlice> smallFileSlice = Option.fromJavaOptional(table.getSliceView()
            .getLatestFileSlicesBeforeOrOn(partitionPath, latestCommitTime.getTimestamp(), false)
            .filter(
                // 根据文件大小、配置获取小文件
                fileSlice -> fileSlice.getLogFiles().count() < 1 && fileSlice.getBaseFile().get().getFileSize() < config
                    .getParquetSmallFileLimit())
            .min((FileSlice left, FileSlice right) ->
                left.getBaseFile().get().getFileSize() < right.getBaseFile().get().getFileSize() ? -1 : 1));
        if (smallFileSlice.isPresent()) {
          allSmallFileSlices.add(smallFileSlice.get());
        }
      } else {
        // If we can index log files, we can add more inserts to log files for fileIds including those under
        // pending compaction.
        // 如果找到log文件，那么就会把更多的inserts操作添加到log文件中，这种是针对MOR表
        List<FileSlice> allFileSlices =
            table.getSliceView().getLatestFileSlicesBeforeOrOn(partitionPath, latestCommitTime.getTimestamp(), true)
                .collect(Collectors.toList());
        for (FileSlice fileSlice : allFileSlices) {
          if (isSmallFile(fileSlice)) {
            allSmallFileSlices.add(fileSlice);
          }
        }
      }
      // Create SmallFiles from the eligible file slices
      // 为每一个文件片（包含文件和log）生成位置、大小信息
      for (FileSlice smallFileSlice : allSmallFileSlices) {
        SmallFile sf = new SmallFile();
        if (smallFileSlice.getBaseFile().isPresent()) {
          // TODO : Move logic of file name, file id, base commit time handling inside file slice
          String filename = smallFileSlice.getBaseFile().get().getFileName();
          sf.location = new HoodieRecordLocation(FSUtils.getCommitTime(filename), FSUtils.getFileId(filename));
          sf.sizeBytes = getTotalFileSize(smallFileSlice);
          smallFileLocations.add(sf);
        } else {
          HoodieLogFile logFile = smallFileSlice.getLogFiles().findFirst().get();
          sf.location = new HoodieRecordLocation(FSUtils.getBaseCommitTimeFromLogPath(logFile.getPath()),
              FSUtils.getFileIdFromLogPath(logFile.getPath()));
          sf.sizeBytes = getTotalFileSize(smallFileSlice);
          smallFileLocations.add(sf);
        }
      }
    }
    return smallFileLocations;
  }
```

可以看到，Hudi会合并parquet基本文件和log文件。因为parquet和log文件的合并方式是不一样的。

配置hoodie.parquet.small.file.limit，可以指定小文件的阈值。默认小于100MB认为就是小文件，需要进行合并。

### Hudi Obtaining marker files for all created, merged paths作业

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 获取要创建、合并的路径标记的文件                             |

源码如下：

```
public Set<String> createdAndMergedDataPaths(HoodieEngineContext context, int parallelism) throws IOException {
    Set<String> dataFiles = new HashSet<>();
    // 获取标记的DFS目录路径
    FileStatus[] topLevelStatuses = fs.listStatus(markerDirPath);
    List<String> subDirectories = new ArrayList<>();
    for (FileStatus topLevelStatus: topLevelStatuses) {
      if (topLevelStatus.isFile()) {
        String pathStr = topLevelStatus.getPath().toString();
        // 获取带合并的数据文件
        if (pathStr.contains(HoodieTableMetaClient.MARKER_EXTN) && !pathStr.endsWith(IOType.APPEND.name())) {
          dataFiles.add(translateMarkerToDataPath(pathStr));
        }
      } else {
        // 获取子目录
        subDirectories.add(topLevelStatus.getPath().toString());
      }
    }
    if (subDirectories.size() > 0) {
      // 获取子目录的数量设置并行度
      parallelism = Math.min(subDirectories.size(), parallelism);
      // 获取序列化配置
      SerializableConfiguration serializedConf = new SerializableConfiguration(fs.getConf());
      // 设置作业信息
      context.setJobStatus(this.getClass().getSimpleName(), "Obtaining marker files for all created, merged paths");
      // 将子目录中的带合并的文件添加到数据文件列表
      dataFiles.addAll(context.flatMap(subDirectories, directory -> {
        Path path = new Path(directory);
        FileSystem fileSystem = path.getFileSystem(serializedConf.get());
        RemoteIterator<LocatedFileStatus> itr = fileSystem.listFiles(path, true);
        List<String> result = new ArrayList<>();
        while (itr.hasNext()) {
          FileStatus status = itr.next();
          String pathStr = status.getPath().toString();
          // 过滤出来后缀为.marker、以及部位APPEND的名字
          if (pathStr.contains(HoodieTableMetaClient.MARKER_EXTN) && !pathStr.endsWith(IOType.APPEND.name())) {
            result.add(translateMarkerToDataPath(pathStr));
          }
        }
        return result.stream();
      }, parallelism));
    }
    return dataFiles;
  }
```

大概意思就是，获取到所有的.marker且不以APPEND结尾的数据文件。

### Hudi Generates list of file slices to be cleaned作业

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 生成清理文件分片的列表                                       |

源码如下：

```
/**
   * Generates List of files to be cleaned.
   * 生成要清理的文件列表
   *
   * @param context HoodieEngineContext
   * @return Cleaner Plan
   */
  HoodieCleanerPlan requestClean(HoodieEngineContext context) {
    try {
      CleanPlanner<T, I, K, O> planner = new CleanPlanner<>(context, table, config);
      // 获取比hoodie.cleaner.commits.retaine（默认为：24）更早的HoodieInstant，默认为24次提交
      Option<HoodieInstant> earliestInstant = planner.getEarliestCommitToRetain();
      // 根据清理策略hoodie.cleaner.policy（默认为：KEEP_LATEST_COMMITS，保持最后的提交）获取要清理的分区
      List<String> partitionsToClean = planner.getPartitionPathsToClean(earliestInstant);
      if (partitionsToClean.isEmpty()) {
        LOG.info("Nothing to clean here.");
        return HoodieCleanerPlan.newBuilder().setPolicy(HoodieCleaningPolicy.KEEP_LATEST_COMMITS.name()).build();
      }
      // 根据配置或者分区数量设置并行度
      LOG.info("Total Partitions to clean : " + partitionsToClean.size() + ", with policy " + config.getCleanerPolicy());
      int cleanerParallelism = Math.min(partitionsToClean.size(), config.getCleanerParallelism());
      LOG.info("Using cleanerParallelism: " + cleanerParallelism);
      context.setJobStatus(this.getClass().getSimpleName(), "Generates list of file slices to be cleaned");
      Map<String, List<HoodieCleanFileInfo>> cleanOps = context
          .map(partitionsToClean, partitionPathToClean -> Pair.of(partitionPathToClean, planner.getDeletePaths(partitionPathToClean)), cleanerParallelism)
          .stream()
          .collect(Collectors.toMap(Pair::getKey, y -> CleanerUtils.convertToHoodieCleanFileInfoList(y.getValue())));
      // 生成清理计划
      return new HoodieCleanerPlan(earliestInstant
          .map(x -> new HoodieActionInstant(x.getTimestamp(), x.getAction(), x.getState().name())).orElse(null),
          config.getCleanerPolicy().name(), CollectionUtils.createImmutableMap(),
          CleanPlanner.LATEST_CLEAN_PLAN_VERSION, cleanOps);
    } catch (IOException e) {
      throw new HoodieIOException("Failed to schedule clean operation", e);
    }
  }
```

根据清理策略清理超过提交次数的HUDI INSTANT。

### Hudi Perform cleaning of partitions作业

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 执行清理。                                                   |

源码如下：

```
/**
   * 执行过期的commit清理
   * @param context
   * @param cleanerPlan
   * @return
   */
  @Override
  List<HoodieCleanStat> clean(HoodieEngineContext context, HoodieCleanerPlan cleanerPlan) {
    JavaSparkContext jsc = HoodieSparkEngineContext.getSparkContext(context);
    // 获取每个分区下待清理的文件
    int cleanerParallelism = Math.min(
        (int) (cleanerPlan.getFilePathsToBeDeletedPerPartition().values().stream().mapToInt(List::size).count()),
        config.getCleanerParallelism());
    LOG.info("Using cleanerParallelism: " + cleanerParallelism);
    // 设置job状态
    context.setJobStatus(this.getClass().getSimpleName(), "Perform cleaning of partitions");
    // 转换路径与要清理的文件信息
    List<Tuple2<String, PartitionCleanStat>> partitionCleanStats = jsc
        .parallelize(cleanerPlan.getFilePathsToBeDeletedPerPartition().entrySet().stream()
            .flatMap(x -> x.getValue().stream().map(y -> new Tuple2<>(x.getKey(),
                new CleanFileInfo(y.getFilePath(), y.getIsBootstrapBaseFile()))))
            .collect(Collectors.toList()), cleanerParallelism)
        .mapPartitionsToPair(deleteFilesFunc(table))
        .reduceByKey(PartitionCleanStat::merge).collect();
    Map<String, PartitionCleanStat> partitionCleanStatsMap = partitionCleanStats.stream()
        .collect(Collectors.toMap(Tuple2::_1, Tuple2::_2));
    // Return PartitionCleanStat for each partition passed.
    // 执行清理，并返回清理状态
    return cleanerPlan.getFilePathsToBeDeletedPerPartition().keySet().stream().map(partitionPath -> {
      PartitionCleanStat partitionCleanStat = partitionCleanStatsMap.containsKey(partitionPath)
          ? partitionCleanStatsMap.get(partitionPath)
          : new PartitionCleanStat(partitionPath);
      HoodieActionInstant actionInstant = cleanerPlan.getEarliestInstantToRetain();
      return HoodieCleanStat.newBuilder().withPolicy(config.getCleanerPolicy()).withPartitionPath(partitionPath)
          .withEarliestCommitRetained(Option.ofNullable(
              actionInstant != null
                  ? new HoodieInstant(HoodieInstant.State.valueOf(actionInstant.getState()),
                  actionInstant.getAction(), actionInstant.getTimestamp())
                  : null))
          .withDeletePathPattern(partitionCleanStat.deletePathPatterns())
          .withSuccessfulDeletes(partitionCleanStat.successDeleteFiles())
          .withFailedDeletes(partitionCleanStat.failedDeleteFiles())
          .withDeleteBootstrapBasePathPatterns(partitionCleanStat.getDeleteBootstrapBasePathPatterns())
          .withSuccessfulDeleteBootstrapBaseFiles(partitionCleanStat.getSuccessfulDeleteBootstrapBaseFiles())
          .withFailedDeleteBootstrapBaseFiles(partitionCleanStat.getFailedDeleteBootstrapBaseFiles())
          .build();
    }).collect(Collectors.toList());
  }
```

### Hudi Looking for files to compact作业

该作业是由format类型为hudi的writeStream触发。

| ![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) |
| ------------------------------------------------------------ |
| 查询MOR要合并的文件                                          |

源码如下：

```
/**
   * 生成MOR合并计划（Parquet与Avro log合并）
   * @param context
   * @param hoodieTable
   * @param config
   * @param compactionCommitTime
   * @param fgIdsInPendingCompactionAndClustering
   * @return
   * @throws IOException
   */
  @Override
  public HoodieCompactionPlan generateCompactionPlan(HoodieEngineContext context,
                                                     HoodieTable<T, JavaRDD<HoodieRecord<T>>, JavaRDD<HoodieKey>, JavaRDD<WriteStatus>> hoodieTable,
                                                     HoodieWriteConfig config, String compactionCommitTime,
                                                     Set<HoodieFileGroupId> fgIdsInPendingCompactionAndClustering)
      throws IOException {
    JavaSparkContext jsc = HoodieSparkEngineContext.getSparkContext(context);
    // 注册日志文件、parquet文件计数器
    totalLogFiles = new LongAccumulator();
    totalFileSlices = new LongAccumulator();
    jsc.sc().register(totalLogFiles);
    jsc.sc().register(totalFileSlices);
    // 校验是否为MOR类型表
    ValidationUtils.checkArgument(hoodieTable.getMetaClient().getTableType() == HoodieTableType.MERGE_ON_READ,
        "Can only compact table of type " + HoodieTableType.MERGE_ON_READ + " and not "
            + hoodieTable.getMetaClient().getTableType().name());
    // TODO : check if maxMemory is not greater than JVM or spark.executor memory
    // TODO - rollback any compactions in flight
    HoodieTableMetaClient metaClient = hoodieTable.getMetaClient();
    LOG.info("Compacting " + metaClient.getBasePath() + " with commit " + compactionCommitTime);
    List<String> partitionPaths = FSUtils.getAllPartitionPaths(context, config.getMetadataConfig(), metaClient.getBasePath());
    // filter the partition paths if needed to reduce list status
    // 获取合并策略hoodie.compaction.strategy（默认为：LogFileSizeBasedCompactionStrategy）
    partitionPaths = config.getCompactionStrategy().filterPartitionPaths(config, partitionPaths);
    if (partitionPaths.isEmpty()) {
      // In case no partitions could be picked, return no compaction plan
      return null;
    }
    SliceView fileSystemView = hoodieTable.getSliceView();
    LOG.info("Compaction looking for files to compact in " + partitionPaths + " partitions");
    context.setJobStatus(this.getClass().getSimpleName(), "Looking for files to compact");
    List<HoodieCompactionOperation> operations = context.flatMap(partitionPaths, partitionPath -> {
      return fileSystemView
          // 获取分区最新的文件分片
          .getLatestFileSlices(partitionPath)
          // 过滤掉等待合并或者小文件聚类的文件组
          .filter(slice -> !fgIdsInPendingCompactionAndClustering.contains(slice.getFileGroupId()))
          .map(s -> {
            // 计数要合并的日志文件
            List<HoodieLogFile> logFiles =
                s.getLogFiles().sorted(HoodieLogFile.getLogFileComparator()).collect(Collectors.toList());
            totalLogFiles.add((long) logFiles.size());
            totalFileSlices.add(1L);
            // Avro generated classes are not inheriting Serializable. Using CompactionOperation POJO
            // for spark Map operations and collecting them finally in Avro generated classes for storing
            // into meta files.
            // 获取分片对应的Base文件（parquet文件）
            // 创建一个Base文件与log文件合并任务操作
            Option<HoodieBaseFile> dataFile = s.getBaseFile();
            return new CompactionOperation(dataFile, partitionPath, logFiles,
                config.getCompactionStrategy().captureMetrics(config, s));
          })
          .filter(c -> !c.getDeltaFileNames().isEmpty());
          // 构建操作(设置必要参数)
    }, partitionPaths.size()).stream().map(CompactionUtils::buildHoodieCompactionOperation).collect(toList());
    LOG.info("Total of " + operations.size() + " compactions are retrieved");
    LOG.info("Total number of latest files slices " + totalFileSlices.value());
    LOG.info("Total number of log files " + totalLogFiles.value());
    LOG.info("Total number of file slices " + totalFileSlices.value());
    // Filter the compactions with the passed in filter. This lets us choose most effective
    // compactions only
    // 创建计划
    HoodieCompactionPlan compactionPlan = config.getCompactionStrategy().generateCompactionPlan(config, operations,
        CompactionUtils.getAllPendingCompactionPlans(metaClient).stream().map(Pair::getValue).collect(toList()));
    ValidationUtils.checkArgument(
        compactionPlan.getOperations().stream().noneMatch(
            op -> fgIdsInPendingCompactionAndClustering.contains(new HoodieFileGroupId(op.getPartitionPath(), op.getFileId()))),
        "Bad Compaction Plan. FileId MUST NOT have multiple pending compactions. "
            + "Please fix your strategy implementation. FileIdsWithPendingCompactions :" + fgIdsInPendingCompactionAndClustering
            + ", Selected workload :" + compactionPlan);
    if (compactionPlan.getOperations().isEmpty()) {
      LOG.warn("After filtering, Nothing to compact for " + metaClient.getBasePath());
    }
    return compactionPlan;
  }
```

该JOB用于生成合并parquet文件和avro文件计划。

## Hudi配置一览

关于Hudi的配置，大家可以在github中找到：

```
hudi/hudi-client/hudi-client-common/src/main/java/org/apache/hudi/config/
```

这里的配置肯定是最新的，因为文档往往会滞后于源码，有些配置在文档中没有。

### 通用配置文件

Hudi中的配置文件还是蛮多的。

| ![img](https://mmbiz.qpic.cn/sz_mmbiz_png/frAIQPLLOycoDia9uFYqIpcEzCiaIMZbvY5HRwIHn6COs2dTj3wQzTvTW6XiactkR0AH9g3v2kNmNugJq9CYZKlibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) |
| ------------------------------------------------------------ |
| 一共有13类配置                                               |

以下是简单说明：

- HoodieBootstrapConfig：控制bootstrap引导程序，无需等待所有数据加载到Hudi，就可以使用Hudi。
- **HoodieClusteringConfig**：控制小文件合并

- **HoodieCompactionConfig**：控制Parquet和Avro合并、以及cleaner相关配置
- HoodieHBaseIndexConfig：控制HBase索引相关配置

- **HoodieIndexConfig**：控制Hudi索引相关配置
- HoodieMemoryConfig：控制操作Hudi的内存配置

- HoodieMetricsConfig：Hudi的相关监控指标配置
- HoodieMetricsDatadogConfig：DataLog指标配置

- HoodieMetricsPrometheusConfig：Prometheus指标配置
- HoodiePayloadConfig：控制Hudi负载相关指标配置。

- **HoodieStorageConfig**：控制Hudi存储相关指标配置。
- HoodieWriteCommitCallbackConfig：控制Hudi提交写入回调行为配置。

- **HoodieWriteConfig**：控制Hudi写入相关配置。

### Spark相关配置

与Spark相关配置在这：

```
hudi/hudi-spark-datasource/hudi-spark-common/src/main/scala/org/apache/hudi/DataSourceOptions.scala
```

配置项请参考：http://hudi.apache.org/docs/configurations.html#read-options