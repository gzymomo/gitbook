[TOC]

# [JDBC Request](https://www.cnblogs.com/imyalost/p/5947193.html)

Jmeter中取样器（Sampler）是与服务器进行交互的单元。一个取样器通常进行三部分的工作：向服务器发送请求，记录服务器的响应数据和记录响应时间信息

有时候工作中我们需要对数据库发起请求或者对数据库施加压力，那么这时候就需要用到**JDBC Request**

JDBC Request可以向数据库发送一个请求（sql语句），一般它需要配合JDBC Connection Configuration配置元件一起使用

首先，还是先建立一个测试计划，添加线程组

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012152908796-540030749.png)

为了方便，这里线程数我设置为1，然后在线程组上面右键单击选择配置元件→ **JDBC Connection Configuration（JDBC连接配置）**

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012155343906-806209958.png)

 

JDBC Connection Configuration界面如下：

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012161853687-1752227876.png)

**Variable Name（变量名）：**这里写入数据库连接池的名字

**Database URL：**数据库连接地址

**JDBC Driver class：**数据库驱动（可以将需要连接的数据库驱动jar包复制到jmeter的lib/目录下，然后在设置测试计划界面，最下面的Library中导入）

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012162444750-1317114640.png)

**Username：**数据库登录名

**Password：**数据库登陆密码

这里顺带说说不同数据库的驱动类和URL格式：

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012163845984-1752650747.png)

 

设置好JDBC连接配置后，添加JDBC请求，界面如下：

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012164257625-1606549546.png)

**Variable name：**这里写入数据库连接池的名字（和JDBC Connection Configuration名字保持一致 ）

**Query：**里面填入查询数据库数据的SQL语句（填写的SQL语句末尾不要加“；”）

**parameter valus：**数据的参数值

**parameter types：**数据的参数类型

**cariable names：**保存SQL语句返回结果的变量名

**result cariable name：**创建一个对象变量，保存所有返回结果

**query timeout：**查询超时时间

**handle result set：**定义如何处理由callable statements语句返回的结果

 

完成了上面的操作后，就可以添加监听器，来查看我们的请求是否成功了

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012170644140-12896310.png)

这是请求内容，即SQL语句

![img](https://images2015.cnblogs.com/blog/983980/201610/983980-20161012170704250-559440878.png)

这是响应数据，正确的显示了我查询的该表的对应字段的数据