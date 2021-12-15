## MySQL备份工具之Xtrabackup 

xtrabackup是percona公司专门针对mysql  数据库开发的一款开源免费的物理备份(热备)工具,可以对innodb和xtradb等事务引擎数据库实现非阻塞(即不锁表)方式的备份,也可以针对myisam等非事务引擎锁表方式备份,是商业备份工具InnoDB Hotbackup的一个很好的替代品。

## 1、介绍

### 1.1 主要特点

- 物理备份工具，拷贝数据文件
- 备份和恢复数据的速度非常快，安全可靠
- 在备份期间执行的事务不会间断，备份`innodb`数据不影响业务
- 备份期间不增加太多数据库的性能压力
- 支持对备份的数据自动校验
- 运行全量，增量，压缩备份及流备份
- 支持在线迁移表以及快速创建新的从库
- 运行几乎所有版本的`mysql`和`maridb`

### 1.2 相关词汇

文件扩展名

| 文件扩展名  | 文件作用说明                                     |
| :---------- | :----------------------------------------------- |
| .idb文件    | 以独立表空间存储的InnoDB引擎类型的数据文件扩展名 |
| .ibdata文件 | 以共享表空间存储的InnoDB引擎类型的数据文件扩展名 |
| .frm文件    | 存放于表相关的元数据(meta)信息及表结构的定义信息 |
| .MYD文件    | 存放MyISAM引擎表的数据文件扩展名                 |
| .MYI文件    | 存放MyISAM引擎表的索引信息文件扩展名             |

名词

- redo日志`redo`日志，也称事务日志，是`innodb`引擎的重要组成部分，作用是记录`innodb`引擎中每一个数据发生的变化信息。主要用于保证`innodb`数据的完整性，以及丢数据后的恢复，同时可以有效提升数据库的`io`等性能。`redo`日志对应的配置参数为`innodb_log_file_size`和`innodb_log_files_in_group`
- Undo日志`Undo`是记录事务的逆向逻辑操作或者向物理操作对应的数据变化的内容，`undo`日志默认存放在共享表空间里面的`ibdata*`文件，和`redo`日志功能不同`undo`日志主要用于回滚数据库崩溃前未完整提交的事务数据,确保数据恢复前后一致。
- LSN`LSN`，全拼`log sequence number`,中文是日志序列号，是一个`64`位的整型数字，`LSN`的作用是记录`redo`日志时，使用`LSN`唯一标识一条变化的数据。
- checkpoint 用来标识数据库崩溃后,应恢复的`redo log`的起始点

### 1.3 XtraBackup备份原理

1. `checkpoint`，记录`LSN`号码
2. `information schema.xxx`备份
3. 拷贝`innoDB`文件，过程中发生的新变化`redo`也会被保存，保存至备份路径
4. `Binlog`只读，`FTWRL`（global read lock）
5. 拷贝`Non InnoDB`，拷贝完成解锁
6. 生成备份相关的信息文件：`binlog`、`LSN`
7. 刷新`Last LSN`
8. 完成备份

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

备份时经历的阶段：

- InnoDB表：

- - 热备份：业务正常发生的时候，影响较小的备份方式
  - `checkpoint`：将已提交的数据页刷新到磁盘，记录一个`LSN`号码
  - 拷贝`InnoDB`表相关的文件(`ibdata1`、`frm`、`ibd`...)
  - 备份期间产生的新的数据变化`redo`也会备份走

- 非InnoDB表：

- - 温备份：锁表备份
  - 触发`FTWRL`全局锁表
  - 拷贝非`InnoDB`表的数据
  - 解锁

再次统计`LSN`号码，写入到专用文件`xtrabackup checkpoint`记录二进制日志位置 所有备份文件统一存放在一个目录下，备份完成

### 1.4 XtraBackup恢复步骤

