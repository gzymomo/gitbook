## [从单体架构到微服务架构、代码拆分（maven 聚合）](https://www.cnblogs.com/l-y-h/p/14105682.html)



## 一、架构演变



### 1、系统架构、集群、分布式系统 简单理解

（1）什么是系统架构？

```
【什么是系统架构？】
    系统架构 描述了 在应用程序内部，如何根据 业务、技术、灵活性、可扩展性、可维护性 等因素，将系统划分成不同的部分并使这些部分相互分工、协作，从而提高系统的性能。
    
【简单的理解：】
    系统架构是 程序运行 的基石、其决定了程序是否能正确、有效的构建 以及 稳定的运行。
```

 

（2）集群

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【什么是集群？】
    计算机集群简称集群，是一种计算机系统，它通过一组松散集成的计算机软件或硬件连接起来、高度紧密地协作完成计算工作。
    在某种意义上，他们可以被看作是一台计算机。
    集群系统中的单个计算机通常称为节点，通常通过局域网连接（或者其它的连接方式）。
    集群计算机通常用来改进单个计算机的计算速度或可靠性。

【简单的理解：】
    通过多台计算机完成同一个工作，达到更高的效率。
    两机或多机内容、工作过程等完全一样。如果一台死机，另一台可以起作用。
    同一个业务，部署在多个服务器上(不同的服务器运行同样的代码，干同一件事)。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（3）分布式系统

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【什么是分布式系统？】
    分布式系统是一组计算机，通过网络相互连接，传递消息与通信后，协调它们的行为而形成的系统。
    组件之间彼此进行交互从而实现一个共同的目标。

【简单的理解：】
    通过多台计算机相互作用完成一个工作，每个计算机负责一个功能模块，将多个计算机组合起来从而完成一个大任务。
    一个业务拆分为多个子业务，部署在不同的服务器上(不同的服务器，运行不同的代码，为了同一个目的)。
注：
    分布式与集群不冲突，可以存在分布式集群，即将一个项目拆分为不同的功能模块后，对各个不同的功能模块实现集群。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（4）架构演变
　　Dubbo 官网将系统架构分为 单体架构、垂直架构、分布式服务架构、流计算架构。
　　可参考：http://dubbo.apache.org/docs/v2.7/user/preface/background/
注：
　　下面的 系统演变过程 以此为 基础 进行展开，有不对的地方还望不吝赐教。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208203924300-1450749443.png)

 

 

 



### 2、系统架构需要考虑的因素（高可用、高并发、高性能）

（1）高可用性（High Availability）

```
【高可用性：】
    高可用性 指的是 尽量缩短 维护操作（计划内）以及 系统故障（非计划内）而导致的停机时间，从而保证 系统 正常运行。
    高可用性 具有高度 的容错性、恢复性。
注：
    容错性 指的是 软件发生故障时 仍能正常运行的 能力。
    恢复性 指的是 软件发生故障时 恢复到故障之前状态的 能力。
```

 

（2）高并发（High Concurrency）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【高并发性：】
    高并发性 指的是 保证系统能够同时并行处理 很多请求。
    通过 水平拓展 或者 垂直拓展的方式 可以提高系统高并发能力。
 
【垂直拓展：】
    垂直拓展，提高当前系统的能力 以适应 需求。
又可分为两种：
    1、提升硬件能力，强化 服务器硬件，比如：机械硬盘 更换为 固态硬盘，增加 CPU 核数、拓展系统内存 等。
    2、提升软件能力，优化 软件性能，比如：使用缓存 减少磁盘 I/O 次数，优化代码使用的数据结构 从而减少响应时间等。
注：
    单机性能提升还是有限的，成本充足情况下，增加服务器的使用还是比较合适的。
 
【水平拓展：】
    水平拓展，也即 横向拓展，增加系统个数 以适应 需求，只需要增加服务器 的数量，就可以线性的增加系统的并发能力。
    当然也不能一直增加服务器，服务器数量增多，维护、成本的压力也就上来了。
              
【高并发相关指标：】
    响应时间（Response Time）：指的是 系统对 用户输入或者请求 进行处理并响应请求的时间。
    吞吐量（Throughput）：指的是 系统单位时间内 处理请求的数量（吞吐量一般与响应时间成反比，即吞吐量越大，响应时间越短）。
    每秒查询率（Query Per Second，QPS）：指的是 系统 每秒响应的请求数。
    并发用户数：指的是 系统 某时刻支持 正常使用系统的用户数。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（3）高性能（High Performance）

```
【高性能：】
    高性能 指的是 程序处理速度快、占用内存少、CPU 占用率低。
    也即系统性能强悍、运算能力强、响应时间短。
```

 



### 3、架构演变 -- 单体应用架构（传统架构、三层架构、集群）

（1）传统架构

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【背景：】
    互联网开发早期，所有的 业务、功能模块 代码都写在一个项目中，然后 编译、打包并部署到容器（比如：Tomcat）中运行。
    此时称为 All in One，即 所有模块代码均写在一起（比如：Servlet、JSP 等代码）、技术上不分层。

【优点：】
    所有功能均在同一个应用程序中，所以只需要部署 一个应用程序即可，减少了 部署节点、以及成本。

【缺点（出现的问题）：】
    代码可维护性差（所有代码均写在一个项目中，没有层次，相互调用复杂，不易修改）。
    容错性差（代码写在 JSP 中，发生错误时 服务器可能直接宕机 且 用户可以直接看到 错误信息）。
    并发量小。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204108911-611173080.png)

 

 

 

