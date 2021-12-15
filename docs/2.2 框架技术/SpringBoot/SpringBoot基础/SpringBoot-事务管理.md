[TOC]

# 1、事务管理简介 
一组业务操作ABCD，要么全部成功，要么全部不成功。

## 1.1 特性：ACID
- 原子性：整体
- 一致性：完成
- 隔离性：并发
- 持久性：结果

## 1.2 隔离问题
- 脏读：一个事务读到另一个事务没有提交的数据
- 不可重复读：一个事务读到另一个事务已提交的数据（update）
- 虚读(幻读)：一个事务读到另一个事务已提交的数据（insert）

## 1.3 隔离级别
- read uncommitted：读未提交。
- read committed：读已提交。解决脏读。
- repeatable read：可重复读。解决：脏读、不可重复读。
- serializable ：串行化。都解决，单事务。

# 2、Spring事务管理
## 2.1 顶级接口
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvDVGH3pGWT1uWocNCl2c6JeLhAzwAJASmZzhtUVyXOrESVm9sgCRO59BK9TutHc9hScxgVogj2ljA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1. PlatformTransactionManager
平台事务管理器，spring要管理事务，必须使用事务管理器进行事务配置时，必须配置事务管理器。

2. TransactionDefinition
事务详情（事务定义、事务属性），spring用于确定事务具体详情，
例如：隔离级别、是否只读、超时时间 等
进行事务配置时，必须配置详情。spring将配置项封装到该对象实例。

3. TransactionStatus
事务状态，spring用于记录当前事务运行状态。例如：是否有保存点，事务是否完成。
spring底层根据状态进行相应操作。

## 2.2 事务状态
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvDVGH3pGWT1uWocNCl2c6JejQv3ng5Xa5LZEEZVbfjeVlqGuGOfPObH0ybgkIUxWmWByTjDhge9fw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.3 事务定义
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvDVGH3pGWT1uWocNCl2c6Jes1xPYvyfRcv42ob0jSTfibicR288e72cVbCqj6rUV0yo25iaiaQvj6YNhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```java
PROPAGATION_REQUIRED , required , 必须  【默认值】
    支持当前事务，A如果有事务，B将使用该事务。
    如果A没有事务，B将创建一个新的事务。
PROPAGATION_SUPPORTS ，supports ，支持
    支持当前事务，A如果有事务，B将使用该事务。
    如果A没有事务，B将以非事务执行。
PROPAGATION_MANDATORY，mandatory ，强制
    支持当前事务，A如果有事务，B将使用该事务。
    如果A没有事务，B将抛异常。
PROPAGATION_REQUIRES_NEW ， requires_new ，必须新的
    如果A有事务，将A的事务挂起，B创建一个新的事务
    如果A没有事务，B创建一个新的事务
PROPAGATION_NOT_SUPPORTED ，not_supported ,不支持
    如果A有事务，将A的事务挂起，B将以非事务执行
    如果A没有事务，B将以非事务执行
PROPAGATION_NEVER ，never，从不
    如果A有事务，B将抛异常
    如果A没有事务，B将以非事务执行
PROPAGATION_NESTED ，nested ，嵌套
    A和B底层采用保存点机制，形成嵌套事务。
掌握：PROPAGATION_REQUIRED、PROPAGATION_REQUIRES_NEW、PROPAGATION_NESTED
```