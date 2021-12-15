# [JDBC Request之Query Type](https://www.cnblogs.com/imyalost/p/6498029.html)

工作中遇到这样一个问题：

需要准备10W条测试数据，利用jmeter中的JDBC Request向数据库中批量插入这些数据（只要主键不重复就可以，利用函数助手中的**Random**将主键的ID末尾五位数随机插入）；

响应数据报错：Can not issue data manipulation statements with executeQuery().后来查阅了很多资料，才发现跟JDBC Request中的Query Type类型选择有关；

最后得出的结论是：如果SQL语句是update、insert等更新语句，应该使用statement的execute()方法；如果使用statement的executeQuery()就会出现报错。

**PS：**之前的博客有简单介绍JDBC Request怎样使用，传送门：[**http://www.cnblogs.com/imyalost/p/5947193.html**](http://www.cnblogs.com/imyalost/p/5947193.html)

 

下面主要说jmeter的JDBC Request请求中的**Query Type**：

JDBC Request界面如下：

![img](https://images2015.cnblogs.com/blog/983980/201703/983980-20170303114140907-2147233133.png)

其中Query Type（SQL语句类型）包含十个类型，每个类型作用都不同，下面分别介绍。

**1、Select statement**

这是一个查询语句类型；如果JDBC Request中的Query内容为**一条**查询语句，则选择这种类型。

**PS：**多个查询语句(不使用参数的情况下)可以放在一起顺序执行，需要设置Query Type为：Callable Statement；

  如果Query Type为：select Statement，则只执行第一条select语句。

 

**2、Update statement**

这是一个更新语句类型（包含insert和update）；如果JDBC Request中的Query内容为**一条**更新语句，则选择这种类型。

**PS：**如果该类型下写入多条update语句，依然只执行第一条（原因同上，具体下面介绍）。

 

**3、Callable statement**

这是一个可调用语句类型，CallableStatement 为所有的 DBMS 提供了一种以标准形式调用已储存过程的方法。

已储存过程储存在数据库中，对已储存过程的调用是 CallableStatement 对象所含的内容。

这种调用是用一种换码语法来写的，有两种形式：一种形式带结果参数，另一种形式不带结果参数；结果参数是一种输出 (OUT) 参数，是已储存过程的返回值。

两种形式都可带有数量可变的输入（IN 参数）、输出（OUT 参数）或输入和输出（INOUT 参数）的参数，问号将用作参数的占位符。 

在 JDBC 中调用已储存过程的语法如下所示。注意，方括号表示其间的内容是可选项；方括号本身并不是语法的组成部份。 

{call 过程名[(?, ?, ...)]}，返回结果参数的过程的语法为： {? = call 过程名[(?, ?, ...)]}；

不带参数的已储存过程的语法类似：{call 过程名}。

更详细的使用方法可参考这篇文章：[**http://blog.csdn.net/imust_can/article/details/6989954**](http://blog.csdn.net/imust_can/article/details/6989954)

 

**4、Prepared select statement**

statement用于为一条SQL语句生成执行计划（这也是为什么select statement只会执行第一条select语句的原因），如果只执行一次SQL语句，statement是最好的类型；

Prepared statement用于绑定变量重用执行计划，对于多次执行的SQL语句，Prepared statement无疑是最好的类型（生成执行计划极为消耗资源，两种实现速度差距可能成百上千倍）；

**PS：**PreparedStatement的第一次执行消耗是很高的. 它的性能体现在后面的重复执行。

更详细的解释请参考这一篇文章：[**http://blog.csdn.net/jiangwei0910410003/article/details/26143977**](http://blog.csdn.net/jiangwei0910410003/article/details/26143977)

 

**5、Prepared update statement**

Prepared update statement和Prepared select statement的用法是极为相似的，具体可以参照第四种类型。

 

**6、Commit**

commit的意思是：将未存储的SQL语句结果写入数据库表；而在jmeter的JDBC请求中，同样可以根据具体使用情况，选择这种Query类型。

 

**7、Rollback**

rollback指的是：撤销指定SQL语句的过程；在jmeter的JDBC请求中，同样可以根据需要使用这种类型。

 

**8、AutoCommit(false)**

MySQL默认操作模式就是autocommit自动提交模式。表示除非显式地开始一个事务，否则每条SQL语句都被当做一个单独的事务自动执行；

我们可以通过设置autocommit的值改变是否是自动提交autocommit模式；

而AutoCommit(false)的意思是AutoCommit（假），即将用户操作一直处于某个事务中，直到执行一条commit提交或rollback语句才会结束当前事务重新开始一个新的事务。

 

**9、AutoCommit(true)**

这个选项的作用和上面一项作用相反，即：无论何种情况，都自动提交将结果写入，结束当前事务开始下一个事务。

 

**10、编辑（${}）**

jmeter中的JDBC请求中的SQL语句是无法使用参数的，比如： SELECT * FROM ${table_name} 是无效的。

如果需实现同时多个不同用户使用不同的SQL，可以通过把整条SQL语句参数化来实现；（把SQL语句放在csv文件中，然后在JDBC Request的Query 中使用参数代替 ${SQL_Statement}）。