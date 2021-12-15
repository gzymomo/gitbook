[TOC]

# 1、SpringBoot集成Nacos
项目中加入使用Nacos配置中心的依赖nacos-config-spring-boot-starter。
pom.xml
```xml
<dependency>
   <groupId>com.alibaba.boot</groupId>
   <artifactId>nacos-config-spring-boot-starter</artifactId>
   <version>0.2.1</version>
</dependency>
```

配置文件中需要配置Nacos服务的地址，如下所示。
```yml
spring.application.name=springboot2-nacos-config
nacos.config.server-addr=127.0.0.1:8848
```
在启动类，加入@NacosPropertySource注解其中包含两个属性，如下：

- dataId：这个属性是需要在Nacos中配置的Data Id。
- autoRefreshed：为true的话开启自动更新。

在使用Nacos做配置中心后，需要使用@NacosValue注解获取配置，使用方式与@Value一样，完整启动类代码如下所示。
```java
import com.alibaba.nacos.api.config.annotation.NacosValue;
import com.alibaba.nacos.spring.context.annotation.config.NacosPropertySource;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@NacosPropertySource(dataId = "springboot2-nacos-config", autoRefreshed = true)
@RestController
public class Springboot2NacosConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(Springboot2NacosConfigApplication.class, args);
    }

    @NacosValue(value = "${nacos.test.propertie:123}", autoRefreshed = true)
    private String testProperties;

    @GetMapping("/test")
    public String test(){
        return testProperties;
    }
}
```

