[为什么数据库字段要使用NOT NULL？](https://www.cnblogs.com/ilovejaney/p/14619604.html)

基于目前大部分的开发现状来说，我们都会把字段全部设置成`NOT NULL`并且给默认值的形式。

通常，对于默认值一般这样设置：

1. 整形，我们一般使用0作为默认值。
2. 字符串，默认空字符串
3. 时间，可以默认`1970-01-01 08:00:01`，或者默认`0000-00-00 00:00:00`，但是连接参数要添加`zeroDateTimeBehavior=convertToNull`，建议的话还是不要用这种默认的时间格式比较好

但是，考虑下原因，为什么要设置成NOT NULL？

来自高性能Mysql中有这样一段话：

> 尽量避免NULL
>
> 很多表都包含可为NULL（空值）的列，即使应用程序并不需要保存NULL也是如此，这是因为可为NULL是列的默认属性。通常情况下最好指定列为NOT NULL，除非真的需要存储NULL值。
>
> 如果查询中包含可为NULL的列，对MySql来说更难优化，因为可为NULL的列使得索引、索引统计和值比较都更复杂。可为NULL的列会使用更多的存储空间，在MySql里也需要特殊处理。当可为NULL的列被索引时，每个索引记录需要一个额外的字节，在MyISAM里甚至还可能导致固定大小的索引（例如只有一个整数列的索引）变成可变大小的索引。
>
> 通常把可为NULL的列改为NOT NULL带来的性能提升比较小，所以（调优时）没有必要首先在现有schema中查找并修改掉这种情况，除非确定这会导致问题。但是，如果计划在列上建索引，就应该尽量避免设计成可为NULL的列。
>
> 当然也有例外，例如值得一提的是，InnoDB使用单独的位（bit）存储NULL值，所以对于稀疏数据有很好的空间效率。但这一点不适用于MyISAM。

书中的描述说了几个主要问题，我这里暂且抛开MyISAM的问题不谈，这里我针对InnoDB作为考量条件。

1. 如果不设置NOT NULL的话，NULL是列的默认值，如果不是本身需要的话，尽量就不要使用NULL
2. 使用NULL带来更多的问题，比如索引、索引统计、值计算更加复杂，如果使用索引，就要避免列设置成NULL
3. 如果是索引列，会带来的存储空间的问题，需要额外的特殊处理，还会导致更多的存储空间占用
4. 对于稀疏数据又更好的空间效率，稀疏数据指的是**很多值为NULL，只有少数行的列有非NULL值**的情况

### 默认值

对于MySql而言，如果不主动设置为NOT NULL的话，那么插入数据的时候默认值就是NULL。

NULL和NOT NULL使用的空值代表的含义是不一样，NULL可以认为这一列的值是未知的，空值则可以认为我们知道这个值，只不过他是空的而已。

举个例子，一张表中的某一条`name`字段是NULL，我们可以认为**不知道名字是什么**，反之如果是空字符串则可以认为**我们知道没有名字，他就是一个空值**。

而对于大多数程序的情况而言，没有什么特殊需要非要字段要NULL的吧，NULL值反而会对程序造成比如空指针的问题。

对于现状大部分使用`MyBatis`的情况来说，我建议使用默认生成的`insertSelective`方法或者纯手动写插入方法，可以避免新增NOT NULL字段导致的默认值不生效或者插入报错的问题。

### 值计算

**聚合函数不准确**

对于NULL值的列，使用聚合函数的时候会忽略NULL值。

现在我们有一张表，`name`字段默认是NULL，此时对`name`进行`count`得出的结果是1，这个是错误的。

`count(*)`是对表中的行数进行统计，`count(name)`则是对表中非NULL的列进行统计。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6es1zx01j30wa0ka40f.jpg)

**=失效**

对于NULL值的列，是不能使用`=`表达式进行判断的，下面对`name`的查询是不成立的，必须使用`is NULL`。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6ekbjkgjj30n20egmyi.jpg)

**与其他值运算**

NULL和其他任何值进行运算都是NULL，包括表达式的值也是NULL。

`user`表第二条记录`age`是NULL，所以`+1`之后还是NULL，`name`是NULL，进行`concat`运算之后结果还是NULL。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6eopq2baj30pk0l8gnj.jpg)

可以再看下下面的例子，任何和NULL进行运算的话得出的结果都会是NULL，想象下你设计的某个字段如果是NULL还不小心进行各种运算，最后得出的结果。。。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6f78yuiij312m09yaax.jpg)

