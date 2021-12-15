[从入门到精通-Redis，图文并茂、分布式锁、主从复制、哨兵机制、Cluster集群、缓存击穿、缓存雪崩、持久化方案、缓存淘汰策略 附案例源码](https://www.cnblogs.com/chenyanbin/p/12073107.html)



# NoSql介绍与Redis介绍[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#nosql介绍与redis介绍)

## 什么是Redis?[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么是redis?)

　　**Redis**是用**C语言**开发的一个**开源**的高性能**键值对**(key-value)**内存数据库**。

　　它提供**五种数据类型**来存储值：**字符串类型、散列类型、列表类型、集合类型、有序类型**。

　　它是一种**NoSql**数据库。

## 什么是NoSql？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么是nosql？)

- NoSql，即Not-Only Sql(不仅仅是SQL)，泛指**非关系型的数据库**。
- 什么是关系型数据库？数据结构是一种有行有列的数据库。
- NoSql数据库是为了解决**高并发、高可用、高可扩展、大数据存储**问题而产生的数据库解决方案。
- NoSql可以作为关系型数据库的良好补充，但是**不能替代关系型数据库**。

## NoSql数据库分类[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#nosql数据库分类)

### 键值(key-value)存储数据库[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#键值(key-value)存储数据库)

- 相关产品：Tokyo Cabinet/Tyrant、**Redis**、Voldemort、Berkeley Db等
- 典型应用：内存缓存，主要用于处理大量数据的高访问负载
- 数据模型：一系列键值对
- **优势：快速查询**
- **劣势：存储的数据缺少结构化**

### 列存储数据库[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#列存储数据库)

- 相关产品：Cassandra、**Hbase**、Riak
- 典型应用：分布式的文件系统
- 数据模型：以列簇式存储，将同一列数据存在一起
- 优势：查找速度快，可扩展性强，更容易进行分布式扩展
- 劣势：功能相对局限

### 文档型数据库[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#文档型数据库)

- 相关产品：CouchDB、**MongoDB**
- 典型应用：web应用(与key-value类似，value是结构化的)
- 数据模型：一系列键值对
- 优势：数据结构要求不严格
- 劣势

### 图形(Graph)数据库[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#图形(graph)数据库)

- 相关数据库：Neo4J、InfoGrid、Infinite、Graph
- 典型应用：社交网络
- 数据模型：图结构
- 优势：利用图结构先关算法
- 劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

# Redis历史发展[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis历史发展)

　　**2008年**，意大利的一家创业公司Merzia推出了一款给予MySql的网站实时统计系统LLOOGG，然而没过多久该公司的创始人**Salvatore Sanfilippo**便对MySql的性能感到失望，于是他决定亲力为LLOOGG量身**定做一个数据库**，并**于2009年开发完成，这个数据库就是Redis**。

　　不过**Salvatore Sanfilippo**并**不满足**只**将Redis****用**于**LLOOGG**这一款产品，而是**希望更多的人使用它**，于是**在同一年Salvatore Sanfilippo将Redis开源发布**。

　　并**开始****和Redis**的另一名**主要**的代码**贡献者**Pieter Noordhuis一起**继续**着**Redis**的**开发**，**直到今天**。

　　Salvatore Sanfilippo自己也没有想到，短短的几年时间，Redis就拥有了庞大的用户群体。Hacker News在2012年发布一份数据库的使用请款调查，结果显示有近12%的公司在使用Redis。**国内如新浪微博、街旁网、知乎网、国外如GitHub、Stack、Overflow、Flickr等都是Redis的用户**。

　　**VmWare**公司从**2010年**开始赞助Redis的开发，Salvatore Sanfilippo和Pieter Noordhuis也分别**在3月和5月****加入VMware**，**全职开发Redis**。

# Redis的应用场景[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis的应用场景)

- 内存数据库(登录信息、购物车信息、用户浏览记录等)
- 缓存服务器(商品数据、广告数据等等)(最多使用)
- 解决分布式集群架构中的Session分离问题(Session共享)
- 任务队列。(秒杀、抢购、12306等等)
- 支持发布订阅的消息模式
- 应用排行榜
- 网站访问统计
- 数据过期处理(可以精确到毫秒)

# Redis安装及配置[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis安装及配置)

- 官网地址：https://redis.io/
- 中文官网地址：http://www.redis.cn
- 下载地址：http://download.redis.io/releases/

## Linux环境下安装Redis[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#linux环境下安装redis)

**注：将下载后的Redis拖进Linux需要安装下，VMware Tools，[参考链接](https://www.cnblogs.com/gucb/p/11525557.html)**

### 将下载后的Redis拖进linux[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#将下载后的redis拖进linux)

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222002115544-1123107779.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222002115544-1123107779.jpg)

### 安装C语言需要的GCC环境[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装c语言需要的gcc环境)

```
yum install gcc-c++
```

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222002943416-1694504890.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222002943416-1694504890.jpg)

### 解压Redis源码压缩包[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#解压redis源码压缩包)

```
tar -zxf redis-4.0.11.tar.gz
```

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222003400492-1812135889.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222003400492-1812135889.jpg)

### 编译Redis源码[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#编译redis源码)

```
make
```

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222003903607-1373453763.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222003903607-1373453763.jpg)

### 安装Redis[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装redis)

```
make install PREFIX=/user/local/redis

格式：make install PREFIX=安装目录
```

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222005321704-1439459785.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222005321704-1439459785.jpg)

# Redis启动[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis启动)

## redis设置密码[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis设置密码)

### 更改redis.conf配置[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#更改redis.conf配置)

[![img](https://img2018.cnblogs.com/i-beta/1504448/202001/1504448-20200107221254945-1859042256.png)](https://img2018.cnblogs.com/i-beta/1504448/202001/1504448-20200107221254945-1859042256.png)

 

### 重启服务[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#重启服务)

[![img](https://img2018.cnblogs.com/i-beta/1504448/202001/1504448-20200107221118992-183116905.png)](https://img2018.cnblogs.com/i-beta/1504448/202001/1504448-20200107221118992-183116905.png)

## 前端启动[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#前端启动)

- 启动命令：redis-server，直接运行bin/redis-server将以前端模式启动。

[![img](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222010331027-1611478245.jpg)](https://img2018.cnblogs.com/common/1504448/201912/1504448-20191222010331027-1611478245.jpg)

#### 关闭服务

```
ctrl+c
```

启动缺点：客户端窗口关闭，则redis-server程序结束，不推荐使用

## 后端启动(守护进程启动)[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#后端启动(守护进程启动))

### 拷贝redis[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#拷贝redis)

```
cp redis.conf /usr/local/redis/bin

格式：cp 拷贝文件夹 拷贝路径
```

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222010836185-752763862.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222010836185-752763862.png)

###  修改redis.conf，将daemonize由no改为yes[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 修改redis.conf，将daemonize由no改为yes)

```
vim redis.conf
```

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222011217563-839218892.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222011217563-839218892.png)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222011346093-1938119308.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222011346093-1938119308.png)

###  执行命令[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 执行命令)

```
 ./redis-server redis.conf

格式：启动服务 指定配置文件
```

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222013818063-1488884897.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222013818063-1488884897.png)

###  关闭服务（粗暴方式）[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 关闭服务（粗暴方式）)

```
kill -9 42126

格式：kill -9 进程号
```

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222014015633-1534155563.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222014015633-1534155563.png)

###  正常关闭[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 正常关闭)

