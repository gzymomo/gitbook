## 脱敏配置Quick Start——Spring 显示配置

以下介绍基于Spring如何快速让系统支持脱敏配置。

### 1、引入依赖

```xml
<!-- for spring namespace -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
```

### 2、创建脱敏配置规则对象

在创建数据源之前，需要准备一个EncryptRuleConfiguration进行脱敏的配置，以下是一个例子，对于同一个数据源里两张表card_info，pay_order的不同字段进行AES的加密

```java
private EncryptRuleConfiguration getEncryptRuleConfiguration() {
Properties props = new Properties();

//自带aes算法需要
props.setProperty("aes.key.value", aeskey);
EncryptorRuleConfiguration encryptorConfig = new EncryptorRuleConfiguration("AES", props);

//自定义算法
//props.setProperty("qb.finance.aes.key.value", aeskey);
//EncryptorRuleConfiguration encryptorConfig = new EncryptorRuleConfiguration("QB-FINANCE-AES", props);

EncryptRuleConfiguration encryptRuleConfig = new EncryptRuleConfiguration();
encryptRuleConfig.getEncryptors().put("aes", encryptorConfig);

//START: card_info 表的脱敏配置
{
    EncryptColumnRuleConfiguration columnConfig1 = new EncryptColumnRuleConfiguration("", "name", "", "aes");
    EncryptColumnRuleConfiguration columnConfig2 = new EncryptColumnRuleConfiguration("", "id_no", "", "aes");
    EncryptColumnRuleConfiguration columnConfig3 = new EncryptColumnRuleConfiguration("", "finshell_card_no", "", "aes");
    Map<String, EncryptColumnRuleConfiguration> columnConfigMaps = new HashMap<>();
    columnConfigMaps.put("name", columnConfig1);
    columnConfigMaps.put("id_no", columnConfig2);
    columnConfigMaps.put("finshell_card_no", columnConfig3);
    EncryptTableRuleConfiguration tableConfig = new EncryptTableRuleConfiguration(columnConfigMaps);
    encryptRuleConfig.getTables().put("card_info", tableConfig);
}

//END: card_info 表的脱敏配置

//START: pay_order 表的脱敏配置
{
    EncryptColumnRuleConfiguration columnConfig1 = new EncryptColumnRuleConfiguration("", "card_no", "", "aes");
    Map<String, EncryptColumnRuleConfiguration> columnConfigMaps = new HashMap<>();
    columnConfigMaps.put("card_no", columnConfig1);
    EncryptTableRuleConfiguration tableConfig = new EncryptTableRuleConfiguration(columnConfigMaps);
    encryptRuleConfig.getTables().put("pay_order", tableConfig);
}

log.info("脱敏配置构建完成:{} ", encryptRuleConfig);
return encryptRuleConfig;
}
```

说明：

**1、** 创建 EncryptColumnRuleConfiguration  的时候有四个参数，前两个参数分表叫plainColumn、cipherColumn，其意思是数据库存储里面真实的两个列（名文列、脱敏列），对于新的系统，只需要设置脱敏列即可，所以以上示例为plainColumn为”“。

**2、** 创建EncryptTableRuleConfiguration  的时候需要传入一个map，这个map存的value即#1中说明的EncryptColumnRuleConfiguration  ，而其key则是一个逻辑列，对于新系统，此逻辑列即为真实的脱敏列。Sharding  Shpere在拦截到SQL改写的时候，会按照用户的配置，把逻辑列映射为名文列或者脱敏列（默认）如下的示例

[![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZffWPDTNara23zX7cQiaVrP6prrN5GfV6UCPEBiazSMcP7A2RrTUU7nzNBYZP46JlpD4swAyjaGpnARg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247487551&idx=1&sn=18f64ba49f3f0f9d8be9d1fdef8857d9&scene=21#wechat_redirect)

### 3、使用Sharding Sphere的数据源进行管理

把原始的数据源包装一层

```java
@Bean("tradePlatformDataSource")
public DataSource dataSource(@Qualifier("druidDataSource") DataSource ds) throws SQLException {
    return EncryptDataSourceFactory.createDataSource(ds, getEncryptRuleConfiguration(), new Properties());

}
```

## 脱敏配置Quick Start——Spring Boot版

以下步骤使用Spring Boot管理，可以仅用配置文件解决：

**1、** 引入依赖

```xml
<!-- for spring boot -->

<dependency>
<groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>

<!-- for spring namespace -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-namespace</artifactId>
    <version>${sharding-sphere.version}</version>
</dependency>
```

**2、** Spring 配置文件

```yaml
spring.shardingsphere.datasource.name=ds
spring.shardingsphere.datasource.ds.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds.url=xxxxxxxxxxxxx
spring.shardingsphere.datasource.ds.username=xxxxxxx
spring.shardingsphere.datasource.ds.password=xxxxxxxxxxxx

# 默认的AES加密器
spring.shardingsphere.encrypt.encryptors.encryptor_aes.type=aes
spring.shardingsphere.encrypt.encryptors.encryptor_aes.props.aes.key.value=hkiqAXU6Ur5fixGHaO4Lb2V2ggausYwW

# card_info 姓名 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.name.cipherColumn=name
spring.shardingsphere.encrypt.tables.card_info.columns.name.encryptor=encryptor_aes

# card_info 身份证 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.id_no.cipherColumn=id_no
spring.shardingsphere.encrypt.tables.card_info.columns.id_no.encryptor=encryptor_aes

# card_info 银行卡号 AES加密
spring.shardingsphere.encrypt.tables.card_info.columns.finshell_card_no.cipherColumn=finshell_card_no
spring.shardingsphere.encrypt.tables.card_info.columns.finshell_card_no.encryptor=encryptor_aes

# pay_order 银行卡号 AES加密
spring.shardingsphere.encrypt.tables.pay_order.columns.card_no.cipherColumn=card_no
spring.shardingsphere.encrypt.tables.pay_order.columns.card_no.encryptor=encryptor_aes
```