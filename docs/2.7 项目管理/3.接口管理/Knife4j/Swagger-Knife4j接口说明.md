# API详细说明

注释汇总

| 作用范围           | API                | 使用位置                         |
| ------------------ | ------------------ | -------------------------------- |
| 对象属性           | @ApiModelProperty  | 用在出入参数对象的字段上         |
| 协议集描述         | @Api               | 用于controller类上               |
| 协议描述           | @ApiOperation      | 用在controller的方法上           |
| Response集         | @ApiResponses      | 用在controller的方法上           |
| Response           | @ApiResponse       | 用在 @ApiResponses里边           |
| 非对象参数集       | @ApiImplicitParams | 用在controller的方法上           |
| 非对象参数描述     | @ApiImplicitParam  | 用在@ApiImplicitParams的方法里边 |
| 描述返回对象的意义 | @ApiModel          | 用在返回对象类上                 |

# 一、接口添加作者

有时候在开发接口时,我们希望给该接口添加一个作者,这样前端或者别个团队来对接该接口时,如果该接口返回的数据或者调用有问题,都能准确找到该人,提升效率

添加作者需要使用`knife4j`提供的增强注解`@ApiOperationSupport`

接口代码示例如下：

```java
@ApiOperationSupport(author = "xiaoymin@foxmail.com")
@ApiOperation(value = "写文档注释我是认真的")
@GetMapping("/getRealDoc")
public Rest<RealDescription> getRealDoc(){
    Rest<RealDescription> r=new Rest<>();
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    r.setData(new RealDescription());
    return r;
}
```

在文档中显示效果如下：

