**作者介绍**：薛超，中移物联网数据库运维高级工程师，10年数据库运维经历，2017年加入中移物联网，负责数字化产品部数据库相关工作，专注于数据库优化和推动架构演进；近年来主要研究国产新型数据库，目前所在部门全部业务已经去"O"，在此过程中，积累了大量新型数据库的运维经验。

中移物联网有限公司是中国移动通信集团有限公司出资成立的全资子公司。公司按照中国移动整体战略布局，围绕“物联网业务服务的支撑者、专用模组和芯片的提供者、物联网专用产品的推动者”的战略定位，专业化运营物联网专用网络，设计生产物联网专用模组和芯片，打造车联网、智能家居、智能穿戴等特色产品，开发运营物联网连接管理平台OneLink和物联网开放平台OneNET，推广物联网解决方案，形成了五大方向业务布局和物联网“云-网-边-端 ”全方位的体系架构。

## **业务背景**

车联网是中移物联网的一个非常典型的场景。在这种场景下，我们需要存储车联网设备的轨迹点，还要支持对轨迹进行查询。

轨迹数据有几个典型的特点：

- 高频写入，每天写入约两亿条轨迹数据；
- 低频查询，通常是由用户触发，查询最近几天的轨迹；
- 企业用户有定制轨迹存储周期的需求；
- 需要考虑双活容灾场景；
- 不针对OLAP需求，数据价值密度低。

##  **存储系统演进过程**

我们的存储系统也经历了几个阶段的演进。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wSibyzVcUE2EqhjSxibfgsI1Vx5d0cBc3EMczicfIvmydysZOMr8YgREhqnNQpyNX3WPiclzGJ6REnbcBepNhpggQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



最初我们使用Oracle小型机单表分区存储数据，建立了366个分区，因此也只有一年的存储容量（按分区来删除数据）。使用Oracle时，我们按日期分区，查询的时候带入1-366的分区数字进行查询，如果不清理历史数据，2020.7.21和2021.7.21会在一个分区内，不便于管理。

2017年响应集团去IOE的要求，轨迹数据不能再使用高性能的Oracle，经过研发和运维团队的调研，决定使用Mycat来搭建MySQL集群。

2019年产品对各个子业务提出了不同存储周期的要求，面向行业的业务存储时间要求更长。因此我们按子业务做了拆分，每个子业务使用自己的独立库。但是，这个时候Mycat配置复杂、难以管理的问题就开始浮现了。

到了2020年，运维团队开始调研国产数据库TiDB，为了验证TiDB的写入性能和稳定性，我们开始将轨迹数据同时写入到TiDB。

2020年10月，我们开始重新考虑轨迹存储方案。

## **原有方案的痛点**

轨迹服务的Mycat集群方案自2017年上线后已经稳定运行2年多，但是在实际商用过程中，我们还是发现了很多问题：

- 首先是扩容风险，Mycat本身是一个构建在MySQL之上的比较简单的中间件，并不支持自动扩容；
- 数据删除比较麻烦，如果每个客户都要定制自己的存储年限，很难设计；
- Mycat中间件配置复杂，每个逻辑库都会有大量复杂的配置；
- 整体方案已经被业内淘汰。

之后我们配合运维团队在轨迹存储上尝试了TiDB，经过几个月的写入测试，验证了TiDB的写入性能和运行稳定性。

优点：

- 写入性能强，扩容简单；
- 支持高可用，灾备完善；
- 支持两地多中心。

问题：

- 要求使用SSD，存储成本过高，不适合轨迹存储这种低价值的数据；
- 不能解决行业客户轨迹数据存储周期定制化的需要。

我们不妨再来看一下轨迹数据存储的典型特点：首先是轨迹数据写入量大，每天约2亿条轨迹，峰值写入8000条/秒；其次是企业对数据存储周期有定制化需求（比如1年、3年或者5年等）；最后还要支持多中心双活特性。而传统关系型数据库很难满足这些需求。

