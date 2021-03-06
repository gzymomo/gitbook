#### 1. 行构造表达式示例

SELECT * FROM t1 WHERE (column1,column2) = (1,1);

其中 (column1,column2) 与 (1,1)就是行构造表达式。

(column1,column2) = (1,1) 这个表达式，在逻辑上，等同于：

column1 = 1 AND column2 = 1

这个也比较好理解，但是，如果把

(column1,column2) > (1,1) 理解为 column1 > 1 AND column2 > 1，那就错了。实际上应该是

column1 > 1 OR ((column1 = 1) AND (column2 > 1))

如果不相信的话，看下面这个案例：

```
mysql> select * from t_test;
+------+------+------+
| c1   | c2   | c3   |
+------+------+------+
|    1 |    1 |    1 |
|    1 |    1 |    0 |
|    1 |    2 |    2 |
|    1 |    2 |   -1 |
+------+------+------+
4 rows in set (0.00 sec)

mysql> select * from t_test where c1=1 AND (c2,c3) > (1,1);
+------+------+------+
| c1   | c2   | c3   |
+------+------+------+
|    1 |    2 |    2 |
|    1 |    2 |   -1 |
+------+------+------+
2 rows in set (0.00 sec)

mysql> select * from t_test where c1 = 1 AND (c2 > 1 OR ((c2 = 1) AND (c3 > 1)));
+------+------+------+
| c1   | c2   | c3   |
+------+------+------+
|    1 |    2 |    2 |
|    1 |    2 |   -1 |
+------+------+------+
2 rows in set (0.00 sec)
```

(c2,c3) > (1,1)，结果把c3为-1的记录也查出来了，是不是与期望有点落差，但事实就是这样。它的逻辑是先比较字段c2，如果c2 > 1，直接返回true，不再比较c3。如果c2=1，再比较字段c3。

#### 2. 行构造表达式优化

关于行构造表达式的优化，主要还是转换成多个单独的条件表达式，由 or and 这类逻辑运算符连接，这样才能最大可能用上索引。