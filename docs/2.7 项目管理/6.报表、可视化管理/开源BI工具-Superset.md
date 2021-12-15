- [开源BI工具-Superset](https://blog.51cto.com/u_15187242/2749157)

## 简介

- Superset的Airbnb开源的数据可视化工具，目前属于Apache孵化器项目，主要用于数据分析师进行数据可视化工作
  - PS，Airbnb在数据方面做的很棒，相关的博客B格也很高，他们的博客名字居然叫『Airbnb Engineering & Data Science』，可见对于数据科学的重视

- 在github上搜索数据可视化，Superset的star数已经远远超过其他可视化工具，文章的最后，我们也会对调研过的可视化工具进行若干对比

  ![目前颜值最高的开源BI工具-Superset_Superset_02](https://s4.51cto.com/images/blog/202104/30/8bf92a710fa06dd982f868c1dc3733c1.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  image

目前支持的图表类型

![目前颜值最高的开源BI工具-Superset_Superset_03](https://s4.51cto.com/images/blog/202104/30/b0217be64c3810c83b5db0003387ac52.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

111

 

![目前颜值最高的开源BI工具-Superset_Superset_04](https://s3.51cto.com/images/blog/202104/30/f68b4432c80f97f584d3297add6cc81f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

## 如何搭建

- 官方项目地址
  - https://github.com/apache/incubator-superset
  - 不推荐直接在服务器上安装，因为依赖的包很多，一方面可能少装，其次是把环境搞乱了，很难维护后续

- 第三方Docker项目
  - https://github.com/amancevice/superset
  - 推荐，使用docker-compose安装，支持SQLLite/PG/MySQL方式的元数据存储
  - 整个Docker包括3部分：1. 元数据 2. Redis缓存 3. Superset本身
  - 以SQLLite为例，简单的安装方式如下：

```
 mkdir /data1/superset 
 cd /data1/superset 

  git clone https://github.com/amancevice/superset.git 

  cd /data1/superset/superset/examples/sqlite 
  mkdir superset
  # 这个是SQLLite的数据文件，映射到Docker内部
  touch superset/superset.db

  # 这一步必须要做，否则Docker可能没有读写权限
  chmod 777 superset/superset.db
  
  # 启动Redis
  docker-compose up -d redis
  # 启动Superset
  docker-compose up -d superset
  # Superset本身启动需要几十秒，需要观察下才能执行下一步
  docker-compose ps   
  # 进行初始化，根据提示设置用户名密码
  docker-compose exec superset superset-demo1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.
```



- 我们考虑到元数据的安全性，就使用了自己的MySQL和Redis服务，基本思路就是使用docker-compose跑起来后，把相关的表结构dump到我们线上，再修改Superset的配置指向

## 问题与坑

### 中文的支持

- 如果你想采用MySQL作为Superset的元数据，请务必修改所MySQL表结构里的charcter，默认是latin1，连中文的Dashboard名字都不支持
  - 多说一句，直接alter table modify是不可以的，需要把数据dump出来，sed修改一下，再灌入『资深DBA友情提醒』
- ClickHouse方面
  - Superset使用的是cloudflare/sqlalchemy-clickhouse驱动，默认是支持中文的，但是在Python2的Superset版本上，会出现中文无法正常解析的问题，所以，如果使用ClickHouse，请使用基于Python3版本的Superset

### 超时问题

- 部分数据分析的SQL，需要很久才能返回，默认的Superset是30秒超时，需要酌情修改
- 在配置文件里修改

```
SUPERSET_WEBSERVER_TIMEOUT = 300
CACHE_DEFAULT_TIMEOUT = 60 * 60 * 24
SQLLAB_TIMEOUT = 3001.2.3.
```



- 我们遇到最慢的可能就是几百秒，如果再慢，就建议把数据做二次提取了

### 默认的limit 5000条

- 在配置文件里修改

```
config.py:ROW_LIMIT = 10000001.
```



### 时区问题

- 使用Docker启动的Superset，请务必修改Docker时区

### 部分画图Bug

- 老版本，地图着色异常，升级新版后解决

- 0.23.2在时间序列堆叠图中，存在Y轴移除的Bug，如图：

  ![目前颜值最高的开源BI工具-Superset_Superset_05](https://s8.51cto.com/images/blog/202104/30/ce1ff93c7967cd79fa7265f2cd5319a8.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  image

- 通过如下方式可以临时解决：

  ![目前颜值最高的开源BI工具-Superset_Superset_06](https://s8.51cto.com/images/blog/202104/30/636984beb1bd000e67aeb0ddff0415c5.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

  image

### 地图渲染

- Superset的地图渲染使用『ISO 3166-1』进行映射，相关文档见：Country Map Tools

## 什么时候用Grafana，什么时候用Superset

- 时间序列，选Grafana
- 数据量很大，用Grafana
- 静态的日报、报表，Superset表现力很好

## 我们的创新

- 有如下一个场景，我们很难以解决：
  - 暂时不开发专门的数据图表系统（其实是有一个的，但是有一些缺陷，而且用户很少主动来门户系统查看数据，都依赖邮件）
  - 邮件发送一个HTML页面嵌入的截图
  - 关键在这个截图的生成上，答案就是，数据分析师使用Superset对各种业务配置Dashboard，后台使用Python定期截图，嵌入到HTML页面，这样，就可以发送一封样式美观大方的数据汇总邮件了
  - 遇到几个问题：
  - 好在大多数老板都Mac系统，忽略
  - 你配置Dashboard别搞那么长不就行了~
  - 发送时要强制刷新，可能会遇到查询超时问题，上面已经说了，如何解决
  - 截图可能会失败：增加重试机制，目前稳定性可以达到4个9
  - Windows的邮件客户端，对长图支持有限制，会导致图片变形
  - 邮件是不支持炫酷的HTML页面的（复杂的CSS样式支持），所以，想要偷懒嵌入一个页面发过去，不可以~
  - 业务繁多，有的希望有饼图，有的希望有时间序列图，有的希望有时间序列堆叠图，这些各式各样的需求很难高效满足
  - 有些数据报表，需要每天使用邮件的方式发送各个产品负责人以及相关老板
  - 有人问，为啥必须发邮件，而不能走系统。
  - 答：你让老板天天登陆系统来看么？未来也许可以，目前的阶段不现实。
  - 邮件可以非常直观的方式，是一种非常友好的通信方式，所以邮件我们必须支持。
  - 那么就引发1个问题：
  - 于是乎，我们相出了如下方案：

## 不足之处

- 权限管理
  - 对于数据这种敏感的东西，实际使用过程中，肯定是各自看各自的数据，你并不希望别人看到你的数据
  - 目前的权限设置比较混乱，官方提供了一个复杂的权限控制，但是并不好用
- 想快速复制一个图表？难，从SQL层面再走一遍吧

## 其他选项

- metabase

  - https://github.com/metabase/metabase

  - 目前不支持ClickHouse

    ![目前颜值最高的开源BI工具-Superset_Superset_07](https://s8.51cto.com/images/blog/202104/30/ebd70a67e5e6038949416843a408e8ea.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- Redash

  - https://github.com/getredash/redash

  - 支持ClickHouse

  - 美观程度相比Superset不够精美

  - 支持简单的报警规则

  - 可以把Dashboard分享出去

  - 支持的图表类型有限

    ![目前颜值最高的开源BI工具-Superset_Superset_08](https://s3.51cto.com/images/blog/202104/30/25378cc8e7eaa88328a5bb556d82c1ba.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- Zeppelin

  - https://github.com/apache/zeppelin

  - 来自Apache项目

  - 支持ClickHouse

  - 炫酷程度2颗星

  - Zeppelin更像是一个notebook，而不是一个单纯的BI工具

    ![目前颜值最高的开源BI工具-Superset_Superset_09](https://s7.51cto.com/images/blog/202104/30/0b6ad97e1d8baa7c35333d0f78cb16b1.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- SQLPad

  - https://github.com/rickbergfalk/sqlpad

    ![目前颜值最高的开源BI工具-Superset_Superset_10](https://s9.51cto.com/images/blog/202104/30/babead3faef2c48612153f7ac71c2f8e.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- Franchise

  - https://github.com/HVF/franchise

    ![目前颜值最高的开源BI工具-Superset_Superset_11](https://s4.51cto.com/images/blog/202104/30/6dde8a171c1069339fd5d9d3a5b197b5.gif?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    68747470733a2f2f6672616e63686973652e636c6f75642f696d616765732f6c616e64696e672d6769662e676966

- CBoard

  - https://github.com/yzhang921/CBoard

  - 国人开发的一款可视化工具

  - 交互设计的不错，但是实际用起来感觉很奇怪

  - Java系

    ![目前颜值最高的开源BI工具-Superset_Superset_12](https://s9.51cto.com/images/blog/202104/30/46db7847dba207bcff368b4b1df1ff3a.gif?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- Davinci

  - 宜信开发的达芬奇，也是Java系

  - 功能还是比较全面的，只是在国内还没有大范围的使用

  - https://github.com/edp963/davinci

    ![目前颜值最高的开源BI工具-Superset_Superset_13](https://s4.51cto.com/images/blog/202104/30/fd3c56129eade6ca5da28ae40fef9086.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

    image

- 商业：
  - 多少商业课程都是在教这个，不过互联网讲究的是免费，怎么会用『????』
  - 国内做的一流的BI工具，我们也接触过，很炫酷，也比较实用，目前不支持ClickHouse
  - FineBI
  - Tableau
- 其他可视化资源：
  - https://github.com/thenaturalist/awesome-business-intelligence
  - https://github.com/topics/business-intelligence?o=desc&s=stars

## 最后

- 目前superset迭代进度很快，建议定期跟进更新
- 部分版本存在无法平滑更新的问题，比如最新的0.25.2版本，元数据增加了很多表，部分表的字段也做了调整，很难100%平滑升级