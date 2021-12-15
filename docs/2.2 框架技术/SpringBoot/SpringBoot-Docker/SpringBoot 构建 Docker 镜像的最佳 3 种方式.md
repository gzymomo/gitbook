[SpringBoot 构建 Docker 镜像的最佳 3 种方式](https://www.cnblogs.com/fengpinglangjingruma/p/13994545.html)



本文将介绍3种技术，通过 Maven 把 SpringBoot 应用构建成 Docker 镜像。

（1）使用 spring-boot-maven-plugin 内置的 **build-image**.

（2）使用 Google 的 **jib-maven-plugin**。

（3）使用 **dockerfle-maven-plugin**。

# 一、Spring Boot maven 插件 的 build-image

Spring Boot 预装了自己的用于构建 Docker 镜像的插件，我们无需进行任何更改，因为它就在 pom.xml 中的 spring-boot-starter-parent。

你不需要写 Dockerfile，也不用操别的心，plugin 都帮你做了，例如 Spring 建议的安全、内存、性能等问题。

只需要简单的执行：

```
mvn spring-boot:build-image
```

执行完成后会看到成功提示信息：

![img](https://upload-images.jianshu.io/upload_images/15462057-56339753a21fb9fb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行容器测试：

```
docker run -p 9090:8080 -t demo-application:0.0.1-SNAPSHOT
```

注意：这里映射的本机端口是`9090`。

# 二、jib-maven-plugin

Jib 是一个 Maven 和 Gradle 插件，用来创建 Docker 镜像。

这个插件有一个非常明显的特点：**不需要本地安装 Docker**，这对持续集成是非常方便的，Jib 可以直接推送到指定的 Docker 仓库。

Jib 同样也不需要写 Dockerfile。

使用起来也非常方便，不需要改代码，也可以不改动 pom.xml。

只需要执行：

```
mvn compile com.google.cloud.tools:jib-maven-plugin:2.3.0:dockerBuild
```

- mvn compile

是我们很熟悉的 maven 编译指令。

- com.google.cloud.tools:jib-maven-plugin:2.3.0

指定了使用 Jib 插件

- dockerBuild

是 Jib 插件的执行目标，`dockerBuild` 指定了 Jib 使用我们本地安装的 Docker。

执行完成后会看到成功提示信息：

![img](https://upload-images.jianshu.io/upload_images/15462057-9b927310f9bf6c7c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动容器测试：

```
docker run -p 9091:8080 -t demo-application:0.0.1-SNAPSHOT
```

注意：这里映射的本机端口是`9091`。

# 三、dockerfile-maven-plugin

这个插件就需要我们写 Dockerfile 了，Domo 项目中已经准备好了。

Dockerfile 需要放在项目的根目录下，和 pom.xml 在一起。

![img](https://upload-images.jianshu.io/upload_images/15462057-5e7aba3c83a8cbe9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后，需要在 pom.xml 中添加一些配置，以便指定仓库、tag 标签，还有上面 Dockerfile 中定义的 `JAR_FILE`。

![img](https://upload-images.jianshu.io/upload_images/15462057-7da0e1a7f1b91c3d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

执行 `mvn package` 时就会自动构建镜像了，执行完成后就会看到提示信息：

![image](https://upload-images.jianshu.io/upload_images/15462057-a3078002303a0fdd?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

运行容器：

```
docker run -p 9092:8080 -t demo-application:0.0.1-SNAPSHOT
```

注意：映射的本机端口是 `9092`。



# 小结

这3个里面最方便的是 SpringBoot 原生的方式，什么都不需要自己做，直接就能用。

最有特点的是 Jib，不需要你本地安装 Docker，可以直接推送到指定的仓库，而且使用起来也很简单。

看起来最麻烦的就是 dockerfile-maven-plugin 这个插件了，需要写 Dockerfile，还得添加配置，**但是**，实际上他是最好用的，因为前2个与网络环境有关系。