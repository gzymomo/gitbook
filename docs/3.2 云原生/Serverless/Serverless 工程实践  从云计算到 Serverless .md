## Serverless 工程实践 | 从云计算到 Serverless

# 云计算的诞生

在 1961 年麻省理工学院百周年纪念典礼上，约翰·麦卡锡（1971 年图灵奖获得者）第一次提出了 “Utility Computing”  的概念，这个概念可以认为是云计算的一个“最初的”，“超前的” 遐想模型；1984 年，SUN 公司联合创始人 John  Gage（约翰·盖奇）提出了“网络就是计算机（The Network is the  Computer）”的重要猜想，用于描述分布式计算技术带来的新世界；到了 1996  年，康柏（Compaq）公司的一群技术主管在讨论计算业务的发展时，首次使用了 Cloud Computing 这个词，并认为商业计算会向  Cloud Computing 的方向转移。**这也是 “云计算” 从雏形到正式被提出的基本过程。**

自 “云计算” 被提出之后，其可谓是如同雨后春笋般，蓬勃发展：

-  2003 年到 2006 年间，谷歌发表了 The Google File System、MapReduce: Simplified Data Processing on Large Clusters、Bigtable: A Distributed Storage System for Structured Data 等文章，这些文章指明了 HDFS（分布式文件系统），MapReduce（并行计算）和  Hbase（分布式数据库）的技术基础以及未来机会，至此奠定了云计算的发展方向。

- 2006 年，Google 首席执行官埃里克·施密特（Eric Schmidt）在搜索引擎大会（SESSanJose 2006）首次公开正式的提出  “云计算”（Cloud  Computing）的概念，同年亚马逊第一次将其弹性计算能力作为云服务进行售卖，这也标志着云计算这种新的商业模式正式诞生。两年后，即  2008年，微软发布云计算战略和平台 Windows Azure Platform，尝试将技术和服务托管化、线上化。

- 2009 年，UC Berkeley 发表了：Above the Clouds: A Berkeley View of Cloud  Computing，在该文章中，明确指出：云计算是一个即将实现的古老梦想，是计算作为基础设施这一长久以来梦想的新称谓，它在最近正快速变为商业现实。在该文章中，明确的为云计算做了定义：云计算包含互联网上的应用服务及在数据中心提供这些服务的软硬件设施。同时在该文章中，也提出了云计算所面临的挑战和机遇，更对云计算的未来发展方向进行了大胆预测。

**同年，阿里软件在江苏南京建立首个“电子商务云计算中心”（即现在的阿里云）。至此，云计算进入到了更加快速的发展阶段。**

# 从云计算到 Serverless

云计算飞速发展的阶段，云计算的形态也在不断的演进，从 IaaS 到 PaaS，再到 SaaS，云计算逐渐的 “找到了正确的发展方向”。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uvCkteiagYWWibGYlSGicibrFRjUHHKdRCHrPTMtvpia19WKZjkA5icoibulr0TZm014dvEJH4QGEbIsrNg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



2012 年由 Iron.io 的副总裁 Ken Form 所写的一篇名为《Why The Future of Software and Apps is Serverless》 的文章中，提出了一个新的观点：即使云计算的已经逐渐的兴起，但是大家仍然在围绕着服务器转。不过，这不会持续太久，  云应用正在朝着无服务器方向发展，这将对应用程序的创建和分发产生重大影响。**并首次将 “Serverless” 这个词带进了大众的视野。**

一直到 2014 年 Amazon 发布了 AWS Lambda 让 “Serverless”  这一范式提高到一个全新的层面，为云中运行的应用程序提供了一种全新的系统体系结构，至此再也不需要在服务器上持续运行进程以等待 HTTP 请求或  API 调用，而是可以通过某种事件机制触发代码执行，通常这只需要在 AWS 的某台服务器上配置一个简单的功能。此后 Ant Stanley 在  2015 年 7 月名为 Server are Dead…的文章中更是围绕着 AWS Lambda 及刚刚发布的 AWS API Gateway 这两个服务解释了他心目中的 Serverless，并说 Servers are dead … they just don’t know it  yet.

2015 年，在 AWS 的 re:Invent 大会上，Serverless 的这个概念更是反复的出现，其中包括了 The Serverless  Company Using AWS Lambda 和 JAWS：The Monstrously Scalable Serverless  Framework 的这些演讲。