（2）三层架构

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【背景：】
    为了解决 传统架构 可维护性差、容错性差、并发量小 等问题，引入了 分层的概念。
    分层 即 将代码 划分出 几个层级，每个层级 干不同的事，从而提高代码的可维护性。

那么如何分层呢？分几层呢？
    在开发早期，所有的逻辑代码没有明显的区分，代码间相互调用、职责不清，页面逻辑、业务逻辑、数据库访问逻辑 等混合在一起，即一层架构，此时的代码维护、迭代工作肯定无比麻烦。
    随着时代的发展，数据库访问逻辑 被逐步的划分出来，但页面逻辑、业务逻辑 仍然混合在一起，即二层架构，此时简化了数据访问的操作，提高了系统的可维护性。
    继续发展，从功能、代码组织的角度进行划分，将三种逻辑分开，也即三层架构出现。

【三层架构（MVC）：】
从功能、代码组织的角度出发，按系统不同职责进行划分成三个层次：
    表示层：关注 数据显示 以及 用户交互。
    业务逻辑层：关注 业务逻辑处理。
    数据访问层：关注 数据的存储与访问。   
注：
    此处的三层架构并非 物理分层，而是逻辑上的分层（所有代码仍然在一个项目中进行 开发、编译、部署，仍是单体架构）。

【优点：】
    提高了可维护性。每一层的功能具体化，解决了系统间 相互调用复杂、职责不清的问题，有效降低了层与层间的依赖关系，降低了维护、迭代的成本。
    MVC 分层开发，提高了系统的容错性。
    服务器分离部署。数据库 以及 应用程序 可以部署在 不同的服务器 上。 
   
【缺点（出现的问题）：】
    并发量仍然不高（随着用户访问量增加，单台应用服务器无法满足需求）。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204145766-1056020792.png)

 

 

 

（3）集群

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【背景：】
    为了解决 三层架构 的并发量问题，引入了 集群的概念。
    单台服务器不能满足需求，那么就使用 多台 服务器构成 集群 提供服务。
    
【优点：】
    使用 多台服务器 构成集群 同时提供服务，提高了并发量。
    提高了容错性（一台服务器挂了，还有其他服务器可以提供服务，保证程序的正常运行）。

【缺点（出现的问题）：】
    用户的请求 发送给 哪台服务器？
    如何保证请求可以 平均的发送给 各个服务器？
    数据如何进行共享、缓存？
    数据需要模糊查询时，如何提高数据库查询效率？
    数据库访问压力如何解决？
    数据量过大时，应该如何存储？
    
【解决：】
    可以通过 Nginx 解决 请求的发送 以及 分发（负载均衡） 问题。
    可以通过 Redis 解决 数据共享 以及 缓存 问题。
    可以通过 ElacticSearch 解决 数据搜索 问题。
    可以通过 MyCat 使用主从复制、读写分离，减轻数据库压力，通过分库分表 的方式，按照指定的方式存储数据。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204222982-1549996353.png)

 

 

 

（4）解决集群出现的问题

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【问题一：】
    用户 登录并访问 服务器时，会产生一个 session，且伴随着用户访问的全过程 直至 用户关闭浏览器结束此次访问。
    当一个服务器突然宕机，若不对 session 进行处理，那么其 session 必定丢失，也即 用户需要重新 进行登录等 一系列操作，用户体验感将极差。
    
    那么 session 如何共享？也即 数据共享问题？    

【解决：】
方式一：
    使用 Tomcat 广播 session，从而实现 session 共享。
    每个 tomcat 会在局域网中广播自己的 session 信息并监听其他 tomcat 广播的 session 信息，一旦 session 发生变化，其他的 tomcat 就能监听并同步 session。
注：
    此方式只适用于 并发量小的 小项目。
    并发量大时，比如用户量为 1000 万，那每个服务器都需要广播、维护 session，那么将会导致服务器大量资源都用来处理 session，这样肯定不适合大型项目。
    
方式二：
    使用 Redis 存储 session，从而实现 session 共享。
    使用 Redis 存储 session，当服务器需要使用 session 时，直接从 Redis 中获取，这样只需要关心 Redis 的维护即可，减轻了 服务器的压力。
    同时，Redis 还可以用来进行 数据缓存，减少数据库访问次数（提高响应时间）。 
注：
    此方式适合于 大型的项目。
    SpringBoot 整合 Redis 可参考：https://www.cnblogs.com/l-y-h/p/13163653.html#_label0
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204300497-2119195637.png)

 

 

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【问题二：】
    使用集群，即存在多个服务器，一个用户请求 肯定会被某一个服务器进行处理，此时就需要考虑 服务器 处理请求的问题了。
    
    那么 用户的请求 发送给 哪台服务器处理？如何保证请求可以 平均的发送给 集群中的各个服务器？
    
【解决：】
    可以通过 Nginx 解决 请求的发送 以及 分发（负载均衡） 问题。
注：
    Nginx 反向代理、负载均衡等基本概念可参考：https://www.cnblogs.com/l-y-h/p/12844824.html#_label0
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204322990-1297949255.png)

 

 

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【问题三：】
    通过上面介绍的 Nginx + Redis 的方式，提高了应用层的性能。
    应用层问题解决了，数据库的访问压力怎么解决？如何提高数据库的负载能力？数据库数据量大时如何存储？
    
