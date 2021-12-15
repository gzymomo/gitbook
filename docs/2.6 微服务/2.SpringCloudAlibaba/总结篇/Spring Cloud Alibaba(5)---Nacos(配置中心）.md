[Spring Cloud Alibaba(5)---Nacos(配置中心）](https://www.cnblogs.com/qdhxhz/p/14658922.html)

##  一、Nacos 服务端初始化

#### 1、启动Nacos客户端

有关Nacos搭建我这里不在陈述，上面博客有写，或者直接看官网如何搭建：[Nacos 官网](https://nacos.io/zh-cn/docs/quick-start.html)

```
sh startup.sh -m standalone
```

#### 2、添加配置

启动好Nacos之后，在Nacos添加如下的配置

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170307971-331066086.jpg)

具体含义下面会做介绍



##  二、springBoot整合Nacos(配置中心） 

`说明` 这里贴出的代码是在上篇博客 **Spring Cloud Alibaba(4)---Nacos(注册中心）** 中项目的基础上添加。

#### 1、pom.xml

```xml
  <!--添加nacos配置中心-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```

#### 2、Controller层

```java
@RestController
@RequestMapping("api/v1/config")
public class ConfigTestController {

    /**
     * nacos获取配置
     */
    @Value("${user.name}")
    private String name;

    @RequestMapping("test-config")
    public Object findByGoodsId() {
        return name;
    }

}
```

#### 3、bootstrap.yml

```yml
spring:
  application:
    name: mall-goods

  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848 #Nacos配置中心地址
        file-extension: yaml #文件拓展格式

  profiles:
    active: dev
```

#### 4、测试

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170324665-817530068.jpg)

成功获取Nacos配置的数据



##  三、Nacos配置管理的模型

对于Nacos配置管理, **通过Namespace, Group, DataId能够定位到一个配置集**。

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170336897-1442835440.jpg)

#### 1、配置管理模型概念

1)`Namespace`(命名空间)

命名空间(namespace)可用于对不同的环境进行配置隔离. 例如: 可以隔离开发环境, 测试环境, 生成环境. 因为他们的配置可能各不相同. 或者是隔离不同的用户, 不同的开发人员使

用同一个Nacos管理各自的配置, 可通过namespace进行隔离。**不同的命名空间下, 可以存在相同名称的配置分组(Group)或配置项(Data Id)**

**默认值：public**

2)`Group`(配置分组)

配置分组就是上图中的Group. 配置分组是对配置集进行分组. 通过一个有意义的字符串(如: buy, trade)来表示. **不同的配置分组下可以有相同的配置集(Data ID)**。

**默认值:DEFAULT_GROUP**

3)`DataId`(配置集)

在系统中, **通常一个配置文件, 就是一个配置集**。一个配置集可以包含系统的各种配置信息. 例如:一个配置集可能包含系统的数据源、连接池， 日志等级的配置信息。每个配置集

都可以定义一个有意义的名称, 就是配置集的Id, 即Data Id

4)`配置项`

**配置集中包含的一个个配置内容, 就是配置项**。 他代表具体的可配置的参数. 通常以key=value的形式存在.

#### 2、通俗理解

这里通俗去理解这几个概念含义

```
 Namespace: 代表不同的环境, 如: 开发、测试， 生产等
 Group: 可以代表某个项目, 如XX就业项目, XX电商项目
 DataId: 每个项目下往往有若干个工程, 每个配置集(DataId)是一个工程的主配置文件(比如这里的mall-goods.yaml就是一个配置集)
```

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170404623-1222922394.jpg)

#### 3、页面理解

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170420857-559674748.jpg)

```
1）代表 Namespace(命名空间)，默认就创建好public，这里创建好了一个dev
2）代表 Group(配置分组)，这里默认分组 DEFAULT_GROUP
3）代表 DataId(配置集)，这里有个配置mall-goods.yaml，配置集里有配置项user.name: "我是张三的好朋友李四"
```



##  四、补充

#### 1、为什么要用bootstrap.yaml

为什么用bootstrap.yaml而不用application.xml官方有说明

```
必须使用 bootstrap.properties 配置文件来配置Nacos Server 地址
```

虽然 **bootstrap.yaml** 和 **application.xml** 都属于配置文件，功能也一样。但技术上，bootstrap.yml由父Spring ApplicationContext加载。父ApplicationContext会在

application.yml之前被加载。当使用 Spring Cloud 的时候，配置信息一般是从 config server 加载的，为了取得配置信息（比如密码等），你需要一些提早的或引导配置。

因此，把 config server信息放在 bootstrap.yml，用来加载真正需要的配置信息。

`说明` bootstrap.properties 和 bootstrap.yaml到没有什么区别，只是格式上有点不一样。

#### 2、DataId(配置集)和微服务对于关系

我们在Nacos配置的配置集叫: **mall-goods.yaml**，它是如何和我们项目匹配上的呢？

我们再来看下我们的 **bootstrap.yml** 的配置

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210414170431496-2113991990.jpg)

我们前面说了，通过namespace, group, dataId能够定位到一个配置集。从这个配置中我们没有去指明具体namespace，那就代表采用默认的 public。没有制定group，

代表采用默认DEFAULT_GROUP。那么配置集就为

```
${spring.application.name}.${file-extension:properties} #这里就相当于 mall-goods.yaml
${spring.application.name}-${profile}.${file-extension:properties} # 这里就相当于 mall-goods-dev.yaml 
```

如果同时配置的话，mall-goods-dev.yaml会覆盖mall-goods.yaml中的配置

#### 3、补充

其实我这里还有很多细节没讲，比如怎么指定分组，指定命名空间和一些其它规则，具体可以看官网说明，讲的还挺清楚的。

官方讲解：[Spring Cloud Alibaba Nacos Config](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)

还有一些Nacos集群搭建、Naocos将数据存储到mysql数据库的知识，这里也不说了。自己可以网上查查。

`GitHub地址`:[spring-cloud-alibaba-study](https://github.com/yudiandemingzi/spring-cloud-alibaba-study/tree/nacos)