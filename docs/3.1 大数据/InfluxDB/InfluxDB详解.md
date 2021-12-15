- [InfluxDB详解](https://blog.csdn.net/fuhanghang/article/details/105484610)
- [简析时序数据库 InfluxDB](https://www.cnblogs.com/buttercup/p/15204096.html)

时序数据`TimeSeries`是一连串随**时间**推移而发生变化的**相关**事件。

常见时序数据有：

- 监控日志：机器的 CPU 负载变化
- 用户行为：用户在电商网站上的访问记录
- 金融行情：股票的日内成交记录

　这类数据具有以下特点：

- 必然带有时间戳，可能存在时效性
- 数据量巨大，并且生成速度极快
- 更关注数据变化的趋势，而非数据本身

InfluxDB（时序数据库）（influx，[ˈɪnflʌks]，流入，涌入），常用的一种使用场景：<font color='red'>监控数据统计</font>。每毫秒记录一下电脑内存的使用情况，然后就可以根据统计的数据，利用图形化界面（InfluxDB V1一般配合Grafana）制作内存使用情况的折线图；<font color='red'>可以理解为按时间记录一些数据（常用的监控数据、埋点统计数据等），然后制作图表做统计；</font>

InfluxDB自带的各种特殊函数如求标准差，随机取样数据，统计数据变化比等，使数据统计和实时分析变得十分方便，适合用于包括**DevOps监控**，**应用程序指标**，**物联网传感器数据**和实时分析的后端存储。类似的数据库有Elasticsearch、Graphite等。

# 一、什么是InfluxDB

InfluxDB是一个由InfluxData开发的<font color='red'>开源时序型数据</font>。它由Go写成，着力于<font color='red'>高性能地查询与存储时序型数据</font>。InfluxDB被广泛应用于存储系统的<font color='red'>监控数据，IoT行业的实时数据等场景。</font>

**InfluxDB有三大特性：**

1. Time Series （时间序列）：你可以使用与时间有关的相关函数（如最大，最小，求和等）
2. Metrics（度量）：你可以<font color='red'>实时对大量数据进行计算</font>
3. Eevents（事件）：它支持任意的事件数据

**特点**

- 为时间序列数据专门编写的自定义高性能数据存储。 <font color='red'>TSM引擎具有高性能的写入和数据压缩</font>
- Golang编写，没有其它的依赖
- 提供简单、高性能的写入、查询<font color='red'> http api，Native HTTP API, 内置http支持，使用http读写</font>
- 插件支持其它数据写入协议，例如 graphite、collectd、OpenTSDB
- 支持<font color='red'>类sql查询语句</font>
- tags可以索引序列化，提供快速有效的查询
- Retention policies自动处理过期数据
- Continuous queries自动聚合，提高查询效率
- schemaless(无结构)，可以是任意数量的列
- Scalable可拓展
- min, max, sum, count, mean,median 一系列函数，方便统计
- Built-in Explorer 自带管理工具

# 2、对常见关系型数据库（MySQL）的基础概念对比

| 概念         | MySQL    | InfluxDB                                                    |
| ------------ | -------- | ----------------------------------------------------------- |
| 数据库（同） | database | database                                                    |
| 表（不同）   | table    | measurement（测量; 度量）                                   |
| 列（不同）   | column   | tag(带索引的，非必须)、field(不带索引)、timestemp(唯一主键) |

**2.Influxdb相关名词**

> database：数据库；
>  measurement：数据库中的表；
>  points：表里面的一行数据。
>  influxDB中独有的一些概念：Point由时间戳（time）、数据（field）和标签（tags）组成。

Point相当于传统数据库里的一行数据，如下表所示：

| Point属性            | 传统数据库中的概念                                          |
| -------------------- | ----------------------------------------------------------- |
| time（时间戳）       | 每个数据记录时间，是数据库中的主索引(会自动生成)            |
| fields（字段、数据） | 各种**记录值（没有索引的属性）也就是记录的值**：温度， 湿度 |
| tags（标签）         | 各种**有索引的属性**：地区，海拔                            |

注意

<font color='red'>在influxdb中，**字段必须存在**。因为字段是没有索引的。如果使用字段作为查询条件，会扫描符合查询条件的所有字段值，性能不及tag。类比一下，fields相当于SQL的没有索引的列。</font>
<font color='red'> tags是可选的，但是强烈建议你用上它，因为tag是有索引的，tags相当于SQL中的有索引的列。**tag value只能是string类型**。</font>


 还有一个重要的名词：series
 所有在数据库中的数据，<font color='red'>都需要通过图表来表示，series（系列）</font>表示这个表里面的所有的数据可以在图标上画成几条线（注：线条的个数由tags排列组合计算出来）

- 示例数据如下： 其中census是measurement，butterflies和honeybees是field key，location和scientist是tag key

```
name: census
————————————
time                 butterflies     honeybees     location     scientist
2015-08-18T00:00:00Z      12             23           1         langstroth
2015-08-18T00:00:00Z      1              30           1         perpetua
2015-08-18T00:06:00Z      11             28           1         langstroth
2015-08-18T00:06:00Z      11             28           2         langstroth
```

示例中有三个tag set

# 3、注意点

- tag 只能为字符串类型
- field 类型无限制
- 不支持join
- 支持连续查询操作（汇总统计数据）：CONTINUOUS QUERY
- 配合Telegraf服务（Telegraf可以监控系统CPU、内存、网络等数据）
- 配合Grafana服务（数据展现的图像界面，将influxdb中的数据可视化）

# 4、常用InfluxQL

```sql
-- 查看所有的数据库
show databases;
-- 使用特定的数据库
use database_name;
-- 查看所有的measurement
show measurements;
-- 查询10条数据
select * from measurement_name limit 10;
-- 数据中的时间字段默认显示的是一个纳秒时间戳，改成可读格式
precision rfc3339; -- 之后再查询，时间就是rfc3339标准格式
-- 或可以在连接数据库的时候，直接带该参数
influx -precision rfc3339
-- 查看一个measurement中所有的tag key 
show tag keys
-- 查看一个measurement中所有的field key 
show field keys
-- 查看一个measurement中所有的保存策略(可以有多个，一个标识为default)
show retention policies;
```

# 5、InfluxDB Java Demo

[GitHub Demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FMuscleape%2Finfluxdb_demo)

## InfluxDB 安装

1. 下载地址，

　　　　64bit：https://dl.influxdata.com/influxdb/releases/influxdb-1.7.4_windows_amd64.zip

　　　　chronograf：https://dl.influxdata.com/chronograf/releases/chronograf-1.7.8_windows_amd64.zip

​    2.解压安装包

​     

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjE0MTE2OC0xODY1NDY0ODI3LnBuZw?x-oss-process=image/format,png)

修改配置文件

​    InfluxDB 的数据存储主要有三个目录。默认情况下是 meta, wal 以及 data 三个目录，服务器运行后会自动生成。

- meta 用于存储数据库的一些元数据，meta 目录下有一个 meta.db 文件。
- wal 目录存放预写日志文件，以 .wal 结尾。
- data 目录存放实际存储的数据文件，以 .tsm 结尾。

如果不使用influxdb.conf配置的话，那么直接双击打开influxd.exe就可以使用influx，此时上面三个文件夹的目录则存放在Windows系统的C盘User目录下的.Influx目录下，默认端口为8086，以下为修改文件夹地址，以及端口号方法。

1.修改以下部分的路径
![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjE1MzI1Ni0xNjI4MTA0NTg1LnBuZw?x-oss-process=image/format,png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjE1MzI1Ni0xNjI4MTA0NTg1LnBuZw?x-oss-process=image/format,png)

2. 如果需要更改端口号，则修改以下部分配置

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjIwMTM3OC0xMzgzMTA3NzYwLnBuZw?x-oss-process=image/format,png)

