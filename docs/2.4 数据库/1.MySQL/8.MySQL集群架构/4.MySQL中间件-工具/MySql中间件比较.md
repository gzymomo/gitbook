[TOC]

![](https://upload-images.jianshu.io/upload_images/1552893-80d37d30ad5ffba3.png?imageMogr2/auto-orient/strip|imageView2/2/w/690/format/webp)

# 1、Mycat
前身 Cobar，开源，较为活跃。

特点：
 1. 遵守Mysql原生协议，跨语言，跨数据库的通用中间件代理。
 2. 基于心跳的自动故障切换，支持读写分离，支持 MySQL 一双主多从，以及一主多从
 3. 有效管理数据源连接，基于数据分库，而不是分表的模式。
 4. 基于 NIO 实现，有效管理线程，高并发问题。
 5. 支持数据的多片自动路由与聚合，支持 sum , count , max 等常用的聚合函数。
 6. 支持2表 join，甚至基于 caltlet 的多表 join。
 7. 支持通过全局表，ER 关系的分片策略，实现了高效的多表 join 查询。
 8. 支持多租户方案。
 9. 支持分布式事务（弱xa）
 10. 支持全局序列号，解决分布式下的主键生成问题。
 11. 分片规则丰富，插件化开发，易于扩展。
 12. 强大的 web，命令行监控。
 13. 支持前端作为 MySQL 通用代理，后端 JDBC 方式支持 Oracle、DB2、SQL Server 、 mongodb 。

集群基于 ZooKeeper 管理，在线升级，扩容，智能优化，大数据处理（2.0开发版）。