【解决：】
    采用读写分离、主从复制的方式，提高数据库负载能力。
    采用 master-slave 方式，master 负责 进行增删改 等写操作，slave 进行 读操作，并通过主从复制的方式 将 master 数据同步到 slave 中。
    设置多个 slave，从而提高数据库查询、负载能力。
    
    采用分库分表的方式，进行数据存储。
注：
    垂直拆分数据表：根据常用字段 以及 不常用字段 将数据表划分为多个表，从而减少单个表的数据大小。
    水平拆分数据表：根据时间、地区 或者 业务逻辑进行拆分。
    垂直拆分仍有局限性，水平拆分便于业务拓展。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204347317-794798833.png)

 

 

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【问题四：】
    虽然进行了读写分离，数据量过大时，执行 模糊查询 的效率低。对于大型的网站（电商等），搜索是其核心模块，若一个查询执行半天才能返回结果，那么用户体验将是极差的。
    
    那么如何提高查询的效率？
    
【解决：】
    使用 搜索引擎技术，比如：ElacticSearch、solr。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204413382-1092097506.png)

 

 

 

（5）单体架构总结

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【单体架构：】
    单体架构，就是将所有 业务、功能模块 都写在一个项目中，编译、打包并部署到容器（比如：Tomcat）中运行。
    应用程序、数据库 可以分开部署，可以通过部署 应用程序集群、数据库集群 的方式提高系统性能。
    能简化 增删改查 工作的 数据访问框架（ORM） 也是提高系统性能的关键。
注：
    随着业务扩大、需求增加，单体架构将会变得臃肿、耦合性高，可维护性、可扩展性、灵活性都在逐步降低，
    难以满足业务快速迭代的需求，且成本不断攀升，单体架构的时代已成为过去。
    
【优点：】
    开发、测试、部署简单，维护成本低。适用于 用户、数据 规模小的 项目。
    通过拓展集群的方式 可以保证 高并发、高可用。

【缺点：】
    可维护性、可拓展性差。（随着业务增加、功能迭代，代码会变得臃肿、耦合）
    技术栈受限，且只能通过 拓展集群 的方式提高系统性能（维护成本高）。
    协同开发不方便（存在修改相同业务代码的情况、导致冲突）。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 



### 4、架构演变 -- 垂直应用架构（水平拆分、垂直拆分）

（1）背景

　　上面介绍的 单体架构 已不能满足实际场景（拓展能力有限、代码臃肿），那么需要进行代码重构，对单体架构代码 进行拆分，那么如何进行拆分 能保证 高可用、高并发、高性能 呢？
　　前面也提到了 高并发 可以通过 垂直拓展、水平拓展 来实现，此处不妨也对单体架构的代码 进行 水平拆分 以及 垂直拆分。

注：
　　拓展 与 拆分 是两种概念。拓展是对外部改造，拆分是对内部改造。

　　　　垂直拓展：增加服务器硬件性能，提高服务器执行应用的能力。
　　　　水平拓展：增加服务器数量，多节点部署应用。

　　　　垂直拆分：根据业务 对 系统进行划分。
　　　　水平拆分：根据逻辑分层 对 系统进行划分（比如：前后端分离、MVC 分层）。

（2）水平拆分
　　水平拆分 根据 逻辑分层 对系统进行划分，将一个大的 单体应用程序 拆分成 多个小应用程序，每个小应用程序 作为 单独的 jar 包，需要使用时，引入相关 jar 包即可。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【单体应用举例：】
    现有一个 SSM 单体应用，如何进行水平拆分？
    
【水平拆分思路：】
    按照逻辑分层对代码进行拆分，将 controller、service、dao 层分别抽取出来，并打成 jar 包，需要使用时 直接引入 jar 包即可。
    
【使用 Maven 聚合的方式可以演示：】
Step1：构建一个 ssm_parent 工程（父工程，聚合下面的子工程，pom）
Step2：构建一个 ssm_bean 工程（子工程，存放 实体类，jar）
Step3：构建一个 ssm_mapper 工程（子工程，存放持久层类以及接口，jar）
Step4：构建一个 ssm_service 工程（子工程，存放 业务逻辑层类以及接口，jar）
Step5：构建一个 ssm_controller 工程（子工程，存放 控制层类，war）

也即目录结构如下：
ssm_parent（pom）
    ssm_bean（jar）  或者 ssm_pojo  名称随意取，见名知意 即可。
    ssm_mapper（jar） 或者 ssm_dao
    ssm_service（jar）
    ssm_controller（war） 或者 ssm_web
注：
    ssm_mapper 需要引入 ssm_bean.jar。
    ssm_service 需要引入 ssm_mapper.jar。
    ssm_controller 需要引入 ssm_service.jar。
    可以通过 ssm_controller 或者 ssm_parent 启动项目。
    
【水平拆分优点：】
    模块可以复用，减少代码冗余度。
    代码分离部署，可以根据实际情况，增加 或者 减少 某些业务层的部署量。（比如：dao 层访问量过大，可以多部署几个 dao 服务减轻压力。单体应用则是整体部署，增大了服务器部署容量。）
    
【水平拆分缺点：】
    各个模块业务仍然交互在一起，修改某个模块业务时，整个 jar 包（非修改模块）需要重新 测试 、部署，增加了 测试 与 维护 的压力。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204528849-928081525.png)

 

 

 

（3）垂直拆分：
　　垂直拆分 是 根据 业务 对系统进行划分，将一个大的 单体应用程序 拆分成 若干个 互不相干的小应用程序（也即 垂直应用架构）。