1. 做恢复前准备
2. 做数据合并，增量和全备份的数据合并
3. 全备数据，先把全备的`redo lo`文件内容和全备数据合并，并且`read only`不进行回滚
4. 把第一次增量的`redo log`变化加载到第一次增量数据再与全量数据做合并
5. 把第二次增量的`redo log`变化加载到第二次增量数据备份，在与全量和第一次增量的合并再进行合并， 最后把脏数据进行提交或回滚
6. 恢复`binlog`的文件内容

## 2、安装

### 2.1 安装依赖包

```
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL libev
```

### 2.2 下载软件并安装

这里使用的是清华源，官方地址下载较慢。官方最新的是`8.0`版本，此版本只适用于`mysql8.0`版本的数据库，所以这里下载支持`mysql5.6`的版本

```
# wget -c https://mirrors.tuna.tsinghua.edu.cn/percona/centos/7/os/x86_64/percona-xtrabackup-24-2.4.18-1.el7.x86_64.rpm
# yum localinstall -y percona-xtrabackup-24-2.4.18-1.el7.x86_64.rpm
```

## 3、全量备份和恢复

### 3.1 前提

- 数据库处于运行状态
- `xtrabackup`能连接上数据库：在`mysql`配置文件`client`下指定`socket`位置标签或者在使用时指定

```
[client]
socket=/tmp/mysql.sock
```

- 读取配置文件`mysqld`下的`datadir`参数

```
[mysqld]
datadir=/usr/local/mysql/data
```

- 开启了`binlog`

```
log-bin = /data/mysql/mysql-bin
binlog_format="ROW"
expire_logs_days=3
```

- `xtrabackup`是服务器端工具，不能远程备份

### 3.2 全备

```
# innobackupex --user=root --password=123456 /backup/xbk/
```

在做全备时为了控制生成的目录名称，可以添加参数`--no-timestamp`并保留日期

```
# innobackupex --user=root --password=123456 --no-timestamp /backup/xbk/full_`date +%F`
```

### 3.3 备份结果

在备份目录下查看备份的文件，除了`mysql`自身的数据文件外，还有这样几个文件

```
# pwd
/backup/xbk/2020-03-25_10-26-16
# ll
...
-rw-r-----. 1 root root       27 Mar 25 10:53 xtrabackup_binlog_info
-rw-r-----. 1 root root      147 Mar 25 10:53 xtrabackup_checkpoints
-rw-r-----. 1 root root      480 Mar 25 10:53 xtrabackup_info
-rw-r-----. 1 root root 31987200 Mar 25 10:53 xtrabackup_logfile
```

- xtrabackup_binlog_info 备份时刻的`binlog`位置 记录的是备份时刻，`binlog`的文件名字和当时的结束的`position`，可以用来作为截取`binlog`时的起点

```
# cat xtrabackup_binlog_info 
mysql-bin.000001        192790323
```

- xtrabackup_checkpoints

- - 备份时刻，立即将已经`commit`过的，内存中的数据页刷新到磁盘`CKPT`开始备份数据，数据文件的`LSN`会停留在`to_lsn`位置
  - 备份时刻有可能会有其他的数据写入，已备走的数据文件就不会再发生变化了
  - 在备份过程中，备份软件会一直监控着`redo`的`undo`，如果一旦有变化会将日志也一并备走，并记录`LSN`到`last_lsn`，从`to_lsn`——>`last_lsn`就是，备份过程中产生的数据变化

```
# cat xtrabackup_checkpoints 
backup_type = full-backuped
from_lsn = 0        # 上次所到达的LSN号(对于全备就是从0开始,对于增量有别的显示方法)
to_lsn = 14194921406   # 备份开始时间(ckpt)点数据页的LSN
last_lsn = 14200504300   # 备份结束后，redo日志最终的LSN
compact = 0
recover_binlog_info = 0
flushed_lsn = 14177446392
```

- xtrabackup_info 备份的全局信息