随着 Serverless 这个概念的进一步发酵，2016 年 10 月在伦敦举办了第一届的  ServerlessConf，在两天时间里面，来自全世界 40 多位演讲嘉宾为开发者分享了关于这个领域进展，并且对未来进行了展望，提出来了  Serverless 的发展机会以及所面临的挑战，**这场大会是针对 Serverless 领域的第一场具有较大规模的会议，在 Serverless 的发展史上具有里程碑的意义。**

截止到 2017 年，各大云厂商基本上都已经在 Serverless 进行了基础的布局，尤其是国内的几大云厂商，也都先后在这一年迈入  “Serverless时代”。从 IaaS 到 PaaS 再到 SaaS 的过程中，云计算所表现出的去服务器化越来越明显，那么 Ken Form 所提出来的 Serverless 又是什么，它在云计算发展的过程中，又在扮演什么角色呢？



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uvCkteiagYWWibGYlSGicibrFR3ia8Cy7TQpPPqAnGOuveicwkjHmPbQD4YKlHq2siaG77nrTBsfCHz3g6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



IaaS、PaaS、SaaS的区别

# 什么是 Serverless

云计算的十余年发展让整个互联网行业发生了翻天覆地的变化，Serverless 作为云计算的产物，或者说是云计算在某个时代的表现，被很多人认为是真正意义上的云计算，伯克利团队甚至断言 Serverless 将会是引领云计算下一个十年的新范式。

Serverless 翻译成中文是无服务器，所谓的无服务器并非是说不需要依靠服务器等资源，而是说**开发者再也不用过多考虑服务器的问题，可以更专注在产品代码上，同时计算资源也开始作为服务出现，**而不是作为服务器的概念出现，Serverless 是一种构建和管理基于微服务架构的完整流程，允许用户在服务部署级别而不是服务器部署级别来管理用户的应用部署。

与传统架构的不同之处在于，**它完全由第三方管理，由事件触发，**存在于无状态（Stateless），暂存（可能只存在于一次调用的过程中）在计算容器内，Serverless 部署应用无须涉及更多的基础设施建设，就可以基本实现自动构建、部署和启动服务。

近些年来，微服务（Micro  Service）是软件架构领域另一个热门的话题，如果说微服务是以专注于单一责任与功能的小型功能块为基础，利用模组化的方式组合出复杂的大型应用程序，那么可以进一步认为 Serverless 架构可以提供一种更加 “代码碎片化” 的软件架构范式，而这一部分称之为 Function as a  Services（FaaS）。而所谓的“函数”提供的是相比微服务更加细小的程序单元。

例如，可以通过微服务代表为某个客户执行所有 CRUD 操作所需的代码，而 FaaS 中的函数可以代表客户所要执行的每个操作：创建、读取、更新以及删除。当触发 “创建账户”  事件后，将通过函数的方式执行相应的“函数”。单就这一层意思来说，可以简单地将 Serverless 架构与 FaaS  概念等同起来。但是就具体的概念深刻探索的话，Serverless 和 FaaS 还是不同的，Serverless 和 FaaS  被广为接受的关系是：

**Serverless = FaaS + BaaS （+ .....）**

在这个关系中，可以看到 Serverless 的组成除了 FaaS 和 BaaS 之外，还有一系列的省略号，其实这是 Serverless 给予给大家的遐想空间，给予这个时代的一些期待。

# Serverless 的发展历程

从 2012 年，Serverless 概念被正式提出之后，2014 年 AWS 带领 Lambda 开启了 Serverless 的商业化，再到 2017 年各大厂商纷纷布局 Serverless 领域，再到 2019 年，Serverless 成为热点议题在 KubeCon  中被众多人参与探讨，Serverless 随着时间的不断推进，各种技术部的不断进步，正在逐渐的朝着更完整，更清晰的方向发展，随着 5G  时代的到来，Serverless 将会在更多领域发挥至关重要的作用。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/hIibsapxkj1uvCkteiagYWWibGYlSGicibrFR6VE5WhAEiaBmUgAbQqkicqqwQBQJpfk9SYfibvvytA2v2RL4rYcMGvq7w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



从 IaaS 到 FaaS 再到 SaaS，再到如今的 Serverless；从虚拟空间到云主机，从自建数据库等业务，到云数据库等服务，云计算的发展是迅速的，未来的方向和形态却是模糊的，没人知道云计算的终态是什么。