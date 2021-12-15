[TOC]

# 1、Too many connections（连接数过多，导致连接不上数据库，业务无法正常进行）
问题还原：
```sql
mysql> show variables like '%max_connection%';
| Variable_name   | Value |
| max_connections | 151   |

mysql> set global max_connections=1;Query OK, 0  rows affected (0.00 sec)

[root@node4 ~]# mysql -uzs -p123456 -h 192.168.56.132
ERROR 1040 (00000): Too many connections
```
解决问题的思路：
1. 首先先要考虑在我们 MySQL 数据库参数文件里面，对应的max_connections 这个参数值是不是设置的太小了，导致客户端连接数超过了数据库所承受的最大值。
● 该值默认大小是151，我们可以根据实际情况进行调整。
● 对应解决办法：set global max_connections=500
但这样调整会有隐患，因为我们无法确认数据库是否可以承担这么大的连接压力，就好比原来一个人只能吃一个馒头，但现在却非要让他吃 10 个，他肯定接受不了。反应到服务器上面，就有可能会出现宕机的可能。
所以这又反应出了，我们在新上线一个业务系统的时候，要做好压力测试。保证后期对数据库进行优化调整。
2. 其次可以限制Innodb 的并发处理数量，如果 innodb_thread_concurrency = 0（这种代表不受限制） 可以先改成 16或是64 看服务器压力。如果非常大，可以先改的小一点让服务器的压力下来之后,然后再慢慢增大,根据自己的业务而定。个人建议可以先调整为 16 即可。
MySQL 随着连接数的增加性能是会下降的，可以让开发配合设置 thread pool，连接复用。在MySQL商业版中加入了thread pool这项功能,另外对于有的监控程序会读取 information_schema 下面的表，可以考虑关闭下面的参数
```sql
innodb_stats_on_metadata=0set global innodb_stats_on_metadata=0
```

## 1.2 法二
查看从这次 mysql 服务启动到现在，同一时刻并行连接数的最大值：
`show status like 'Max_used_connections';`

更改最大连接数只能从表面上解决问题，随着我们开发人员的增多，Sleep 连接也会更多，到时候万一又达到了 1000 的上限，难道我们又得改成 10000 吗？这显然是非常不可取的。所以杀掉多余的 Sleep 连接。

**杀掉Sleep连接**
我们可以通过 `show processlist` 命令来查看当前的所有连接状态。

Mysql 数据库有一个属性 wait_timeout 就是 sleep 连接最大存活时间，默认是 28800 s，换算成小时就是 8 小时。
执行命令：
```sql
show global variables like '%wait_timeout';
```
将他修改成一个合适的值，这里我改成了 250s。当然也可以在配置文件中修改，添加 wait_timeout = 250。这个值可以根据项目的需要进行修改，以 s 为单位。我在这里结合 navicat 的超时请求机制配置了 240s。
执行命令：
```sql
set global wait_timeout=250;
```
这样，就能从根本上解决 Too Many Connections 的问题了

# 2、数据库密码忘记的问题
```sql
[root@zs ~]# mysql -uroot -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
[root@zs ~]# mysql -uroot -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```
我们有可能刚刚接手别人的 MySQL 数据库，而且没有完善的交接文档。root 密码可以丢失或者忘记了。

解决思路：
目前是进入不了数据库的情况，所以我们要考虑是不是可以跳过权限。因为在数据库中，mysql数据库中user表记录着我们用户的信息。

解决方法：
启动 MySQL 数据库的过程中，可以这样执行：
```shell
/usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf  --skip-grant-tables &
```
这样启动，就可以不用输入密码，直接进入 mysql 数据库了。然后在修改你自己想要改的root密码即可。
```sql
update mysql.user set password=password('root123') where user='root';
```

