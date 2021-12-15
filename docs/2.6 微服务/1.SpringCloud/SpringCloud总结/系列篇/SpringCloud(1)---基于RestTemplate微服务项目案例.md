[SpringCloud(1)---基于RestTemplate微服务项目案例](https://www.cnblogs.com/qdhxhz/p/9349950.html)

在写SpringCloud搭建微服务之前，我想先搭建一个不通过springcloud只通过SpringBoot和Mybatis进行模块之间额通讯。然后在此基础上再添加SpringCloud框架。

**下面先对案例做个说明**

   该项目有一个maven父模块，其中里面有三个子模块：

​                  **serverspringcloud**：整体父工程。

​               **serverspringcloud-api**：公共子模块，放公共实体对象。

 **serverspringcloud-provider-dept-8001**：部门微服务提供者。

 **serverspringcloud-consumer-dept-80**：部门微服务消费者。调用部分微服务提供者接口进行CRUD操作。

 

## 一、构建父工程

**主要步骤**:

  (1) 创建一个Maven父工程并命名serverspringcloud

  (2)  打包方式为POM

  (3) 在pom.xml中定义各依赖的版本号（若Module中pom.xml的依赖没有指定版本号，则会根据父工程的版本号加入依赖）

#### 1、 创建一个Maven父工程

![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722132114477-74335401.png)

#### 2、 打包方式为POM

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722211115366-1700379075.png)

#### 3、 在pom.xml中定义各依赖的版本号

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

 

## 二、构建**serverspringcloud-api(**公共子模块)

 **主要步骤**

   (1) 在父工程下新建Maven的Module，打包方式为jar

   (2) 在该Module下pom.xml中加入其它需要的依赖

   (3) 完成后先clean一下Maven项目，然后再install提供给其它模块调用 

####  1、在父工程下新建Maven的Module，打包方式为jar

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722133152520-1975660115.png)

在创建完子模块后看下pom.xml的Overview视图的一些信息。

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722133253873-1249020168.png)

####  2、 在该Module下pom.xml中加入其它需要的依赖

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

####  3、我在这里面添加了一个Dept实体

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722134037620-591891389.png)

#### Dept实体

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) Dept类

 

##  三、创建部门微服务提供者

 步骤：这个就比较复杂了，具体看下面的图

![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722134747004-245682793.png)

下面就展示几个比较重要的环节

####  1、pom.xml文件

  **（1）先看下pom.xml的Overview视图的一些信息。**

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722135049418-758930536.png)

 **（2）pom.xml**

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

#### 2、application.yml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
server:
  port: 8001            #端口号
  
mybatis:
  config-location: classpath:mybatis/mybatis.cfg.xml        # mybatis配置文件所在路径
  type-aliases-package: com.jincou.springcloud.entities     # 所有Entity别名类所在包
  mapper-locations:
  - classpath:mybatis/mapper/**/*.xml                       # mapper映射文件
    
spring:
   application:
    name: serverspringcloud-dept             #这个名字很重要后期如果注入eureka就很重要
   datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/cloudDB01            # 数据库名称
    username: root
    password: root
    dbcp2:
      min-idle: 5                                           # 数据库连接池的最小维持连接数
      initial-size: 5                                       # 初始化连接数
      max-total: 5                                          # 最大连接数
      max-wait-millis: 200                                  # 等待连接获取的最大超时时间
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 3、MySQL表信息

![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722135947002-269943376.png)

#### 4、DAO接口信息

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Mapper
public interface DeptDao
{
    public boolean addDept(Dept dept);//添加部门

    public Dept findById(Long id);   //通过id找该部门数据

    public List<Dept> findAll();     //查看所有部门
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####  5、Controller层类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
public class DeptController
{
    @Autowired
    private DeptService service;

   //添加部门接口
    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(@RequestBody Dept dept)
    {
        return service.add(dept);
    }
    //通过部门id查找部门信息
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") Long id)
    {
        return service.get(id);
    }
    //查找所有部门信息
    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list()
    {
        return service.list();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####  6、测试

 先做个小测试，看数据库连接是否成功，调用api模块是否成功。

![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722140812046-1674077758.png)

说明测试成功！

 

##  四、创建部门微服务消费者

**主要步骤如图**

 ![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722141122577-839639559.png)

#### 1、pom.xml文件

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) pom.xml

#### 2、application.yml

```
server:
  port: 80
```

#### 3、ConfigBean配置类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@Configuration
public class ConfigBean //    @Configuration配置   ConfigBean = applicationContext.xml
{ 
    @Bean
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    } 
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### 4、DeptController_Consumer类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
@RestController
public class DeptController_Consumer
{

    private static final String REST_URL_PREFIX = "http://localhost:8001";

    /**
     * 使用 使用restTemplate访问restful接口非常的简单粗暴无脑。 (url, requestMap,
     * ResponseBean.class)这三个参数分别代表 REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。
     */
    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping(value = "/consumer/dept/add")
    public boolean add(Dept dept)
    {
        return restTemplate.postForObject(REST_URL_PREFIX + "/dept/add", dept, Boolean.class);
    }

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id)
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/get/" + id, Dept.class);
    }

    @SuppressWarnings("unchecked")
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list()
    {
        return restTemplate.getForObject(REST_URL_PREFIX + "/dept/list", List.class);
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

#### **5、测试**

![img](https://images2018.cnblogs.com/blog/1090617/201807/1090617-20180722141934526-1060126218.png)

测试成功，当我调用消费者接口的时候，它会再去调用提供者的接口。

 

## 五、总结

   整个项目终于跑通，然后再来屡一下思路，其实还是蛮简单的。

 （1）通过maven构建父子工程，一个个子模块就是一个个独立的进程（因为他们端口号都不一样），也就是微服务。

  （2）模块之间的调用只要把你需要的模块放到你的pom.xml中，这样就会打成jar包，就可以供该模块调用。

  （3）接口之间的调用只要通过RestTemplate工具类就可以了。