3. 修改配置后启动方式

InfluxDB 使用时需要首先打开Influxd.exe，直接打开会使用默认配置，需要使用已配置的配置文件的话，需要指定conf文件进行启动，启动命令如下：

influxd.exe -config influxdb.conf（cmd目录为influxDB目录）

启动可写成bat文件，内容如下：

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjI0NzEwNC0xNjgxNTIwNTA1LnBuZw?x-oss-process=image/format,png)

打开成功画面：

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjI1Mzk2MC0xMjg4ODc4NTUxLnBuZw?x-oss-process=image/format,png)

Influxd成功启动后，即可打开influx.exe，若使用默认配置，则直接打开即可，使用配置文件的情况下，在cmd中输入influx命令（cmd目录为influxDB目录），启动可写成bat文件，文件内容如下：

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjMxMTE3OS0xNTY4NDgxNDczLnBuZw?x-oss-process=image/format,png)![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjMxMTE3OS0xNTY4NDgxNDczLnBuZw?x-oss-process=image/format,png)

-port是使用特定port号启动

启动成功画面显示如下：

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MjMyNTczMS00NzQ4NTIzNTYucG5n?x-oss-process=image/format,png)

备注：运行influx.exe 时，influxd.exe不可关闭。（influxd.exe是守护进程）

