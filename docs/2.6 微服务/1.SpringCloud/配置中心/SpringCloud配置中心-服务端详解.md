[TOC]

[博客园：东北小狐狸：Config 配置中心与客户端的使用与详细](https://www.cnblogs.com/hellxz/p/9306507.html)

# 1、基础架构
![](https://images2018.cnblogs.com/blog/1149398/201807/1149398-20180713180005353-1826517410.jpg)

- 远程git仓库：用来存储配置文件的地方，多环境配置文件使用 hellxztest-{环境名}.yml
- Config Server：分布式配置中心，指定了git仓库uri、搜索路径、访问账号和密码
- 微服务应用：配置客户端（Config Client），指定应用名、配置中心url、环境名、分支名等

# 2、启动流程
1. 微服务应用启动，根据bootstrap.yml（properties）中配置的应用名（application）、环境名（profile）、分支名（label），向Config Server请求配置信息
2. Config Server 根据自己bootstrap.yml（properties）中的Git（或SVN）仓库信息加上客户端传来的配置定位信息去查配置信息的路径
3. Config Server 执行git clone命令，将配置信息下载到本地Git仓库中，将配置信息加载到Spring的ApplicationContext读取内容返回给客户端（微服务应用）
4. 客户端将内容加载到ApplicationContext，配置内容的优先级大于客户端内部的配置内容，进行忽略

**特殊情况**： 当Config Server因为网络原因无法连接到Git或SVN时，客户端的请求过来后，会先连接Git或SVN，如果没连上，就使用本地仓库的配置文件内容进行返回给客户端

# 3、Git配置仓库
## 3.1 本地仓库

Spring Cloud Config默认使用Git，对Git的配置也最简单，这里Config Server只用到了uri、username、password这三个参数就可以读取配置了，通过Git的版本控制可以使Config Server适应特殊的场景。

测试时我们也可以使用本地仓库的方式，使用file://前缀，那么uri的配置就可以写作
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: file://${user.home}/config-repo
```
**注意**：
1. Windows系统需要使用file:///前缀
2. ${user.home}代表当前用户的家目录

## 3.2 占位符配置URI

- ｛application｝、｛profile｝、{label}这些占位符除了标识配置文件规则外，还可以对Git的uri配置，
- 如：spring.cloud.config.server.git.uri=https://github.com/hellxz/SpringCloudlearn/config-repo/{application}

此时spring.application.name的值会填充到这个uri中，从而达到动态获取不同位置的配置.

## 3.3 匹配并配置多个仓库
Spring Cloud Config Server除了使用{应用名}/{环境名}来匹配配置仓库外，还支持通过带有通配符的表达式来匹配。
当有多个匹配规则的时候，可以用逗号分隔多个{应用名}/{环境名}配置规则。
以官方文档例子加减举例：
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo   #默认的仓库
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
            test: 
              pattern: 
                - '*/development'
                - '*/staging'
              uri: https://github.com/development/config-repo
```
如果{应用名}/{环境名}不能匹配到仓库，那么就在默认的uri下去查找配置文件。

上边的例子中，

- simple 仓库自动匹配到 simple/*
- special 仓库的pattern，第一个是应用名以special开头，环境名以dev开头；第二个是应用名包含special，环境名以dev开头；多个匹配到同一uri的pattern用逗号分割
- local 仓库的的pattern也会自动补全为local*/*
- test仓库中的 pattern 是以通配符开始的，需要使用单引号

**注意**：配置多个仓库时，Config Server 在启动时会直接克隆第一个仓库的配置库，其他配置库只有请求时才会clone到本地

## 3.4 子目录存储
通过spring.cloud.config.server.git.searchPaths来定位到Git仓库的子目录中，相当于在uri后加上searchPaths的目录。

searchPaths参数的配置也支持使用{应用名}、{环境名}、{分支名}占位符

比如spring.cloud.config.server.git.searchPaths={应用名}，通过这样的配置，我们能让每一个应用匹配到自己的目录中。

举例：
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          searchPaths: '{application}'
```

## 3.5 访问权限

使用Git仓库的时候，使用HTTP认证需要使用username和password属性来配置账户

举例
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          username: trolley
          password: strongpassword
还可以使用SSH认证，Config Server本地的.ssh文件或使用私钥等进行配置，此处仅为提出这个用法，不作深究，使用方法见http://cloud.spring.io/spring-cloud-static/Finchley.RELEASE/single/spring-cloud.html#_git_ssh_configuration_using_properties
```

# 4、本地仓库
使用版本控制方式将配置文件clone到本地，往往都是克隆到一些临时目录中，但是操作系统往往会清理这些临时目录，这可以导致一些我们不期待的情况，比如丢配置，为了避免出现这种问题，可以通过spring.cloud.config.server.svn.basedir去更改本地仓库的位置。

# 5、本地文件系统
不同于Git和SVN以及他们的本地仓库的存储方式，Spring Cloud Config 提供了本地文件系统的存储方式来保存配置信息，实现方式很简单，设置属性spring.profiles.active=native，Config Server会从应用的src/main/resources目录下搜索配置文件。如果需要指定配置文件的路径，可以通过spring.cloud.config.server.native.searchLocations属性来指定具体配置文件位置。

> 放在本地应用中不如直接配置在bootstrap.yml中，这样一想，这个Config Server端就没什么用了，虽然有这个功能，但还是推荐使用Git和SVN仓库的方式。

# 6、健康监测
配置好了Git或SVN之后，为了能确保能连通仓库，我们需要为其实现健康监测功能，来判断Git或SVN仓库是否可以访问

我们配置了spring.cloud.config.server.git.uri，而且依赖了spring-boot-actuator的依赖，会自动mapped到/health的端点，我们可以访问测试一下
![](https://images2018.cnblogs.com/blog/1149398/201807/1149398-20180713180052874-2015438255.jpg)

> 这里没有配具体的仓库路径，不然会显示出更多的信息

如果我们无法连接到配置仓库的uri那么status就会变成DOWN

在org.springframework.cloud.config.server.config包中有一个ConfigServerHealthIndicator的健康检测器，这个检测器会不断地检查配置的uri是否可以连通。

如果不想使用这个健康检测器，也可以通过使用spring.cloud.config.server.health.enabled=false来禁用它。

# 7、属性覆盖
Config Server提供一个叫Property Overrides即属性覆盖的功能，通过spring.cloud.config.server.overrides属性来设置键值对的参数，这些参数会以Map形式加载到所有Config Client的配置中。

官方举例：
```yml
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```

通过属性覆盖配置的参数不会被Config Client修改，并且Config Client获取配置信息时就会得到这些信息，使用这种特性可以方便我们为客户端应用配置一些共同属性或默认属性。这些属性不是强制的，可以通过改变客户端中的更高优先我级的配置方式来选择是否使用这些属性覆盖的默认值。

# 8、安全保护
配置中心存储的内容很敏感，所以必需做一些安保措施，使用Spring Security更方便一些。

只需要引入依赖并配置用户名和密码
```xml
        <!--Spring Security依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-security</artifactId>
        </dependency>
```
默认情况下会得到一名为user的用户，并在配置中心启动的时候在log中打印出来随机密码，当然大多数情况下我们不会使用这个随机密码，我们可以在配置文件中指定用户和密码
```yml
security:
  user:
    name: user
    password: xxxxxxx
```
通过上边的配置，配置中心已经开启了安全保护，这时候连接配置中心的客户端没有密码的情况下会返回401错误

只需要在客户端中加入账号密码来通过安全校验，举例
```yml
spring:
  cloud:
    config:
      username: user
      password: xxxxxxx
```