```
./redis-cli shutdown
```

### 修改redis配置文件(解决IP绑定问题)[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#修改redis配置文件(解决ip绑定问题))

```
# bind 127.0.0.1 绑定的IP才能fangwenredis服务器，注释掉该配置
protected-mode yes 是否开启保护模式，由yes改为no
```

### 其他命令说明[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#其他命令说明)

```
redis-server ：启动redis服务
redis-cli ：进入redis命令客户端
redis-benchmark： 性能测试的工具
redis-check-aof ： aof文件进行检查的工具
redis-check-dump ：  rdb文件进行检查的工具
redis-sentinel ：  启动哨兵监控服务
```

# Redis客户端[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis客户端)

## 自带命令行客户端[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#自带命令行客户端)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222195540066-1931343008.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222195540066-1931343008.png)

## 语法[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#语法)

```
./redis-cli -h 127.0.0.1 -p 6379 
```

## 修改redis.conf配置文件(解决ip绑定问题)[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#修改redis.conf配置文件(解决ip绑定问题))

```
#bind 127.0.0.1 绑定的ip才能访问redis服务器，注释掉该配置

protected-mode yes 是否开启保护模式，由yes改为no
```

## 参数说明[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#参数说明)

- -h：redis服务器的ip地址
- -p：redis实例的端口号

## 默认方式[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#默认方式)

**如果不制定主机和端口号也可以**

```
./redis-cli

默认的主机地址是：127.0.0.1
默认的端口号是：6379
```

# Redis数据类型[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis数据类型)

## 官网命令大全网址[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#官网命令大全网址)

http://www.redis.cn/commands.html

- String(字符类型)
- Hash(散列类型)
- List(列表类型)
- Set(集合类型)
- SortedSet(有序集合类型，简称zset)

**注：命令不区分大小写，而key是区分大小写的。**

## String类型[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#string类型)

### 赋值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#赋值)

语法：SET key value

### 取值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#取值)

语法：GET key

### 取值并赋值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#取值并赋值)

语法：GETSET key value

### 演示[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#演示)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222200450823-410512864.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222200450823-410512864.png)

###  数值增减[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 数值增减)

### 前提条件：[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#前提条件：)

1. 当**value**为**整数数据**时，才能使用以下命令操作数值的增减。
2. 数值增减都是**原子**操作。

#### 递增数字

语法：**INCR key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222200935828-2000193662.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222200935828-2000193662.png)

####  增加指定的整数

语法：**INCRBY key increment**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201114798-154352781.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201114798-154352781.png)

####  递减数值

语法：**DECR key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201224536-1147328638.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201224536-1147328638.png)

#### 减少指定的整数 

语法：**DECRBY key decrement**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201341749-1662237044.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201341749-1662237044.png)

###  仅当不存在时赋值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 仅当不存在时赋值)

**注：该命令可以实现分布式锁的功能，后续讲解！！！！**

语法：**setnx key value**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201825200-1623331154.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222201825200-1623331154.png)

### 向尾部追加值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#向尾部追加值)

注：**APPEND**命令，**向键**值的**末尾追加value**。**如果键不存**在则该**键的值设置为value**，即相当于set key value。返回值是追加后字符串的总长度。

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202450938-2075866676.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202450938-2075866676.png)

###  获取字符串长度[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取字符串长度)

注：strlen命令，返回键值的长度，如果键不存在则返回0

 语法：**STRLEN key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202634864-649334692.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202634864-649334692.png)

### 同时设置/获取多个键值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#同时设置/获取多个键值)

语法：

1. **MSET key value [key value ....]**
2. **MGET key [key ....]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202937260-1685836530.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222202937260-1685836530.png)

###  应用场景之自增主键[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 应用场景之自增主键)

需求：**商品编号、订单号采用INCR命令生成。**

设计：key明明要有一定的设计

实现：定义商品编号key：items:id

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222211944158-144595926.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222211944158-144595926.png)

##  Hash类型[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# hash类型)

　　Hash叫散列类型，它提供了字段和字段值的映射。字段值只能是字符串类型，不支持散列类型、集合类型等其他类型。

**[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212113345-81285081.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212113345-81285081.png)**

### 赋值 [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#赋值 )

　　**HSET**命令**不区分插入**和**更新**操作，当执行**插入**操作时HSET命令**返回1**，当执行**更新**操作时**返回0**。

#### 一次只能设置一个字段值

语法：**HSET key field value**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212552494-609613098.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212552494-609613098.png)

####  一次设置多个字段值

语法：**HMSET key field value [field value ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212744518-678723842.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222212744518-678723842.png)

####  当字段不存在时

类似HSET，区别在于如何字段存在，该命令不执行任何操作

语法：**HSETNX key field value**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213020424-286882757.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213020424-286882757.png)

### 取值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#取值)

#### 一次只能获取一个字段值

语法：**HGET key field**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213217307-955879843.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213217307-955879843.png)

####  一次可以获取多个字段值

语法：**HMGET key field [field ....]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213337690-1313492019.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213337690-1313492019.png)

#### 获取所有字段值

语法：**HGETALL key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213457044-1951080079.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213457044-1951080079.png)

###  删除字段[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 删除字段)

可以删除一个或多个字段，返回值是被删除的字段个数

语法：**HDEL key field [field ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213651281-858272075.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222213651281-858272075.png)

###  增加数字[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 增加数字)

语法：**HINCRBY key field increment**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214001738-331663953.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214001738-331663953.png)

###  判断字段是否存在[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 判断字段是否存在)

语法：**HEXISTS key field**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214205910-2098226231.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214205910-2098226231.png)

### 只获取字段名或字段值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#只获取字段名或字段值)

语法：

1. **HKEYS key**
2. **HVALS key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214353740-246209435.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214353740-246209435.png)

###  获取字段数量[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取字段数量)

语法：**HLEN key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214451050-981035825.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214451050-981035825.png)

###  获取所有字段[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取所有字段)

作用：获取hash的所有信息，包括key和value

语法：**hgetall key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214612104-2010751801.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222214612104-2010751801.png)

###  应用之存储商品信息[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 应用之存储商品信息)

**注意事项：存在哪些对象数据，特别是对象属性经常发生增删改操作的数据。**

商品信息字段

　　【商品id，商品名称，商品描述，商品库存，商品好评】

定义商品信息的key

　　商品id为1001的信息在Redis中的key为：[items.1001]

#### 示例

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215014828-1481714081.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215014828-1481714081.png)

##  List类型[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# list类型)

　　ArrayList使用**数组方式**存储数据，所以根据索引查询数据速度快，而新增或者删除元素时需要涉及到位移操作，所以比较慢。

　　LinkedList使用**双向链表方式**存储数据，每个元素都记录前后元素的指针，所以插入、删除数据时只是更改前后元素的指针即可，速度非常快。然后通过下标查询元素时需要从头开始索引，所以比较慢，但是如果查询前几个元素或后几个元素速度比较快。

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215457354-1734006376.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215457354-1734006376.png)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215508663-750071829.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222215508663-750071829.png)

###  List介绍[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# list介绍)

　　Redis的列表类型(list)可以存储一个有序的字符串列表，常用的操作是**向列表两端添加元素，或者获取列表的某一个片段**。

　　列表类型内部是使用**双向链表(double linked list)**实现的，所以向列表两端添加元素的时间复杂度为0/1，获取越接近两端的元素速度就越快。意味着即使是一个有几千万个元素的列表，获取头部或尾部的10条记录也是极快的。

