[Springboot项目架构设计](https://www.cnblogs.com/lucky_hu/p/14458118.html)



### 架构的艺术

  架构是一门艺术,和开发语言无关。架构是整个项目的顶层设计。特别是在Devops的流行的今天,开发人员也要参与项目部署和运维的工作。站在项目架构的角度,架构不仅要考虑项目的开发,还要考虑项目的部署。
 本文更多地聚焦于项目开发的架构,即中小型项目的搭建。

  架构的功能:

- 方便团队分工
- 职责分离
- 可扩展
- 重用
- 方便排错

### 理解阿里应用分层架构

  在国内Java应用方面,阿里应该是权威。在阿里《Java开发规范》中有一个推荐的分层。



​    [![img](https://img.zhikestreet.com/20210227110310.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](https://img.zhikestreet.com/20210227110310.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)



- 开放 API 层： 可直接封装 Service 接口暴露成 RPC 接口； 通过 Web 封装成 http 接口； 网关控制层等。

- 终端显示层：各个端的模板渲染并执行显示的层。当前主要是 velocity 渲染，JS 渲染，JSP 渲染，移
   动端展示等。

- Web 层：主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。

- Service 层：相对具体的业务逻辑服务层。

- Manager 层：通用业务处理层，它有如下特征：

  1） 对第三方平台封装的层，预处理返回结果及转化异常信息，适配上层接口。
   2） 对 Service 层通用能力的下沉，如缓存方案、中间件通用处理。
   3） 与 DAO 层交互，对多个 DAO 的组合复用。

- DAO 层：数据访问层，与底层 MySQL、Oracle、Hbase、OB 等进行数据交互。

- 第三方服务：包括其它部门 RPC 服务接口，基础平台，其它公司的 HTTP 接口，如淘宝开放平台、支
   付宝付款服务、高德地图服务等。

- 外部数据接口：外部（应用）数据存储服务提供的接口，多见于数据迁移场景中。

分层领域模型规约：

- DO（Data Object）：此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
- DTO（Data Transfer Object）：数据传输对象，Service 或 Manager 向外传输的对象。
- BO（Business Object）：业务对象，可以由 Service 层输出的封装业务逻辑的对象。
- Query：数据查询对象，各层接收上层的查询请求。注意超过 2 个参数的查询封装，禁止使用 Map 类
   来传输。
- VO（View Object）：显示层对象，通常是 Web 向模板渲染引擎层传输的对象。

#### manager层的使用

  其它层的关系,大家都很容易理解。但是manager层和service层容易搞混。这也是初学者容易迷惑的地方,笔者也是反复揣摩和多次查阅了相关资料,有了一个比较合理的理解。

  笔者认为,controller调用service层,service层调用dao层(或Mapper层)。manager层出现的目的就是供其他service层使用，就是说service不能直接调用service，而是通过manager来做调用。

- manager层是对service层的公共能力的提取。
- manager层还可以下沉内部的common/helper等。

  笔者所在团队,Java项目的搭建也是参照的这套标准。但是没有放之四海而皆准的标准,在实际搭建中都会根据实际业务和团队文化而因地制宜的有所改造。

  这里值得注意的是manager层,我想可能很多项目是没有使用这个层的。或者直接使用的common/helper作为替代。

### superblog项目架构

  鉴于以上的阐述,在设计superblog的架构上就基本心中有数了。我们姑且将博客中的创作称为作品,这里就以作品创建为例来展示一下项目各层之间的调用关系。

笔者把Superblog创作定义为以下类型:

- Article(文章)
- Weekly(周刊)
- Solution(解决方案)



​    [![img](https://img.zhikestreet.com/superblog_product_call_chain.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](https://img.zhikestreet.com/superblog_product_call_chain.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)



  以上这种分层和调用关系和开发语言没有太多关系,无论是C#还是Java都是适用的,这就想三层框架，ｍｖｃ架构一样,是一种设计理念和设计模式。

  架构的关系明确之后,搭建框架就是水到渠成的事情。

  讲到这里,终于要轮到Springboot上场了。下面我们就以Springboot来搭建一套开发架构。

  本项目包含一个父工程super-blog和 blog-base, blog-pojo,blog-dao, blog-manager, blog-service,blog-web,blog-webapi。

- blog-base,blog-pojo 为其他模块的公共模块,blog-base依赖blog-pojo;
- 七个子模块都依赖父模块,blog-dao 依赖 blog-base;
- blog-service 依赖 blog-dao，也就间接依赖 blog-base;
- blog-web和blog-webapi是并列的。都依赖 blog-service,也就间接依赖blog-base和blog-dao。
- blog-manager是blog-service中公共的业务层。



​    [![img](https://img.zhikestreet.com/superblog-mutimodule.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](https://img.zhikestreet.com/superblog-mutimodule.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)



> 模块的概念在不同语言之间叫法不同。在`.NET Core`项目中,每一层都是一个程序集,程序集之间通过引用来表达调用关系。在Springboot项目中,每一层都是一个模块,各个模块之间通过maven配置依赖关系。

比如,以下配置就表示父工程下有哪些子模块。

```xml
<!--声明子模块-->
    <modules>
        <module>blog-pojo</module>
        <module>blog-dao</module>
		    <module>blog-service</module>
		    <module>blog-manager</module>
		    <module>blog-base</module>
		    <module>blog-webapp</module>
	</modules>
```

而这个配置又表示了一种子模块之间的依赖关系。

```xml
	<!-- 添加 blog-service 的依赖 -->
		<dependency>
			<groupId>com.zhike</groupId>
			<artifactId>blog-service</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
```

  值得注意的是,在Springboot之间通常使用单向依赖,避免产生相互引用。双向引用在编译阶段不会报错,但是运行时就会异常。

  按照惯例,也要将搭建好的项目架构展示一下:



​    [![img](https://img.zhikestreet.com/20210227180906.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](https://img.zhikestreet.com/20210227180906.png?imageView2/0/q/75|watermark/2/text/NTJJbnRlcnZpZXc=/font/5a6L5L2T/fontsize/240/fill/IzBFMDkwNQ==/dissolve/100/gravity/SouthEast/dx/10/dy/10)