使用Nacos修改配置：
点击右侧加号，添加我们刚刚创建的data id 的服务，并将配置由123修改为111，如图所示。
![](https://upload-images.jianshu.io/upload_images/9953332-f7575fe51adee7dd?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

然后点击右下角发布按钮，访问对应的接口进行测试。
![](https://upload-images.jianshu.io/upload_images/7253165-6127194cc3f5c3be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2、Nacos配置
## 2.1 命名空间
nacos使用namespace进行环境隔离，可以指定不同的环境，更好的管理开发、测试、生产的配置文件管理。
![](https://img-blog.csdnimg.cn/20190415134503979.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hhbmdiaW4xMjM=,size_16,color_FFFFFF,t_70)

## 2.2 资源配置
![](https://img-blog.csdnimg.cn/20190415134525906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hhbmdiaW4xMjM=,size_16,color_FFFFFF,t_70)

### 2.2.1 Data ID
Data ID的格式如下:`${prefix}-${spring.profile.active}.${nacos.config.file-extension}`

- prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
- spring.profile.active 即为当前环境对应的 profile。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}。
- nacos.config.file-extension的默认值为properties
- 当spring.profiles.active未配置时，则匹配${spring.application.name}.properties
- 若设置了spring.profiles.active而Nacos中存在${spring.application.name}.properties时，若还存在${spring.application.name}-${spring.profiles.active}.properties，则默认匹配后者，若不存在，则会自动匹配前者
- file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension来配置。目前只支持 properties 和 yaml 类型。
- 由于Nacos建议且默认用spring.application.name作为Data Id的前缀，若要在不同服务中共享项目统一配置，则可以通过配置nacos.config.shared-dataids或nacos.config.refreshable-dataids来添加共享配置，前者不支持自动刷新，后者支持

### 2.2.2 Group
默认为DEFAULT_GROUP,可以对不同类型的微服务配置文件进行分组管理。配置文件通过，可以用作多环境、多模块、多版本之间区分配置。
spring.cloud.nacos.config.group=AAA来指定。

### 2.2.3 Namespace
- 推荐使用命名空间来区分不同环境的配置，因为使用profiles或group会是不同环境的配置展示到一个页面，而Nacos控制台对不同的Namespace做了Tab栏分组展示，如下图：
![](https://upload-images.jianshu.io/upload_images/7253165-020f9781f3c5e0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 注意配置Namespace的时候不是通过名称，而是通过命名空间的ID(上图所示)，可通过如下配置来设置服务使用的命名空间：

示例：
```yml
nacos:
  service-address: 127.0.0.1
  port: 8848
  config:
    server-addr: ${nacos.service-address}:${nacos.port}
    namespace: 9af36d59-2efd-4f43-8a69-82fb37fc8094  # 命名空间ID 不是命名空间名称
```


### 2.2.4 配置内容
配置文件格式支持一下几种TEXT、JSON、XML、YAML、HTML、Properties

## 2.3 配置操作
### 2.3.1 历史版本
资源文件每次修改都会记录一个历史版本，历史记录默认保存时间为30天，可以根据历史记录看到每次更新的内容。还可以让指定的记录文件回滚至上一个版本。
![](https://img-blog.csdnimg.cn/20190415134543308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hhbmdiaW4xMjM=,size_16,color_FFFFFF,t_70)

### 2.3.2 监听查询
可以监听每个具体资源文件由哪些ip进行访问。
![](https://img-blog.csdnimg.cn/20190415134555471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW5nY2hhbmdiaW4xMjM=,size_16,color_FFFFFF,t_70)

# 3、扩展配置注意事项
## 3.1 客户端配置文件类型设置
在bootstrap.properties文件中
spring.cloud.nacos.config.file-extension=properties,yml,yaml
属性声明从配置中心中读取的配置文件格式
该配置的缺省值为properties，即默认是读取properties格式的配置文件。当客户端没有配置该属性，并且在nacos server添加的是yml格式的配置文件，则给客户端会读取不到配置文件，导致启动失败。
非properties配置格式，必须添加如下配置才可生效
`spring.cloud.nacos.config.file-extension=yml`

## 3.2 根据profile设置不同的环境配置
springboot中我们可以通过配置spring.profiles.active 实现在开发、测试、生产环境下采用不同的配置文件
同样，我们同科可以在nacos server分别创建
```yml
${application.name}-dev.properties
${application.name}-test.properties
${application.name}-prod.properties
```
然后通过命令启动jar时 设置spring.profiles.active来实现不同环境下使用不同的配置文件。
`java -jar nacos-client-0.0.1-SNAPSHOT.jar --spring.profiles.active=test`


## 3.3 自定义group
在同一个group下，配置文件名不能重复，所以当需要创建文件名称相同的两个配置文件时，将两个配置文件创建在不同的group下即可。当我们再同一个group下创建一个已有的配置文件时，nacos会将其视为配置文件的修改，而不是新建。
因此我们可以把group作为一个project名称，相当于pom中的artifactId来标示不同的工程，每个工程拥有不同的配置文件即可。
![](https://img-blog.csdnimg.cn/20190307183015665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pqY2phdmE=,size_16,color_FFFFFF,t_70)

如果创建了新的group那么客户端需要显式的配置group信息否则默认DEFAULT_GROUP空间中会出现找不到或者配置信息不符合你真实想法的情况。
```yml
#spring.cloud.nacos.config.file-extension=yaml
spring.cloud.nacos.config.group=bamboo_group
```

## 3.4 自定义 namespace 命名空间
相应的如果是服务，我们一般是按照一个服务一个隔离空间的，比如公司有两个不同的业务项目都有amdin服务，那么为了避免不会发生冲突，服务配置中就使用命名空间作为隔离开来。
![](https://img-blog.csdnimg.cn/20190307183623202.png)

注：该配置必须放在 bootstrap.properties 文件中。此外 spring.cloud.nacos.config.namespace的值是 namespace 对应的 id，id 值可以在 Nacos 的控制台获取。并且在添加配置时注意不要选择其他的 namespace，否则将会导致读取不到正确的配置。

## 3.5 服务中心使用mysql保存数据
1. 在mysql server 新建数据库：nocas（名字自己随意）
2. 在nacos server的 conf目录下找到nacos-mysql.sql 文件，并在创建的nacos数据库下执行表nacos-mysql.sql中的SQL语句
**nacos-mysql.sql**

```sql

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(20) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(512) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

![](https://www.showdoc.cc/server/api/common/visitfile/sign/de5df98dc3371ab7bfce64efd09056f8?showdoc=.jpg)


![](https://www.showdoc.cc/server/api/common/visitfile/sign/d00cb0fe7f3f0b5d9f9dbdf0c5559ff8?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/8870723a50ae075215c13ca929ec0bc7?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/10dd8f9528881e13736cae5f2499d9e3?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/common/visitfile/sign/e79af593ceafc216a1b83860f65dbb9c?showdoc=.jpg)

3. 修改nacos server application.properties配置文件，修改后如下图所示
```yml
# spring
server.contextPath=/nacos
server.servlet.contextPath=/nacos
server.port=8848
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://数据库IP:端口号/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=数据库用户名
db.password=数据库密码
```


4. 重启Nacos server并添加配置文件，就可以看到mysql数据库数据表中出现了自己的配置文件内容

# 4、项目示例
NacosConfig.java代码实现：
```java
import com.alibaba.nacos.api.config.annotation.NacosValue;
import com.alibaba.nacos.spring.context.annotation.config.NacosPropertySource;
import lombok.Data;
import org.springframework.stereotype.Component;

/**
 * 从Nacos外部拉取配置, 修改配置，自动会刷新应用的配置
 *
 * @author raysonfang
 */
@NacosPropertySource(dataId = "rayson", autoRefreshed = true)
@Data
@Component
public class NacosConfig {

    @NacosValue(value = "${service.name:1}", autoRefreshed = true)
    private String serviceName;
}
```
注解说明：
@NacosPropertySource注解其中包含两个属性，如下：

- dataId：这个属性是需要在Nacos中配置的Data Id。
- autoRefreshed：为true的话开启自动更新。

在使用Nacos做配置中心后，需要使用@NacosValue注解获取配置，使用方式与@Value一样。
其中${service.name:1}的service.name是属性key, 1是默认值。

**针对@NacosValue的value必须赋默认值的情况下，解决方法如下：**
1. @NacosValue(value = "${service.name:}")   ----在配置项的表达式后面加一个冒号,当配置项没有该项的时候，就会采用默认值空而不是抛出错误。
2. [@Value() 设置默认值后，配置值无法生效的一个解决方法。](https://blog.csdn.net/hx765287443/article/details/84062637)