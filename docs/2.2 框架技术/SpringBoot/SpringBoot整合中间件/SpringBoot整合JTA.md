[TOC]

博客园[知了一笑]：[SpringBoot2 整合JTA组件，多数据源事务管理](https://www.cnblogs.com/cicada-smile/p/13289306.html)

# 一、JTA组件简介
## 1、JTA基本概念
JTA即Java-Transaction-API，JTA允许应用程序执行分布式事务处理，即在两个或多个网络计算机资源上访问并且更新数据。JDBC驱动程序对JTA的支持极大地增强了数据访问能力。

XA协议是数据库层面的一套分布式事务管理的规范,JTA是XA协议在Java中的实现,多个数据库或是消息厂商实现JTA接口,开发人员只需要调用SpringJTA接口即可实现JTA事务管理功能。

JTA事务比JDBC事务更强大。一个JTA事务可以有多个参与者，而一个JDBC事务则被限定在一个单一的数据库连接。下列任一个Java平台的组件都可以参与到一个JTA事务中

## 2、分布式事务
分布式事务（DistributedTransaction）包括事务管理器（TransactionManager）和一个或多个支持 XA 协议的资源管理器 ( Resource Manager )。

资源管理器是任意类型的持久化数据存储容器，例如在开发中常用的关系型数据库：MySQL，Oracle等，消息中间件RocketMQ、RabbitMQ等。

事务管理器提供事务声明，事务资源管理，同步，事务上下文传播等功能，并且负责着所有事务参与单元者的相互通讯的责任。JTA规范定义了事务管理器与其他事务参与者交互的接口，其他的事务参与者与事务管理器进行交互。

# 二、SpringBoot整合JTA
项目整体结构图
![](https://img2020.cnblogs.com/blog/1691717/202007/1691717-20200712181129569-1930578029.png)

## 1、核心依赖
```xml
<!--SpringBoot核心依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--JTA组件核心依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

## 2、环境配置
这里jtaManager的配置，在日志输出中非常关键。

```yml
spring:
  jta:
    transaction-manager-id: jtaManager
  # 数据源配置
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    data01:
      driverClassName: com.mysql.jdbc.Driver
      dbUrl: jdbc:mysql://localhost:3306/data-one
      username: root
      password: 000000
    data02:
      driverClassName: com.mysql.jdbc.Driver
      dbUrl: jdbc:mysql://localhost:3306/data-two
      username: root
      password: 000000
```

## 3、核心容器
这里两个数据库连接的配置手法都是一样的，可以在源码中自行下载阅读。基本思路都是把数据源交给JTA组件来统一管理，方便事务的通信。

数据源参数
```java
@Component
@ConfigurationProperties(prefix = "spring.datasource.data01")
public class DruidOneParam {
    private String dbUrl;
    private String username;
    private String password;
    private String driverClassName;
}
```
JTA组件配置
```java
package com.jta.source.conifg;

@Configuration
@MapperScan(basePackages = {"com.jta.source.mapper.one"},sqlSessionTemplateRef = "data01SqlSessionTemplate")
public class DruidOneConfig {

    private static final Logger LOGGER = LoggerFactory.getLogger(DruidOneConfig.class) ;

    @Resource
    private DruidOneParam druidOneParam ;

    @Primary
    @Bean("dataSourceOne")
    public DataSource dataSourceOne () {

        // 设置数据库连接
        MysqlXADataSource mysqlXADataSource = new MysqlXADataSource();
        mysqlXADataSource.setUrl(druidOneParam.getDbUrl());
        mysqlXADataSource.setUser(druidOneParam.getUsername());
        mysqlXADataSource.setPassword(druidOneParam.getPassword());
        mysqlXADataSource.setPinGlobalTxToPhysicalConnection(true);

        // 事务管理器
        AtomikosDataSourceBean atomikosDataSourceBean = new AtomikosDataSourceBean();
        atomikosDataSourceBean.setXaDataSource(mysqlXADataSource);
        atomikosDataSourceBean.setUniqueResourceName("dataSourceOne");
        return atomikosDataSourceBean;
    }

    @Primary
    @Bean(name = "sqlSessionFactoryOne")
    public SqlSessionFactory sqlSessionFactoryOne(
            @Qualifier("dataSourceOne") DataSource dataSourceOne) throws Exception{
        // 配置Session工厂
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSourceOne);
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        sessionFactory.setMapperLocations(resolver.getResources("classpath*:/dataOneMapper/*.xml"));
        return sessionFactory.getObject();
    }

    @Primary
    @Bean(name = "data01SqlSessionTemplate")
    public SqlSessionTemplate sqlSessionTemplate(
            @Qualifier("sqlSessionFactoryOne") SqlSessionFactory sqlSessionFactory) {
        // 配置Session模板
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

## 4、测试对比
这里通过两个方法测试结果做对比，在两个数据源之间进行数据操作时，只需要在接口方法加上@Transactional注解即可，这样保证数据在两个数据源间也可以保证一致性。
```java
@Service
public class TransferServiceImpl implements TransferService {

    @Resource
    private UserAccount01Mapper userAccount01Mapper ;

    @Resource
    private UserAccount02Mapper userAccount02Mapper ;

    @Override
    public void transfer01() {
        userAccount01Mapper.transfer("jack",100);
        System.out.println("i="+1/0);
        userAccount02Mapper.transfer("tom",100);
    }

    @Transactional
    @Override
    public void transfer02() {
        userAccount01Mapper.transfer("jack",200);
        System.out.println("i="+1/0);
        userAccount02Mapper.transfer("tom",200);
    }
}
```

# 三、JTA组件小结
在上面JTA实现多数据源的事务管理，使用方式还是相对简单，通过两阶段的提交，可以同时管理多个数据源的事务。但是暴露出的问题也非常明显，就是比较严重的性能问题,由于同时操作多个数据源,如果其中一个数据源获取数据的时间过长,会导致整个请求都非常的长,事务时间太长,锁数据的时间就会太长，自然就会导致低性能和低吞吐量。

因此在实际开发过程中，对性能要求比较高的系统很少使用JTA组件做事务管理。作为一个轻量级的分布式事务解决方案，在小的系统中还是值得推荐尝试的。

最后作为Java下的API，原理和用法还是值得学习一下，开阔眼界和思路。

# 四、源代码地址
GitHub·地址
https://github.com/cicadasmile/middle-ware-parent
GitEE·地址
https://gitee.com/cicadasmile/middle-ware-parent