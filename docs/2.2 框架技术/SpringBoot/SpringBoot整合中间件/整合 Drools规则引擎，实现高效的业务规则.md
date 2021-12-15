[TOC]

# 1、Drools引擎简介
Drools是一个基于java的规则引擎，开源的，可以将复杂多变的规则从硬编码中解放出来，以规则脚本的形式存放在文件中，使得规则的变更不需要修正代码重启机器就可以立即在线上环境生效。具有易于访问企业策略、易于调整以及易于管理的特点，作为开源业务规则引擎，符合业内标准，速度快、效率高。

## 1.1 规则语法
(1)、演示drl文件格式
```java
package droolRule ;
import org.slf4j.Logger
import org.slf4j.LoggerFactory ;
dialect  "java"
rule "paramcheck1"
    when 
    then
        final Logger LOGGER = LoggerFactory.getLogger("param-check-one 规则引擎") ;
        LOGGER.info("参数");
end
```
(2)、语法说明
```java
· 文件格式
可以 .drl、xml文件，也可以Java代码块硬编码;
· package
规则文件中，package是必须定义的，必须放在规则文件第一行;
· import
规则文件使用到的外部变量，可以是一个类，也可以是类中的可访问的静态方法;
· rule
定义一个规则。paramcheck1规则名。规则通常包含三个部分：属性、条件、结果;
```

# 2、SpringBoot整合Drools
## 2.1 项目结构
![](https://mmbiz.qpic.cn/mmbiz_jpg/uUIibyNXbAvCUOPWjoxWFPFuabZJchdCsaoS1fHuJIF2dOB5S9lt6DJE8NpEoiaUbuAiantJN2Z6V45u9WqjBwN3Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.2 核心依赖
```xml
<!--drools规则引擎-->
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-core</artifactId>
    <version>7.6.0.Final</version>
</dependency>
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-compiler</artifactId>
    <version>7.6.0.Final</version>
</dependency>
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-templates</artifactId>
    <version>7.6.0.Final</version>
</dependency>
<dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-api</artifactId>
    <version>7.6.0.Final</version>
</dependency>
<dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-spring</artifactId>
    <version>7.6.0.Final</version>
</dependency>
```
## 2.3 配置文件
```java
@Configuration
public class RuleEngineConfig {
    private static final Logger LOGGER = LoggerFactory.getLogger(RuleEngineConfig.class) ;
    private static final String RULES_PATH = "droolRule/";
    private final KieServices kieServices = KieServices.Factory.get();
    @Bean
    public KieFileSystem kieFileSystem() throws IOException {
        KieFileSystem kieFileSystem = kieServices.newKieFileSystem();
        ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
        Resource[] files = resourcePatternResolver.getResources("classpath*:" + RULES_PATH + "*.*");
        String path = null;
        for (Resource file : files) {
            path = RULES_PATH + file.getFilename();
            LOGGER.info("path="+path);
            kieFileSystem.write(ResourceFactory.newClassPathResource(path, "UTF-8"));
        }
        return kieFileSystem;
    }
    @Bean
    public KieContainer kieContainer() throws IOException {
        KieRepository kieRepository = kieServices.getRepository();
        kieRepository.addKieModule(kieRepository::getDefaultReleaseId);
        KieBuilder kieBuilder = kieServices.newKieBuilder(kieFileSystem());
        kieBuilder.buildAll();
        return kieServices.newKieContainer(kieRepository.getDefaultReleaseId());
    }
    @Bean
    public KieBase kieBase() throws IOException {
        return kieContainer().getKieBase();
    }
    @Bean
    public KieSession kieSession() throws IOException {
        return kieContainer().newKieSession();
    }
    @Bean
    public KModuleBeanFactoryPostProcessor kiePostProcessor() {
        return new KModuleBeanFactoryPostProcessor();
    }
}
```
这样环境整合就完成了。

# 3、演示案例
## 3.1 规则文件
规则一
```java
dialect  "java"
rule "paramcheck1"
    salience 99
    when queryParam : QueryParam(paramId != null && paramSign.equals("+"))
        resultParam : RuleResult()
    then
        final Logger LOGGER = LoggerFactory.getLogger("param-check-one 规则引擎") ;
        LOGGER.info("参数:getParamId="+queryParam.getParamId()+";getParamSign="+queryParam.getParamSign());
        RuleEngineServiceImpl ruleEngineService = new RuleEngineServiceImpl() ;
        ruleEngineService.executeAddRule(queryParam);
        resultParam.setPostCodeResult(true);
end
```
规则二
```kava
dialect  "java"
rule "paramcheck2"
    salience 88
    when queryParam : QueryParam(paramId != null && paramSign.equals("-"))
        resultParam : RuleResult()
    then
        final Logger LOGGER = LoggerFactory.getLogger("param-check-two 规则引擎") ;
        LOGGER.info("参数:getParamId="+queryParam.getParamId()+";getParamSign="+queryParam.getParamSign());
        RuleEngineServiceImpl ruleEngineService = new RuleEngineServiceImpl() ;
        ruleEngineService.executeRemoveRule(queryParam);
        resultParam.setPostCodeResult(true);
end
```
规则说明：
A、salience 的值越大，越优先执行；
B、规则流程：如果paramId不为null，参数标识是+号，执行添加规则，-号，执行移除规则操作。

## 3.2 规则执行代码
```java
@Service
public class RuleEngineServiceImpl implements RuleEngineService {
    private static final Logger LOGGER = LoggerFactory.getLogger(RuleEngineServiceImpl.class) ;
    @Override
    public void executeAddRule(QueryParam param) {
        LOGGER.info("参数数据:"+param.getParamId()+";"+param.getParamSign());
        ParamInfo paramInfo = new ParamInfo() ;
        paramInfo.setId(param.getParamId());
        paramInfo.setParamSign(param.getParamSign());
        paramInfo.setCreateTime(new Date());
        paramInfo.setUpdateTime(new Date());
        ParamInfoService paramInfoService = (ParamInfoService)SpringContextUtil.getBean("paramInfoService") ;
        paramInfoService.insertParam(paramInfo);
    }
    @Override
    public void executeRemoveRule(QueryParam param) {
        LOGGER.info("参数数据:"+param.getParamId()+";"+param.getParamSign());
        ParamInfoService paramInfoService = (ParamInfoService)SpringContextUtil.getBean("paramInfoService") ;
        ParamInfo paramInfo = paramInfoService.selectById(param.getParamId());
        if (paramInfo != null){
            paramInfoService.removeById(param.getParamId()) ;
        }
    }
}
```
## 3.3 规则调用接口
```java
@RestController
@RequestMapping("/rule")
public class RuleController {
    @Resource
    private KieSession kieSession;
    @Resource
    private RuleEngineService ruleEngineService ;
    @RequestMapping("/param")
    public void param (){
        QueryParam queryParam1 = new QueryParam() ;
        queryParam1.setParamId("1");
        queryParam1.setParamSign("+");
        QueryParam queryParam2 = new QueryParam() ;
        queryParam2.setParamId("2");
        queryParam2.setParamSign("-");
        // 入参
        kieSession.insert(queryParam1) ;
        kieSession.insert(queryParam2) ;
        kieSession.insert(this.ruleEngineService) ;
        // 返参
        RuleResult resultParam = new RuleResult() ;
        kieSession.insert(resultParam) ;
        kieSession.fireAllRules() ;
    }
}
```