　　每个小应用程序 就是一个单独的 web 项目。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【单体应用举例：】
    现有一个 SSM 单体应用，如何进行垂直拆分？
    
【垂直拆分思路：】
    按照业务对代码进行拆分，比如：现在 SSM 中存在 后台管理业务、用户业务、菜单业务 等。
    则将 这些业务 分别抽取出来，各自构成 web 工程。
    
【使用 Maven 聚合的方式可以演示：】
Step1：构建一个 ssm_parent 工程（父工程，聚合下面的子模块，pom）
Step2：构建一个 ssm_admin 模块（子模块，后台管理模块）
Step3：构建一个 ssm_user 模块（子模块，用户模块）
Step4：构建一个 ssm_menu 模块（子模块，菜单模块） 

也即目录结构如下：
ssm_parent（pom）
    ssm_admin 
    ssm_user 
    ssm_menu 

【垂直拆分优点：】
    提高了可维护性（需求变更时，修改对应的模块即可）。
    提高了可拓展性（拓展业务时，增加新的模块即可，可以针对访问量 增大、减少 某个模块的部署量）。
    提高了协同开发能力（不同的团队可以开发 不同的模块）。
    
【垂直拆分缺点：】
    某个模块修改时（比如：页面频繁更换），需要对整个模块进行重新部署，增大了维护的难度。
    随着业务模块增加，各模块间必然需要进行业务交互，模块交互问题又是一个头疼的问题。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204606166-8071439.png)

 

 

 

（4）垂直应用架构总结

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【垂直应用架构：】
    垂直应用架构，按照业务将原来的 大单体应用 拆分成 若干个互不想干的 小单体应用。
    用于加速前端页面开发的 web 框架（MVC）也是提高开发效率的关键。
    
【优点：】
    系统间相互独立，极大地解决了 耦合问题。
    可以针对不同的业务进行 优化、可以针对不同的业务搭建集群。

【缺点：】
    使用集群时，负载均衡相对而言比较复杂，且拓展成本高、有瓶颈。
    随着功能增加，模块随之增多，一些通用的服务、模块也会增多，代码冗余。
    随着业务增加，应用之间难免会进行 数据交互，若某个应用端口、IP 变更，需要手动进行代码更改（增加维护成本）。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 



### 5、架构演变 -- 分布式服务架构、SOA 架构、微服务架构

（1）分布式服务架构
　　随着 垂直应用架构 模块增多，模块之间的交互不可避免，为了解决 这个问题，引出了 分布式服务架构，将 核心业务 抽取出来并独立部署，各服务之间通过 远程调用框架（PRC）进行通信。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【分布式服务架构：】
    分布式服务架构，按照业务 拆分成不同的子业务，并独立部署在不同的服务器上。

【垂直应用架构问题一：】
    客户对页面要求变化大，每次修改后，都需要对应用重新部署，是比较麻烦的事情。
    
【解决：】
    采用 前后端分离开发，将 界面 与 业务逻辑分开（水平拆分），此时只需关心界面的修改即可。
    
【垂直应用架构问题二：】
    随着业务模块增加，各个模块必然会进行交互，如何交互？业务部署在不同服务器上，该如何交互？
    
【解决：】
   采用 RPC/HTTP/HttpClient 框架进行远程服务调用。
   
【分布式架构问题：】
    新架构的改变必然带来新的技术问题。比如： 分布式事务、分布式锁、分布式日志管理 等。
    随之而来的就是 分布式服务治理中间件： Dubbo（RPC） 以及 SpringCloud（HTTP）。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204713733-1048289627.png)

 

 

 

（2）SOA 架构
　　随着 服务的 增多，若不对 服务进行管理，那么容易导致 服务资源浪费（比如：用户模块访问量大 只部署了 10 台服务器，而 菜单模块访问量小，却部署了 20 台服务器）、且服务间调用混乱（100 个服务相互调用若没有条理，那将是一件非常头疼的事情）。
　　为了提高 机器利用率 并 对服务进行管理，引出了 SOA 的概念。

注：

　　此处只是简单的介绍了下概念，详情请自行 谷歌、百度 了解（有时间再详细研究研究）。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【SOA：】
    SOA 是 Service-Oriented Architecture 的简写，即 面向服务架构。
    指的是 根据实际业务，将系统拆分成合适的、独立部署的服务，各服务相互独立，通过 调用中心 完成 各服务之间的 调用 以及 管理。

【缺点：】
    依赖于 中心化服务发现机制。
    SOA 采用 SOAP 协议（HTTP + XML），而 XML 报文存在大量冗余数据，影响传输效率。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208204949311-877551189.png)

 

 

 

（3）微服务
　　微服务是 基于 SOA 架构演变而来，去除了 SOA 架构中 ESB 消息总线，采用 HTTP + JSON（RESTful ）进行传输。其划分粒度比 SOA 更精细。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【微服务架构：】
    微服务架构 指的是 将单个应用程序 划分为 若干个互不相干的小应用，每个小应用都是一个服务，服务之间相互协调、配合，从而为用户提供最终价值。
    每个服务运行在独立的进程中，服务之间通常采用 轻量级的通信机制（通常是基于 HTTP 的 RESTful API）,每个服务均是基于 具体业务进行构建，并可以独立的 部署到生产环境中。
    
