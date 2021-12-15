**业务背景：**

　　最近公司研发了一款对并发要求比较高的web项目，需要对其压力测试，模拟线上可能存在的问题，这个过程中遇到一些很多问题，这里重新梳理一下思路，希望能给遇到同样问题的小伙伴提供一个参考。

**工具描述：**

　　压力工具使用的是：Loadrunner

　　服务器监控使用的是：nmon

　　数据库：oracle

　　web容器：Tomcat + war 

 

项目就好像是一个木桶，性能测试要找到最短的那个板，也就是系统的瓶颈，但是有些部分应用的一些参数配置可能会变成系统的瓶颈。

**圈重点，以下是这个性能测试过程中踩的参数配置的坑**

**1、数据库线程数**

```
oracle是连接数限制的，默认是150个，如果不把这个放开，只需要超过150个连接就开始等待了，等待时间超过了，就认为失败了。开始的时候没有注意到这块，连接数一直上不来，放开了就好了。
查看当前的数据库连接数：select count(*) from v$process ; 
修改数据库最大连接数：alter system set processes = 300 scope = spfile;
修改完之后要重启数据库：
关闭数据库：shutdown immediate;
启动数据库：startup;
另外测试过程需要关注数据库的当前连接数来辅助判断问题
当前的session连接数：select count(*) from v$session

```

**2、tomcat连接数配置**

　tomcat连接数的配置在 tomcat根目录/conf/server.xml中，

```
 <Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443" URIEncoding="UTF-8" maxConnections="200" acceptCount="100"/>
```

  maxConnections：最大连接数

  acceptCount：最大等待数

  tomcat最大连接数取决于maxConnections这个值加上acceptCount这个值，在连接数达到了maxConenctions之后，tomcat仍会保持住连接，但是不处理，等待其它请求处理完毕之后才会处理这个请求。

**3、数据库连接池配置**

  项目使用的是druid连接池，主要关注以下配置，最大连接数以及等待时间

```
  <!-- 配置初始化大小、最小、最大 --> 
  <property name="initialSize" value="1" /> 
  <property name="minIdle" value="1" /> 
  <property name="maxActive" value="10" />
  <!-- 配置获取连接等待超时的时间 --> 
  <property name="maxWait" value="10000" />
```

 

最后补充一点系统性能的优化方案（一知半解，先列一下，后续继续学习）：

1、SQL执行效率太低

这块可以考虑优化sql本身，比如说查询频繁的加索引，插入频繁的去索引，如果都频繁还可以考虑读写分离，加缓存处理。

2、代码发生死锁（调整代码业务逻辑）

在高并发测试中比较容易检查出来的就是死锁的问题，这块需要开发检查自身的代码逻辑，加一些事务锁之类。

这里附上ORACLE查询锁表及解锁的SQL语句：

```
-- 查看锁表进程SQL语句：
select * from vsessiont1,vsession t1, vsessiont1,vlocked_object t2 where t1.sid = t2.SESSION_ID;

-- 查看导致锁表的sql语句是那一条
select l.session_id sid,
s.serial#,
l.locked_mode,
l.oracle_username,
s.user#,
l.os_user_name,
s.machine,
s.terminal,
a.sql_text,
a.action
from vsqlareaa,vsqlarea a, vsqlareaa,vsession s, v$locked_object l
where l.session_id = s.sid
and s.prev_sql_addr = a.address
order by sid, s.serial#;

–- 杀掉锁表进程：
–- 通过上面的查询获取SID和serial#，替换下面的x,y,就可以解除被锁的状态
alter system kill session ‘x,y’;
```