### 向列表两端添加元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#向列表两端添加元素)

#### 向列表左边添加元素

语法：**LPUSH key value [value ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220025257-983722706.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220025257-983722706.png)

####  向列表右边添加元素

语法：**RPUSH key value [value ....]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220148214-284592605.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220148214-284592605.png)

###  查看列表[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 查看列表)

语法：**LRANGE key start stop**

　　LRANGE命令是列表类型最常用的命令之一，获取列表中的某一片段，将返回start、stop之间的所有元素(包括两端的元素)，索引从0开始。索引可以是负数，**“-1”代表最后一边的一个元素**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220454270-1493619033.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220454270-1493619033.png)

###  从列表两端弹出元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 从列表两端弹出元素)

LPOP命令从列表左边弹出一个元素，会分两步完成：

1. 将列表左边的元素从列表中移除
2. 返回被移除的元素值

语法：

1. **LPOP key**
2. **RPOP key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220718561-418251812.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220718561-418251812.png)

### 获取列表中元素的个数[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取列表中元素的个数)

语法：**LLEN key** [![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220822571-1012217498.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222220822571-1012217498.png)

###  删除列表中指定个数的值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 删除列表中指定个数的值)

　　LREM命令会删除列表中前count个数为value的元素，返回实际删除的元素个数。根据count值不同，该命令的执行方式会有所不同。

语法：**LREM key count value**

1. 当count>0时，LREM会从列表左边开始删除
2. 当count<0时，LREM会从列表右边开始删除
3. 当count=0时，LREM会删除所有值为value的元素

### 获取/设置指定索引的元素值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取/设置指定索引的元素值)

#### 获取指定索引的元素值

语法：**LINDEX key index**

### 设置指定索引的元素值[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#设置指定索引的元素值)

语法：**LSET key index value**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222221837291-1061254107.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222221837291-1061254107.png)

### 向列表中插入元素 [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#向列表中插入元素 )

　　该命令首先会在列表中从左到右查询值为pivot的元素，然后根据第二个参数是BEFORE还是AFTER来决定将value插入到该元素的前面还是后面。

语法：**LINSERT key BEFORE|AFTER pivot value**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222222456437-887011735.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222222456437-887011735.png)

###  将元素从一个列表转移到另一个列表中[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 将元素从一个列表转移到另一个列表中)

语法：**RPOPLPUSH source destination**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222222659395-657050176.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222222659395-657050176.png)

###  应用之商品评论列表[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 应用之商品评论列表)

需求1：用户针对某一商品发布评论，一个商品会被不同的用户进行评论，存储商品评论时，要按时间顺序排序。

需要2：用户在前端页面查询该商品的评论，需要按照时间顺序降序排序。

思路：

　　使用list存储商品评论信息，key是该商品的id，value是商品评论信息商品编号为1001的商品评论key【items:comment:1001】 [![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222223111333-1649727843.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222223111333-1649727843.png)

##  Set类型[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# set类型)

set类型即**集合类型**，其中的数据时**不重复且没有顺序**。

集合类型和列表类型的对比：

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224130645-773233242.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224130645-773233242.png)

 　集合类型的常用操作是向集合中加入或删除元素、判断某个元素是否存在等，由于集合类型的Redis内部是使用值为空散列标实现，所有这些操作的时间复杂度都为0/1。

　　Redis还提供了多个集合之间的交集、并集、差集的运算。

### 添加/删除元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#添加/删除元素)

语法：**SADD key member [member ...]**

语法：**SREM key member [member ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224602044-701811264.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224602044-701811264.png)

### 获取集合中的所有元素 [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取集合中的所有元素 )

语法：**SMEMBERS key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224744113-953004652.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224744113-953004652.png)

###  判断元素是否在集合中[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 判断元素是否在集合中)

语法：**SISMEMBER key member**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224852660-84775417.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224852660-84775417.png)

###  集合运算命令[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 集合运算命令)

#### 集合的差集运算 A-B

属于A并且不属于B的元素构成的集合

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224958976-1949849206.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222224958976-1949849206.png)

 语法：**SDIFF key [key ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225114435-502281447.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225114435-502281447.png)

#### 集合的交集运算 A∩B

属于A且属于B的元素构成的集合。 

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225322631-154776024.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225322631-154776024.png)

 语法：**SINTER key [key ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225421404-1460233157.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225421404-1460233157.png)

#### 集合的并集运算 A ∪ B

属于A或者属于B的元素构成的集合

 [![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225526763-1067055555.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225526763-1067055555.png)

 

 语法：**SUNION key [key ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225618078-1460111861.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225618078-1460111861.png)

### 获取集合中的元素个数[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取集合中的元素个数)

语法：**SCARD key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225738349-136486373.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225738349-136486373.png)

### 从集合中弹出一个元素 [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#从集合中弹出一个元素 )

注意：集合是无序的，所有spop命令会从集合中随机选择一个元素弹出

语法：**SPOP key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225907641-876177040.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222225907641-876177040.png)

##  SortedSet类型zset[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# sortedset类型zset)

　　在集合类型的基础上，有序集合为集合中的每个元素都关联一个分数，这使得我们不仅可以完成插入、删除和判断元素是否存在集合中，还能够获得最高或最低的前N个元素、获取指定分数范围内的元素等与分苏有关的操作。

在某些方面有序集合和列表类型有些相似。

1. 二者都是有序的。
2. 二者都可以获得某一范围的元素

但是二者有着很大的区别：

1. 列表类型是通过链表实现的，后去靠近两端的数据速度极快，而当元素增多后，访问中间数据的速度会变慢。
2. 有序集合类型使用散列实现，所有即使读取位于中间部分的数据也很快。
3. 列表中不能简单的调整某个元素的位置，但是有序集合可以(通过更改分数实现)。
4. 有序集合要比列表类型更耗内存。

### 添加元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#添加元素)

　　向有序集合中加入一个元素和该元素的分数，如果该元素已经存在则会用新的分数替换原有的分数。返回值是新加入到集合中的元素个数，不不含之前已经存在的元素。

语法：**ZADD key score member [score member ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233153788-1247462090.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233153788-1247462090.png)

### 获取排名在某个范围的元素列表[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取排名在某个范围的元素列表)

按照元素分数从小到大的顺序返回索引从start到stop之间的所有元素(包含两端的元素)

语法：**ZRANGE key start stop [WITHSCORES]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233516433-1857757797.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233516433-1857757797.png)

 如果需要获取元素的分数的可以在命令尾部加上WITHSCORES参数

### [![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233628597-350325994.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233628597-350325994.png) 获取元素的分数[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取元素的分数)

语法：**ZSCORE key member**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233746308-1959359455.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222233746308-1959359455.png)

###  删除元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 删除元素)

移除有序集key中的一个或多个成员，不存在的成员将被忽略。

当key存在但不是有序集类型时，返回错误。

语法：**ZREM key member [member ...]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234041755-2052960235.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234041755-2052960235.png)

###  获取指定分数范围的元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取指定分数范围的元素)

语法：**ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234309648-1010675361.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234309648-1010675361.png)

###  增加某个元素的分数[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 增加某个元素的分数)

返回值是更改后的分数

语法：**ZINCRBY key increment member**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234500849-1282932410.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234500849-1282932410.png)

###  获取集合中元素的数量[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取集合中元素的数量)