```
# cat xtrabackup_info 
uuid = c04f3d33-6e43-11ea-9224-005056ac7d7c
name = 
tool_name = innobackupex
tool_command = --user=root --password=... /backup/xbk/
tool_version = 2.4.18
ibbackup_version = 2.4.18
server_version = 5.6.46-log
start_time = 2020-03-25 10:26:16
end_time = 2020-03-25 10:53:05
lock_time = 0
binlog_pos = filename 'mysql-bin.000001', position '192790323'
innodb_from_lsn = 0
innodb_to_lsn = 14194921406
partial = N
incremental = N
format = file
compact = N
compressed = N
encrypted = N
```

- xtrabackup_logfile 备份过程中的`redo`，关联在备份期间对`InnoDB`表产生的新变化

### 3.4 全备份的恢复

恢复流程：

- xbk备份执行的瞬间,立即触发`ckpt`,已提交的数据脏页,从内存刷写到磁盘,并记录此时的`LSN`号
- 备份时，拷贝磁盘数据页，并且记录备份过程中产生的`redo`和`undo`一起拷贝走,也就是`checkpoint LSN`之后的日志
- 在恢复之前，模拟`Innodb`“自动故障恢复”的过程，将`redo`（前滚）与`undo`（回滚）进行应用
- 恢复过程是`cp`备份到原来数据目录下

模拟数据库宕机，删除数据

```
# pkill mysqld
# rm -rf datadir=/usr/local/mysql/data/*
```

`prepare`预处理备份文件，将`redo`进行重做，已提交的写到数据文件，未提交的使用`undo`回滚掉。模拟了`CSR`的过程

```
# innobackupex --apply-log  /backup/xbk/2020-03-25_10-26-16
```

数据恢复并启动数据库

```
# cp -a /backup/xbk/2020-03-25_10-26-16/* /usr/local/mysql/data/
# chown -R mysql.mysql /usr/local/mysql/data/
# /etc/init.d/mysqld start
```

## 4、增量备份和恢复

### 4.1 前提

增量必须依赖于全备 每次增量都是参照上次备份的`LSN`号码（xtrabackup checkpoints），在此基础上变化的数据页进行备份 会将备份过程中产生新的变化的`redo`一并备份走 恢复时增量备份无法单独恢复，必须基于全备进行恢复。必须将所有的增量备份，按顺序全部合并到全备中

### 4.2 增量备份

- 全量备份

```
# innobackupex --user=root --password --no-timestamp /backup/full >&/tmp/xbk_full.log
```

- 第一次模拟新数据变化

```
db01 [(none)]>create database cs charset utf8;
db01 [(none)]>use cs
db01 [cs]>create table t1 (id int);
db01 [cs]>insert into t1 values(1),(2),(3);
db01 [cs]>commit;
```

- 第一次增量备份

```
# innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/full  /backup/inc1 &>/tmp/inc1.log
```

参数：--incremental	增量备份，后面跟要增量备份的路径 --incremental-basedir=DIRECTORY	基目录，增量备份使用，上一次（全备）增量备份所在目录

- 第二次模拟新数据变化

```
db01 [cs]>create table t2 (id int);
db01 [cs]>insert into t2 values(1),(2),(3);
db01 [cs]>commit;
```

- 第二次增量备份

```
# innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/inc1  /backup/inc2  &>/tmp/inc2.log
```

- 第三次模拟新数据变化

```
db01 [cs]>create table t3 (id int);
db01 [cs]>insert into t3 values(1),(2),(3);
db01 [cs]>commit;
db01 [cs]>drop database cs;
```

### 4.3 备份恢复

恢复流程：

- 挂出维护页，停止当天的自动备份脚本
- 检查备份：full+inc1+inc2+最新的完整二进制日志
- 进行备份整理（细节），截取关键的二进制日志（从备份——误删除之前）
- 测试库进行备份恢复及日志恢复
- 应用进行测试无误，开启业务

模拟数据库宕机，删除数据

```
# pkill mysqld
# rm -rf datadir=/usr/local/mysql/data/*
```

确认备份完整性，对比每个备份集中的`checkpoints`文件