【本质：】
    微服务的目的是有效的拆分应用（将功能分散到各个服务中，降低系统耦合度），实现敏捷开发 与 部署。
    微服务关键点在于 系统要提供一套基础的架构，使得微服务可以独立部署、运行、升级，且各个服务 在结构 上松耦合，在功能上为一个统一的整体。
注：
    统一指的是：统一的安全策略、统一的权限管理、统一的日志处理 等。
    
【优点：】
    微服务每个模块就等同于一个独立的项目，可以使用不同的开发技术，使开发模式更灵活。
    每个模块都有独立的数据库，可以选择不同的存储方式。比如：redis、mysql。
    微服务的拆分粒度 比 SOA 更精细，复用性更强（提高开发效率）。

【缺点：】
    微服务过多，服务的管理成本将随之提高。
    技术要求变高（分布式事务、分布式锁、分布式日志、SpringCloud、Dubbo 等一系列知识都需要学习）。    
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 



### 6、什么是 SpringCloud？

（1）相关地址：

```
【SpringCloud 官网地址：】
    https://spring.io/projects/spring-cloud

【SpringCloud 中文文档：】
    https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md
```

 

（2）基本认识
　　Spring Cloud 是分布式微服务架构下的一站式解决方案，是各个微服务架构技术实现的集合体。
注：
　　Spring Boot 可以快速构建单个微服务。
　　Spring Cloud 将多个 Spring Boot 构建的微服务整合并管理起来，并提供一系列处理（比如：服务发现、配置管理、消息总线、负载均衡、断路器、数据监控等）。

 



### 7、微服务问题 以及 技术实现

（1）背景
　　SpringCloud 是解决 微服务架构 而存在的，其针对 微服务架构 一系列技术问题 都做出了相关实现，当然随着技术的进步，有些技术已经停止更新、维护了，逐步被新技术替代（学无止境）。
　　此处从整体上了解一下 SpringCloud 有哪些技术，后续再逐步深入。

 

（2）微服务问题

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【微服务相关问题：】
    服务注册与发现、服务配置中心
    服务调用、服务负载均衡
    服务网关
    服务熔断、服务降级
    服务总线
    ...
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（3）技术实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【技术实现：】
    服务注册与发现：
        Eureka     停止维护了，不推荐使用。
        ZooKeeper
        Consul
        Nacos      阿里开源的产品，推荐使用
        
    服务配置中心：
        Config
        Nacos      推荐使用
        
    服务调用、负载均衡:
         Ribbon        停止更新了（维护状态），不推荐使用。
        Loadbalancer  作为 Ribbon 的替代产品。
        Feign          停止更新了（维护状态），不推荐使用
        OpenFeign      Spring 推出的 Feign 的替代产品（推荐使用）。

    服务网关：
        Zuul          停止维护了，不推荐使用。
        Zuul2         还没出来（已经凉凉了）。
        Gateway       Spring 推出的替代产品（推荐使用）。 
        
    服务降级：
        Hystrix       停止维护了，不推荐使用。
        Resilience4j  替代产品，国外使用多。
        Sentienl      替代产品，国内使用多（推荐使用）。
        
    服务总线：
        Bus
        Nacos          推荐使用。
    
注：
    Nacos 功能还是比较强大的（可以替换多个组件），重点关注。
    各个组件具体功能后续介绍，此处暂时略过。。。
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（4）SpringCloud 版本选择
　　进入官网，可以查看到当前最新版本的 SpringCloud。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205235914-671334555.png)

 

 

 

　　SpringCloud 是基于 SpringBoot 开发的，其历史版本与 SpringBoot 对应版本如下。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205300651-344762033.png)

 

 

 

当然，还是需要进入不同版本的 SpringCloud （Reference Doc.）查看官方推荐配置。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205318816-1136739420.png)

 

 

 

 

[回到顶部](https://www.cnblogs.com/l-y-h/p/14105682.html#_labelTop)

##  二、代码拆分演示（maven 聚合）



### 1、构建普通的 web 项目

（1）基本说明

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【项目基本说明：】
    此项目仅供参考，不与数据库进行交互。
    创建 ControllerA、ServiceA 表示业务 A。
    创建 ControllerB、ServiceB 表示业务 B。   

使用 IDEA 利用 maven 构建 SSM 项目 可参考：
    https://www.cnblogs.com/l-y-h/p/12030104.html
    或者 https://www.cnblogs.com/l-y-h/p/14010034.html#_label0_3  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（2）使用 IDEA 利用 maven 构建一个普通的 web 工程
　　可参考：https://www.cnblogs.com/l-y-h/p/11454933.html
Step1：创建一个 web 工程（选择 maven-archetype-webapp 模板，会自动生成 webapp 文件夹）

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205506720-512766791.png)

 

 

 

Step2：引入 依赖（Spring MVC）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>5.2.8.RELEASE</version>
</dependency>

【pom.xml（注意：<packaging>war</packaging> 是 war、非 pom）】
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>ssm</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>ssm Maven Webapp</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205539703-278380008.png)

 

 

 