语法：**ZCARD key**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234559501-597115170.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234559501-597115170.png)

###  获得指定分数范围内的元素个数[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获得指定分数范围内的元素个数)

语法：**ZCOUNT key min max**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234720353-86493855.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234720353-86493855.png)

###  按照排名范围删除元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 按照排名范围删除元素)

语法：**ZREMRANGEBYRANK key start stop**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234922779-1452594854.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222234922779-1452594854.png)

###  按照分数范围删除元素[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 按照分数范围删除元素)

语法：**ZREMRANGEBYSCORE key min max**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235110405-860150005.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235110405-860150005.png)

###  获取元素的排名[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 获取元素的排名)

从小到大

语法：**ZRANK key member**

从大到小

语法：**ZREVRANK key member**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235348836-711778404.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235348836-711778404.png)

 

###  应用之商品销售排行榜[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 应用之商品销售排行榜)

需求：根据商品销售对商品进行排序显示

思路：定义商品销售排行榜(sorted set集合)，key为items:sellsort，分数为商品小数量。

写入商品销售量：

\>商品编号1001的销量是9，商品编号1002的销量是10

\>商品编号1001销量家1

\>商品销量前10名

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235718340-959062345.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191222235718340-959062345.png)

 

##  通用命令[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 通用命令)

### keys[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#keys)

语法：**keys pattern**

### del[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#del)

语法：**DEL key**

### exists[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#exists)

作用：确认一个key是否存在

语法：**exists key**

### expire[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#expire)

　　Redis在实际使用过程中更多的用作缓存，然后缓存的数据一般都是需要设置生存时间的，即：到期后数据销毁。

```
EXPIRE key seconds             设置key的生存时间（单位：秒）key在多少秒后会自动删除

TTL key                     查看key生于的生存时间

PERSIST key                清除生存时间 

PEXPIRE key milliseconds    生存时间设置单位为：毫秒

例子：
192.168.101.3:7002> set test 1        设置test的值为1
OK
192.168.101.3:7002> get test            获取test的值
"1"
192.168.101.3:7002> EXPIRE test 5    设置test的生存时间为5秒
(integer) 1
192.168.101.3:7002> TTL test            查看test的生于生成时间还有1秒删除
(integer) 1
192.168.101.3:7002> TTL test
(integer) -2
192.168.101.3:7002> get test            获取test的值，已经删除
(nil)
```

### rename[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#rename)

作用：**重命名key**

语法：**rename oldkey newkey**

### type[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#type)

作用：**显示指定key的数据类型**

语法：**type key**

# Redis事务[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis事务)

## 事务介绍[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#事务介绍)

- Redis的事务是通过**MULTI，EXEC，DISCARD和WATCH**这四个命令来完成。
- Redis的单个命令都是**原子性**的，所以这里确保事务性的对象是**命令集合**。
- Redis将命令集合序列化并确保处于一事务的**命令集合连续且不被打断**的执行。
- Redis**不支持回滚**的操作。

## 相关命令[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#相关命令)

- ### MULTI[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#multi)

　　　　注：**用于标记事务块的开始**。

　　　　Redis会将后续的命令逐个放入队列中，然后使用**EXEC命令**原子化地执行这个命令序列。

　　　　语法：**MULTI**

- ### EXEC[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#exec)

　　　　在一个**事务中执行所有先前放入队列的命令**，然后恢复正常的连接状态。

　　　　语法：**EXEC**

- ### DISCARD[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#discard)

　　　　清楚所有先前在一个事务中放入队列的命令，然后恢复正常的连接状态。

　　　　语法：**DISCARD**

- ### WATCH[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#watch)

　　　　当某个**事务需要按条件执行**时，就要使用这个命令将给定的**键设置为受监控**的**状态**。

　　　　语法：**WATCH key [key ....]**

　　　　注：该命令可以实现redis的**乐观锁**

- ### UNWATCH[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#unwatch)

　　　　清除所有先前为一个事务监控的键。

　　　　语法：**UNWATCH**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223142731515-1595805701.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223142731515-1595805701.png)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223144852865-714681855.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223144852865-714681855.png)

 

###  事务失败处理[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 事务失败处理)

- Redis语法错误(编译器错误)

**[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223145343486-1781516846.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223145343486-1781516846.png)**

 

-  Redis类型错误(运行期错误)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223145411115-84909057.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223145411115-84909057.png)

 

### 为什么redis不支持事务回滚？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#为什么redis不支持事务回滚？)

1. 大多数事务失败是因为**语法错误或者类型错误**，这两种错误，再开发阶段都是可以避免的
2. Redis为了**性能方面**就忽略了事务回滚

# Redis实现分布式锁[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis实现分布式锁)

## 锁的处理[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#锁的处理)

　　单应用中使用锁：单线程多线程

　　　　**synchronize、Lock**

　　分布式应用中使用锁：多进程

## 分布式锁的实现方式[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#分布式锁的实现方式)

1. 数据库的乐观锁
2. 给予zookeeper的分布式锁
3. **给予redis的分布式锁**

## 分布式锁的注意事项[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#分布式锁的注意事项)

1. **互斥性**：在任意时刻，只有一个客户端能持有锁
2. **同一性**：加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。
3. **避免死锁**：即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

## 实现分布式锁[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#实现分布式锁)

### 获取锁[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#获取锁)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223160628749-370417310.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191223160628749-370417310.png)

#### 方式一(使用set命令实现)

#### 方式二(使用setnx命令实现)

```
package com.cyb.redis.utils;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class jedisUtils {
    private static String ip = "192.168.31.200";
    private static int port = 6379;
    private static JedisPool pool;
    static {
        pool = new JedisPool(ip, port);
    }
    public static Jedis getJedis() {
        return pool.getResource();
    }
    public static boolean getLock(String lockKey, String requestId, int timeout) {
        //获取jedis对象，负责和远程redis服务器进行连接
        Jedis je=getJedis();
        //参数3：NX和XX
        //参数4：EX和PX
        String result = je.set(lockKey, requestId, "NX", "EX", timeout);
        if (result=="ok") {
            return true;
        }
        return false;
    }

    public static synchronized boolean getLock2(String lockKey, String requestId, int timeout) {
        //获取jedis对象，负责和远程redis服务器进行连接
        Jedis je=getJedis();
        //参数3：NX和XX
        //参数4：EX和PX
        Long result = je.setnx(lockKey, requestId);
        if (result==1) {
            je.expire(lockKey, timeout); //设置有效期
            return true;
        }
        return false;
    }
}
```

### 释放锁[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#释放锁)

```
package com.cyb.redis.utils;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class jedisUtils {
    private static String ip = "192.168.31.200";
    private static int port = 6379;
    private static JedisPool pool;
    static {
        pool = new JedisPool(ip, port);
    }
    public static Jedis getJedis() {
        return pool.getResource();
    }
    /**
     * 释放分布式锁
     * @param lockKey
     * @param requestId
     */
    public static void releaseLock(String lockKey, String requestId) {
        Jedis je=getJedis();
        if (requestId.equals(je.get(lockKey))) {
            je.del(lockKey);
        }
    }
}
```

# Redis持久化方案[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis持久化方案)

## 导读[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#导读)

　　Redis是一个内存数据库，为了保证数据的持久性，它提供了两种持久化方案。

1. RDB方式(默认)
2. AOF方式

## RDB方式[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#rdb方式)

　　RDB是Redis**默认**采用的持久化方式。

