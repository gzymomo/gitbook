[Spring Boot - 多模块多环境配置](https://segmentfault.com/a/1190000037660396)

### 多模块

#### 模块

在[模块化编程](https://en.wikipedia.org/wiki/Modular_programming)中，开发者将程序分解成离散功能块(discrete chunks of functionality)，并称之为模块。

#### 多模块的优点

每个模块具高内聚的特性，使得校验、调试、测试轻而易举。 精心编写的模块提供了可靠的抽象和封装界限，使得每个模块都具有条理清楚的设计和明确的目的。

#### 实现多模块

1. 创建maven工程
2. 配置多模块
3. 添加模块依赖

##### 创建maven工程

![image](https://segmentfault.com/img/bVcIbeQ)
![image](https://segmentfault.com/img/bVcIbeR)
![image](https://segmentfault.com/img/bVcIbeT)

##### 配置多模块

在pom中，增加modules节点，模块名<module>任意名称</module>，可以配置多个；

```xml
<modules>
  <module>seckill-api</module>
  <module>seckill-biz</module>
</modules>
```

![image](https://segmentfault.com/img/bVcIbfh)

通常到这里，多模块就配置完毕了。但现实中，我们的模块间是需要相互依赖的，同时每个模块还要依赖第三方模块；

##### 添加模块依赖

seckill-api（api层）要依赖seckill-biz（业务层），在api模块的pom文件中，增加如下配置

```xml
<dependencies>
 <dependency> 
     <groupId>com.sifou.courses</groupId>
     <artifactId>seckill-biz</artifactId>
     <version>1.0-SNAPSHOT</version>
 </dependency>
</dependencies>
```

假定，api和biz模块都依赖lombok，validation-api这两个第三方模块（包），如何实现？

- 方案1：在每个（biz & api）模块中，增加依赖；
- 方案2：在父模块增加依赖；

相信大家都会选择方案2；在root工程中的pom文件，增加如下配置；

```xml
<properties>
    <lombok.version>1.18.8</lombok.version>
    <javax.validation>2.0.1.Final</javax.validation>
</properties>

<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
    </dependency>
    <dependency>
        <groupId>javax.validation</groupId>
        <artifactId>validation-api</artifactId>
        <version>${javax.validation}</version>
    </dependency>
</dependencies>
```

到这里，配置完成；（是不是很清晰，请投币，点赞）
![image](https://segmentfault.com/img/bVcIbhr)
还可以用mvn dependency:tree命令，来查看依赖关系（必备核心技能，解决包冲突，解决包版本失效）

```java
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] com.sifou.courses.seckill
[INFO] seckill-biz
[INFO] seckill-api
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building com.sifou.courses.seckill 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:3.1.2:tree (default-cli) @ com.sifou.courses.seckill ---
[INFO] com.sifou.courses:com.sifou.courses.seckill:pom:1.0-SNAPSHOT
[INFO] +- org.projectlombok:lombok:jar:1.18.8:compile
[INFO] \- javax.validation:validation-api:jar:2.0.1.Final:compile
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building seckill-biz 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:3.1.2:tree (default-cli) @ seckill-biz ---
[INFO] com.sifou.courses:seckill-biz:jar:1.0-SNAPSHOT
[INFO] +- org.projectlombok:lombok:jar:1.18.8:compile
[INFO] \- javax.validation:validation-api:jar:2.0.1.Final:compile
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building seckill-api 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:3.1.2:tree (default-cli) @ seckill-api ---
[INFO] com.sifou.courses:seckill-api:jar:1.0-SNAPSHOT
[INFO] +- com.sifou.courses:seckill-biz:jar:1.0-SNAPSHOT:compile
[INFO] +- org.projectlombok:lombok:jar:1.18.8:compile
[INFO] \- javax.validation:validation-api:jar:2.0.1.Final:compile
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] com.sifou.courses.seckill .......................... SUCCESS [  1.007 s]
[INFO] seckill-biz ........................................ SUCCESS [  0.040 s]
[INFO] seckill-api ........................................ SUCCESS [  0.036 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.862 s
[INFO] Finished at: 2020-10-29T23:18:09+08:00
[INFO] Final Memory: 27M/230M
[INFO] ------------------------------------------------------------------------

Process finished with exit code 0
```

### 多环境

在工作中，我们面临开发、测试、生产等等多个环境，要完美实现多环境，总共可以分文两个大的步骤；

- 在工程中支持多环境配置；
- 在真实环境中实现多环境启动；

#### 支持多环境配置

1. 创建properties文件
2. 指定环境参数

##### 创建properties文件

在resources文件夹下创建三个以properties为后缀的文件
例如：
application-dev.properties：开发环境
application-test.properties：测试环境
application-prod.properties：生产环境

##### 指定环境参数

**spring.profiles.active=test**

到这里，多环境配置完成；
在Spring Boot中多环境配置文件名必须满足：**application-{profile}.properties**的固定格式，其中**{profile}**对应你的环境标识；

> 例如：
> application-**dev**.properties：开发环境
> application-**test**.properties：测试环境
> application-**prod**.properties：生产环境

application.properyies通过spring.profiles.active来具体激活一个或者多个配置文件，如果没有指定任何profile的配置文件的话，spring boot默认会启动application-default.properties；而哪个配置文件运行：

spring.profiles.active=test

就会加载application-test.properties配置文件内容

#### 多环境启动

刚刚讲了在工程中如何配置，那么在真正的环境中如何启动？莫非，改配置吗？？？当然不是，正解如下。

```java
-Dspring.profiles.active=${PROFILE}
```

在启动脚本中，增加上面这个，按环境来指定要加载的配置文件；