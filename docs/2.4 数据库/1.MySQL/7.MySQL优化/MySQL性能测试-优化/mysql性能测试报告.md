

# 1. 测试目的

对mysql数据库进行基准测试，各性能指标进行定量的、可复现的、可对比的测试。

基准测试可以理解为针对系统的一种压力测试。但基准测试不关心业务逻辑，更加简单、直接、易于测试，

数据可以由工具生成，不要求真实；而压力测试一般考虑业务逻辑()，要求真实的数据。



# 2. **测试环境**

## 2.1 **软件配置**

**![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200315165132226-1867262920.png)**

## 2.1 **硬件配置**

**![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200315165245154-1244337498.png)**

# 3.**测试工具**

## 3.1 **工具介绍sysbench**

​    本次测试采用通用工具SysBench，是跨平台的基准测试工具，支持多线程，支持多种数据库；

对数据库的基准测试的作用，就是分析在当前的配置下（包括硬件配置、OS、数据库设置等） 数据库的性能表现，

从而找出MySQL的性能阈值，并根据实际系统的要求调整硬件配置。

## 3.2 **测试指标**

**TPS**：Transaction Per Second，事务数/秒

一台数据库服务器在单位时间内处理事务的个数，每个事务包含18条SQL语句。

**QPS**：Query Per Second， 查询量/秒

每秒执行的查询次数，是对服务器在规定时间内所处理查询量多少的衡量标准，即数据库每秒执行的SQL数，包含insert、select、update、delete等。

**响应时间**：包括平均响应时间、最小响应时间、最大响应时间、时间百分比等，其中时间百分比参考意义较大，如前95%的请求的最大响应时间。

**并发量**：同时处理的查询请求的数量。

# 4. 安装步骤

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 #cd /opt
 2 #下载sysbench包
 3 #wget -c https://github.com/akopytov/sysbench/archive/1.0.12.zip -O "sysbench-1.0.12.zip"
 4 #安装依赖项
 5 #yum install autoconf libtool mysql mysql-devel vim unzip
 6 #解压文件包
 7 #unzip  sysbench-1.0.12.zip
 8 #编译
 9 #cd sysbench-1.0.12
10 #./autogen.sh
11 #./configure
12 #make
13 #make install
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 上述命令依次执行，安装完成。查找测试脚本所在路径：

 \#find / -name '*oltp.lua*'

 例如：/opt/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua

 然后进入 cd /opt/sysbench-1.0.12，开始测试。

# 5. **参数说明**

sysbenh测试工具命令，根据测试需要调整参数sysbench [options]... [testname] [command]

sysbench --help 查看命令的基本参数

表5-1

| 选项[**options**] | 备注                     |
| ----------------- | ------------------------ |
| --test            | 脚本路径 oltp.lua        |
| --mysql-db        | 测试库名称               |
| --mysql-host      | 数据库IP地址             |
| --mysql-port      | 端口号 默认3306          |
| --mysql-user      | 数据库用户名，一般是root |
| --mysql-password  | 数据库密码               |

在老版本的sysbench中，可以通过--test参数指定测试的脚本；

而在新版本中，--test参数已经声明为废弃，可以不使用--test，而是直接指定脚本。如下两种结果相同

 **![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200315170949761-1776739939.png)**

表5-2

| 测试项[**testname**]     | 备注                                      |
| ------------------------ | ----------------------------------------- |
| --oltp-tables-count      | 建表数量                                  |
| --oltp-table-size        | 每张表的数据量，默认[10000]               |
| --time=N                 | 限制的总执行时间，默认为10s               |
| --threads=N              | 需要使用的线程数，默认为1                 |
| --report-interval=N      | 指定间隔（秒）报告统计信息，默认为0，禁用 |
| --oltp-test-mode=complex | 测试模式，默认complex                     |

**--oltp-test-mode** 运行模式包括：

**simple**  模式下只测试简单的查询；

**nontrx**  不仅测试查询，还测试插入更新等，但是不使用事务；

**complex** 模式下测试最全面，会测试增删改查，而且会使用事务。

备注：如果需要特别测试服务器只读性能，或不使用事务时的性能，可以

选择simple模式或nontrx模式。

表5-3

| 命令[**command**] | 备注         |
| ----------------- | ------------ |
| **prepare**       | 准备测试数据 |
| **run**           | 执行测试     |
| **cleanup**       | 清理数据     |

# 6. **测试方法**

## 6.1 **准备测试数据**

先登陆mysql数据库：

**#**mysql -uroot -p密码 -h 主机ip

执行show global variables统计各参数配置:show global status

创建测试库sbtest:

\>create database sbtest;

查看所有库名:

\>show databases;

**![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200315172810178-1277676711.png)**

