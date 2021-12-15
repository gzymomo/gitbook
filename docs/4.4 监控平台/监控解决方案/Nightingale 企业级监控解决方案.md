夜莺（Nightingale）是一个企业级监控解决方案。旨在满足云原生时代企业级的监控需求。Nightingale 在产品完成度、系统高可用、以及用户体验方面，达到了企业级的要求，可满足不同规模用户的场景，小到几台服务，大到数十万都可以完美支撑。兼顾云原生和裸金属，支持应用监控和系统监控，插件机制灵活，插件丰富完善，具有高度的灵活性和可扩展性。

Nightingale 在 [Open-Falcon](https://github.com/open-falcon) 的基础上，结合滴滴内部的最佳实践，在性能、可维护性、易用性方面做了大量的改进，作为集团统一的监控解决方案，支撑了滴滴内部数十亿监控指标，覆盖了从系统、容器、到应用等各层面的监控需求，周活跃用户数千。五年磨一剑，取之开源，回馈开源。

![img](https://static.oschina.net/uploads/img/202003/23102919_4Wnp.png)

Nightingale 采用树状节点导航，我们称之为对象树。对象树本质上是一种对监控对象的分组管理机制，方便查找和查看监控对象，以及对监控对象设置监控策略等管理动作。 一棵典型的树可从上到下描述为组织架构关系、产品服务模块关系、机房和机器挂载关系，该导航树可根据用户需求自行灵活定制。

![img](https://static.oschina.net/uploads/img/202003/23102919_axij.png)

监控策略应用到某个节点后，该节点下的所有子节点挂载的所有的机器都会应用这个策略，任何一台机器触发相关阈值都会产生告警。

![img](https://static.oschina.net/uploads/img/202003/23102919_G5Rb.png)

监控大盘的定制做了大幅易用性改进，支持了图表阈值，支持了图表分类，新增图表和排序管理都是可见即所得的方式，巡检大盘的定制从此不再是困难。

Nightingale 是在 Open-Falcon 的基础上衍化发展而来，Open-Falcon 作为国内使用最广泛的监控解决方案之一，为 Nightingale 的设计开发提供了大量的借鉴意义。

#### 与 Open-Falcon 的不同点

- **告警引擎重构**：Open-Falcon 的告警策略，在监控数据推送上来的同时会触发策略判断，这种「推」的模式优势是策略的判断时效性非常高，但是不利于更高级的告警策略的支持和扩展，比如多条件的组合报警就很难支持。Nightingale 转为推拉结合模式，通过推模式保证大部分策略判断的效率，通过拉模式支持了与条件告警和nodata告警。
- **引入了导航对象树**：将 Open-Falcon 采用的扁平 HostGroup，转为 Nightingale 的导航对象树，对象树本质上是一种对监控对象的分组管理机制，方便查找和查看监控对象，以及对监控对象设置监控策略等管理动作。 同时在 Nightingale 中，去除了告警模板的概念，告警策略直接与树节点绑定，简化设计，大幅提升灵活度和易用性。
- **索引模块升级换代**：Open-Falcon 使用 MySQL 存储 metrics 的索引数据，在扩展性和灵活性上存在瓶颈。Nightingale 根据监控需求，设计开发了全新的内存索引模块 index，查询方式更多样，查询效率更高，避免了原来 MySQL 索引数据达到亿级别时面临的维护优化工作。
- **时序数据库优化**：在 Open-Falcon 存储模块 Graph 的基础上，引入 Facebook 的 Gorilla 压缩方案，近期几个小时的数据采用内存存储，大幅提升数据查询效率，长期数据仍然使用 rrdtool 数据格式存储在硬盘上。同时进一步完善了时序数据库的性能和稳定性。
- **告警引擎高可用改进**：告警引擎 judge 模块通过心跳机制做到了故障自动摘除，再也不用担心单个 judge 宕机导致部分策略失效，需要人工介入的问题，index 模块也是采用类似方式保证可用性。
- **原生内置日志监控功能**：Nightingale 客户端原生内置了日志匹配和指标抽取能力，在 web 控制台页面上支持了日志匹配规则的配置，同时也支持读取目标机器特定目录下的配置文件的方式，让业务指标监控更为易用。
- **可运维性增强**：将 portal (falcon-plus 中的 api)、uic、dashboard、hbs、alarm 合并为一个模块：monapi，简化了系统整体部署难度，原来的部分模块间调用变成进程内方法调用，性能更高。
- **配置文件中心化**：配置文件做了易用性改造，抽取数据库通用配置到 mysql.yml，抽取端口实例地址等关联配置到 address.yml，大批配置在代码里给了默认值，使得配置文件更清晰，易于维护。

#### 与 Open-Falcon 的相同点

- 数据模型没有变化，仍然是 metric、endpoint、tags 的组织方式，agent 基本是可以复用的，Nightingale 中的 agent 叫 collector，融合了原来 Open-Falcon 的 agent 和 falcon-log-agent 的逻辑，各种监控插件也都是可以复用的。
- 数据流向和整体处理逻辑是类似的，仍然使用灵活的推模型，分为数据存储和告警判断两条链路。

#### Nightingale 架构[ ](https://n9e.didiyun.com/blog/2020/02/29/nightingale-v1.0.0-release/#nightingale架构)

![img](https://static.oschina.net/uploads/img/202003/23102919_kPCa.png)

- collector即agent，可以采集机器常见指标，原生支持日志监控，支持插件机制，支持业务通过接口直接上报数据；
- transfer提供rpc接口接收collector上报的数据，然后通过一致性哈希，将数据转发给多台tsdb和多台judge；
- tsdb即open-falcon中的graph组件，用于存储历史数据，支持配置为双写模式提升系统容灾能力，tsdb会把监控数据转发一份给index建索引；
- index是内存索引模块，替换原来的mysql方案，在内存里构建索引，便于后续数据检索，在检索的灵活性和检索性能方面大幅提升；
- judge是告警引擎，从monapi(portal)同步监控策略，然后对接收到的数据做告警判断，如满足阈值，则生成告警事件推送到redis队列；
- monapi(alarm)从redis队列中读取judge生成的事件，进行二次处理，补充一些元信息，生成告警消息，重新推送回redis队列；
- 各发送组件，比如mail-sender、sms-sender等，从redis读取告警消息，发送告警，抽象出各类sender是为了后续定制方便；
- monapi集成了原来多个模块的功能，提供接口给js调用，api前缀为/api/portal，数据查询走transfer，去除了 open-falcon 中原来的query组件，api前缀为/api/transfer，索引查询的api前缀/api/index，于是，在前端统一搭建nginx，即可通过不同location将请求转发到不同后端；
- 数据库仍然使用MySQL，主要存储的内容包括：用户信息、团队信息、树节点信息、告警策略、监控大盘、屏蔽策略、采集策略、部分组件心跳信息等；

#### 仍在进行中的工作[ ](https://n9e.didiyun.com/blog/2020/02/29/nightingale-v1.0.0-release/#仍在进行中的工作)

1. 提供监控指标聚合组件，现在的架构可以解决机器级、模块级的监控，但是集群维度的监控指标，是需要聚合整个集群的所有模块、机器的指标，做一些加和、求平均之类的操作，相关聚合组件，我们在紧锣密鼓的开源过程中；
2. 与k8s无缝集成的工作，也在进行之中；
3. 完善更多监控插件，之前Open-Falcon社区里的很多插件都是可以直接用的，我们会尽量补充社区没有的插件，并对社区已有的插件，进行二次整理和维护，让Nightingale周边更完善；

#### 联系我们[ ](https://n9e.didiyun.com/blog/2020/02/29/nightingale-v1.0.0-release/#联系我们)

- 我们的官网是 [https://n9e.didiyun.com](https://n9e.didiyun.com/)，相关文档会首发于此。
- 您可以在 [Github](https://github.com/didi/nightingale) 上关注 [Nightingale](https://github.com/didi/nightingale)，欢迎您试用和参与社区。
- 您可以通过[滴滴云的夜莺镜像](http://n9e.didiyun.com/docs/install/didiyun/)，一键安装和体验。

#### 致谢和说明

- [Open-Falcon](http://open-falcon.org/) 是小米运维团队开源的企业级监控解决方案，在国内广泛使用。
- [Nightingale](http://n9e.didiyun.com/) 采用 Apache-2.0 开源协议，Copyright © 滴滴 2020。