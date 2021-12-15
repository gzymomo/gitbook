> MySQL 5.7.8 版本开始支持JSON类型，在JSON类型支持之前，通常使用字符串类型存储JSON数据，相对于字符串，原生的JSON类型能够对数据的有效性进行验证。JSON类型独特的内部存储格式和索引，能够更加高效的访问JSON数据节点。另外MySQL提供丰富的JSON类型相关函数，JSON数据的查询与操作更加便捷。

#### 1、JSON类型优势：

- 自动验证JSON数据的有效性，不符合JSON规范的数据将会提示报错。
- 优化存储格式，JSON类型在存储时被转换成一种内部的格式，能够快速地访问JSON文档数据的元素。
- 丰富的JSON类型相关函数，JSON数据操作更加便捷。

#### 2、常用的JSON类型：

- JSON数组，如：[1,2]
- JSON对象，如：{"key":"value"}

**示例：**

创建一个包含JSON类型的表：
create table tb(c json);

插入JSON数据：
insert into tb values('[1,2]');
insert into tb values('{"key":"value"}');

查询表中JSON数据的值：
select c->"$.key" from tb;

查出来的结果是带引号的，想去除引号的话，把->换成->>，如下：
select c->>"$.key" from tb;

#### 3、JSON函数：

- JSON_TYPE()
- JSON_VALID()
- JSON_ARRAY()
- JSON_OBJECT()
- JSON_MERGE()
- JSON_EXTRACT()
- JSON_SET()
- JSON_INSERT()
- JSON_REPLACE()
- JSON_REMOVE()
- JSON_SEARCH()
- JSON_CONTAINS_PATH()

##### 3.1 JSON_TYPE()

JSON_TYPE() 函数返回 JSON数据的类型，JSON数组返回ARRAY，JSON对象返回OBJECT，如下所示：

```
mysql> select JSON_TYPE('[1,2]');
+--------------------+
| JSON_TYPE('[1,2]') |
+--------------------+
| ARRAY              |
+--------------------+
mysql> select JSON_TYPE('{"key":"value"}');
+------------------------------+
| JSON_TYPE('{"key":"value"}') |
+------------------------------+
| OBJECT                       |
+------------------------------+
```

##### 3.2 JSON_VALID()

JSON_VALID() 函数用于判断数据是否为有效的JSON数据，有效返回1，无效返回0。如下所示：

```
mysql> select JSON_VALID('["abc"]');
+-----------------------+
| JSON_VALID('["abc"]') |
+-----------------------+
|                     1 |
+-----------------------+

mysql> select JSON_VALID('{abc}');
+---------------------+
| JSON_VALID('{abc}') |
+---------------------+
|                   0 |
+---------------------+
```

##### 3.3 JSON_ARRAY()

JSON_ARRAY()函数用于构造一个JSON数组，如下：

```
mysql> SELECT JSON_ARRAY('a', 1, NOW());
+----------------------------------------+
| JSON_ARRAY('a', 1, NOW())              |
+----------------------------------------+
| ["a", 1, "2020-02-18 19:09:41.000000"] |
+----------------------------------------+
```

##### 3.4 JSON_OBJECT()

JSON_OBJECT()函数用于构造一个JSON对象，如下：

```
mysql> SELECT JSON_OBJECT('key1', 1, 'key2', 'abc');
+---------------------------------------+
| JSON_OBJECT('key1', 1, 'key2', 'abc') |
+---------------------------------------+
| {"key1": 1, "key2": "abc"}            |
+---------------------------------------+
```

##### 3.5 JSON_MERGE()

JSON_MERGE()函数用于合并两个JSON数据，如下：

```
mysql> SELECT JSON_MERGE('["a", 1]', '{"key": "value"}');
+--------------------------------------------+
| JSON_MERGE('["a", 1]', '{"key": "value"}') |
+--------------------------------------------+
| ["a", 1, {"key": "value"}]                 |
+--------------------------------------------+
```

##### ***\*3.6 JSON_EXTRACT()\****

JSON_EXTRACT()函数用于精准查询JSON数据中的某个元素，比如获取某个key对应的value，如下：

```
mysql> SELECT JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name');
+---------------------------------------------------------+
| JSON_EXTRACT('{"id": 14,"name":"Aztalan"}', '$.name') |
+---------------------------------------------------------+
| "Aztalan"                                               |
+---------------------------------------------------------+
```

##### 3.7 JSON_SET()

JSON_SET()函数用于设置JSON中的某个值，如下：

```
mysql> SELECT JSON_SET('["x","y"]', '$[0]', 'a');
+------------------------------------+
| JSON_SET('["x","y"]', '$[0]', 'a') |
+------------------------------------+
| ["a", "y"]                         |
+------------------------------------+
```

##### 3.8 JSON_INSERT()

JSON_INSERT()函数用于在JSON数据中插入新的值，如下：

```
mysql> SELECT JSON_INSERT('["x","y"]', '$[2]', 'a');
+---------------------------------------+
| JSON_INSERT('["x","y"]', '$[2]', 'a') |
+---------------------------------------+
| ["x", "y", "a"]                       |
+---------------------------------------+
```

##### 3.9 JSON_REPLACE()

JSON_REPLACE()函数用于替换JSON数据中的某个值，示例如下：

```
mysql> SELECT JSON_REPLACE('["x","y"]', '$[0]', 'a');
+----------------------------------------+
| JSON_REPLACE('["x","y"]', '$[0]', 'a') |
+----------------------------------------+
| ["a", "y"]                             |
+----------------------------------------+
```

##### 3.10 JSON_REMOVE()

JSON_REMOVE()函数用于移除JSON数据中的某个值，示例如下：

```
mysql> SELECT JSON_REMOVE('["x","y"]', '$[0]');
+----------------------------------+
| JSON_REMOVE('["x","y"]', '$[0]') |
+----------------------------------+
| ["y"]                            |
+----------------------------------+
```

##### 3.11 JSON_SEARCH()

JSON_SEARCH()函数用于搜索JSON数据中的某个值，参数one表示只搜索符合条件的一个结果，all表示搜索所有结果，示例如下：

```
mysql> SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT JSON_SEARCH(@j, 'one', 'abc');
+-------------------------------+
| JSON_SEARCH(@j, 'one', 'abc') |
+-------------------------------+
| "$[0]"                        |
+-------------------------------+
1 row in set (0.00 sec)

mysql> SELECT JSON_SEARCH(@j, 'all', 'abc');
+-------------------------------+
| JSON_SEARCH(@j, 'all', 'abc') |
+-------------------------------+
| ["$[0]", "$[2].x"]            |
+-------------------------------+
1 row in set (0.00 sec)
```

##### 3.12 JSON_CONTAINS_PATH()

JSON_CONTAINS_PATH()函数用于搜索符合条件的JSON路径，如果搜索到符合条件的路径，返回1，否则返回0。示例如下：

```
mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e') as t;
+------+
| t    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e') as t;
+------+
| t    |
+------+
|    0 |
+------+
1 row in set (0.00 sec)

mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.c.d') as t;
+------+
| t    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a.d') as t;
+------+
| t    |
+------+
|    0 |
+------+
1 row in set (0.00 sec)
```

#### 4、最后

MySQL 支持JSON类型的时间并不算太久，功能还很简单，甚至还有许多未知的Bug，更多MySQL JSON相关资料请参考官方文档：

https://dev.mysql.com/doc/refman/5.7/en/json.html