exit退出mysql，进入目录 cd /opt/sysbench-1.0.12

开始测试前，首先要生成测试数据，

执行以下命令：

\#sysbench --test=/opt/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua --mysql-db=sbtest --mysql-host=ip --mysql-port=3306 --mysql-user=root --mysql-password=密码 --oltp-test-mode=complex --oltp-tables-count=200 --oltp-table-size=100000 --report-interval=2 prepare

其中，prepare表示准备数据，在的sbtest库生成测试数据，

创建200张表，每张表10万条数据，每2秒显示一次报告。

**注意：**这里不需要设置运行时间（time），执行时间受建表数量影响。

运行过程如下图所示：

![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200316081322602-1626876740.png)

##  6.2 **建表语句**

数据准备阶段建表结构如下：

CREATE TABLE `sbtest` (

`id` INTEGER UNSIGNED NOT NULL AUTO_INCREMENT,

`k` INTEGER UNSIGNED DEFAULT '0' NOT NULL,

`c` CHAR(120) DEFAULT '' NOT NULL,

`pad` CHAR(60) DEFAULT '' NOT NULL,

PRIMARY KEY (`id`)

) ENGINE=InnoDB;

说明：以上SQL无需执行建表，在下面介绍测试方法，数据准备阶段，执行命令时工具自动安照上面描述格式创建表。

## 6.3 执行测试方法

测试命令如下：

\#sysbench ./tests/include/oltp_legacy/oltp.lua --mysql-db=sbtest --mysql-host=ip --mysql-port=3306 --mysql-user=root --mysql-password=密码

--oltp-test-mode=complex --time=60 --threads=1 --report-interval=2 run

其中， run表示运行测试，单线程 运行60秒。

--oltp-tables-count（表个数）

--oltp-table-size（表数据量）

这两个参数用于准备阶段创建表的大小，在运行阶段不受影响，故run阶段可以取消这两个参数，结果不受影响。

输出结果如下图：

![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200316082016110-715891716.png)

由上图可知，单线程下TPS为33，QPS为660；

### 6.3.1 **测试事务包含的SQL**

Sysbench自带测试脚本，路径如下/opt/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua

默认提交的事务中包含18条SQL语句：

主键SELECT语句，10条：

SELECT c FROM ${rand_table_name} where id=${rand_id};

范围SELECT语句，4条：

SELECT c FROM ${rand_table_name} WHERE id BETWEEN ${rand_id_start} AND ${rand_id_end};

SELECT SUM(K) FROM ${rand_table_name} WHERE id BETWEEN ${rand_id_start} AND ${rand_id_end};

SELECT c FROM ${rand_table_name} WHERE id BETWEEN ${rand_id_start} AND ${rand_id_end} ORDER BY c;

SELECT DISTINCT c FROM ${rand_table_name} WHERE id BETWEEN ${rand_id_start} AND ${rand_id_end} ORDER BY c;

UPDATE语句，2条：

UPDATE ${rand_table_name} SET k=k+1 WHERE id=${rand_id}

UPDATE ${rand_table_name} SET c=${rand_str} WHERE id=${rand_id}

DELETE语句，1条：

DELETE FROM ${rand_table_name} WHERE id=${rand_id}

INSERT语句，1条：

INSERT INTO ${rand_table_name} (id, k, c, pad) VALUES (${rand_id},${rand_k},${rand_str_c},${rand_str_pad});

### 6.3.2 **清理测试数据**

\#sysbench ./tests/include/oltp_legacy/oltp.lua --mysql-db=sbtest --mysql-host=ip

--mysql-port=3306 --mysql-user=root --mysql-password=密码 --oltp-tables-count=600 cleanup

其中，cleanup表示清理数据，这里不需要设置运行时间（time），只需设置表的数量即可，

此时需表的数量>=准备阶段表的数量,才能全部清除测试数据，以免对下次结果产生影响；

# 7. **测试用例**

## 7.1 **测试用例（一）**

 

| **测试用例1：创建6张表每表预置1000万行数据测试** |                                                              |
| ------------------------------------------------ | ------------------------------------------------------------ |
| **测试目的**                                     | 对大表进行增删改查场景下的QPS和TPS                           |
| **前置条件**                                     | 7张表，每张表1000万行数据，每次修改并发数                    |
| **步骤**                                         | 1.准备测试数据（8张表会提示空间不足，因为存储只有40G）sysbench--test=/opt/sysbench1.0.12/tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=ip --mysql-port=3306--mysql-user=root --mysql-password=密码 --oltp-test-mode=complex  --oltp-tables-count=7 --oltp-table-size=10000000 --report-interval=2 prepare2.运行测试sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest  --mysql-host=IP --mysql-port=3306--mysql-user=root  --mysql-password=密码--oltp-test-mode=complex --time=600 --threads=32 --report-interval=2 --oltp-tables-count=64 run3.记录结果，并依次增加线程数**threads**分别记录线程数为 1/ 16/ 32/ 64/ 128 ……情况下的指标及资源情况4.清除测试数据sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root  --mysql-password=密码--oltp-tables-count=7 cleanup |
| **参数化变量**                                   | --threads：线程数，增加线程数观察瓶颈                        |
| **获取指标**                                     | 1、**TPS**2、**QPS**3、**总事务数**4、**前95%延迟时间**      |