　　RDB方式是通过**快照**(snapshotting)完成的，当**符合一定条件**时Redis会自动将内存中的数据进行快照并持久化到硬盘。

### RDB触发条件[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#rdb触发条件)

1. 符合自定义配置的快照规则
2. 执行save或者bgsave命令
3. 执行flushall命令
4. 执行主从复制操作

### 在redis.conf中设置自定义快照规则[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#在redis.conf中设置自定义快照规则)

1、RDB持久化条件

　　格式：save <seconds> <changes>

示例:

　　save 900 1：表示15分钟(900秒)内至少1个键更改则进行快照。

　　save 300 10：表示5分钟(300秒)内至少10个键被更改则进行快照。

　　save 60 10000：表示1分钟内至少10000个键被更改则进行快照。

2、配置dir指定rdb快照文件的位置

```
# Note that you must specify a directory here, not a file name.
dir ./
```

3、配置dbfilename指定rdb快照文件的名称

```
# The filename where to dump the DB
dbfilename dump.rdb
```

### 说明[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#说明)

1. Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存
2. 根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将记录1千万个字符串类型键，大小为1GB的快照文件载入到内存中需要花费20-30秒钟。

## 快照的实现原理[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#快照的实现原理)

### 快照过程[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#快照过程)

1. **redis**使用f**ork函数复制**一份**当前进程**的**副本**(子进程)
2. **父进程继续接受**并处理**客户端发来的命令**，而**子进程**开始**将内存中的数据写**入到**硬盘****中**的**临时文件**。
3. 当**子进程写入完**所有**数据后**会**用该临时文件替换旧的RDB文件**，至此，一次快照操作完成。

### 注意[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#注意)

1. redis在进行**快照的过程中不会修改RDB文件**，只有快照结束后才会将旧的文件替换成新的，也就是说任何时候RDB文件都是完整的。
2. 这就使得我们可以通过定时备份RDB文件来实现redis数据库的备份，**RDB文件是经过压缩的二进制文件**，占用的空间会小于内存中的数据，更加利于传输。

## RDB优缺点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#rdb优缺点)

### 缺点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缺点)

　　使用RDB方式实现持久化，一旦redis异常退出，就会**丢失最后一次快照以后更改的所有数据**。这个时候我们就需要根据具体的应用场景，通过组合设置自动快照条件的方式将可能发生的数据损失控制在能够接受范围。如果数据相对来说比较重要，希望将损失降到最小，则可以使用**AOF**方式进行持久化

### 优点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#优点)

　　RDB可以最大化redis的性能：父进程在保存RDB文件时唯一要做的就是fork出一个字进程，然后这个子进程就会处理接下来的所有保存工作，父进程无需执行任何磁盘I/O操作。同时这个也是一个缺点，如果数据集比较大的时候，fork可能比较耗时，造成服务器在一段时间内停止处理客户端的请求。

## AOF方式[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#aof方式)

### 介绍[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#介绍)

　　默认情况下Redis没有开启AOF(append only file)方式的持久化

　　开启AOF持久化后每执行一条会**更改Redis中的数据命令**，Redis就会将该命令写入硬盘中的AOF文件，这一过程显示会**降低Redis的性能**，但大部分下这个影响是能够接受的，另外使用**较快的硬盘可以提高AOF的性能**。

### 配置redis.conf[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#配置redis.conf)

设置appendonly参数为yes

```
appendonly yes
```

AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的

```
dir ./
```

默认的文件名是appendonly.aof，可以通过appendfilename参数修改

```
appendfilename appendonly.aof
```

### AOF重写原理(优化AOF文件)[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#aof重写原理(优化aof文件))

1. Redis可以在AOF文件体积**变得过大**时，自动地**后台对AOF进行重写**
2. 重写后的新AOF文件包含了恢复当前数据集所需的**最小命令集合**。
3. 整个重写操作是绝对安全的，因为Redis在**创建新的AOF文件**的过程中，会**继续将命令追加到现有的AOF文件**里面，即使重写过程中发生停机，现有的AOF文件也不会丢失。而**一旦新AOF文件创建完毕**，Redis就会从**旧AOF**文件**切换到新AOF**文件，并开始对新AOF文件进行追加操作。
4. AOF文件有序地保存了对数据库执行的所有写入操作，这些写入操作以Redis协议的格式保存，因此AOF文件的内容非常容易被人读懂，对文件进行分析(parse)也很轻松。

### 参数说明[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#参数说明)

1. \#auto-aof-rewrite-percentage 100：表示当前aof文件大小超过上次aof文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以启动时aof文件大小为基准。
2. \#auto-aof-rewrite-min-size 64mb：表示限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化

### 同步磁盘数据[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#同步磁盘数据)

　　Redis每次更改数据的时候，aof机制都会将命令记录到aof文件，但是实际上由于**操作系统的缓存机制**，**数据**并**没**有**实时写入到硬盘**，而是**进入硬盘缓存**。**再通过硬盘缓存机制去刷新到保存文件中**。

### 参数说明[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#参数说明)

1. appendfsync always：每次执行写入都会进行同步，这个是最安全但是效率比较低
2. appendfsync everysec：每一秒执行
3. appendfsync no：不主动进行同步操作，由于操作系统去执行，这个是最快但是最不安全的方式

### AOF文件损坏以后如何修复[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#aof文件损坏以后如何修复)

　　服务器可能在程序正在对AOF文件进行写入时停机，如果停机造成AOF文件出错(corrupt)，那么Redis在重启时会拒绝载入这个AOF文件，从而确保数据的一致性不会被破坏。

　　当发生这种情况时，可以以以下方式来修复出错的AOF文件：

　　　　1、为现有的AOF文件创建一个备份。

　　　　2、使用Redis附带的redis-check-aof程序，对原来的AOF文件进行修复。

　　　　3、重启Redis服务器，等待服务器字啊如修复后的AOF文件，并进行数据恢复。

## 如何选择RDB和AOF[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#如何选择rdb和aof)

1. 一般来说，如果对数据的安全性要求非常高的话，应该同时使用两种持久化功能。
2. 如果可以承受数分钟以内的数据丢失，那么可以只使用RDB持久化。
3. 有很多用户都只使用AOF持久化，但并不推荐这种方式：因为定时生成RDB快照(snapshot)非常便于进行数据库备份，并且**RDB恢复数据集的速度也要比AOF恢复的速度要快**。
4. 两种持久化策略可以同时使用，也可以使用其中一种。如果同时使用的话，那么Redis启动时，会**优先使用AOF**文件来还原数据。

# Redis的主从复制[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis的主从复制)

## 什么是主从复制[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么是主从复制)

　　持久性保证了即使redis服务重启也不会丢失数据，因为redis服务重启后将硬盘上持久化的数据恢复到内存中，但是当redis服务器的硬盘损坏了可能导致数据丢失，不过通过redis的主从复制机制旧可以避免这种单点故障，如下图：

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224105716436-1188739390.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224105716436-1188739390.png)

 

 说明：

1. 主redis中的数据有两个副本(replication)即从redis1和从redis2，即使一台redis服务器宕机其他两台redis服务也可以继续提供服务。
2. 主redis中的数据和从redis上的数据保持实时同步，当主redis写入数据时通过主从复制机制会复制到两个从redis服务上。
3. 只有一个主redis，可以有多个从redis。
4. 主从复制不会阻塞master，在同步数据时，master可以继续处理client请求
5. 一个redis可以即是主从，如下图：