全备份的`checkpoints`文件内容如下，可以发现`to_lsn`和`last_lsn`中间相差`9`。这个数字差在`5.7`版本前为`0`，两者相等，在`5.7`版本后开启`GTID`后有了这个差值，作为内部使用。所以如果是满足这个条件，那么可以认为备份期间并没有新的数据修改。同样的，在增量备份的备份集下的文件也是如此，且增量备份`from_lsn`号与相邻的上一个备份的`last_lsn`减去`9`是一致的。

```
# cat full/xtrabackup_checkpoints
backup_type = full-backuped
from_lsn = 0
to_lsn = 337979814
last_lsn = 337979823
compact = 0
recover_binlog_info = 0
# cat inc1/xtrabackup_checkpoints
backup_type = incremental
from_lsn = 337979814
to_lsn = 337985758
last_lsn = 337985767
compact = 0
recover_binlog_info = 0
# cat inc2/xtrabackup_checkpoints
backup_type = incremental
from_lsn = 337985758
to_lsn = 337991702
last_lsn = 337991711
compact = 0
recover_binlog_info = 0
```

合并整理所有（apply-log）备份（full+inc1+inc2）到全备：

- 基础全备整理

`--redo-only`参数表示只应用`redo`，不进行`undo`，防止`LSN`号发生变化，除最后一次的备份合并外都需要加此参数

```
# innobackupex --apply-log --redo-only /data/backup/full
```

- 合并增量到全备中 合并完可以发现每个备份集中的`check_points`文件的`last_lsn`相同，说明合并成功

```
# 合并inc1到full中
# innobackupex --apply-log --redo-only --incremental-dir=/data/backup/inc1 /data/backup/full
# 合并inc2到full中(最后一次增量)
# innobackupex --apply-log  --incremental-dir=/data/backup/inc2 /data/backup/full
```

- 最后一次整理全备

```
# innobackupex --apply-log  /data/backup/full
```

- 数据恢复并启动数据库

```
# cp -a /backup/full/* /usr/local/mysql/data/
# chown -R mysql.mysql /usr/local/mysql/data/
# /etc/init.d/mysqld start
```

- 截取删除时刻 到`drop`之前的 `binlog`

查看最后一次增量备份中的文件内容

```
# cat /data/backup/inc2/xtrabackup_binlog_info
mysql-bin.000020 1629 9b8e7056-4d4c-11ea-a231-000c298e182d:1-19. df04d325-5946-11ea-000c298e182d:1-7
# mysqlbinlog --skip-gtids --start-position=1629 /data/binlog/mysql-bin.000020 >/data/backup/binlog.sql
或
# mysqlbinlog --skip-gtids --include-gtids='9b8e7056-4d4c-11ea-a231-000c298e182d:1-19' /data/binlog/mysql-bin.000020 >/data/backup/binlog.sql
```

登录`mysql`，恢复最后的`sql`

```
Master [(none)]>set sql_log_bin=0;
Master [(none)]>source /data/backup/binlog.sql
Master [(none)]>set sql_log_bin=1;
```

恢复完成。

## 5、生产案例

### 5.1 生产场景

现有一个生产数据库，总数据量`3TB`，共`10`个业务，`10`个库`500`张表。周三上午`10`点，误`DROP`了`taobao.1`业务核心表`20GB`，导致`taobao`库业务无法正常运行。采用的备份策略是：周日`full`全备，周一到周五`inc`增量备份，`binlog`完整 针对此种场景，怎么快速恢复业务，还不影响其他业务？

### 5.2 实现思路

迁移表空间

```
create table t1;
alter table taobao.t1 discard tablespace;
alter table taobao.t1 import tablespace;
```

1、要想恢复单表，需要表结构和数据 首先合并备份到最新的备份 如何获取表结构？借助工具mysqlfrm

```
yum install -y mysql-utilities
```

2、获取建表语句

```
# mysqlfrm —diagnostic t2.frm
create table `t2` (
`id` int(11) default null
) engine=InnoDB;
```