配置文件具体内容详解：

官方介绍：https://docs.influxdata.com/influxdb/v1.2/administration/config/

转自：https://www.cnblogs.com/guyeshanrenshiwoshifu/p/9188368.html

###  全局配置

```sql
reporting-disabled = false  # 该选项用于上报influxdb的使用信息给InfluxData公司，默认值为false

bind-address = ":8088"  # 备份恢复时使用，默认值为8088
```

### 1、meta相关配置

```yaml
[meta]
dir = "/var/lib/influxdb/meta"  # meta数据存放目录
retention-autocreate = true  # 用于控制默认存储策略，数据库创建时，会自动生成autogen的存储策略，默认值：true
logging-enabled = true  # 是否开启meta日志，默认值：true
```

### 2、data相关配置

```yaml
[data]
dir = "/var/lib/influxdb/data"  # 最终数据（TSM文件）存储目录
wal-dir = "/var/lib/influxdb/wal"  # 预写日志存储目录
query-log-enabled = true  # 是否开启tsm引擎查询日志，默认值： true
cache-max-memory-size = 1048576000  # 用于限定shard最大值，大于该值时会拒绝写入，默认值：1000MB，单位：byte
cache-snapshot-memory-size = 26214400  # 用于设置快照大小，大于该值时数据会刷新到tsm文件，默认值：25MB，单位：byte
cache-snapshot-write-cold-duration = "10m"  # tsm引擎 snapshot写盘延迟，默认值：10Minute
compact-full-write-cold-duration = "4h"  # tsm文件在压缩前可以存储的最大时间，默认值：4Hour
max-series-per-database = 1000000  # 限制数据库的级数，该值为0时取消限制，默认值：1000000
max-values-per-tag = 100000  # 一个tag最大的value数，0取消限制，默认值：100000
```

### 3、coordinator查询管理的配置选项

```yaml
[coordinator]
write-timeout = "10s"  # 写操作超时时间，默认值： 10s
max-concurrent-queries = 0  # 最大并发查询数，0无限制，默认值： 0
query-timeout = "0s  # 查询操作超时时间，0无限制，默认值：0s
log-queries-after = "0s"  # 慢查询超时时间，0无限制，默认值：0s
max-select-point = 0  # SELECT语句可以处理的最大点数（points），0无限制，默认值：0
max-select-series = 0  # SELECT语句可以处理的最大级数（series），0无限制，默认值：0
max-select-buckets = 0  # SELECT语句可以处理的最大"GROUP BY time()"的时间周期，0无限制，默认值：0
```

### 4、retention旧数据的保留策略

```yaml
[retention]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "30m"  # 检查时间间隔，默认值 ："30m"
```

### 5、shard-precreation分区预创建

```yaml
[shard-precreation]
enabled = true  # 是否启用该模块，默认值 ： true
check-interval = "10m"  # 检查时间间隔，默认值 ："10m"
advance-period = "30m"  # 预创建分区的最大提前时间，默认值 ："30m"
```

### 6、monitor 控制InfluxDB自有的监控系统

默认情况下，InfluxDB把这些数据写入_internal  数据库，如果这个库不存在则自动创建。 _internal  库默认的retention策略是7天，如果你想使用一个自己的retention策略，需要自己创建。

```yaml
[monitor]
store-enabled = true  # 是否启用该模块，默认值 ：true
store-database = "_internal"  # 默认数据库："_internal"
store-interval = "10s  # 统计间隔，默认值："10s"
```

### 7、admin web管理页面