**[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224110209678-396170238.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224110209678-396170238.png)**

 

## 主从配置[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#主从配置)

### 主redis配置[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#主redis配置)

　　无需特殊配置

### 从redis配置[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#从redis配置)

　　修改从服务器上的redis.conf文件

```
# slaveof <masterip> <masterport>
slaveof 192.168.31.200 6379
```

　　上边的配置说明当前【从服务器】对应的【主服务器】的ip是192.168.31.200，端口是6379.

## 实现原理[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#实现原理)

1. slave第一次或者重连到master发送一个**SYNC**的命令。
2. master收到SYNC的时候，会做两件事
   1. 执行bgsave(rdb的快照文件)
   2. master会把新收到的修改命令存入到缓冲区

**缺点：没有办法对master进行动态选举**

# Redis Sentinel哨兵机制[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis-sentinel哨兵机制)

## 简介[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#简介)

　　Sentinel(哨兵)进程是用于监控redis集群中Master主服务器工作的状态，在Master主服务器发生故障的时候，可以实现Master和Slave服务器的切换，保证系统的高可用，其已经被集成在redis2.6+的版本中，Redis的哨兵模式到2.8版本之后就稳定了下来。

## 哨兵进程的作用[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#哨兵进程的作用)

1. **监控**(Monitoring)：哨兵(Sentinel)会不断地检查你的Master和Slave是否运作正常。

2. **提醒**(Notification)：当被监控的某个Redis节点出现问题时，哨兵(Sentinel)可以通过API向管理员或者其他应用程序发送通知。

3. **自动故障迁移**

   (Automatic failover)：当一个Master不能正常工作时，哨兵(Sentinel)会开始一次自动故障迁移操作。

   1. 它会将失效Master的其中一个Slave升级为新的Master，并让失效Master的其他Slave改为复制新的Master；
   2. 当客户端视图连接失效的Master时，集群也会向客户端返回新Master的地址，使得集群可以使用现在的Master替换失效的Master。
   3. Master和Slave服务器切换后，Master的redis.conf、Slave的redis.conf和sentinel.conf的配置文件的内容都会发生相应的改变，即Master主服务器的redis.conf配置文件中会多一行Slave的配置，sentinel.conf的监控目标会随之调换。

## 哨兵进程的工作方式[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#哨兵进程的工作方式)

1. 每个Sentinel(哨兵)进程以**每秒钟一次**的频率**向整个集群**中的**Master主服务器**，**Slave从服务器以及其他Sentinel(哨兵)进程**发送一个**PING命令**。
2. 如果一个实例(instance)距离最后一次有效回复PING命令的时间超过down-after-milliseconds选项所指定的值，则这个实例会被Sentinel(哨兵)进程标记为**主观下线**(**SDOWN**)。
3. 如果一个Master主服务器被标记为主观下线(SDOWN)，则正在监视这个Master主服务器的**所有Sentinel(哨兵)**进程要以每秒一次的频率**确认Master主服务器**确实**进入**了**主观下线状态**。
4. 当有**足够数量的Sentinel(哨兵)进程(**大于等于配置文件指定的值)在指定的时间范围内**确认Master主服务器进入了主观下线状态(SDOWN)**，则Master**主服务器**会被标记为**客观下线(ODOWN)**。
5. 在一般情况下，每个Sentinel(哨兵)进程会以每10秒一次的频率向集群中的所有Master主服务器、Slave从服务器发送INFO命令。
6. 当Master主服务器被Sentinel(哨兵)进程标记为客观下线(ODOWN)时，Sentinel(哨兵)进程向下线的Master主服务器的所有Slave从服务器发送INFO命令的频率会从10秒一次改为每秒一次。
7. 若没有足够数量的Sentinel(哨兵)进程同意Master主服务器下线，Master主服务器的客观下线状态就会被移除。若Master主服务器重新向Sentinel(哨兵)进程发送PING命令返回有效回复，Master主服务器的主观下线状态就会被移除。

## 实现[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#实现)

### 修改从机的sentinel.conf[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#修改从机的sentinel.conf)

```
sentinel monitor mymaster  192.168.127.129 6379 1
```

### 启动哨兵服务器[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#启动哨兵服务器)

```
./redis-sentinel sentinel.conf
```

# Redis Cluster集群[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis-cluster集群)

## redis-cluster架构图[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis-cluster架构图)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224143609847-1937041246.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224143609847-1937041246.png)

 

##  架构细节[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 架构细节)

1. 所有的redis节点彼此互联(PING-PING机制)，内部使用二进制协议优化传输速度和带宽。
2. 节点的fail是通过集群中超过半数的节点检测失效时才生效。
3. 客户端与redis节点直连，不需要中间proxy层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。
4. redis-cluster把所有的物理节点映射到[0-16383]slot上，cluster负责维护node<->slot<->value

```
    Redis集群中内置了16384个哈希槽，当需要在Redis集群中放置一个key-value时，redis先对key使用crc16算法算出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16384之间的哈希槽，redis会根据节点数量大致均等的将哈希槽映射到不同节点。
```

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224144810186-1588931493.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224144810186-1588931493.png)

 

## redis-cluster投票：容错 [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#redis-cluster投票：容错 )

**[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224144913406-1314459663.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224144913406-1314459663.png)**

 

1.  集群中所有master参与投票，如果半数以上master节点与其中一个master节点通信超过(cluster-node-timeout)，认为该master节点挂掉。
2. 什么时候整个集群不可用(cluster_state:fail)？
   1. 如果集群任意master挂掉，且当前master没有slave，则集群进入fail状态。也可以理解成集群的[0-16384]slot映射不完全时进入fail状态。
   2. 如果集群超过半数以上master挂掉，无论是否有slave，集群进入fail状态。

# **安装Ruby环境**[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装ruby环境)

## 导读[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#导读)

　　redis集群需要使用**集群管理脚本redis-trib.rb**，它的执行相应依赖ruby环境。

## 安装[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装)

### 安装ruby[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装ruby)

```
yum install ruby
yum install rubygems
```

### 将redis-3.2.9.gen拖近Linux系统[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#将redis-3.2.9.gen拖近linux系统)

### 安装ruby和redis的接口程序redis-3.2.9.gem[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装ruby和redis的接口程序redis-3.2.9.gem)

```
gem install redis-3.2.9.gem
```

### 复制redis-3.2.9/src/redis-trib.rb 文件到/usr/local/redis目录[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#复制redis-3.2.9/src/redis-trib.rb-文件到/usr/local/redis目录)

```
cp redis-3.2.9/src/redis-trib.rb /usr/local/redis/ -r
```

## 安装Redis集群(RedisCluster)[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#安装redis集群(rediscluster))

　　Redis集群最少需要三台主服务器,三台从服务器,端口号分别为7001~7006。

### 创建7001实例，并编辑redis.conf文件，修改port为7001。[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#创建7001实例，并编辑redis.conf文件，修改port为7001。)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224151316949-1760203584.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224151316949-1760203584.png)

 

### 修改redis.conf配置文件，打开Cluster-enable yes [#](https://www.cnblogs.com/chenyanbin/p/12073107.html#修改redis.conf配置文件，打开cluster-enable-yes )

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224151354802-472806930.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224151354802-472806930.png)

 

###  重复以上2个步骤，完成7002~7006实例的创建，注意端口修改[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 重复以上2个步骤，完成7002~7006实例的创建，注意端口修改)

