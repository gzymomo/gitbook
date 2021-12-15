[SpringBoot整合JSP和模板引擎Freemarker/Thymeleaf](https://www.cnblogs.com/xsge/p/13915356.html)



　模板引擎的作用就是，我们来写一个页面模板， 比如有些值，是动态的，我们在通过写一些表达式，来动态的显示这些值，就像我们早前常用的JSP。模板引擎的作用其实都一样，只不过呢，不同模板引擎之间，他们可能这个语法有点不一样。Spring Boot支持FreeMarker、Groovy、Thymeleaf和Mustache四种模板解析引擎，官方推荐使用Thymeleaf。

　　对于模版引擎而言，`SpringBoot`默认存放模版文件的路径为`src/main/resources/templates`，当然也可以通过配置文件进行修改的。

　　当然了，使用JSP也是可以的，**但官方已经不建议使用`JSP`了**，本文也会讲解下`SpringBoot`下`JSP`的支持的，因为有很多老的项目还是使用`JSP模板的`居多。（不推荐使用JSP也是因为JSP需要编译转换，而其他模板引擎则无需编译转换）

# 1.SpringBoot整合Java Server Page

　　JSP（全称Java Server Page）,是web开发最早期的模板引擎产品，随着时代的更新，已渐渐老去，当然目前还未完全退出市场。SpringBoot微服务架构，所有项目都是以jar文件方式打包部署，嵌入式的Tomcat（简化版，不支持JSP），所以SpringBoot默认是不支持JSP的，那么如果想要整合JSP，就需要独立引入整合依赖，和基础配置。

 　`SpringBoot`默认存放模版文件的路径为`src/main/resources/templates，但由于SpringBoot默认是不支持JSP的，所以我们不能将JSP文件放在templates目录下。`

1. 引入JSP核心引擎

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 <!-- SpringBoot整合JSP：SpringBoot默认整合Tomcat，所以不需要指定JSP版本 -->
    2 <dependency>
    3     <groupId>org.apache.tomcat.embed</groupId>
    4     <artifactId>tomcat-embed-jasper</artifactId>
    5 </dependency>
    6 <!-- JSP依赖JSTL -->
    7 <dependency>
    8     <groupId>javax.servlet</groupId>
    9     <artifactId>jstl</artifactId>
   10 </dependency>
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

2. 创建JSP整合目录webapp以及WEB-INF

   在Java，resources同级目录下新建webapp目录及子目录WEB-INF

   

    

    至于web.xml配置文件，是可要可不要了。

3. 修改yml文件，配置MVC试图解析器

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
   1 spring:
   2   # 配置整合JSP
   3   mvc:
   4     view:
   5       # 配置视图解析器前缀
   6       prefix: /WEB-INF/
   7       # 配置视图解析器后缀
   8       suffix: .jsp
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

4. 创建Controller跳转访问JSP

   ```
   因为是返回页面 所以不能是@RestController
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
   // @RestController  该注解会将任何返回值都作为JSON暴露，不经过视图解析器　　
   @Controller    // SpringBoot整合JSP，不能使用@RestController
   public class PersonController {
       
       @RequestMapping({"/indexJSP","/listJSP"})
       public String getIndexJSP() {
           System.out.println("JSP访问测试。。。。。");
           return "index";
       }
   }
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

5. 启动SpringBoot主程序，访问测试
   [http://localhost:8080/indexJSP
   ](http://localhost:8080/indexJSP)

> 1.说明：在完成上面的步骤之后，运行项目就可以访问jsp界面，但是别高兴的太早，当你打成一个jar包的时候，你会发现你不能访问jsp了，原因是SpringBoot打包时，是不会主动打包webapp目录的，你需要在你的pom文件手动配置把webapp的文件打包进去。详情参考附录。
> 2.常见错误：没有手动重启服务器！导致无法解析，错误代码500，显示信息：template might not exist or might not be accessible by any of the configured Template Resolvers（模板可能不存在，或者任何配置的模板解析程序都无法访问）
> 3.如果只是整合JSP，不整合使用其他模板，那么请不要引入其他模板依赖，也无需配置其他模板信息，否则会错错错！！！（SpringBoot默认支持的模板引擎，自然也有对应的默认配置，一旦你添加了对应的依赖，SpringBoot将遵循约定大于配置，自动注入，那你的JSP也就无法访问了）。

# 2.SpringBoot整合Freemarker

　　FreeMarker 是一款*模板引擎*： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。[官网参考](http://freemarker.foofun.cn/)

 　在Spring Boot中使用FreeMarker 只需在pom中加入FreeMarker 的starter即可。

1. 修改pom文件引入FreeMarker 的starter

   ```
   1 <!-- SpringBoot整合Freemarker -->
   2 <dependency>
   3     <groupId>org.springframework.boot</groupId>
   4     <artifactId>spring-boot-starter-freemarker</artifactId>
   5 </dependency>
   ```

2. 修改yml文件配置FreeMarker

   （可以不配，SpringBoot默认支持该模板）

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 spring:  
    2   # Freemarker配置详解（都含有默认配置）
    3   freemarker:
    4     # 模板加载路径（默认配置就是templates下，可以手动配置，多值使用“,”分割）
    5     template-loader-path:
    6     - classpath:/templates/
    7     # 配置缓存，开发阶段应该配置为false 因为经常会改,部署后建议开启
    8     cache: false
    9     # 配置编码设置
   10     charset: UTF-8
   11     # 建议模版是否存在
   12     check-template-location: true
   13     # Content-Type 值
   14     content-type: text/html
   15     # 模板后缀：默认为ftlh（这里测试更改为了html）,如果你设置了模板后缀，那么新建的模板文件必须为指定后缀
   16     # suffix: .html
   17     # 是否启用Freemarker
   18     enabled: true 
   19     # 详解配置可以查看FreeMarkerProperties类
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   > 配置说明：这里是向广大朋友概述/举例常见配置，并不是所有的都需要，SpringBoot含有默认配置，详情参考**`org.springframework.boot.autoconfigure.freemarker.FreeMarkerProperties`**类，上面的配置中，实际仅配置关闭缓存即可。

3. 新建FreeMarker模板

   　　在src/main/resources/templates/下新建模板引擎。

   

   　　FreeMarker模板默认后缀为ftlh，如果没有自定义配置，请遵循约定大于配置的初衷，新建模板且后缀设定为.ftlh。如果有自定义配置，则新建模板后缀为自定义的。例如：当前案例上面配置指定后缀为.html，那么新建的模板后缀就是.html，不必担心，会不会不支持FreeeMarker语法的问题。（模板引擎最终还是会转换为HTML的）

   Feemarker语法：

   参考官网

   ，

   参考中文网

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 <!DOCTYPE html>
    2 <html>
    3 <head>
    4 <meta charset="UTF-8">
    5 <title>Insert title here</title>
    6 </head>
    7 <body>
    8     Hello Freemarker！！！
    9     <#-- FreeMarker注释：获取数值（注意：这里的${}是FreeMarker语法，类似于EL，要清楚这不是EL表达式，且这里的值不是null，否则会报错） -->
   10     ${name }
   11 </body>
   12 </html>
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

4. 编写Controller控制器，实现跳转访问

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 @Controller // 因为是返回页面 所以不能是@RestController
    2 public class FreemarkerController {
    3     
    4     @RequestMapping("/indexFtlh")
    5     private String freemarkerShowIndex(String name, Model model) {
    6         // 将接受的参数通过model共享（实际保存在request中）
    7         model.addAttribute("name", name);
    8         return "indexFtlh";
    9     }
   10 }
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

5. 启动主程序测试

   访问：http://localhost:8080/indexFtlh?name=xsge

   

   > 常见错误：没有手动重启服务器！导致无法解析，错误代码500，显示信息：template might not exist or might not be accessible by any of the configured Template Resolvers（模板可能不存在，或者任何配置的模板解析程序都无法访问）

# 3.SpringBoot整合Thymeleaf

　　`Thymeleaf`是一个`XML/XHTML/HTML5`模板引擎，可用于Web与非Web环境中的应用开发。`Thymeleaf`的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。在Spring Boot中使用Thymeleaf只需在pom中加入Thymeleaf的starter即可。
　　Thymeleaf[官方网站](https://www.thymeleaf.org/)，[GitHup](https://github.com/thymeleaf)（Thymeleaf）

1. 修改pom，引入Thymeleaf的starter依赖

   ```
   1 <!-- SpringBoot整合Thymeleaf -->
   2 <dependency>
   3     <groupId>org.springframework.boot</groupId>
   4     <artifactId>spring-boot-starter-thymeleaf</artifactId>
   5 </dependency>
   ```

   默认的Thymeleaf版本为2.x.x.RELEASE版本，这里推荐使用3.0以上版本。建议在pom中将Thymeleaf的版本修改为3.0.x.RELEASE以上。（网上很多人说需要改，当然我没有改过，如果你想要更改高版本，可以参考——个人觉得没必要较真必须改，应当遵循你的项目情况而定，难道开发SpringBoot的人傻吗？）

   ```
   1 <properties>  
   2   <thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>  
   3   <thymeleaf-layout-dialect.version>2.0.4</thymeleaf-layout-dialect.version>  
   4 </properties>  
   ```

2. 修改yml文件配置基本信息

   （可以不配，SpringBoot默认推荐该模板）

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 Spring:
    2   #开启模板缓存（默认值：true）
    3   thymeleaf:
    4     cache: false 
    5     # 在呈现效果前检查模板是否存在.
    6     check-template: true 
    7     # 检查模板位置是否正确（默认值:true）
    8     check-template-location: true
    9     # Content-Type的值（默认值：text/html）
   10     servlet:
   11       content-type: text/html
   12     # 开启MVC Thymeleaf视图解析（默认值：true）
   13     enabled: true
   14     # 模板编码
   15     encoding: UTF-8
   16     # 要被排除在解析之外的视图名称列表
   17     # excluded-view-names: 
   18     # 要运用于模板之上的模板模式。另见StandardTemplate-ModeHandlers(默认值：HTML5)
   19     mode: HTML5
   20     # 在构建URL时添加到视图名称前的前缀（模板加载路径:默认值：classpath:/templates/）
   21     prefix: classpath:/templates/
   22     # 在构建URL时添加到视图名称后的后缀（默认值：.html）
   23     suffix: .html
   24     # Thymeleaf模板解析器在解析器链中的顺序。默认情况下，它排第一位。顺序从1开始，只有在定义了额外的TemplateResolver Bean时才需要设置这个属性。
   25     # template-resolver-order: 
   26     # 可解析的视图名称列表
   27     # view-names:
   28     # - 
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

3. 新建模板

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 <!DOCTYPE html>
    2 <html xmlns:th="http://www.thymeleaf.org">
    3 <head>
    4 <meta charset="UTF-8">
    5 <title>Insert title here</title>
    6 </head>
    7 <body>
    8     <div th:text="${url}"></div>
    9 </body>
   10 </html>
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   在html页面中引入thymeleaf命名空间，即，此时在html模板文件中动态的属性使用th:命名空间修饰 。（避免thymeleaf语法报错，标注警告等问题）
   以下简单了解：详情请参阅博主模板语法笔记。

   > ### th属性
   >
   > html有的属性，Thymeleaf基本都有，而常用的属性大概有七八个。其中th属性执行的优先级从1~8，数字越低优先级越高。
   >
   > 1. th:text ：设置当前元素的文本内容，相同功能的还有th:utext，两者的区别在于前者不会转义html标签，后者会。优先级不高：order=7
   > 2. th:value：设置当前元素的value值，类似修改指定属性的还有th:src，th:href。优先级不高：order=6
   > 3. th:each：遍历循环元素，和th:text或th:value一起使用。注意该属性修饰的标签位置，详细往后看。优先级很高：order=2
   > 4. th:if：条件判断，类似的还有th:unless，th:switch，th:case。优先级较高：order=3
   > 5. th:insert：代码块引入，类似的还有th:replace，th:include，三者的区别较大，若使用不恰当会破坏html结构，常用于公共代码块提取的场景。优先级最高：order=1
   > 6. th:fragment：定义代码块，方便被th:insert引用。优先级最低：order=8
   > 7. th:object：声明变量，一般和*{}一起配合使用，达到偷懒的效果。优先级一般：order=4
   > 8. th:attr：修改任意属性，实际开发中用的较少，因为有丰富的其他th属性帮忙，类似的还有th:attrappend，th:attrprepend。优先级一般：order=5

4. 编写控制器

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

   ```
    1 @Controller
    2 public class HtmlController {
    3 
    4     @RequestMapping("/indexHtml")
    5     public String indexHtml(Model model) {
    6         model.addAttribute("url","XSGE个人网站：http://www.xsge123.com");
    7         System.out.println("ceshi");
    8         return "indexHtml";
    9     }
   10 }
   ```

   [![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

5. 测试访问

   启动SpringBoot主程序，单开浏览器访问：http://localhost:8080/indexHtml

   

   > 常见错误：没有手动重启服务器！导致无法解析，错误代码500，显示信息：template might not exist or might not be accessible by any of the configured Template Resolvers（模板可能不存在，或者任何配置的模板解析程序都无法访问）

# 附录　　

 　SpingBoot整合JSP，打包问题：两种总结

1.网上很多说法是，在pom.xml中配置打包地址（这种方法可行）实现如下：

　　A，修改pom文件，在插件中配置JSP资源路径。

　　B，修改SpringBoot打包插件版本为（1.4.2.RELEASE)

　　C，运行Mavne命令打包

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 <!-- SpringBoot打包插件 -->
 2 <build>
 3     <plugins>
 4         <plugin>
 5             <groupId>org.springframework.boot</groupId>
 6             <artifactId>spring-boot-maven-plugin</artifactId>
 7             <version>1.4.2.RELEASE</version>
 8         </plugin>
 9     </plugins>
10 
11     <resources>
12         <resource>
13             <directory>src/main/java</directory>
14             <includes>
15                 <include>**/**</include>
16             </includes>
17         </resource>
18         <resource>
19             <directory>src/main/resources</directory>
20             <includes>
21                 <include>**/**</include>
22             </includes>
23             <filtering>false</filtering>
24         </resource>
25         <!-- 打包时将jsp文件拷贝到META-INF目录下-->
26         <resource>
27             <!-- 指定resources插件处理哪个目录下的资源文件 -->
28             <directory>src/main/webapp</directory>
29             <includes>
30                 <include>**/**</include>
31             </includes>
32             <!--注意此次必须要放在此目录下才能被访问到-->
33             <targetPath>META-INF/resources</targetPath>
34             <filtering>false</filtering>
35         </resource>
36     </resources>
37 </build>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

> 很多人可能都在网上看到过类似的配置，但几乎没有明确版本！然而，这种配置方式在SpringBoot版本升级后，官方便提出不推荐使用，在SpringBoot1.4.x版本之后，推荐整合JSP时打包为war包方式，SpringBoot项目整合内置Tomcat插件，所以我们打包的war文件，无需考虑安装服务器问题，就把它当作一个jar文件一样运行即可
> **Java  -jar  文件名.jar/war**

2.无需在pom中过多配置，修改打包方式为war即可。

　　A，修改打包方式为war

　　B，运行Maven命令打包

　　C，启动jar测试访问

```
<packaging>war</packaging>
```