# 3、Mysql乱码问题
## 3.1 Mysql乱码问题
博客园：StrongerW：[MySQL乱码问题(为什么？追根溯源)](https://www.cnblogs.com/wubug/p/13388165.html)

win下my.ini、Linux下my.cnf：
```xml
# Win下 my.ini 有的默认被注释掉，只需要去掉注释就可以
#在[client]下追加：
default-character-set=utf8
#在[mysqld]下追加：
character-set-server=utf8
#在[mysql]下追加：
default-character-set=utf8

# Linux下，这里就有所不同，每个人当初安装MySQL的方式,添加的my.cnf
#是否是用的官网模板还是网上复制的内容填充的，但是方式要添加的内容和win
#大同小异，如果当初指定了相应的默认字符集就无需指定字符集。

#【注】无论是my.ini还是my.cnf里面的mysql相关的配置项一定要在所属的组下面，
比如default-character-set就只能放在[mysql]/[client],不能放在[mysqld]下，
不然会导致mysql服务启动失败，可能如下：
#[Error]start Starting MySQL ..  The server quit without updating PID file
# 所以说mysql服务起不来了，可能是配置文件出现了问题
```
最关键的一项是[mysqld] character-set-server=utf8，其它两项，对于my.cnf只需要追加[mysql] character-set-server=utf8就可以改变 character_set_client、character_set_connection、character_set_results这三项的值.

**character_set_client/connection/results变量**
![](https://img-blog.csdnimg.cn/20200726212905318.png)



## 3.2 数据库总会出现中文乱码的情况
解决思路：
对于中文乱码的情况，记住老师告诉你的三个统一就可以。还要知道在目前的mysql数据库中字符集编码都是默认的UTF8

处理办法：
1. 数据终端，也就是我们连接数据库的工具设置为 utf8
2. 操作系统层面；可以通过 cat /etc/sysconfig/i18n 查看；也要设置为 utf8
3. 数据库层面；在参数文件中的 mysqld 下，加入 character-set-server=utf8。
Emoji 表情符号录入 mysql 数据库中报错。
```java
Caused by: java.sql.SQLException: Incorrect string value: '\xF0\x9F\x98\x97\xF0\x9F...' for column 'CONTENT' at row 1
at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:1074)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4096)
at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:4028)
at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2490)
at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2651)
at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2734)
at com.mysql.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:2155)
at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:1379)
```
解决思路：针对表情插入的问题，一定还是字符集的问题。
处理方法：我们可以直接在参数文件中，加入
```shell
vim /etc/my.cnf
[mysqld]
init-connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
```
注：utf8mb4 是 utf8 的超集。

# 4、MySQL 数据库连接超时的报错
```java
org.hibernate.util.JDBCExceptionReporter - SQL Error:0, SQLState: 08S01
org.hibernate.util.JDBCExceptionReporter - The last packet successfully received from the server was43200 milliseconds ago.The last packet sent successfully to the server was 43200 milliseconds ago, which is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection 'autoReconnect=true' to avoid this problem.
org.hibernate.event.def.AbstractFlushingEventListener - Could not synchronize database state with session
org.hibernate.exception.JDBCConnectionException: Could not execute JDBC batch update
com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Connection.close() has already been called. Invalid operation in this state.
org.hibernate.util.JDBCExceptionReporter - SQL Error:0, SQLState: 08003
org.hibernate.util.JDBCExceptionReporter - No operations allowed after connection closed. Connection was implicitly closed due to underlying exception/error:
** BEGIN NESTED EXCEPTION **
```
大多数做 DBA 的同学，可能都会被开发人员告知，你们的数据库报了这个错误了。赶紧看看是哪里的问题。

这个问题是由两个参数影响的，wait_timeout 和 interactive_timeout。数据默认的配置时间是28800（8小时）意味着，超过这个时间之后，MySQL 数据库为了节省资源，就会在数据库端断开这个连接，Mysql服务器端将其断开了，但是我们的程序再次使用这个连接时没有做任何判断，所以就挂了。

解决思路：
先要了解这两个参数的特性；这两个参数必须同时设置，而且必须要保证值一致才可以。
我们可以适当加大这个值，8小时太长了，不适用于生产环境。因为一个连接长时间不工作，还占用我们的连接数，会消耗我们的系统资源。

解决方法：
可以适当在程序中做判断；强烈建议在操作结束时更改应用程序逻辑以正确关闭连接；然后设置一个比较合理的timeout的值（根据业务情况来判断）

# 5、MySql设置InnoDb级别和密码

```yaml
[mysqld]
# Mysql innodb级别
innodb_force_recovery = 6
# 设置免密登录
skip-grant-tables
```