### 启动所有的实例[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#启动所有的实例)

### 创建Redis集群[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#创建redis集群)

```
./redis-trib.rb create --replicas 1 192.168.242.129:7001 192.168.242.129:7002 192.168.242.129:7003 192.168.242.129:7004 192.168.242.129:7005  192.168.242.129:7006
>>> Creating cluster
Connecting to node 192.168.242.129:7001: OK
Connecting to node 192.168.242.129:7002: OK
Connecting to node 192.168.242.129:7003: OK
Connecting to node 192.168.242.129:7004: OK
Connecting to node 192.168.242.129:7005: OK
Connecting to node 192.168.242.129:7006: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.242.129:7001
192.168.242.129:7002
192.168.242.129:7003
Adding replica 192.168.242.129:7004 to 192.168.242.129:7001
Adding replica 192.168.242.129:7005 to 192.168.242.129:7002
Adding replica 192.168.242.129:7006 to 192.168.242.129:7003
M: d8f6a0e3192c905f0aad411946f3ef9305350420 192.168.242.129:7001
   slots:0-5460 (5461 slots) master
M: 7a12bc730ddc939c84a156f276c446c28acf798c 192.168.242.129:7002
   slots:5461-10922 (5462 slots) master
M: 93f73d2424a796657948c660928b71edd3db881f 192.168.242.129:7003
   slots:10923-16383 (5461 slots) master
S: f79802d3da6b58ef6f9f30c903db7b2f79664e61 192.168.242.129:7004
   replicates d8f6a0e3192c905f0aad411946f3ef9305350420
S: 0bc78702413eb88eb6d7982833a6e040c6af05be 192.168.242.129:7005
   replicates 7a12bc730ddc939c84a156f276c446c28acf798c
S: 4170a68ba6b7757e914056e2857bb84c5e10950e 192.168.242.129:7006
   replicates 93f73d2424a796657948c660928b71edd3db881f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join....
>>> Performing Cluster Check (using node 192.168.242.129:7001)
M: d8f6a0e3192c905f0aad411946f3ef9305350420 192.168.242.129:7001
   slots:0-5460 (5461 slots) master
M: 7a12bc730ddc939c84a156f276c446c28acf798c 192.168.242.129:7002
   slots:5461-10922 (5462 slots) master
M: 93f73d2424a796657948c660928b71edd3db881f 192.168.242.129:7003
   slots:10923-16383 (5461 slots) master
M: f79802d3da6b58ef6f9f30c903db7b2f79664e61 192.168.242.129:7004
   slots: (0 slots) master
   replicates d8f6a0e3192c905f0aad411946f3ef9305350420
M: 0bc78702413eb88eb6d7982833a6e040c6af05be 192.168.242.129:7005
   slots: (0 slots) master
   replicates 7a12bc730ddc939c84a156f276c446c28acf798c
M: 4170a68ba6b7757e914056e2857bb84c5e10950e 192.168.242.129:7006
   slots: (0 slots) master
   replicates 93f73d2424a796657948c660928b71edd3db881f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@localhost-0723 redis]#
```

### 命令客户端连接集群[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#命令客户端连接集群)

命令：

```
./redis-cli -h 127.0.0.1 -p 7001 -c


注：-c表示是以redis集群方式进行连接
./redis-cli -p 7006 -c
127.0.0.1:7006> set key1 123
-> Redirected to slot [9189] located at 127.0.0.1:7002
OK
127.0.0.1:7002>
```

### 查看集群的命令[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#查看集群的命令)

#### 查看集群状态

```
127.0.0.1:7003> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:3
cluster_stats_messages_sent:926
cluster_stats_messages_received:926
```

#### 查看集群中的节点

```
127.0.0.1:7003> cluster nodes
7a12bc730ddc939c84a156f276c446c28acf798c 127.0.0.1:7002 master - 0 1443601739754 2 connected 5461-10922
93f73d2424a796657948c660928b71edd3db881f 127.0.0.1:7003 myself,master - 0 0 3 connected 10923-16383
d8f6a0e3192c905f0aad411946f3ef9305350420 127.0.0.1:7001 master - 0 1443601741267 1 connected 0-5460
4170a68ba6b7757e914056e2857bb84c5e10950e 127.0.0.1:7006 slave 93f73d2424a796657948c660928b71edd3db881f 0 1443601739250 6 connected
f79802d3da6b58ef6f9f30c903db7b2f79664e61 127.0.0.1:7004 slave d8f6a0e3192c905f0aad411946f3ef9305350420 0 1443601742277 4 connected
0bc78702413eb88eb6d7982833a6e040c6af05be 127.0.0.1:7005 slave 7a12bc730ddc939c84a156f276c446c28acf798c 0 1443601740259 5 connected
127.0.0.1:7003>
```

### 维护节点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#维护节点)

　　**集群创建完成后可以继续向集群中添加节点**。

#### 添加主节点

##### 添加7007节点作为新节点

命令：**./redis-trib.rb add-node 127.0.0.1:7007 127.0.0.1:7001**

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152243887-1358118837.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152243887-1358118837.png)

 

#### 查看集群节点发现7007已加到集群中 

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152348292-970339234.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152348292-970339234.png)

 

###  hash槽重新分配[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# hash槽重新分配)

　　**添加完主节点需要对主节点进行hash槽分配，这样该主节才可以存储数据**。

### 查看集群中槽占用情况[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#查看集群中槽占用情况)

　　redis集群有16384个槽，集群中的每个节点分配自己槽，通过查看集群节点可以看到槽占用情况。

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152701194-1459087985.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152701194-1459087985.png)

 

####  给刚添加的7007节点分配槽

第一步：连上集群(连接集群中任意一个可用节点都行)

```
./redis-trib.rb reshard 192.168.101.3:7001
```

第二步：输入要分配的槽数量

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152941490-889359303.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224152941490-889359303.png)

 

 **输入500，表示要分配500个槽**

第三步：输入接收槽的节点id

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153030966-1282854940.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153030966-1282854940.png)

 

输入：15b809eadae88955e36bcdbb8144f61bbbaf38fb

ps：这里准备给7007分配槽，通过cluster node查看7007节点id为：

15b809eadae88955e36bcdbb8144f61bbbaf38fb

第四步：输入源节点id

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153244843-1749626343.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153244843-1749626343.png)

 

 输入：all

第五步：输入yes开始移动槽到目标节点id

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153348607-1314284227.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153348607-1314284227.png)

 

 输入：yes

### 添加从节点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#添加从节点)

　　添加7008从节点，将7008作为7007的从节点

命令：

```
./redis-trib.rb add-node --slave --master-id  主节点id   新节点的ip和端口   旧节点ip和端口
```

执行如下命令：

```
./redis-trib.rb add-node --slave --master-id cad9f7413ec6842c971dbcc2c48b4ca959eb5db4  192.168.101.3:7008 192.168.101.3:7001
```

**cad9f7413ec6842c971dbcc2c48b4ca959eb5db4** 是7007结点的id，可通过cluster nodes查看。

#### nodes查看

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153612560-552075539.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153612560-552075539.png)

 

注意：如果原来该节点在集群中的配置信息已经生成到cluster-config-file指定的配置文件中(如果cluster-config-file没有指定则默认为**nodes.conf**)，这时可能会报错 

```
[ERR] Node XXXXXX is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0
```

**解决办法是删除生成的配置文件nodes.conf，删除后再执行./redis-trib.rb add-node指令**

