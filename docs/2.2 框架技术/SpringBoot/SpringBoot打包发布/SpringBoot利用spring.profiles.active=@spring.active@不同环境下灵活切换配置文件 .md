-  [     SpringBoot利用spring.profiles.active=@spring.active@不同环境下灵活切换配置文件        ](https://www.cnblogs.com/csp1993/p/14477396.html)             

## 一、创建配置文件[#](https://www.cnblogs.com/csp1993/p/14477396.html#一、创建配置文件)

**配置文件结构：**这里建三个配置文件，application.yml作为主配置文件配置所有共同的配置；-dev和-local分别配置两种环境下的不同配置内容，如数据库地址等。
 [![img](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303222836908-890518781.png)](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303222836908-890518781.png)
 application.yml中添加spring.profiles.active配置来动态加载活跃的配置文件：

```

spring:
  profiles:
    active: @spring.active@
```

## 二、POM文件添加PROFILES配置[#](https://www.cnblogs.com/csp1993/p/14477396.html#二、pom文件添加profiles配置)

```

<profiles>
	<profile>
		<id>local</id>
		<properties>
			<spring.active>local</spring.active>
		</properties>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
	</profile>
	<profile>
		<id>dev</id>
		<properties>
			<spring.active>dev</spring.active>
		</properties>
	</profile>
</profiles>
```

以上配置声明有两种配置文件、分别为dev和local。且默认使用local（通过true设置的）。
 这样配置好的项目在maven中就多了一个配置项：
 [![img](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303222944889-1073556343.png)](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303222944889-1073556343.png)

## 三、具体应用[#](https://www.cnblogs.com/csp1993/p/14477396.html#三、具体应用)

### 1、使用mvn命令打包项目打包时[#](https://www.cnblogs.com/csp1993/p/14477396.html#1、使用mvn命令打包项目打包时)

```

  mvn clean package # 清理并打包命令，默认是使用local配置文件。
  mvn clean package -P dev # 清理并指定配置文件打包命令，使用dev配置文件。
```

### 2、手动打包，通过勾选profiles选项切换配置文件[#](https://www.cnblogs.com/csp1993/p/14477396.html#2、手动打包，通过勾选profiles选项切换配置文件)

```

  maven profiles中勾选dev，然后打包，则使用dev配置文件。
```

### 3、本地启动springboot时，以idea为例[#](https://www.cnblogs.com/csp1993/p/14477396.html#3、本地启动springboot时，以idea为例)

如步骤2中勾选所需激活的配置文件后，启动application中的main方法则对应加载勾选中的配置文件。还可以在idea中配置指定加载配置文件，指定后勾选功能失效。方法如下：
 [![img](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303223033067-1780043223.png)](https://img2020.cnblogs.com/blog/761444/202103/761444-20210303223033067-1780043223.png)