```yaml
[admin]
enabled = true  # 是否启用该模块，默认值 ： false
bind-address = ":8083"  # 绑定地址，默认值 ：":8083"
https-enabled = false  # 是否开启https ，默认值 ：false
https-certificate = "/etc/ssl/influxdb.pem"  # https证书路径，默认值："/etc/ssl/influxdb.pem"
```

### 8、http API

```yaml
[http]
enabled = true  # 是否启用该模块，默认值 ：true
bind-address = ":8086"  # 绑定地址，默认值：":8086"
auth-enabled = false  # 是否开启认证，默认值：false
realm = "InfluxDB"  # 配置JWT realm，默认值: "InfluxDB"
log-enabled = true  # 是否开启日志，默认值：true
write-tracing = false  # 是否开启写操作日志，如果置成true，每一次写操作都会打日志，默认值：false
pprof-enabled = true  # 是否开启pprof，默认值：true
https-enabled = false  # 是否开启https，默认值：false
https-certificate = "/etc/ssl/influxdb.pem"  # 设置https证书路径，默认值："/etc/ssl/influxdb.pem"
https-private-key = ""  # 设置https私钥，无默认值
shared-secret = ""  # 用于JWT签名的共享密钥，无默认值
max-row-limit = 0  # 配置查询返回最大行数，0无限制，默认值：0
max-connection-limit = 0  # 配置最大连接数，0无限制，默认值：0
unix-socket-enabled = false  # 是否使用unix-socket，默认值：false
bind-socket = "/var/run/influxdb.sock"  # unix-socket路径，默认值："/var/run/influxdb.sock"
```

### 9、subscriber 控制Kapacitor接受数据的配置

```yaml
[subscriber]
enabled = true  # 是否启用该模块，默认值 ：true
http-timeout = "30s"  # http超时时间，默认值："30s"
insecure-skip-verify = false  # 是否允许不安全的证书
ca-certs = ""  # 设置CA证书
write-concurrency = 40  # 设置并发数目，默认值：40
write-buffer-size = 1000  # 设置buffer大小，默认值：1000
```

### 10、graphite 相关配置

```yaml
[[graphite]]
enabled = false  # 是否启用该模块，默认值 ：false
database = "graphite"  # 数据库名称，默认值："graphite"
retention-policy = ""  # 存储策略，无默认值
bind-address = ":2003"  # 绑定地址，默认值：":2003"
protocol = "tcp"  # 协议，默认值："tcp"
consistency-level = "one"  # 一致性级别，默认值："one
batch-size = 5000  # 批量size，默认值：5000
batch-pending = 10  # 配置在内存中等待的batch数，默认值：10
batch-timeout = "1s"  # 超时时间，默认值："1s"
udp-read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0
separator = "."  # 多个measurement间的连接符，默认值： "."
```

### 11、collectd

```yaml
[[collectd]]
enabled = false  # 是否启用该模块，默认值 ：false
bind-address = ":25826"  # 绑定地址，默认值： ":25826"
database = "collectd"  # 数据库名称，默认值："collectd"
retention-policy = ""  # 存储策略，无默认值
typesdb = "/usr/local/share/collectd"  # 路径，默认值："/usr/share/collectd/types.db"
auth-file = "/etc/collectd/auth_file"
batch-size = 5000
batch-pending = 10
batch-timeout = "10s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。默认值：0
```

### 12、opentsdb

```yaml
[[opentsdb]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":4242"  # 绑定地址，默认值：":4242"
database = "opentsdb"  # 默认数据库："opentsdb"
retention-policy = ""  # 存储策略，无默认值
consistency-level = "one"  # 一致性级别，默认值："one"
tls-enabled = false  # 是否开启tls，默认值：false
certificate= "/etc/ssl/influxdb.pem"  # 证书路径，默认值："/etc/ssl/influxdb.pem"
log-point-errors = true  # 出错时是否记录日志，默认值：true
batch-size = 1000
batch-pending = 5
batch-timeout = "1s"
```

### 13、udp

```yaml
[[udp]]
enabled = false  # 是否启用该模块，默认值：false
bind-address = ":8089"  # 绑定地址，默认值：":8089"
database = "udp"  # 数据库名称，默认值："udp"
retention-policy = ""  # 存储策略，无默认值
batch-size = 5000
batch-pending = 10
batch-timeout = "1s"
read-buffer = 0  # udp读取buffer的大小，0表示使用操作系统提供的值，如果超过操作系统的默认配置则会出错。 该配置的默认值：0　
```