轨迹数据本身也很有特点：

- 数据只查询不修改；
- 海量数据写入；
- 查询时间主要集中在最近一段时间；
- 无需事务支持；
- 查询方式简单，设备之间无关联，不需要join查询；
- 客户希望轨迹存储时间足够长。

我们发现，这个特点天然适合时序数据库。

## **时序数据库选型**

我们研究了业界比较有代表性的几款时序数据库产品。

- **InfluxDB**

在成都的资源池团队有实践，在使用社区版的时候遇到了一些问题，而且该版本不支持集群和高可用。

- **Prometheus**

虽然也是主流时序数据库，但是其独特的拉数据方式并不适合我们的场景，而且其“联邦”集群的机制高可用存疑。

- **TDengine**

国产开源的物联网大数据平台。

在参考DB-Ranking上的排名数据和部门运维专家的意见后，我们进行了部分调研。发现市场上虽然时序数据库百花齐放，但是都没有完美的选择。在评审会议结束后，大家决定把重点放到TDengine上，发现这款产品意外的很不错。运维团队主动出击，还得到了涛思数据官方的免费技术支持。

## **TDengine试用初体验**

TDengine部署非常简单，而且写入速度极高，接近硬盘的连续写入性能。TDengine的高效压缩算法，可以节省大量存储空间。SQL语法，使用非常简单。

数据存储周期的定制化需求，也很容易通过TDengine满足。

TDengine的存储效率也非常高，在选型过程中，我们进行了2000万条数据的模拟测试。下面是MySQL表结构，一共22个字段和一个联合索引。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wSibyzVcUE2EqhjSxibfgsI1Vx5d0cBc374WXHgYyPApgEs6AqJpgPLbhib8ng5qUarZ42VdaCpH8MZy92Z8s5gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



下图是插入2000万条数据之后，MySQL和TDengine占用的磁盘空间对比情况。

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wSibyzVcUE2EqhjSxibfgsI1Vx5d0cBc3bNusDvN1ZOpjg1GyZ3icp2ABeO8ibteIEq9Vz5eicwTo0iapp2ibv3HK31g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以发现，在节省存储空间方面，TDengine的优势极为明显。

因此，我们决定将轨迹数据的存储迁移到TDengine。

## **库表设计**

在库表设计上，我们运用了TDengine自动建表的特性，每个终端设备产生的轨迹点位数据在第一次入库的时候自动创建子表，我们只需要建一个database和一个存储轨迹的STable就万事大吉了。

我们使用的STable结构如下：

```sql
create stable device_statushis (    pos_time TIMESTAMP,    sample_time TIMESTAMP,    record_time TIMESTAMP,    online_status SMALLINT,    alarm_status SMALLINT,    pos_method SMALLINT,    pos_precision SMALLINT,    pos_longitude DOUBLE,    pos_latitude DOUBLE,    pos_altitude DOUBLE,    pos_speed FLOAT,    pos_direction FLOAT,    acc_forward FLOAT,    acc_side FLOAT,    acc_verticle FLOAT,    rollover_level SMALLINT,    power_voltage FLOAT,    acc_status SMALLINT,    satellite_num SMALLINT    )    tags(        device_id BINARY(32)    ) ;
```

建表完成之后，我们还要将历史数据迁移到TDengine中。

我们大约有30T的数据需要从Mycat集群迁移到TDengine，因此我们写了一个小工具来完成这项任务，只用了2天左右的时间完成了全部迁移。

系统的整体架构如图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_png/1wSibyzVcUE2EqhjSxibfgsI1Vx5d0cBc3iakA6tE4mB6pwQkxYYicSDrlxns0qKxGeDSpc38ZYzY04oUnG5HpVQiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##  **整体性能** 

在迁移到TDengine之后，性能表现非常不错：写入峰值1.2-1.3w/s ;存储大约只有MySQL的1/7，查询性能也很突出，单设备单日查询在0.1s以内可以返回结果。