### 查看集群中的节点，刚添加7008为7007的从节点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#查看集群中的节点，刚添加7008为7007的从节点)

[![img](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153929003-1319930718.png)](https://img2018.cnblogs.com/i-beta/1504448/201912/1504448-20191224153929003-1319930718.png)

 

###  删除节点[#](https://www.cnblogs.com/chenyanbin/p/12073107.html# 删除节点)

命令：

```
./redis-trib.rb del-node 127.0.0.1:7005 4b45eb75c8b428fbd77ab979b85080146a9bc017
```

删除已经占用hash槽的节点会失败，报错如下

```
[ERR] Node 127.0.0.1:7005 is not empty! Reshard data away and try again.
```

需要将该节点占用的hash槽分配出去

# Jedis连接集群[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#jedis连接集群)

## 创建JedisCluster类连接Redis集群[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#创建jediscluster类连接redis集群)

```
@Test
public void testJedisCluster() throws Exception {
    //创建一连接，JedisCluster对象,在系统中是单例存在
    Set<HostAndPort> nodes = new HashSet<>();
    nodes.add(new HostAndPort("192.168.242.129", 7001));
    nodes.add(new HostAndPort("192.168.242.129", 7002));
    nodes.add(new HostAndPort("192.168.242.129", 7003));
    nodes.add(new HostAndPort("192.168.242.129", 7004));
    nodes.add(new HostAndPort("192.168.242.129", 7005));
    nodes.add(new HostAndPort("192.168.242.129", 7006));
    JedisCluster cluster = new JedisCluster(nodes);
    //执行JedisCluster对象中的方法，方法和redis一一对应。
    cluster.set("cluster-test", "my jedis cluster test");
    String result = cluster.get("cluster-test");
    System.out.println(result);
    //程序结束时需要关闭JedisCluster对象
    cluster.close();
}
```

## 使用Spring[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#使用spring)

### 配置applicationContext.xml[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#配置applicationcontext.xml)

```
<!-- 连接池配置 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!-- 最大连接数 -->
    <property name="maxTotal" value="30" />
    <!-- 最大空闲连接数 -->
    <property name="maxIdle" value="10" />
    <!-- 每次释放连接的最大数目 -->
    <property name="numTestsPerEvictionRun" value="1024" />
    <!-- 释放连接的扫描间隔（毫秒） -->
    <property name="timeBetweenEvictionRunsMillis" value="30000" />
    <!-- 连接最小空闲时间 -->
    <property name="minEvictableIdleTimeMillis" value="1800000" />
    <!-- 连接空闲多久后释放, 当空闲时间>该值 且 空闲连接>最大空闲连接数 时直接释放 -->
    <property name="softMinEvictableIdleTimeMillis" value="10000" />
    <!-- 获取连接时的最大等待毫秒数,小于零:阻塞不确定的时间,默认-1 -->
    <property name="maxWaitMillis" value="1500" />
    <!-- 在获取连接的时候检查有效性, 默认false -->
    <property name="testOnBorrow" value="true" />
    <!-- 在空闲时检查有效性, 默认false -->
    <property name="testWhileIdle" value="true" />
    <!-- 连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true -->
    <property name="blockWhenExhausted" value="false" />
</bean>
<!-- redis集群 -->
<bean id="jedisCluster" class="redis.clients.jedis.JedisCluster">
    <constructor-arg index="0">
        <set>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7001"></constructor-arg>
            </bean>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7002"></constructor-arg>
            </bean>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7003"></constructor-arg>
            </bean>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7004"></constructor-arg>
            </bean>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7005"></constructor-arg>
            </bean>
            <bean class="redis.clients.jedis.HostAndPort">
                <constructor-arg index="0" value="192.168.101.3"></constructor-arg>
                <constructor-arg index="1" value="7006"></constructor-arg>
            </bean>
        </set>
    </constructor-arg>
    <constructor-arg index="1" ref="jedisPoolConfig"></constructor-arg>
</bean>
```

### 测试代码[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#测试代码)

```
private ApplicationContext applicationContext;
    @Before
    public void init() {
        applicationContext = new ClassPathXmlApplicationContext(
                "classpath:applicationContext.xml");
    }

    // redis集群
    @Test
    public void testJedisCluster() {
        JedisCluster jedisCluster = (JedisCluster) applicationContext
                .getBean("jedisCluster");

        jedisCluster.set("name", "zhangsan");
        String value = jedisCluster.get("name");
        System.out.println(value);
    }
```

# 缓存穿透、缓存击穿、缓存雪崩[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存穿透、缓存击穿、缓存雪崩)

## 缓存数据步骤[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存数据步骤)

1. 查询缓存，如果没有数据，则查询数据库
2. 查询数据库，如果数据不为空，将结果写入缓存

## 缓存穿透[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存穿透)

### 什么叫缓存穿透？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么叫缓存穿透？)

　　一般的缓存系统，都是按照key去缓存查询，如果不存在对应的value，就应该去后端系统查询。如果key对应的value是一定不存在的，并且对key并发请求量很大，就会对后端系统造成很大的压力。这就叫做缓存穿透。

### 如何解决？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#如何解决？)

1. **对查询结果为空的情况也进行缓存，缓存时间设置短一点，或者该key对应的数据insert了之后清楚缓存**。
2. 对一定不存在的key进行过滤。可以把所有的可能存在的key放到一个大的Bitmap中，查询时通过该Bitmap过滤。(布隆表达式)

## 缓存雪崩[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存雪崩)

### 什么叫缓存雪崩？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么叫缓存雪崩？)

　　当缓存服务器重启或者大量缓存集合中某一个时间段失效，这样在失效的时候，也会给后端系统带来很大压力。

### 如何解决？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#如何解决？)

1. 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
2. **不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀**。
3. 做二级缓存，A1为原始缓存，A3为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期。

## 缓存击穿[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存击穿)

### 什么叫缓存击穿？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#什么叫缓存击穿？)

　　对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：“缓存”被击穿的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key。

　　缓存在某个时间点过期的时候，恰好在这个时间点对这个key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

### 如何解决？[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#如何解决？)

　　**使用redis的setnx互斥锁先进行判断，这样其他线程就处于等待状态，保证不会有大并发操作去操作数据库。**

**if(redis.setnx()==1){**

**//查数据库**

**//加入线程**

**}**

# 缓存淘汰策略[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#缓存淘汰策略)

- 当 Redis 内存超出物理内存限制时，内存的数据会开始和磁盘产生频繁的交换 (swap)。交换会让 Redis 的性能急剧下降，对于访问量比较频繁的 Redis 来说，这样龟速的存取效率基本上等于不可用。
- 在生产环境中我们是不允许 Redis 出现交换行为的，为了限制最大使用内存，Redis 提供了配置参数 maxmemory 来限制内存超出期望大小。
- 当实际内存超出 maxmemory 时，Redis 提供了几种可选策略 (maxmemory-policy) 来让用户自己决定该如何腾出新的空间以继续提供读写服务。

## 策略[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#策略)

```
noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。
allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。
allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。
volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。
volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。
volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。
```

## 使用[#](https://www.cnblogs.com/chenyanbin/p/12073107.html#使用)

修改redis.conf的`maxmemory`，设置最大使用内存：

```
maxmemory 1024000
```

修改redis.conf的`maxmemory-policy`，设置redis缓存淘汰机制：

```
maxmemory-policy noeviction
```