3、进入数据库中创建表

```
create table `t2` (
`id` int(11) default null
) engine=InnoDB;
```

4、丢弃新建的表空间

```
alter table t2 discard tablespace;
```

5、将表中的数据`cp`回数据库数据目录

```
cp t2.ibd /data/23306/xbk/
chown mysql:mysql /data/3306/xbk/t2.ibd
```

6、导入表空间

```
alter table t2 import tablespace;
```

7、切割二进制日志到删库前生成`sql`并导入

## 6、备份脚本

### 6.1 备份用户创建

创建一个专用于备份的授权用户

```
create user 'back'@'localhost' identified by '123456';
grant reload,lock tables,replication client,create tablespace,process,super on *.* to 'back'@'localhost' ;
grant create,insert,select on percona_schema.* to 'back'@'localhost';
```

### 6.2 全量备份

mybak-all.sh

```
#!/bin/bash
#全量备份，只备份一次
#指定备份目录
backup_dir="/bak/mysql-xback"
#检查
[[ -d ${backup_dir} ]] || mkdir -p ${backup_dir}
if [[ -d ${backup_dir}/all-backup ]];then
    echo "全备份已存在"
    exit 1
fi
#命令，需要设置
innobackupex --defaults-file=/etc/my.cnf --user=back --password='123456' --no-timestamp ${backup_dir}/all-backup &> /tmp/mysql-backup.log
tail -n 1  /tmp/mysql-backup.log | grep 'completed OK!'
if [[ $? -eq 0 ]];then
    echo "all-backup" > /tmp/mysql-backup.txt
else
    echo "备份失败"
    exit 1
fi
```

### 6.3 增量备份

mybak-section.sh

```
#!/bin/bash
#增量备份
#备份目录
backup_dir="/bak/mysql-xback"
#新旧备份
old_dir=`cat /tmp/mysql-backup.txt`
new_dir=`date +%F-%H-%M-%S`
#检查
if [[ ! -d ${backup_dir}/all-backup ]];then
    echo "还未全量备份"
    exit 1
fi
#命令
/usr/bin/innobackupex --user=back --password='123456' --no-timestamp --incremental --incremental-basedir=${backup_dir}/${old_dir} ${backup_dir}/${new_dir} &> /tmp/mysql-backup.log
tail -n 1  /tmp/mysql-backup.log | grep 'completed OK!'
if [[ $? -eq 0 ]];then
    echo "${new_dir}" > /tmp/mysql-backup.txt
else
    echo "备份失败"
    exit 1
fi
```

### 6.4 binlog备份

单点，备份`binlog`，要指定备份目录位置和其它变量

