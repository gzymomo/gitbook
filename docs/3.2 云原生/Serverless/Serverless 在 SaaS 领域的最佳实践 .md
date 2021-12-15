- [Serverless 在 SaaS 领域的最佳实践](https://mp.weixin.qq.com/s/cJoFMmhNw14Fcx7-TNxUew)



随着互联网人口红利逐渐减弱，基于流量的增长已经放缓，互联网行业迫切需要找到一片足以承载自身持续增长的新蓝海，<font color='red'>产业互联网正是这一宏大背景下的新趋势。我们看到互联网浪潮正在席卷传统行业，云计算、大数据、人工智能开始大规模融入到金融、制造、物流、零售、文娱、教育、医疗等行业的生产环节中，</font>这种融合称为**产业互联网**。而在产业互联网中，有一块不可小觑的领域是 **SaaS 领域**，它是 ToB 赛道的中间力量，比如 CRM、HRM、费控系统、财务系统、协同办公等等。



# 一、SaaS 系统面临的挑战

在消费互联网时代，大家是搜索想要的东西，各个厂商在云计算、大数据、人工智能等技术基座之上建立流量最大化的服务与生态，基于海量内容分发与流量共享为逻辑构建系统。而到了产业互联网时代，供给关系发生了变化，大家是定制想要的东西，需要从供给与需求两侧出发进行双向建设，这个时候系统的灵活性和扩展性面临着前所未有的挑战，尤其是 ToB 的 SaaS 领域。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWuqm1yXr3Yp4L2KIS15LeJAyfxtleic8Le7xwySEBcBn4QZ2pNHd45KXrJK5tIWWafaJSFfexMlqdg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

特别是对于当下的经济环境，SaaS 厂商要明白，不能再通过烧钱的方式，只关注在自己的用户数量上，而更多的要思考如何帮助客户降低成本、提高效率，所以需要将更多的精力放在自己产品的定制化能力上。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWuqm1yXr3Yp4L2KIS15LeJAwqaPO7RxUloe3aaxgX27VWic5NmxvdAnSAouicticTRI6Fe8yRiaR6MooA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 二、如何应对挑战

SaaS 领域中的佼佼者 Salesforce，将 CRM 的概念扩展到 Marketing、Sales、Service，而这三块领域中只有 Sales 有专门的 SaaS 产品，其他两个领域都是各个 ISV 在不同行业的行业解决方案，靠的是什么？毋庸置疑，是 Salesforce 强大的  aPaaS 平台。ISV、内部实施、客户均可以在各自维度通过 aPaaS 平台构建自己行业、自己领域的 SaaS  系统，建立完整的生态。所以在我看来，现在的 Salesforce 已经由一家 SaaS 公司升华为一家 aPaaS  平台公司了。这种演进的过程也印证了消费互联网和产业互联网的转换逻辑以及后者的核心诉求。



然而不是所有 SaaS 公司都有财力和时间去孵化和打磨自己的 aPaaS 平台，但市场的变化、用户的诉求是实实在在存在的。若要生存，就要求变。这个变的核心就是能够让自己目前的 SaaS 系统变得灵活起来，相对建设困难的 aPaaS 平台，我们其实可以**选择轻量且有效的 Serverless 方案来提升现有系统的灵活性和可扩展性，从而实现用户不同的定制需求。**

# 三、Serverless 工作流

Serverless 工作流是一个用来协调多个分布式任务执行的全托管云服务。在  Serverless工作流中，可以用顺序、分支、并行等方式来编排分布式任务，Serverless  工作流会按照设定好的步骤可靠地协调任务执行，跟踪每个任务的状态转换，并在必要时执行您定义的重试逻辑，以确保工作流顺利完成。Serverless  工作流通过提供日志记录和审计来监视工作流的执行，可以轻松地诊断和调试应用。



下面这张图描述了 Serverless 工作流如何协调分布式任务，这些任务可以是函数、已集成云服务 API、运行在虚拟机或容器上的程序。



![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWuqm1yXr3Yp4L2KIS15LeJA0zcGria0BTNs0LuQBNnHuFGK2hOVWTAzg55O6KEf45jNI8XT5HnI0QA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



看完 Serverless 工作流的介绍，大家可能已经多少有点思路了吧。系统灵活性和可扩展性的核心是服务可编排，无论是以前的 BPM 还是现在的  aPaaS。所以基于 Serverless 工作流重构 SaaS  系统灵活性方案的核心思路，是将系统内用户最希望定制的功能进行梳理、拆分、抽离，再配合函数计算（FC）提供无状态的能力，通过 Serverless 工作流进行这些功能点的编排，从而实现不同的业务流程。