### 14、continuous_queries

```yaml
[continuous_queries]
enabled = true  # enabled 是否开启CQs，默认值：true
log-enabled = true  # 是否开启日志，默认值：true
run-interval = "1s"  # 时间间隔，默认值："1s"
```

## **InfluxDB数据库常用命令**

###   1、显示所有数据库

​    show databases

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzExMzY5Ny0xNTIwNDM2OTQzLnBuZw?x-oss-process=image/format,png)

### 2、 创建数据库

  create database test

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzEyMjcyNC0xMDA1NDk1NTIxLnBuZw?x-oss-process=image/format,png)

### 3、 使用某个数据库

use test

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzEzNTc5MS0yNjAzNzM4MDQucG5n?x-oss-process=image/format,png)

### 4、 显示所有表

show measurements

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzE0Njg2OC02MjYwMTYyMDgucG5n?x-oss-process=image/format,png)

没有表则无返回。

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzE1ODY4OS03ODE0NzQ5MjUucG5n?x-oss-process=image/format,png)![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzE1ODY4OS03ODE0NzQ5MjUucG5n?x-oss-process=image/format,png)

### 5、新建表和插入数据

新建表没有具体的语法，只是增加第一条数据时，会自动建立表

insert results,hostname=index1 value=1

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzE1ODY4OS03ODE0NzQ5MjUucG5n?x-oss-process=image/format,png)![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzE1ODY4OS03ODE0NzQ5MjUucG5n?x-oss-process=image/format,png)

这里的时间看不懂，可以设置一下时间显示格式

precision rfc3339

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzIxNjM3Mi0zNTg0Nzg1MTQucG5n?x-oss-process=image/format,png)

### 6、 查询数据

表名有点号时，输入双引号

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzIyNzE3OS0xMjI5NDkwMjIyLnBuZw?x-oss-process=image/format,png)

和sql语法相同，区别：

measurement 数据库中的表

points 表里面的一行数据，Point由时间戳（time）、数据（field）、标签（tags）组成。

### 7、 用户显示

a. 显示所有用户

show users

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzIzNzgwMy04OTE4MTI1MDUucG5n?x-oss-process=image/format,png)

b.新增用户

*--**普通用户*

create user "user" with password 'user'

*--**管理员用户*

create user "admin" with password 'admin' with all privileges

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzI0NjI4Ny0xODU1NTgzMzk5LnBuZw?x-oss-process=image/format,png)

c.删除用户

drop user "user"

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzI1NTY3OS0xMTIyNTcxNDIxLnBuZw?x-oss-process=image/format,png)![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzI1NTY3OS0xMTIyNTcxNDIxLnBuZw?x-oss-process=image/format,png)

很多InfluxDB的文章都说InfluxDB是时序数据库，不支持删除。但实际测试是可以删除的。

连接InfluxDB
![InfluxDB删除数据](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zNC41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODExLzE0L2FlMDFhNDliMDdmY2UwYmVhOGNjYmVhNWM3ZGU4NjJkLnBuZw?x-oss-process=image/format,png)

一张叫uv的表

![InfluxDB删除数据](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zNC41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODExLzE0LzkwODI4M2NjMzIzZmE5ZDg3OTljOGQ0NWM5Nzg5MTVkLnBuZw?x-oss-process=image/format,png)

执行删除后
![InfluxDB删除数据](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zNC41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODExLzE0L2JlY2JkNzI5ZjQ3ZjI3OTAwOTVlMTFkYzhmNThiMDYyLnBuZw?x-oss-process=image/format,png)

## **Chronograf 使用**

1、解压文件后，直接进入安装目录，执行chronograf.exe后；

2、输入：http://localhost:8888（chronograf默认是8888端口）

3、influxDB数据源连接

 ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzMwNTUzNC02NTA4OTE0MDIucG5n?x-oss-process=image/format,png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzQwNDY0Mi03Mjc5ODc2MDMucG5n?x-oss-process=image/format,png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDE4LmNuYmxvZ3MuY29tL2Jsb2cvMTI0NzQ4NS8yMDE5MDIvMTI0NzQ4NS0yMDE5MDIyNTE3MzQxMTg0MC02MTM3MDYyMDkucG5n?x-oss-process=image/format,png)

 