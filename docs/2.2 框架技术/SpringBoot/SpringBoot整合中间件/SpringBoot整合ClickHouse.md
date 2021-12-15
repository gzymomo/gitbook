[TOC]

# 1、ClickHouse简介
Yandex开源的数据分析的数据库，名字叫做ClickHouse，适合流式或批次入库的时序数据。ClickHouse不应该被用作通用数据库，而是作为超高性能的海量数据快速查询的分布式实时处理平台，在数据汇总查询方面(如GROUP BY)，ClickHouse的查询速度非常快。

## 1.1 数据分析能力
- OLAP场景特征

>大多数是读请求
数据总是以相当大的批(> 1000 rows)进行写入
不修改已添加的数据
每次查询都从数据库中读取大量的行，但是同时又仅需要少量的列
宽表，即每个表包含着大量的列
较少的查询(通常每台服务器每秒数百个查询或更少)
对于简单查询，允许延迟大约50毫秒
列中的数据相对较小： 数字和短字符串(例如，每个URL 60个字节)
处理单个查询时需要高吞吐量（每个服务器每秒高达数十亿行）
事务不是必须的
对数据一致性要求低
每一个查询除了一个大表外都很小
查询结果明显小于源数据，换句话说，数据被过滤或聚合后能够被盛放在单台服务器的内存中

- 列式数据存储

行式数据和列式数据对比：
> 分析类查询，通常只需要读取表的一小部分列。在列式数据库中可以只读取需要的数据。数据总是打包成批量读取的，所以压缩是非常容易的。同时数据按列分别存储这也更容易压缩。这进一步降低了I/O的体积。由于I/O的降低，这将帮助更多的数据被系统缓存。

# 2、SpringBoot整个ClickHouse
## 2.1 核心依赖
```xml
<dependency>
    <groupId>ru.yandex.clickhouse</groupId>
    <artifactId>clickhouse-jdbc</artifactId>
    <version>0.1.53</version>
</dependency>
```
## 2.2 配属数据源
```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    click:
      driverClassName: ru.yandex.clickhouse.ClickHouseDriver
      url: jdbc:clickhouse://127.0.0.1:8123/default
      initialSize: 10
      maxActive: 100
      minIdle: 10
      maxWait: 6000
```
## 2.3 Druid连接池配置
```java
@Configuration
public class DruidConfig {
    @Resource
    private JdbcParamConfig jdbcParamConfig ;
    @Bean
    public DataSource dataSource() {
        DruidDataSource datasource = new DruidDataSource();
        datasource.setUrl(jdbcParamConfig.getUrl());
        datasource.setDriverClassName(jdbcParamConfig.getDriverClassName());
        datasource.setInitialSize(jdbcParamConfig.getInitialSize());
        datasource.setMinIdle(jdbcParamConfig.getMinIdle());
        datasource.setMaxActive(jdbcParamConfig.getMaxActive());
        datasource.setMaxWait(jdbcParamConfig.getMaxWait());
        return datasource;
    }
}
```
## 2.4 参数配置类
```java
@Component
@ConfigurationProperties(prefix = "spring.datasource.click")
public class JdbcParamConfig {
    private String driverClassName ;
    private String url ;
    private Integer initialSize ;
    private Integer maxActive ;
    private Integer minIdle ;
    private Integer maxWait ;
    // 省略 GET 和 SET
}
````
这样整合代码就完成了。

# 3、操作案例演示
## 3.1 Mapper接口
```java
public interface UserInfoMapper {
    // 写入数据
    void saveData (UserInfo userInfo) ;
    // ID 查询
    UserInfo selectById (@Param("id") Integer id) ;
    // 查询全部
    List<UserInfo> selectList () ;
}
```
这里就演示简单的三个接口。

## 3.2 Mapper.xml文件
```xml
<mapper namespace="com.click.house.mapper.UserInfoMapper">
    <resultMap id="BaseResultMap" type="com.click.house.entity.UserInfo">
        <id column="id" jdbcType="INTEGER" property="id" />
        <result column="user_name" jdbcType="VARCHAR" property="userName" />
        <result column="pass_word" jdbcType="VARCHAR" property="passWord" />
        <result column="phone" jdbcType="VARCHAR" property="phone" />
        <result column="email" jdbcType="VARCHAR" property="email" />
        <result column="create_day" jdbcType="VARCHAR" property="createDay" />
    </resultMap>
    <sql id="Base_Column_List">
        id,user_name,pass_word,phone,email,create_day
    </sql>
    <insert id="saveData" parameterType="com.click.house.entity.UserInfo" >
        INSERT INTO cs_user_info
        (id,user_name,pass_word,phone,email,create_day)
        VALUES
        (#{id,jdbcType=INTEGER},#{userName,jdbcType=VARCHAR},#{passWord,jdbcType=VARCHAR},
        #{phone,jdbcType=VARCHAR},#{email,jdbcType=VARCHAR},#{createDay,jdbcType=VARCHAR})
    </insert>
    <select id="selectById" parameterType="java.lang.Integer" resultMap="BaseResultMap">
        select
        <include refid="Base_Column_List" />
        from cs_user_info
        where id = #{id,jdbcType=INTEGER}
    </select>
    <select id="selectList" resultMap="BaseResultMap" >
        select
        <include refid="Base_Column_List" />
        from cs_user_info
    </select>
</mapper>
```
这里 create_day 是以字符串的方式在转换，这里需要注意下。

## 3.3 控制层接口
```java
@RestController
@RequestMapping("/user")
public class UserInfoController {
    @Resource
    private UserInfoService userInfoService ;
    @RequestMapping("/saveData")
    public String saveData (){
        UserInfo userInfo = new UserInfo () ;
        userInfo.setId(4);
        userInfo.setUserName("winter");
        userInfo.setPassWord("567");
        userInfo.setPhone("13977776789");
        userInfo.setEmail("winter");
        userInfo.setCreateDay("2020-02-20");
        userInfoService.saveData(userInfo);
        return "sus";
    }
    @RequestMapping("/selectById")
    public UserInfo selectById () {
        return userInfoService.selectById(1) ;
    }
    @RequestMapping("/selectList")
    public List<UserInfo> selectList () {
        return userInfoService.selectList() ;
    }
}
```