```
#!/bin/bash
#
# 注意：执行脚本前修改脚本中的变量
# 功能：cp方式增量备份
#
# 适用：centos6+
# 语言：中文
#
#使用：./xx.sh -uroot -p'123456'，将第一次增量备份后的binlog文件名写到/tmp/binlog-section中，若都没有，自动填写mysql-bin.000001
#过程：增量先刷新binlog日志，再查询/tmp/binlog-section中记录的上一次备份中最新的binlog日志的值
#      cp中间的binlog日志，并进行压缩。再将备份中最新的binlog日志写入。
#恢复：先进行全量恢复，再根据全量备份附带的time-binlog.txt中的记录逐个恢复。当前最新的Binlog日志要去掉有问题的语句，例如drop等。
#[变量]
#mysql这个命令所在绝对路径
my_sql="/usr/local/mysql/bin/mysql"
#mysqldump命令所在绝对路径
bak_sql="/usr/local/mysql/bin/mysqldump"
#binlog日志所在目录
binlog_dir=/usr/local/mysql/data
#mysql-bin.index文件所在位置
binlog_index=${binlog_dir}/mysql-bin.index
#备份到哪个目录
bak_dir=/bak/mysql-binback
#这个脚本的日志输出到哪个文件
log_dir=/tmp/mybak-binlog.log
#保存的天数，4周就是28天
save_day=10
#[自动变量]
#当前年
date_nian=`date +%Y-`
begin_time=`date +%F-%H-%M-%S`
#所有天数的数组
save_day_zu=($(for i in `seq 1 ${save_day}`;do date -d -${i}days "+%F";done))
#开始
/usr/bin/echo >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:开始增量备份" >> ${log_dir}
#检查
${my_sql} $* -e "show databases;" &> /tmp/info_error.txt
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:登陆命令错误" >> ${log_dir}
    /usr/bin/cat /tmp/info_error.txt #如果错误则显示错误信息
    exit 1
fi
#移动到目录
cd ${bak_dir}
bak_time=`date +%F-%H-%M`
bak_timetwo=`date +%F`
#刷新
${my_sql} $* -e "flush logs"
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:刷新binlog失败" >> ${log_dir}
    exit 1
fi
#获取开头和结尾binlog名字
last_bin=`cat /tmp/binlog-section`
next_bin=`tail -n 1 ${binlog_dir}/mysql-bin.index`
echo ${last_bin} |grep 'mysql-bin' &> /dev/null
if [[ $? -ne 0 ]];then
    echo "mysql-bin.000001" > /tmp/binlog-section #不存在则默认第一个
    last_bin=`cat /tmp/binlog-section`
fi
#截取需要备份的binlog行数
a=`/usr/bin/sort ${binlog_dir}/mysql-bin.index | uniq | grep -n ${last_bin} | awk -F':' '{print $1}'`
b=`/usr/bin/sort ${binlog_dir}/mysql-bin.index | uniq | grep -n ${next_bin} | awk -F':' '{print $1}'`
let b--
#输出最新节点
/usr/bin/echo "${next_bin}" > /tmp/binlog-section
#创建文件
rm -rf mybak-section-${bak_time}
/usr/bin/mkdir mybak-section-${bak_time}
for i in `sed -n "${a},${b}p" ${binlog_dir}/mysql-bin.index  | awk -F'./' '{print $2}'`
do
    if [[ ! -f ${binlog_dir}/${i} ]];then
        /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:binlog文件${i} 不存在" >> ${log_dir}
        exit 1
    fi
    cp -rf ${binlog_dir}/${i} mybak-section-${bak_time}/
    if [[ ! -f mybak-section-${bak_time}/${i} ]];then
        /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:binlog文件${i} 备份失败" >> ${log_dir}
        exit 1
    fi
done
#压缩
if [[ -f mybak-section-${bak_time}.tar.gz ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:压缩包mybak-section-${bak_time}.tar.gz 已存在" >> ${log_dir}
    /usr/bin/rm -irf mybak-section-${bak_time}.tar.gz
fi
/usr/bin/tar -cf mybak-section-${bak_time}.tar.gz mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:压缩失败" >> ${log_dir}
    exit 1
fi
#删除binlog文件夹
/usr/bin/rm -irf mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:删除sql文件失败" >> ${log_dir}
    exit 1
fi
#整理压缩的日志文件
for i in `ls | grep "^mybak-section.*tar.gz$"`
   do
    echo $i | grep ${date_nian} &> /dev/null
        if [[ $? -eq 0 ]];then
            a=`echo ${i%%.tar.gz}`
            b=`echo ${a:(-16)}` #当前日志年月日
            c=`echo ${b%-*}`
            d=`echo ${c%-*}`
            #看是否在数组中，不在其中，并且不是当前时间，则删除。
            echo ${save_day_zu[*]} |grep -w $d &> /dev/null
            if [[ $? -ne 0 ]];then
                [[ "$d" != "$bak_timetwo" ]] && rm -rf $i
            fi
        else
            #不是当月的，其他类型压缩包，跳过
            continue
        fi
done
#结束
last_time=`date +%F-%H-%M-%S`
/usr/bin/echo "begin_time:${begin_time}   last_time:${last_time}" >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:增量备份完成" >> ${log_dir}
/usr/bin/echo >> ${log_dir}
```

