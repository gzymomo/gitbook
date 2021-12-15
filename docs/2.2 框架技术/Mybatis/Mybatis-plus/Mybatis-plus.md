[从架构师角度谈谈mybatis-plus可能存在的问题](https://www.cnblogs.com/wangjiming/p/14916081.html)

# 1 关于dao层技术选型

在JAVA领域，可选择的ORM框架还是比较多的，如Spring JDBC,JPA,Hibernate,Mybatis,Mybatis-plus等，其中，mybatis可以算得上是领导者，具有极强的领导作用。

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623229714677-e23c774b-ebd8-464d-bc43-cab6ae491427.png)

# 2 深入分析mybatis plus插件原理

## 2.1 资料

https://mp.baomidou.com/guide/

https://github.com/baomidou/mybatis-plus

## 2.2 mybatis 源码架构解析

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623398821917-fbf4fec6-aa78-4737-82e3-54dacec2e545.png)

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623229714681-dce2421c-bf1e-471d-ba90-ce3c61d1b12f.png)

## 2.3 mybatis-plus 架构与源码

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623399139463-aa714764-444e-4ca1-b616-87268e6be665.png)

# 3 mybatis plus 存在的问题

关于mybatis-plus的优点，这里就不论述了，官网说得很详细，详细参考官网：

https://jiansutech.yuque.com/staff-al4og8/drmyey/brma9c

![img](https://img2020.cnblogs.com/blog/1066923/202106/1066923-20210621215645582-1228209931.png)

## 3.1 增加学习成本

mybatis是公认的，被市场验证过的，程序员普遍认同的，java行业统一认同的ORM框架，但mybatis-plus并未经过市场的充分验证，且更谈不上程序员普遍认同

和java行业统一认同，如此，假设A,B,C,D,E,F公司，都用mybatis(因为是公认的，行业默认标准)，而只有A，C公司用mybatis-plus，在这样情况下，A公司外聘

了B,D,E,F公司的技术人员，这些技术人员加入进来，需要学习mybatis-plus插件，增加了学习成本。

## 3.2 增加升级和维护成本

1.mybatis版本变化，mybatis-plus版本是否会相应变化?

2.对于mybatis-plus每个版本，是否充分经过市场验证和证明？若出现不能用或bug，怎么解决？

## 3.3 “百花齐放”现象

在Java开发领域，有这么一个默认共识：在考虑稳定性，可扩展性和生态性前提下，架构越简单，越单纯，越单一，越佳。举几个例子：

**例子1：架构的干净与统一**

你会更喜欢一个只有一种ORM还是有多种ORM的框架，除此之外，考虑一下新人入手框架的学习成本以及代码的维护成本。

**例子2：开发人员的犹豫性**

假设开发人员接手一个多种ORM组合的框架，那么他在开发需求时，他应该选择哪个ORM框架呢？
**例子3：开发人员的排他性**

假设一个只熟悉Hibernate的开发人员接手到一个混合ORM框架吗(暂且定为二混合，即hibernate和mybatis)，当他在开发新需求时，会更倾向于选择hibernate(即使他

知道，在大多数场景下，mybatis性能优于hibernate性能)，而不是mybatis，当你问他为什么不用mybatis时，他肯定会反驳几个让你无法回对的问题,如框架不是有hibernate

吗，照着copy不就ok了？我不熟悉mybatis，刚好框架也有hibernate，方便快捷？没人给我说尽量用mybatis呀，框架有说明么？......

**例子4:  开发人员的自我证明与炫耀性**

你看现在框架里有多种组合，我也搞一种，A说我封装了一个Json类，B说我封装了算法类，C说我封装了字符串类,D说公司有自己通用的类，E说你们这些都不行，你看

现在hutool(对该框架感兴趣，可参官方文档：https://www.hutool.cn/docs/#/)这么齐全，直接用就好。。。，如此，真是“百花齐放”，难以控制。

## 3.4  DB和架构师视角

根据mybatis-plus源码，随机抽取几个功能接口来分析：

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623393086822-5681a327-1575-457d-8ce9-1e5d6ae538de.png)

1  T selectById(Serializable id);
![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623393239793-0b165e56-f1cd-493b-b6dd-7bbe64c3615d.png)

1.基本性能视角：假设用户信息表有100个字段，100w条数据(其实，100w数据是远远不够的，一般有一定业务量的，都是千万级别)，每次都SELECT * FROM user_info，结果会怎样？
2.dba或架构师优化视角：某天，系统很卡，问题反馈到DAB或架构师处，他们均不能直观的看到sql是怎么写的，也没有时间去研究源码，也没时间去反编译，那么他们如何去优化？

3.联合主键视角：某天，发现仅仅通过主键id不能解决性能问题，需要建立联合索引，此时，该如何解决？

4.表Join视角：从用户维度，我们能拆分成很多部分，如用户基本信息表(静态)，用户信息表(动态)，用户组表，用户资产表，用户权限表，用户部门表，

这些表如何进行join查询，该如何解决？表的水平垂直拆分，也是相同道理。

5.代码评审视角：代码评审最重要的是确定代码业务逻辑的准确性，其中包括两部分：(1)代码及代码逻辑  (2)SQL语句，而mybatis-plus将CRUD封装在框架内，

无法直接看到SQL，那么在评审SQL时，如何评审，更恐怖是的是，如果DBA参与评估的话，他看不到SQL，在他那里评估就过不了。

## 3.5  安全视角

​      以用户信息表为例，假设该表有100个字段，其中手机号，身份证，姓名，年龄，资产为其中字段，现在有个需求：需要开一个接口，统计年龄大于30，

资产100w以上的人id,姓名，手机号，会有什么样的问题？

问题一：将不需要的字段用户身份证，用户资产字段暴露在网络传输中，被第三方抓包工具抓取数据，用户安全信息存在隐患。

问题二：若进行DTO转换，是否存在转换成本，这里会有两个性能IO代价，代价一是从DB库捞取数据成本，成本二是DTO成本。

 

## 3.6  不利于架构发展

​      研究过mybatis-plus源码的同学知道，这个封装版框架是将mapper和service捆绑一起的，然而，在微服务架构，中台架构，尤其是DDD驱动设计中，

service和mapper并不是耦合的(否则会造成domain贫血)，对DDD领域感兴趣的同学，可以去研究领域驱动设计**《实现领域驱动设计 （美）弗农著》，**提供DDD简要架构图：

![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623398473324-73cea093-a3c9-4ac0-be00-b26159885d41.png)  ![img](https://cdn.nlark.com/yuque/0/2021/png/2334047/1623398505358-d5b6bff3-9fca-498f-9f8d-76ffee7c8b52.png)

 