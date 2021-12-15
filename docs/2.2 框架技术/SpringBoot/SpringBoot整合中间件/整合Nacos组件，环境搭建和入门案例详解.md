[TOC]

# 1、Nacos基础简介
Nacos 是构建以“服务”为中心的现代应用架构，如微服务范式、云原生范式等服务基础设施。聚焦于发现、配置和管理微服务。Nacos提供一组简单易用的特性集，帮助开发者快速实现动态服务发现、服务配置、服务元数据及流量管理。敏捷构建、交付和管理微服务平台。

## 1.1 关键特性
- 动态配置服务
- 服务发现和服务健康监测
- 动态 DNS 服务
- 服务及其元数据管理

## 1.2 专业术语解释
- 命名空间
用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。

- 配置集
一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。

- 配置集 ID
Nacos 中的某个配置集的ID。配置集ID是组织划分配置的维度之一。DataID通常用于组织划分系统的配置集。

- 配置分组
Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串对配置集进行（Group）分组，从而区分 Data ID 相同的配置集。

- 配置快照
Nacos 的客户端 SDK 会在本地生成配置的快照。当客户端无法连接到 Nacos Server 时，可以使用配置快照显示系统的整体容灾能力。

- 服务注册
存储服务实例和服务负载均衡策略的数据库。

- 服务发现
使用服务名对服务下的实例的地址和元数据进行探测，并以预先定义的接口提供给客户端进行查询。

- 元数据
Nacos数据（如配置和服务）描述信息，如服务版本、权重、容灾策略、负载均衡策略等。

## 1.3 Nacos生态圈
> Nacos 无缝支持一些主流的开源框架生态：

- Spring Cloud 微服务框架 ;
- Dubbo RPC框架 ;
- Kubernetes 容器应用 ;

# 2、SpringBoot整合Nacos
## 2.1 新建配置
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvCm2iaNtiat9WnmB53Jb9obrTtNPicq3lDCF1anbX00dzzSCzvtsMwibk4CSwFCEmqBZcu6QGUxupb0GQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.2 核心依赖
```xml
<!-- Nacos 组件依赖 -->
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-discovery-spring-boot-starter</artifactId>
    <version>0.2.3</version>
</dependency>
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>0.2.3</version>
</dependency>
```
## 2.3 Yml配置文件
这里把项目作为服务注册到Nacos中。
```yml
nacos:
  config:
    server-addr: 127.0.0.1:8848
  discovery:
    server-addr: 127.0.0.1:8848
```
## 2.4 启动类配置
启动类关联配置中心的dataId标识。
```java
@EnableSwagger2
@SpringBootApplication
@NacosPropertySource(dataId = "WARE_ID", autoRefreshed = true)
public class Application7017 {
    public static void main(String[] args) {
        SpringApplication.run(Application7017.class,args) ;
    }
}
```
## 2.5 核心配置类
```java
import com.alibaba.nacos.api.annotation.NacosInjected;
import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingService;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import javax.annotation.PostConstruct;
@Configuration
public class NacosConfig {
    @Value("${server.port}")
    private int serverPort;
    @Value("${spring.application.name}")
    private String applicationName;
    @NacosInjected
    private NamingService namingService;
    @PostConstruct
    public void registerInstance() throws NacosException {
        namingService.registerInstance(applicationName, "127.0.0.1", serverPort);
    }
}
```
启动成功后查询服务列表：
![](https://mmbiz.qpic.cn/mmbiz_png/uUIibyNXbAvCm2iaNtiat9WnmB53Jb9obrTsAmIsice5StibUulyqOaTXmVAVKcJ6wwNKWzU3VoQtVg5ym7NfcToFRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.6 基础API用例
```java
@Api("Nacos接口管理")
@RestController
@RequestMapping("/nacos")
public class NacosController {

    @NacosValue(value = "${MyName:null}", autoRefreshed = true)
    private String myName;
    @NacosValue(value = "${project:null}", autoRefreshed = true)
    private String project;

    @ApiOperation(value="查询配置信息")
    @GetMapping(value = "/info")
    public String info () {
        return myName+":"+project;
    }

    @NacosInjected
    private NamingService namingService;

    @ApiOperation(value="查询服务列表")
    @GetMapping(value = "/getServerList")
    public List<Instance> getServerList (@RequestParam String serviceName) {
        try {
            return namingService.getAllInstances(serviceName) ;
        } catch (Exception e){
            e.printStackTrace();
        }
        return null ;
    }
}
```