主从，备份`relay-bin`，要指定备份目录位置和其它变量

```
#!/bin/bash
#
# 注意：执行脚本前修改脚本中的变量
# 功能：cp方式增量备份
#
# 适用：centos6+
# 语言：中文
#
#使用：./xx.sh -uroot -p'123456'
#[变量]
#mysql这个命令所在绝对路径
my_sql="/usr/local/mysql/bin/mysql"
#mysqldump命令所在绝对路径
bak_sql="/usr/local/mysql/bin/mysqldump"
#binlog日志所在目录
binlog_dir=/usr/local/mysql/data
#mysql-bin.index文件所在位置
binlog_index=${binlog_dir}/mysql-bin.index
#备份到哪个目录
bak_dir=/bak/mysql-binback
#这个脚本的日志输出到哪个文件
log_dir=/tmp/mybak-binlog.log
#保存的天数，4周就是28天
save_day=10
#[自动变量]
#当前年
date_nian=`date +%Y-`
begin_time=`date +%F-%H-%M-%S`
#所有天数的数组
save_day_zu=($(for i in `seq 1 ${save_day}`;do date -d -${i}days "+%F";done))
#开始
/usr/bin/echo >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:开始增量备份" >> ${log_dir}
#检查
${my_sql} $* -e "show databases;" &> /tmp/info_error.txt
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:登陆命令错误" >> ${log_dir}
    /usr/bin/cat /tmp/info_error.txt #如果错误则显示错误信息
    exit 1
fi
#移动到目录
cd ${bak_dir}
bak_time=`date +%F-%H-%M`
bak_timetwo=`date +%F`
#创建文件
rm -rf mybak-section-${bak_time}
/usr/bin/mkdir mybak-section-${bak_time}
for i in `ls ${binlog_dir}| grep relay-bin`
do
    cp -rf ${binlog_dir}/${i} mybak-section-${bak_time}/
    if [[ ! -f mybak-section-${bak_time}/${i} ]];then
        /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:binlog文件${i} 备份失败" >> ${log_dir}
        exit 1
    fi
done
#压缩
if [[ -f mybak-section-${bak_time}.tar.gz ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:压缩包mybak-section-${bak_time}.tar.gz 已存在" >> ${log_dir}
    /usr/bin/rm -irf mybak-section-${bak_time}.tar.gz
fi
/usr/bin/tar -cf mybak-section-${bak_time}.tar.gz mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) error:压缩失败" >> ${log_dir}
    exit 1
fi
#删除binlog文件夹
/usr/bin/rm -irf mybak-section-${bak_time}
if [[ $? -ne 0 ]];then
    /usr/bin/echo "time:$(date +%F-%H-%M-%S) info:删除sql文件失败" >> ${log_dir}
    exit 1
fi
#整理压缩的日志文件
for i in `ls | grep "^mybak-section.*tar.gz$"`
   do
    echo $i | grep ${date_nian} &> /dev/null
        if [[ $? -eq 0 ]];then
            a=`echo ${i%%.tar.gz}`
            b=`echo ${a:(-16)}` #当前日志年月日
            c=`echo ${b%-*}`
            d=`echo ${c%-*}`
            #看是否在数组中，不在其中，并且不是当前时间，则删除。
            echo ${save_day_zu[*]} |grep -w $d &> /dev/null
            if [[ $? -ne 0 ]];then
                [[ "$d" != "$bak_timetwo" ]] && rm -rf $i
            fi
        else
            #不是当月的，其他类型压缩包，跳过
            continue
        fi
done
#结束
last_time=`date +%F-%H-%M-%S`
/usr/bin/echo "begin_time:${begin_time}   last_time:${last_time}" >> ${log_dir}
/usr/bin/echo "time:$(date +%F-%H-%M-%S) info:增量备份完成" >> ${log_dir}
/usr/bin/echo >> ${log_dir}
```