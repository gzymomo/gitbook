## knife4j简介

knife4j是springfox-swagger的增强UI实现，为Java开发者在使用Swagger的时候，提供了简洁、强大的接口文档体验。knife4j完全遵循了springfox-swagger中的使用方式，并在此基础上做了增强功能，如果你用过Swagger，你就可以无缝切换到knife4j。



- 在pom.xml中增加knife4j的相关依赖；



```xml
<!--整合Knife4j-->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.4</version>
</dependency>
```

- 在Swagger2Config中增加一个@EnableKnife4j注解，该注解可以开启knife4j的增强功能；



```java
/**
 * Swagger2API文档的配置
 */
@Configuration
@EnableSwagger2
@EnableKnife4j
public class Swagger2Config {
    
}
```

- 运行我们的SpringBoot应用，访问API文档地址即可查看：[http://localhost:8088/doc.html](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8088%2Fdoc.html)

![img](https:////upload-images.jianshu.io/upload_images/1939592-d49df2f95aad7587?imageMogr2/auto-orient/strip|imageView2/2/w/1158/format/webp)

image

## 功能特点

> 接下来我们对比下Swagger，看看使用knife4j和它有啥不同之处！

### JSON功能增强

> 平时一直使用Swagger，但是Swagger的JSON支持一直不是很好，JSON不能折叠，太长没法看，传JSON格式参数时，没有参数校验功能。这些痛点，在knife4j上都得到了解决。

- 返回结果集支持折叠，方便查看；

![img](https:////upload-images.jianshu.io/upload_images/1939592-281d44b3733cbdfc?imageMogr2/auto-orient/strip|imageView2/2/w/729/format/webp)

image

- 请求参数有JSON校验功能。

![img](https:////upload-images.jianshu.io/upload_images/1939592-49b1b90ada19a1a3?imageMogr2/auto-orient/strip|imageView2/2/w/645/format/webp)

image

### 登录认证

> knife4j也支持在头部添加Token，用于登录认证使用。

- 首先在`Authorize`功能中添加登录返回的Token；

![img](https:////upload-images.jianshu.io/upload_images/1939592-672932bd8ea4ddb3?imageMogr2/auto-orient/strip|imageView2/2/w/1164/format/webp)

image

- 之后在每个接口中就可以看到已经在请求头中携带了Token信息。

![img](https:////upload-images.jianshu.io/upload_images/1939592-872d566aee12d4d2?imageMogr2/auto-orient/strip|imageView2/2/w/974/format/webp)

image

### 离线文档

> knife4j支持导出离线文档，方便发送给别人，支持Markdown格式。

- 直接选择`文档管理->离线文档`功能，然后选择`下载Markdown`即可；

![img](https:////upload-images.jianshu.io/upload_images/1939592-7455ae9709ad6b3d?imageMogr2/auto-orient/strip|imageView2/2/w/1158/format/webp)

image

- 我们来查看下导出的Markdown离线文档，还是很详细的。

![img](https:////upload-images.jianshu.io/upload_images/1939592-5428c62b569ffb8b?imageMogr2/auto-orient/strip|imageView2/2/w/1088/format/webp)

image

### 全局参数

> knife4j支持临时设置全局参数，支持两种类型query(表单)、header(请求头)。

- 比如我们想要在所有请求头中加入一个参数appType来区分是android还是ios调用，可以在全局参数中添加；

![img](https:////upload-images.jianshu.io/upload_images/1939592-be48d9802ef4877e?imageMogr2/auto-orient/strip|imageView2/2/w/1157/format/webp)

image

- 此时再调用接口时，就会包含`appType`这个请求头了。

![img](https:////upload-images.jianshu.io/upload_images/1939592-a5e59e2309ee1833?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

### 忽略参数属性

> 有时候我们创建和修改的接口会使用同一个对象作为请求参数，但是我们创建的时候并不需要id，而修改的时候会需要id，此时我们可以忽略id这个属性。

- 比如这里的创建商品接口，id、商品数量、商品评论数量都可以让后台接口生成无需传递，可以使用knife4j提供的`@ApiOperationSupport`注解来忽略这些属性；



```java
/**
 * 品牌管理Controller
 * Created by macro on 2019/4/19.
 */
@Api(tags = "PmsBrandController", description = "商品品牌管理")
@Controller
@RequestMapping("/brand")
public class PmsBrandController {
    @Autowired
    private PmsBrandService brandService;

    private static final Logger LOGGER = LoggerFactory.getLogger(PmsBrandController.class);

    @ApiOperation("添加品牌")
    @ApiOperationSupport(ignoreParameters = {"id","productCount","productCommentCount"})
    @RequestMapping(value = "/create", method = RequestMethod.POST)
    @ResponseBody
    public CommonResult createBrand(@RequestBody PmsBrand pmsBrand) {
        CommonResult commonResult;
        int count = brandService.createBrand(pmsBrand);
        if (count == 1) {
            commonResult = CommonResult.success(pmsBrand);
            LOGGER.debug("createBrand success:{}", pmsBrand);
        } else {
            commonResult = CommonResult.failed("操作失败");
            LOGGER.debug("createBrand failed:{}", pmsBrand);
        }
        return commonResult;
    }
}
```

- 此时查看接口文档时，发现这三个属性已经消失，这样前端开发查看接口文档时就不会觉得你定义无用参数了，是不很很好的功能！

![img](https:////upload-images.jianshu.io/upload_images/1939592-60d80244d9f5d1d0?imageMogr2/auto-orient/strip|imageView2/2/w/1017/format/webp)

image

## 参考资料

官方文档：[https://doc.xiaominfo.com/guide/](https://links.jianshu.com/go?to=https%3A%2F%2Fdoc.xiaominfo.com%2Fguide%2F)

## 项目源码地址

[https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-knife4j](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmacrozheng%2Fmall-learning%2Ftree%2Fmaster%2Fmall-tiny-knife4j)



作者：梦想de星空
链接：https://www.jianshu.com/p/02cea650699a
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。