![img](https://xiaoym.gitee.io/knife4j/images/2-0-2/debug-3.png)

在2.0.3版本中,收到开发者反馈希望能在Controller上增加作者的注解

所代表的意思是该Controller模块下所有的接口都是该作者负责开发,当然用@ApiOperationSupport的注解也能覆盖

因此,在2.0.3版本中新增加了`@ApiSupport`注解,该注解目前有两个属性,分别是author(作者)和order(排序)

使用代码示例：

```java
@Api(tags = "2.0.3版本-20200312")
@ApiSupport(author = "xiaoymin@foxmail.com",order = 284)
@RestController
@RequestMapping("/api/nxew203")
public class Api203Constroller {
    
    
}
```

# 二、自定义文档

开发者可以在当前项目中添加多个个文件夹，文件夹中存放`.md`格式的markdown文件,每个`.md`文档代表一份自定义文档说明

**注意**：自定义文档说明必须以`.md`结尾的文件,其他格式文件会被忽略

例如项目结构如下：

![img](https://xiaoym.gitee.io/knife4j/images/1-9-3/construct.png)

每个`.md`文件中，`Knife4j`允许一级(h1)、二级(h2)、三级(h3)标题作为最终的文档标题

比如`api.md`文档：

```markdown
# 自定义文档说明

## 效果说明

`knife4j`为了满足文档的个性化配置,添加了自定义文档功能

开发者可自定义`md`文件扩展补充整个系统的文档说明

开发者可以在当前项目中添加一个文件夹，文件夹中存放`.md`格式的markdown文件,每个`.md`文档代表一份自定义文档说明

**注意**：自定义文档说明必须以`.md`结尾的文件,其他格式文件会被忽略
```

最终在`Knife4j`的界面中,`api.md`的文档标题会是`自定义文档说明`

整个文档效果如下：

![img](https://xiaoym.gitee.io/knife4j/images/knife4j/self-doc1.png)

如果没有按照一级(h1)、二级(h2)、三级(h3)来设置标题,默认标题会是文件名称，如图上的`api2.md`

**如何使用**

在Spring Boot环境中,首先需要在`application.yml`或者`application.properties`配置文件中配置自定义文档目录,支持多级目录

如下：

```yml
knife4j:
  enable: true
  documents:
    -
      group: 1.2.x
      name: 测试自定义标题分组
      # 某一个文件夹下所有的.md文件
      locations: classpath:markdown/*
    -
      group: 1.2.x
      name: 接口签名
      # 某一个文件夹下单个.md文件
      locations: classpath:markdown/sign.md
```

在配置`knife4j.documents`中，该属性是一个集合数组,代表开发者可以添加多个自定义文档分组,因为我们在最终呈现接口文档时，会存在逻辑分组的情况，有时候我们希望不同的逻辑分组下显示不同的逻辑分组文档，所以需要通过该节点下的group(分组名称)进行区分

相关属性说明如下：

| 属性名称 | 是否必须 | 说明                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| group    | true     | 逻辑分组名称,最终在逻辑分组时该属性需要传入                  |
| name     | true     | 自定义文档的分组名称，可以理解为开发者存在多个自定义文档，最终在Ui界面呈现时的一个分组名称 |
| location | true     | 提供自定义`.md`文件的路径或者文件                            |

开发者配置好后,最核心的一步，也是最后最重要的一步，开发者需要在创建`Docket`逻辑分组对象时，通过`Knife4j`提供的工具对象`OpenApiExtensionResolver`将扩展属性进行赋值

示例代码如下：

```java
@Configuration
@EnableSwagger2WebMvc
public class SwaggerConfiguration {

   private final OpenApiExtensionResolver openApiExtensionResolver;

    @Autowired
    public SwaggerConfiguration(OpenApiExtensionResolver openApiExtensionResolver) {
        this.openApiExtensionResolver = openApiExtensionResolver;
    }

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        String groupName="2.X版本";
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .host("https://www.baidu.com")
                .apiInfo(apiInfo())
                .groupName(groupName)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.swagger.bootstrap.ui.demo.new2"))
                .paths(PathSelectors.any())
                .build()
                .extensions(openApiExtensionResolver.buildExtensions(groupName));
        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //.title("swagger-bootstrap-ui-demo RESTful APIs")
                .description("# swagger-bootstrap-ui-demo RESTful APIs")
                .termsOfServiceUrl("http://www.xx.com/")
                .contact("xx@qq.com")
                .version("1.0")
                .build();
    }
}
```

通过上面示例代码，主要步骤如下：

1、通过`@Autowired`注解引入`Knife4j`向Spring容器注入的Bean对象`OpenApiExtensionResolver`

2、最终在`Dcoket`对象构建后，通过调用`Docket`对象的`extensions`方法进行插件赋值

3、插件赋值需要调用`OpenApiExtensionResolver`提供的`buildExtensions`方法，该方法需要一个逻辑分组名称，就是开发者在`yml`配置文件中配置的`group`名称

# 三、访问权限控制

## 3.1 访问页面加权控制

不管是官方的`swagger-ui.html`或者`doc.html`,目前接口访问都是无需权限即可访问接口文档的,很多朋友以前问我能不能提供一个登陆界面的功能,开发者输入用户名和密码来控制界面的访问,只有知道用户名和密码的人才能访问此文档

做登录页控制需要有用户的概念,所以相当长一段时间都没有提供此功能

针对Swagger的资源接口,`Knife4j`提供了简单的**Basic认证功能**

效果图如下：

![img](https://xiaoym.gitee.io/knife4j/images/ac-pwd.png)

允许开发者在配置文件中配置一个静态的用户名和密码,当对接者访问Swagger页面时,输入此配置的用户名和密码后才能访问Swagger文档页面,如果您使用SpringBoot开发,则只需在相应的`application.properties`或者`application.yml`中配置如下：

```yml
knife4j:
  # 开启增强配置 
  enable: true
　# 开启Swagger的Basic认证功能,默认是false
  basic:
      enable: true
      # Basic认证用户名
      username: test
      # Basic认证密码
      password: 123
```

如果用户开启了basic认证功能,但是并未配置用户名及密码,`Knife4j`提供了默认的用户名和密码：

```text
admin/123321
```

# 四、接口排序

针对Controller下的具体接口,排序规则是使用`Knife4j`提供的增强注解`@ApiOperationSupport`中的order字段,代码示例如下：

```java
@ApiOperationSupport(order = 33)
@ApiOperation(value = "忽略参数值-Form类型")
@PostMapping("/ex")
public Rest<LongUser> findAll(LongUser longUser) {
    Rest<LongUser> r=new Rest<>();
    r.setData(longUser);
    return r;
}
```

Knife4j通过Spring Plugin插件体系,对每个接口进行扫描,最终将扫描的`@ApiOperationSupport`注解获取的`order`值通过OpenAPI的扩展属性规范进行赋值

最终在OpenAPI的规范中，接口的path节点下,通过`x-order`属性得到接口的排序，最终前端根据排序值进行排序(顺序)，如下图：

![img](https://xiaoym.gitee.io/knife4j/images/documentation/apiorder.png)

开发者如果遇到排序不生效的问题，可以通过检查接口返回的OpenAPI规范中，接口`path`节点下是否包含`x-order`的扩展属性

# 五、分组排序

Controller之间的排序主要有两种方式,排序的规则是倒序,但是排序的最小值必须大于0

建议优先级是：`@ApiSupport`>`@ApiSort`>`@Api`

> 对于最高级别的值,可以从999开始
>
> @ApiSupport注解自2.0.3版本引入

第一种,使用`@ApiSupport`注解中的属性`order`,代码示例如下：

```java
@Api(tags = "2.0.3版本-20200312")
@ApiSupport(order = 284)
@RestController
@RequestMapping("/api/nxew203")
public class Api203Constroller {
    
    
}
```

第二种情况,使用`knife4j`提供的增强注解`@ApiSort`,代码示例如下：

```java
@Api(tags = "2.0.2版本-20200226")
@ApiSort(286)
@RestController
@RequestMapping("/api/nxew202")
public class Api202Controller {
    
    
}
```

第三种,使用注解`@Api`中的属性`position`(需要注意的是该属性以及过时,建议开发者使用第一种),代码示例如下：

```java
@Api(tags = "2.0.2版本-20200226",position = 286)
@RestController
@RequestMapping("/api/nxew202")
public class Api202Controller {
    
    
}
```

很[接口排序](https://xiaoym.gitee.io/knife4j/documentation/apiSort.html)规则一样,Knife4j也是通过Spring Plugin插件化的方式，扫描接口注解，最终通过扩展OpenAPI的扩展属性`x-order`进行赋值，最终在Ui界面中解析，然后再进行排序后渲染组件。

# 六、动态请求参数

在某些特定的情况下,因为我们对于接口使用的是一种先定义,后展示的规范执行,所以我们在界面上看到的请求参数,全部来源于我们后端接口层如何定义

但是如果后端定义的是一种Map结构,或者是参数并没有定义声明,而希望也能达到一种动态添加参数进行调试的结果,这种体验有点类似于`postman`

`Knife4j`针对上面的需求提供了支持

在`Knife4j`的前端页面中,个性化设置功能里,可以开启对参数的动态调试(`该选项默认是关闭状态`)，如下图：

![img](https://xiaoym.gitee.io/knife4j/images/knife4j/plus/debugDynamic.png)

当在配置中勾选该选项后,我们的接口栏会有变化,如下图：

![img](https://xiaoym.gitee.io/knife4j/images/knife4j/plus/debugDynamic1.png)

在原本已存在的参数栏下会出现一栏空的参数栏,开发者可以输入参数名称、参数值对参数进行添加

不管是参数名称的变化还是参数值的变化,变化后会自动追加一行新的调试栏参数,效果图如下：

![img](https://xiaoym.gitee.io/images/knife4j/plus/dynamicparam3.gif)

# 七、自定义Swagger Models名称

**如何使用**

在Spring Boot环境中,首先需要在`application.yml`或者`application.properties`配置文件中配置自定义名称

如下：

```yml
knife4j:
  enable: true
  setting:
    enableSwaggerModels: true
    swaggerModelName: 我是自定义的Model名称
```

开发者配置好后,最核心的一步，也是最后最重要的一步，开发者需要在创建`Docket`逻辑分组对象时，通过`Knife4j`提供的工具对象`OpenApiExtensionResolver`将扩展属性进行赋值

示例代码如下：

```java
@Configuration
@EnableSwagger2WebMvc
public class SwaggerConfiguration {

   private final OpenApiExtensionResolver openApiExtensionResolver;

    @Autowired
    public SwaggerConfiguration(OpenApiExtensionResolver openApiExtensionResolver) {
        this.openApiExtensionResolver = openApiExtensionResolver;
    }

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        String groupName="2.X版本";
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .host("https://www.baidu.com")
                .apiInfo(apiInfo())
                .groupName(groupName)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.swagger.bootstrap.ui.demo.new2"))
                .paths(PathSelectors.any())
                .build()
                .extensions(openApiExtensionResolver.buildExtensions(groupName));
        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                //.title("swagger-bootstrap-ui-demo RESTful APIs")
                .description("# swagger-bootstrap-ui-demo RESTful APIs")
                .termsOfServiceUrl("http://www.xx.com/")
                .contact("xx@qq.com")
                .version("1.0")
                .build();
    }
}
```

通过上面示例代码，主要步骤如下：

1、通过`@Autowired`注解引入`Knife4j`向Spring容器注入的Bean对象`OpenApiExtensionResolver`

2、最终在`Dcoket`对象构建后，通过调用`Docket`对象的`extensions`方法进行插件赋值

3、插件赋值需要调用`OpenApiExtensionResolver`提供的`buildExtensions`方法，该方法需要一个逻辑分组名称，就是开发者在`yml`配置文件中配置的`group`名称。

# 八、@ApiModel和@ApiModelProperty

- [@ApiModel和@ApiModelProperty用法](https://www.cnblogs.com/anycc/p/12910853.html)

## 8.1 @ApiModel

 使用场景：在实体类上边使用，标记类时swagger的解析类。
 概述：提供有关swagger模型的其它信息，类将在操作中用作类型时自动内省。
 用法：
 ![img](https://img2020.cnblogs.com/blog/397648/202005/397648-20200518152915502-1667156516.png)

## 8.2 @ApiModelProperty

 使用场景：使用在被 @ApiModel 注解的模型类的属性上。表示对model属性的说明或者数据操作更改 。
 概述：添加和操作模型属性的数据。
 用法：
 ![img](https://img2020.cnblogs.com/blog/397648/202005/397648-20200518152933653-887862377.png)