7.2 **测试用例（二）**

##  

| **测试用例2：创建6张表每表预置100万行数据测试** |                                                              |
| ----------------------------------------------- | ------------------------------------------------------------ |
| **测试目的**                                    | 与用例1进行对比，观察表数据量的多少，对结果是否有影响        |
| **前置条件**                                    | 7张表，每张表10万行数据，每次修改并发数                      |
| **步骤**                                        | 1.准备测试数据sysbench--test=/opt/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root --mysql-password=密码 --oltp-test-mode=complex --oltp-tables-count=7 --oltp-table-size=100000 --report-interval=2 prepare2.运行测试sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root --mysql-password=密码--oltp-test-mode=complex --time=600--threads=32 --report-interval=2 --oltp-tables-count=64 run3.记录结果，并依次增加线程数**threads**分别记录线程数为 1/ 16/ 32/ 64/ 128 ……情况下的指标及资源情况4.清除测试数据sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP--mysql-port=3306 --mysql-user=root--mysql-password=密码 --oltp-tables-count=7 cleanup |
| **参数化变量**                                  | --threads：线程数，增加线程数观察瓶颈                        |
| **获取指标**                                    | 1、**TPS****2、QPS****3、总事务数****4、前95%延迟时间**      |

## 7.3 **测试用例（三）**

| **测试用例3：创建200张表每表预置100万行数据** |                                                              |
| --------------------------------------------- | ------------------------------------------------------------ |
| **测试目的**                                  | 与用例2相比，单表数量一致，增加表的数量进行观察              |
| **前置条件**                                  | 200张表，每张表10万行数据，每次修改并发数                    |
| **步骤**                                      | 1.准备测试数据sysbench--test=/opt/sysbench-1.0.12/tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root --mysql-password=密码 --oltp-test-mode=complex --oltp-tables-count=200 --oltp-table-size=100000 --report-interval=2 prepare2.运行测试sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root --mysql-password=密码--oltp-test-mode=complex --time=600 --threads=128 --report-interval=2 --oltp-tables-count=64 run3.记录结果，并依次增加线程数**threads**分别记录线程数为 1/ 16/ 32/ 64/ 128 ……情况下的指标及资源情况4.清除测试数据sysbench ./tests/include/oltp_legacy/oltp.lua--mysql-db=sbtest --mysql-host=IP --mysql-port=3306--mysql-user=root --mysql-password=密码--oltp-tables-count=200 cleanup |
| **参数化变量**                                | --threads 线程数,增加线程数观察瓶颈                          |
| **获取指标**                                  | 1、**TPS****2、QPS****3、总事务数****4、前95%延迟时间**      |

# 8. **测试结果分析**

## 8.1  **6张表每表预置1000万行数据测试**

### 8.1.1 **测试结果**

![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200316085253844-659548666.png)

| **线程数**              | 1    | 16   | 32   | 64   | 96   | 128  | 160  | 256  | 512  |
| ----------------------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| **TPS(/s)**             |      |      |      |      |      |      |      |      |      |
| **QPS(/s)**             |      |      |      |      |      |      |      |      |      |
| **事务总数（万）**      |      |      |      |      |      |      |      |      |      |
| **前95%延迟时间（ms）** |      |      |      |      |      |      |      |      |      |

### 8.1.2 **测试过程记录**

**1）数据准备阶段：**

![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200316085627029-1057622104.png)

说明：准备完成后，表空间约13G；

**2）执行测试阶段：**

①连接数1：

![img](https://img2020.cnblogs.com/i-beta/1963340/202003/1963340-20200316085741733-579401282.png)

### 8.1.3 **资源情况**

查看云服务自带监控；

### 8.1.4 **结果分析**

根据测试结果进行描述；

 

后

续

省

略

。。。。

 

# 9. **测试结果总结**

**总结**：当前mysql测试环境，在并发线程数达到160左右时，TPS约900/s左右，QPS约1.8万/s左右,95%事务平均延迟达到300ms，

再增加并发数据后，QPS变化不大，但延迟时间明显加长。测试过程整体CPU利用率较高，在达到最大TPS及QPS时此时CPU利用率达到93%以上。