**distinct、group by、order by**

对于`distinct`和`group by`来说，所有的NULL值都会被视为相等，对于`order by`来说升序NULL会排在最前

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6f1rxml4j30n40we0v8.jpg)

**其他问题**

表中只有一条有名字的记录，此时查询名字`!=a`预期的结果应该是想查出来剩余的两条记录，会发现与预期结果不匹配。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6gcrl138j30n60fomyn.jpg)

### 索引问题

为了验证NULL字段对索引的影响，分别对`name` 和`age`添加索引。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6hidqy3ej30xe0ne775.jpg)

关于网上很多说如果NULL那么不能使用索引的说法，这个描述其实并不准确，根据引用官方文档[3]里描述，使用is NULL和范围查询都是可以和正常一样使用索引的，实际验证的结果好像也是这样，看以下例子。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6hkl143qj31re0jcdj3.jpg)

然后接着我们往数据库中继续插入一些数据进行测试，当NULL列值变多之后发现索引失效了。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6lyhig4wj31ne0u0dmt.jpg)

我们知道，一个查询SQL执行大概是这样的流程：

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6m7q8w1cj30ny07kmxp.jpg)

首先连接器负责连接到指定的数据库上，接着看看查询缓存中是否有这条语句，如果有就直接返回结果。

如果缓存没有命中的话，就需要分析器来对SQL语句进行语法和词法分析，判断SQL语句是否合法。

现在来到优化器，就会选择使用什么索引比较合理，SQL语句具体怎么执行的方案就确定下来了。

最后执行器负责执行语句、有无权限进行查询，返回执行结果。

从上面的简单测试结果其实可以看到，索引列存在NULL就会存在书中所说的导致优化器在做索引选择的时候更复杂，更加难以优化。

### 存储空间

数据库中的一行记录在最终磁盘文件中也是以行的方式来存储的，对于InnoDB来说，有4种行存储格式：`REDUNDANT`、 `COMPACT`、 `DYNAMIC` 和 `COMPRESSED`。

InnoDB的默认行存储格式是`COMPACT`，存储格式如下所示，虚线部分代表可能不一定会存在。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6pbo6xd7j30v602u3yq.jpg)

变长字段长度列表：有多个字段则以逆序存储，我们只有一个字段所有不考虑那么多，存储格式是16进制，如果没有变长字段就不需要这一部分了。

NULL值列表：用来存储我们记录中值为NULL的情况，如果存在多个NULL值那么也是逆序存储，并且必须是8bit的整数倍，如果不够8bit，则高位补0。1代表是NULL，0代表不是NULL。如果都是NOT NULL那么这个就存在了。

ROW_ID：一行记录的唯一标志，没有指定主键的时候自动生成的ROW_ID作为主键。

TRX_ID：事务ID。

ROLL_PRT：回滚指针。

最后就是每列的值。

为了说明清楚这个存储格式的问题，我弄张表来测试，这张表只有`c1`字段是NOT NULL，其他都是可以为NULL的。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6p3brl0vj329f0u0tdx.jpg)

**可变字段长度列表**：`c1`和`c3`字段值长度分别为1和2，所以长度转换为16进制是`0x01 0x02`，逆序之后就是`0x02 0x01`。

**NULL值列表**：因为存在允许为NULL的列，所以`c2,c3,c4`分别为010，逆序之后还是一样，同时高位补0满8位，结果是`00000010`。

其他字段我们暂时不管他，最后第一条记录的结果就是，当然这里我们就不考虑编码之后的结果了。

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6phs9gt6j30wu0970th.jpg)

这样就是一个完整的数据行数据的格式，反之，如果我们把所有字段都设置为NOT NULL，并且插入一条数据`a,bb,ccc,dddd`的话，存储格式应该这样：

![img](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6pnr75t6j31070a0mxx.jpg)

虽然我们发现NULL本身并不会占用存储空间，但是如果存在NULL的话就会多占用一个字节的标志位的空间。

> 文章参考文档：
>
> 1. https://dev.mysql.com/doc/refman/8.0/en/problems-with-null.html
> 2. https://dev.mysql.com/doc/refman/8.0/en/working-with-null.html
> 3. https://dev.mysql.com/doc/refman/5.6/en/is-null-optimization.html
> 4. https://dev.mysql.com/doc/refman/5.6/en/innodb-row-format.html
> 5. https://www.cnblogs.com/zhoujinyi/articles/2726462.html