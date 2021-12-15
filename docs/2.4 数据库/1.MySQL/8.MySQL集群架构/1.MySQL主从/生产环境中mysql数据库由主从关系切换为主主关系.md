- [生产环境中mysql数据库由主从关系切换为主主关系](https://www.cnblogs.com/zhoujishan/p/14631999.html)



### 一、清除原从数据库数据及主从关系

#### 1.1、关闭主从数据库原有的主从关系

从库停止salve

```
mysql> stop salve;
```

查看主从关系连接状态，确保IO线程和SQL线程停止运行

```
mysql> show slave status\G;
......
             Slave_IO_Running: NO
            Slave_SQL_Running: NO
......
```

#### 1.2、清除从数据库原有数据

从库删除原有tsc数据库

```
mysql> drop database tsc;
```

从库重新创建tsc数据库

```
mysql> create database tsc;
```

### 二、将主库上的数据备份到从库

#### 2.1、备份主库数据到从库

从库备份主库数据到本地，该过程主库不会锁表。

```
[root@mysql-2 ~]# mysqldump -umaster -pSenseTime#2020 -h 192.168.116.128 --single-transaction tsc > /root/tsc.sql;
```

查询备份到从库本地tsc.sql文件的大小,保证数据量充足。

```
[root@mysql-2 ~]# ll /root/tsc.sql -h
```

#### 2.2、在从库使用tsc.sql文件恢复主库数据

查看是否存在tsc数据库

```
mysql> show databases;
```

进入tsc数据库

```
mysql> use tsc;
```

在从数据库上开始备份数据

```
mysql> source /root/tsc.sql;
```

备份完成后，检查从库tsc库中的数据表数量

```
mysql> show tables;
```

检查各数据表中的数据条数

```
mysql> select count(*) from tb_fever_treatment;
+----------+
| count(*) |
+----------+
|    50056 |
+----------+
1 row in set (0.01 sec)
```

### 三、建立主主关系

#### 3.1、修改数据库配置文件并重启生效

修改主库mysql配置文件

```
[root@mysql-1 ~]# vi /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

log_bin=master-bin
binlog_format=MIXED
server_id=1
gtid-mode=on
enforce-gtid-consistency
skip-name-resolve
skip-host-cache
#在原有配置上增加以下内容
auto-increment-offset=2
auto-increment-increment=2
relay_log=mysql-relay-bin
log-slave-updates=on
```

修改从库mysql配置文件

```
[root@mysql-2 ~]# vi /etc/my.cnf

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

log_bin=master-bin
binlog_format=MIXED
server_id=2
gtid-mode=on
enforce-gtid-consistency
skip-name-resolve
skip-host-cache
#在原有配置上增加以下内容
auto-increment-offset=2
auto-increment-increment=2
relay_log=mysql-relay-bin
log-slave-updates=on
```

重启主数据库并验证。

```
[root@mysql-1 ~]# systemctl restart mysqld
[root@mysql-1 ~]# systemctl status mysqld
```

重启从数据库并验证。

```
[root@mysql-2 ~]# systemctl restart mysqld
[root@mysql-2 ~]# systemctl status mysqld
```

#### 3.2、建立数据库主主关系

##### 以mysql-1为主、mysql2为从建立主从关系

查看mysql-1的master状态，记录下file和position的值

```
mysql> show master status\G;
*************************** 1. row ***************************
             File: master-bin.000522
         Position: 438
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 37f084e8-9161-11eb-abf6-000c29f882cb:12-13,
87366c38-9164-11eb-988a-000c293c71e0:1-14,
87366c38-9164-11eb-988a-000c293c71e1:1-10
1 row in set (0.00 sec)
```

在mysql-2上建立主从关系

```
mysql> change master to master_host='192.168.116.128',master_port=3306,master_user='master',master_password='SenseTime#2020',master_log_file='master-bin.000522',master_log_pos=438;
```

在mysql-2上打开主从关系

```
mysql> start slave;
```

在mysql-2上查看主从关系，当IO线程和SQL线程都在运行时则主从建立成功

```
mysql> show slave status\G;
......
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
......
```

##### 以mysql-2为主、mysql-1为从建立主从关系

查看mysql-2的master状态，记录下file和position的值

```
mysql> show master status\G;
*************************** 1. row ***************************
             File: master-bin.002054
         Position: 1948
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 37f084e8-9161-11eb-abf6-000c29f882cb:12-13,
87366c38-9164-11eb-988a-000c293c71e0:1-13,
87366c38-9164-11eb-988a-000c293c71e1:1-10
1 row in set (0.00 sec)
```

在mysql-1上建立主从关系

```
mysql> change master to 
master_host='192.168.116.129',master_port=3306,master_user='master',master_password='SenseTime#2020',master_log_file='master-bin.002054',master_log_pos=1948;
```

在mysql-1上打开主从关系

```
mysql> start slave;
```

在mysql-1上查看主从关系，当IO线程和SQL线程都在运行时则主从建立成功

```
mysql> show slave status\G;
```

#### 3.3、主主关系建立失败回退方案

若主主关系建立失败，则撤销主主关系，mysql-1对外提供服务不受影响，user用户无感知。

在mysql-1上运行:

```
mysql> stop slave;
```

在mysql-2上运行:

```
mysql> stop slave;
```

### 四、开启keepalived服务，为主主数据库创建VIP

#### 4.1、配置mysql-1 keepalived服务的配置文件

```
[root@mysql-1 ~]# vi /etc/keepalived/keepalived.conf 

! configuration File for keepalived

global_defs {
    router_id master
}


vrrp_script check_mysql {
    script "/etc/keepalived/check_mysql_second.sh"
    interval 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 150
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.116.150
    }
    track_script {
        check_mysql
    }
}
```

配置mysql-1的mysqld健康状态脚本并赋予权限

```
[root@mysql-1 ~]# vi /etc/keepalived/check_mysql_second.sh
#!/bin/bash
counter=$(ss -antlp | grep 3306 | wc -l)
if [ "${counter}" -eq 0 ]; then
    /usr/bin/systemctl stop keepalived.service
fi

[root@mysql-1 ~]# chmod 755 /etc/keepalived/check_mysql_second.sh
```

#### 4.2、配置mysql-2的 keepalived服务的配置文件

```
[root@mysql-2 ~]# vi /etc/keepalived/keepalived.conf

! configuration File for keepalived

global_defs {
    router_id master
}


vrrp_script check_mysql {
    script "/etc/keepalived/check_mysql_second.sh"
    interval 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 50
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.116.150
    }
    track_script {
        check_mysql
    }
}
```

配置mysql-2的mysqld健康状态脚本并赋予权限

```
[root@mysql-2 ~]# vi /etc/keepalived/check_mysql.sh
#!/bin/bash
counter=$(ss -antlp | grep 3306 | wc -l)
if [ "${counter}" -eq 0 ]; then
    /usr/bin/systemctl stop keepalived.service
fi

[root@mysql-1 ~]# chmod 755 /etc/keepalived/check_mysql_second.sh
```

#### 4.3、分别在mysql-1、mysql-2上启动keepalived服务

在mysql-1上启动keepalived服务并检查状态

```
[root@mysql-1 ~]# systemctl start keepalived
[root@mysql-1 ~]# systemctl status keepalived
```

在mysql-2上启动keepalived服务并检查状态

```
[root@mysql-2 ~]# systemctl start keepalived
[root@mysql-2 ~]# systemctl status keepalived
```

#### 4.4、检查mysql-1、mysql-2的VIP使用情况

正常情况下VIP要求落在mysql-1上，当mysql-1的mysql退出后，VIP落到mysql-2上，即使mysql-1恢复后也补抢占VIP。

检查mysql-1的VIP使用情况

```
[root@mysql-1 ~]# ip address
```

检查mysql-2的VIP使用情况

```
[root@mysql-2 ~]# ip address
```

#### 4.5、切换客户端访问数据库的IP地址为VIP

需改需要访问mysql程序的配置文件，访问地址由mysql-1的IP（本例中为192.168.116.128）改为VIP（本例中为192.168.116.150）。

### 五、数据库主从架构变更为主主架构的结果验证

#### 5.1、高可用特性验证

数据主主架构的高可用，是指当有client连接mysql集群时，一台mysql主机停服后，另一台mysql能够无缝切换对外提供服务，client侧无感知或感知不强烈。

模拟mysql client用户连接mysql，创建服务器虚拟机（主机名为）user。

在user上访问数据库的VIP，并使用mysql服务。

```
[root@user ~]# mysql -umaster -pSenseTime#2020 -h 192.168.116.150
mysql> select UUID();
+--------------------------------------+
| UUID()                               |
+--------------------------------------+
| 8aa2591c-94ef-11eb-93de-000c29309533 |
+--------------------------------------+
1 row in set (0.01 sec)
```

在mysql-1上停止mysqld服务,并查看IP，发现mysql-1上已经没有VIP

```
[root@mysql-1 ~]# systemctl stop mysqld
[root@mysql-1 ~]# ip address
```

此时mysql-1上的mysqld已经停止运行，但在user服务器上仍然能够正常访问mysql服务，查看UUID，发现UUID已经变化，说明keepalived实现了VIP在mysql-1和mysql-2上的无缝切换。

```
mysql> select UUID();
+--------------------------------------+
| UUID()                               |
+--------------------------------------+
| f78cfe1b-94ef-11eb-9622-000c293c71e0 |
+--------------------------------------+
1 row in set (0.02 sec)
```

恢复mysql-1上的mysqld服务后，发现VIP仍然在mysql-2上，实现了VIP的非抢占特性。

#### 5.2、一致性特征验证

一致性特征是指，mysql client在一个数据库上进行数据操作后，其他数据库上能够同步发生相应的数据变化。

在mysql-1上，对数据库进行数据插入操作。

```
mysql> INSERT INTO tb_fever_treatment(`id`,`temp`,`device_id`) VALUES(123293,35.3,'123213213213');
Query OK, 1 row affected (0.00 sec)
```

在mysql-2上，能够查询到对应的数据。

```
mysql> select id,temp,device_id from tb_fever_treatment where id=123293;
+--------+------+--------------+
| id     | temp | device_id    |
+--------+------+--------------+
| 123293 | 35.3 | 123213213213 |
+--------+------+--------------+
1 row in set (0.00 sec)
```

#### 5.3、数据完整性验证

数据完整性是指在mysql主主建立的过程中，mysql-2是否能够百分百的同步到mysql-1上的数据。

在mysql-1上查询表中的行数：

```
mysql> use tsc;select count(*) from tb_fever_treatment;
Database changed
+----------+
| count(*) |
+----------+
|    50056 |
+----------+
1 row in set (0.04 sec)
```

在mysql-2上查询表中的行数:

```
mysql> use tsc;select count(*) from tb_fever_treatment;
Database changed
+----------+
| count(*) |
+----------+
|    50053 |
+----------+
1 row in set (0.03 sec)
```

发现mysql-2上的数据比mysql-1上的少，即mysql-2上的数据和mysql-1上的数据同步不完整。

由于在生产环境上进行主主改造，需要尽量保证mysql-1服务的连续性，故在[2.1、备份主库数据到从库](https://www.cnblogs.com/zhoujishan/p/14631999.html#2.1、备份主库数据到从库)章节中，备份主库数据时，加入了“--single-transaction”选项，即备份过程不锁表，此时当有数据在主库写入时，无法备份到文件中去，因此在从库使用备份文件同步主库数据时，存在部分数据丢失。

### 六、数据库崩溃恢复方案

生产环境对数据库操作存在一定风险，尽管本文已经对操作步骤进行了一定的规范和操作示例，但为了最大程度保证数据安全，需要预备一套数据库崩溃后的数据恢复方案。

#### 6.1、数据恢复前置条件

- mysqld服务可以正常启动
- 数据库备份文件完整，假设备份文件路径为/root/mysqlDump/databaseDump.sql。

#### 6.2、数据库恢复操作

mysql安装并正常启动后（次试数据库没有数据），根据原有数据文件对数据库进行恢复。

创建数据库:

```
mysql> create database tsc;
```

对数据库进行备份:

```
mysql> use tsc;
mysql> source /root/mysqlDump/databaseDump.sql;
```

检查数据条数，出现如下结果说明数据恢复成功。

```
mysql> use tsc;select count(*) from tb_fever_treatment;
Database changed
+----------+
| count(*) |
+----------+
|    50053 |
+----------+
1 row in set (0.03 sec)
```

#### 6.3验证数据库恢复后的可用性

在需要使用mysql服务的客户端上（本文为user服务器），连接mysql服务并查询。

```
[root@user ~]# mysql -umaster -pSenseTime#2020 -h192.168.116.150 --execute='use tsc;select count(*) from tb_fever_treatment';
mysql: [Warning] Using a password on the command line interface can be insecure.
+----------+
| count(*) |
+----------+
|    50053 |
+----------+
```

正常显示数据条数，说明客户端可以正常查询数据库，数据库恢复成功。