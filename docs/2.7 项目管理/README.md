- [SpringBoot 项目模板：摆脱步步搭建](https://mp.weixin.qq.com/s?__biz=MzUxOTc4NjEyMw==&mid=2247499947&idx=2&sn=b218be474b36544fd595a91882414bdd&chksm=f9f6db4fce815259d4f1b1b40d1a1d0c0f86774b93423afbda1d11fc0540d5df7853727f7512&mpshare=1&scene=24&srcid=0117IJW8jD0GUU8eaP1wNV9U&sharer_sharetime=1610865492908&sharer_shareid=63281a6430fc669a5b286c6a03545e04#rd)
- [Gitee项目示例](https://github.com/e-commerce-sample/ecommerce-order-service)



# 项目简介

- 一两句话描述该项目实现的业务功能
- 需求说明文档地址：[XXXX项目需求说明书](http://)


# 环境

- 开发环境
- 本地构建：列出本地开发过程中所用到的工具命令

| 数据库  | 账号，密码  | 备注 |
| ------- | ----------- | ---- |
| ip:port | root,123456 |      |

- 测试环境
- 访问地址：

| 数据库  | 账号，密码  | 备注 |
| ------- | ----------- | ---- |
| ip:port | root,123456 |      |

- 正式环境（正式环境非管理员禁止使用root账户登录数据库）
- 访问地址：

| 数据库  | 账号，密码  | 备注 |
| ------- | ----------- | ---- |
| ip:port | root,123456 |      |


# 部署架构

- 项目部署架构图（拓扑图）


# 测试策略

- 单元测试：核心的领域模型，包括领域对象(比如Order类)，Factory类，领域服务类等；
> 目录：src/test/java


- 组件测试：不适合写单元测试但是又必须测试的类，比如Repository类，在有些项目中，这种类型测试也被称为集成测试；
> 目录：src/componentTest/java


- API测试：模拟客户端测试各个API接口，需要启动程序。
> 目录：src/apiTest/java


# 技术选型

- 项目的技术栈，包括语言，框架，中间件等
- JDK1.8（必须指明JDK版本，JDK版本会导致部分源码不一致）
- MySQL5.7.*（经测试，5.7版本各项性能优于之前版本，部分性能优于8.0.*版本）

# 领域模型

- 核心的领域概念，针对于当前系统所在的领域

# 编码实践

- 统一的编码实践，比如异常处理原则，分页封装等

# FAQ-开发过程中常见问题的解答

- 代码质量规范文档地址：[代码质量规范书](http://)
- 开发部WIKI技术文档地址：[技术开发WIKI](http://)

- JVM参数设定参考

| 参数说明                                        | 推荐值                    | 备注   |
| ----------------------------------------------- | ------------------------- | ------ |
| -Xms - 堆最大大小                               | -Xms1024m                 | 必填   |
| -Xmx - 堆默认大小                               | -Xmx1024m                 | 必填   |
| -Xmn256m - 新生代大小                           | -Xmn256m                  | 非必填 |
| -XX:MetaspaceSize - 元空间默认大小              | -XX:MetaspaceSize=128m    | 非必填 |
| -XX:MaxMetaspaceSize - 元空间最大大小           | -XX:MaxMetaspaceSize=128m | 非必填 |
| -Xss256k - 棧最大深度大小                       | -Xss256k                  | 非必填 |
| -XX:SurvivorRatio=8 - 新生代分区比例 8:2        | -XX:SurvivorRatio=8       | 非必填 |
| -XX:+UseConcMarkSweepGC - 垃圾收集器，CMS收集器 | -XX:+UseConcMarkSweepGC   | 非必填 |
| -XX:+PrintGCDetails - 打印详细的GC日志          | -XX:+PrintGCDetails       | 非必填 |


注意：
​	保持README的持续更新，一些重要的架构决定可以通过示例代码的形式记录在代码块当中，新开发者可以通过直接阅读这些示例代码快速了解项目的通用实践方式以及架构选择。