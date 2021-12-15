[TOC]

# 1、Mybatis框架
MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

## 1.1 mybatis特点
1. sql语句与代码分离，存放于xml配置文件中，方便管理
2. 用逻辑标签控制动态SQL的拼接，灵活方便
3. 查询的结果集与java对象自动映射
4. 编写原生态SQL，接近JDBC
5. 简单的持久化框架，框架不臃肿简单易学
## 1.2 适用场景
- MyBatis专注于SQL本身，是一个足够灵活的DAO层解决方案。
- 对性能的要求很高，或者需求变化较多的项目，MyBatis将是不错的选择。

# 2、SpringBoot整合MyBatis
## 2.1 核心依赖
```xml
<!-- mybatis依赖 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<!-- mybatis的分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.1.6</version>
</dependency>
```
## 2.2 核心配置
```yml
mybatis:
  # mybatis配置文件所在路径
  config-location: classpath:mybatis.cfg.xml
  type-aliases-package: com.boot.mybatis.entity
  # mapper映射文件
  mapper-locations: classpath:mapper/*.xml
```

# 3、集成分页插件
## 3.1 mybatis配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <plugins>
        <!--mybatis分页插件-->
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql"/>
        </plugin>
    </plugins>
</configuration>
```
## 3.2 分页实现代码
```java
@Override
public PageInfo<ImgInfo> queryPage(int page,int pageSize) {
    PageHelper.startPage(page,pageSize) ;
    ImgInfoExample example = new ImgInfoExample() ;
    // 查询条件
    example.createCriteria().andBEnableEqualTo("1").andShowStateEqualTo(1);
    // 排序条件
    example.setOrderByClause("create_date DESC,img_id ASC");
    List<ImgInfo> imgInfoList = imgInfoMapper.selectByExample(example) ;
    PageInfo<ImgInfo> pageInfo = new PageInfo<>(imgInfoList) ;
    return pageInfo ;
}
```