Step3：配置基本 web 环境（Spring、SpringMVC）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【web.xml】
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!-- step1: 配置全局的参数，启动Spring容器 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <!-- 若没有提供值，默认会去找/WEB-INF/applicationContext.xml。 -->
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- step2: 配置SpringMVC的前端控制器，用于拦截所有的请求  -->
  <servlet>
    <servlet-name>springmvcDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <init-param>
      <param-name>contextConfigLocation</param-name>
      <!-- 若没有提供值，默认会去找WEB-INF/*-servlet.xml。 -->
      <param-value>classpath:dispatcher-servlet.xml</param-value>
    </init-param>
    <!-- 启动优先级，数值越小优先级越大 -->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvcDispatcherServlet</servlet-name>
    <!-- 将DispatcherServlet请求映射配置为"/"，则Spring MVC将捕获Web容器所有的请求，包括静态资源的请求 -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <!-- step3: characterEncodingFilter字符编码过滤器，放在所有过滤器的前面 -->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <!--要使用的字符集，一般我们使用UTF-8(保险起见UTF-8最好)-->
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <!--是否强制设置request的编码为encoding，默认false，不建议更改-->
      <param-name>forceRequestEncoding</param-name>
      <param-value>false</param-value>
    </init-param>
    <init-param>
      <!--是否强制设置response的编码为encoding，建议设置为true-->
      <param-name>forceResponseEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <!--这里不能留空或者直接写 ' / ' ，否则可能不起作用-->
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!-- step4: 配置过滤器，将post请求转为delete，put -->
  <filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>

【applicationContext.xml】
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!-- step1: 配置包扫描方式。扫描所有包，但是排除Controller层 -->
    <context:component-scan base-package="com.lyh.ssm">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>

【dispatcher-servlet.xml】
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- step1: 配置Controller扫描方式 -->
    <!-- 使用组件扫描的方式可以一次扫描多个Controller,只需指定包路径即可 -->
    <context:component-scan base-package="com.lyh.ssm" use-default-filters="false">
        <!-- 一般在SpringMVC的配置里，只扫描Controller层，Spring配置中扫描所有包，但是排除Controller层。
        context:include-filter要注意，如果base-package扫描的不是最终包，那么其他包还是会扫描、加载，如果在SpringMVC的配置中这么做，会导致Spring不能处理事务，
        所以此时需要在<context:component-scan>标签上，增加use-default-filters="false"，就是真的只扫描context:include-filter包括的内容-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>

    <!-- step2: 配置视图解析器 -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- step3: 标准配置 -->
    <!-- 将springmvc不能处理的请求交给 spring 容器处理 -->
    <mvc:default-servlet-handler/>
    <!-- 简化注解配置，并提供更高级的功能 -->
    <mvc:annotation-driven />
</beans>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205615769-757035203.png)

 

 

 

Step4：编写 基本 业务代码。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ControllerA】
package com.lyh.ssm.controller;

import com.lyh.ssm.service.ServiceA;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerA {
    @Autowired
    private ServiceA serviceA;

    @GetMapping("/testA")
    public String testContollerA() {
        return serviceA.testA();
    }
}

【ControllerB】
package com.lyh.ssm.controller;

import com.lyh.ssm.service.ServiceB;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerB {
    @Autowired
    private ServiceB serviceB;

    @GetMapping("/testB")
    public String testControllerB() {
        return serviceB.testB();
    }
}

【ServiceA】
package com.lyh.ssm.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceA {
    public String testA() {
        return "test serviceA";
    }
}

【ServiceB】
package com.lyh.ssm.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceB {
    public String testB() {
        return "test serviceB";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205646593-1181685477.png)

 

 

 

Step5：配置、启动 tomcat。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205707264-283267077.png)

 

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205723984-1359352378.png)

 

 

 

Step6：访问 testA()、testB()。正常访问 也即基本 web 工程已搭建完成。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205743454-737391054.png)

 

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205755971-1441817431.png)

 

 

 

 



### 2、水平拆分 web 项目

（1）基本说明

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【水平拆分说明：】
    将上面简单的 web 工程 做水平拆分，按照逻辑分层对代码进行拆分，
    将 controller、service 层分别抽取出来，并做成 war 包或者 jar 包，需要时引入依赖即可。
    
【使用 Maven 聚合的方式可以演示：】
Step1：构建一个 ssm_parent 工程（父工程，聚合下面的子工程，pom）
Step2：构建一个 ssm_service 工程（子工程，存放 业务逻辑层类以及接口，jar）
Step3：构建一个 ssm_controller 工程（子工程，存放 控制层类，war）

也即目录结构如下：
ssm_parent（pom）
    ssm_service（jar）
    ssm_controller（war）
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（2）使用 maven 聚合的方式进行水平拆分
Step1：使用 maven 创建一个父工程（maven-archetype-site-simple）。
　　删除 src 文件夹（无用文件夹），选择 maven-archetype-site-simple 目的是使 pom.xml 中 packaging 为 <packaging>pom</packaging>。
注：
　　模板随意选择，选择 maven-archetype-quickstart 亦可，保证 <packaging>pom</packaging>。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205904580-795903685.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205920026-2131649517.png)

 

 

Step2：在 ssm_parent 上 右键 选择 创建 Module。
　　并使用 maven 模板（maven-archetype-quickstart）创建一个模块（ssm_service）。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205940443-1942285174.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208205950819-1257026768.png)

 

 

Step3：将 普通 web 工程中 service 代码 抽取出来，并放入 ssm_service 中。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ServiceA】
package com.lyh.ssm.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceA {
    public String testA() {
        return "test serviceA";
    }
}

【ServiceB】
package com.lyh.ssm.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceB {
    public String testB() {
        return "test serviceB";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210018893-2122174482.png)

 

 

Step4：在 父工程（ssm_parent）或者 当前工程（ssm_service）中引入 SpringMVC 依赖包。

```
【SpringMVC 依赖:】
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>5.2.8.RELEASE</version>
</dependency>
```

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210043782-2113600069.png)

 

 

Step5：在 ssm_parent 上 右键 选择 创建 Module。
　　并使用 maven 模板（maven-archetype-webapp）创建一个模块（ssm_controller）。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210107090-884335293.png)

 

 

Step6：将普通 web 工程中 相关代码（controller、配置文件）抽取出来，放入 ssm_controller 中。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ControllerA】
package com.lyh.ssm.controller;

import com.lyh.ssm.service.ServiceA;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerA {
    @Autowired
    private ServiceA serviceA;

    @GetMapping("/testA")
    public String testContollerA() {
        return serviceA.testA();
    }
}

【ControllerB】
package com.lyh.ssm.controller;

import com.lyh.ssm.service.ServiceB;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerB {
    @Autowired
    private ServiceB serviceB;

    @GetMapping("/testB")
    public String testControllerB() {
        return serviceB.testB();
    }
}

【applicationContext.xml】
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">

    <!-- step1: 配置包扫描方式。扫描所有包，但是排除Controller层 -->
    <context:component-scan base-package="com.lyh.ssm">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>

【dispatcher-servlet.xml】
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- step1: 配置Controller扫描方式 -->
    <!-- 使用组件扫描的方式可以一次扫描多个Controller,只需指定包路径即可 -->
    <context:component-scan base-package="com.lyh.ssm" use-default-filters="false">
        <!-- 一般在SpringMVC的配置里，只扫描Controller层，Spring配置中扫描所有包，但是排除Controller层。
        context:include-filter要注意，如果base-package扫描的不是最终包，那么其他包还是会扫描、加载，如果在SpringMVC的配置中这么做，会导致Spring不能处理事务，
        所以此时需要在<context:component-scan>标签上，增加use-default-filters="false"，就是真的只扫描context:include-filter包括的内容-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
    </context:component-scan>

    <!-- step2: 配置视图解析器 -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- step3: 标准配置 -->
    <!-- 将springmvc不能处理的请求交给 spring 容器处理 -->
    <mvc:default-servlet-handler/>
    <!-- 简化注解配置，并提供更高级的功能 -->
    <mvc:annotation-driven />
</beans>

【web.xml】
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

  <!-- step1: 配置全局的参数，启动Spring容器 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <!-- 若没有提供值，默认会去找/WEB-INF/applicationContext.xml。 -->
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- step2: 配置SpringMVC的前端控制器，用于拦截所有的请求  -->
  <servlet>
    <servlet-name>springmvcDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <init-param>
      <param-name>contextConfigLocation</param-name>
      <!-- 若没有提供值，默认会去找WEB-INF/*-servlet.xml。 -->
      <param-value>classpath:dispatcher-servlet.xml</param-value>
    </init-param>
    <!-- 启动优先级，数值越小优先级越大 -->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvcDispatcherServlet</servlet-name>
    <!-- 将DispatcherServlet请求映射配置为"/"，则Spring MVC将捕获Web容器所有的请求，包括静态资源的请求 -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>

  <!-- step3: characterEncodingFilter字符编码过滤器，放在所有过滤器的前面 -->
  <filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <!--要使用的字符集，一般我们使用UTF-8(保险起见UTF-8最好)-->
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
      <!--是否强制设置request的编码为encoding，默认false，不建议更改-->
      <param-name>forceRequestEncoding</param-name>
      <param-value>false</param-value>
    </init-param>
    <init-param>
      <!--是否强制设置response的编码为encoding，建议设置为true-->
      <param-name>forceResponseEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <!--这里不能留空或者直接写 ' / ' ，否则可能不起作用-->
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <!-- step4: 配置过滤器，将post请求转为delete，put -->
  <filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210139483-313158800.png)

 

 

Step7：在 ssm_controller 中引入 ssm_service.jar 包。

```
<dependency>
  <groupId>com.lyh.ssm</groupId>
  <artifactId>ssm_service</artifactId>
  <version>1.0-SNAPSHOT</version>
</dependency>
```

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210210608-160256414.png)

 

 

Step8：使用 tomcat 启动 ssm_controller 工程，即可访问项目。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210229156-1935349779.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210243158-424853534.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210254497-667918261.png)

 

 

Step9：使用 tomcat7 插件来启动 maven 聚合工程。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【在父工程 pom.xml 中引入 tomcat7 插件】
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.tomcat.maven</groupId>
      <artifactId>tomcat7-maven-plugin</artifactId>
      <version>2.2</version>
    </plugin>
  </plugins>
</build>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210318413-1518909890.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210331192-1575307964.png)

 

 



### 3、垂直拆分 web 项目

（1）基本说明

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【垂直拆分说明：】
    将上面简单的 web 工程 做垂直拆分，按照业务对代码进行拆分，
    将 业务A、业务 B 分别抽取出来，并做成独立的 war 包。
注：
    若拆分后仍使用配置文件的方式进行项目构建，那么代码冗余将非常多。
    所以一般均使用 SpringBoot 简化开发（约定 > 配置）。
    
【使用 Maven 聚合的方式可以演示：】
Step1：构建一个 ssm_parent 工程（父工程，聚合下面的子工程，pom）
Step2：构建一个 ssm_serviceA 工程（子工程，存放 业务A，war）
Step3：构建一个 ssm_serviceB 工程（子工程，存放 业务B，war）

也即目录结构如下：
ssm_parent（pom）
    ssm_serviceA（war）
    ssm_serviceB（war）
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（2）使用 maven 聚合的方式进行垂直拆分
　　此处 以 SpringBoot 作为项目构建的基础。

Step1：使用 maven 构建一个父工程 ssm_parent。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210411604-795898296.png)

 

 

Step2：ssm_parent 项目上右键选择 New -> Module 并选择 Spirng Initializr。
　　使用 SpringBoot 创建 serviceA 项目。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210430587-714458339.png)

 

 

Step3：从普通 web 工程中 抽取出 业务A 的代码放入 ssm_serviceA 中。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ControllerA】
 package com.lyh.ssm.aservice.controller;

import com.lyh.ssm.aservice.service.ServiceA;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerA {
    @Autowired
    private ServiceA serviceA;

    @GetMapping("/testA")
    public String testContollerA() {
        return serviceA.testA();
    }
} 

【ServiceA】
package com.lyh.ssm.aservice.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceA {
    public String testA() {
        return "test serviceA";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

Step4：修改 父工程（ssm_parent） 以及 当前工程（ssm_serviceA）pom.xml 并引入依赖。
　　修改 ssm_serviceA 的 <parent>，使其指向父工程（ssm_parent）。
　　在父工程中通过 <module> 管理 子工程。
　　在父工程中 通过 <dependencyManagement> 声明依赖、管理版本。
　　在子工程中 通过 <dependency> 引入需要的依赖。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ssm_parent 的 pom.xml 为：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lyh.ssm</groupId>
    <artifactId>ssm_parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>ssm_serviceA</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <web.version>2.4.0</web.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>${web.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${web.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>

【ssm_serviceA 的 pom.xml 为：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.lyh.ssm</groupId>
        <artifactId>ssm_parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>aservice</artifactId>
    <name>ssm_serviceA</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210518259-1768962699.png)

 

 

Step5：修改 ssm_serviceA 的配置文件（端口号、服务名等）
　　修改 application.properties 或者 application.yml 文件。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【application.yml】
server:
  port: 9000

spring:
  application:
    name: ssm_serviceA
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210547685-1591720584.png)

 

 

Step6：同理，创建 SpingBoot 项目 ssm_serviceB，并从 普通 web 工程中 抽取 业务B 代码放入其中。同样修改 pom.xml（指向父工程，引入 web 依赖） 以及 配置文件。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【ControllerB】
package com.lyh.ssm.bservice.controller;

import com.lyh.ssm.bservice.service.ServiceB;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ControllerB {
    @Autowired
    private ServiceB serviceB;

    @GetMapping("/testB")
    public String testContollerB() {
        return serviceB.testB();
    }
}

【ServiceB】
package com.lyh.ssm.bservice.service;

import org.springframework.stereotype.Service;

@Service
public class ServiceB {
    public String testB() {
        return "test serviceB";
    }
}

【ssm_serviceB 的 pom.xml 为：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.lyh.ssm</groupId>
        <artifactId>ssm_parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>bservice</artifactId>
    <name>bservice</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

【application.yml】
server:
  port: 9010

spring:
  application:
    name: ssm_serviceB
    
【ssm_parent 的 pom.xml 为：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lyh.ssm</groupId>
    <artifactId>ssm_parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>ssm_serviceA</module>
        <module>ssm_serviceB</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <web.version>2.4.0</web.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>${web.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${web.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210622668-288431690.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210632079-111486868.png)

 

 

Step7：通过两个项目的启动类 可以 分别启动两个项目。
　　或者 直接 mvn install 父工程（ssm_parent），然后执行打包好的 jar 包。

![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210650559-2016021306.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210703954-621960290.png)

 

 ![img](https://img2020.cnblogs.com/blog/1688578/202012/1688578-20201208210713292-839892222.png)

 

 



### 4、使用 maven 聚合的注意事项

（1）父工程的 pom.xml 文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【聚合项目 父工程：】
    使用 maven 创建聚合工程时，父工程中 使用 <packaging>pom</packaging> 指定类型为 pom。
    父工程一般 用于进行 依赖声明 以及 版本控制。
注：
    通过 <properties> 标签 以及 ${} 可以进行依赖（jar）版本的管理。
    使用 <dependencyManagement> 标签可以进行依赖声明。
    使用 <modules> 标签管理 子模块。
    
【父工程 pom.xml 举例：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lyh.ssm</groupId>
    <artifactId>ssm_parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>ssm_serviceA</module>
        <module>ssm_serviceB</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <web.version>2.4.0</web.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>${web.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${web.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（2）子工程的 pom.xml 文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
【聚合项目 子工程：】
    使用 maven 创建子工程时，使用 <parent> 标签指定 父工程。
    使用 <dependencies> 标签按需引入依赖，若父工程使用 <dependencyManagement> 进行版本控制，则子工程引入依赖时可以不指定版本 <version>（便于统一管理）。

【子工程 pom.xml 举例：】
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.lyh.ssm</groupId>
        <artifactId>ssm_parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <artifactId>bservice</artifactId>
    <name>bservice</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

（3）<dependencyManagement> 与 <dependencies> 区别
　　<dependencyManagement> 一般出现在 父工程中，用于 声明依赖 以及 版本控制，但是并没有真正引入依赖。
　　<dependencies> 是真正的引入依赖。
　　父工程中 使用 <dependencyManagement> 进行了版本控制，若子工程中 <dependencies> 引入依赖时使用 <version> 指定了版本，则以子工程的版本为主。若子工程中没有使用 <version>，则以父工程定义的 version 为主。