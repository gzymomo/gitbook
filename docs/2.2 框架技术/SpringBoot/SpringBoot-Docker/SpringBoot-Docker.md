[TOC]

微信公众号：Aditya Bhuyan K8S中文社区
# 1、Docker关键概念
Docker有四个关键概念：images, layers, Dockerfile 和 Docker cache 。简而言之，Dockerfile描述了如何构建Docker镜像。镜像由许多层组成。Dockerfile从基础镜像开始，并添加了其他层。当新内容添加到镜像时，将生成一个新层。所构建的每个层都被缓存，因此可以在后续构建中重复使用。当Docker构建运行时，它可以从缓存中获取重复使用任何已有层。这就减少了每次构建所需的时间和空间。任何已更改或以前尚未构建的内容都将根据需要进行构建。

![](https://www.showdoc.cc/server/api/common/visitfile/sign/697f74a013e27b8827dd13e76ab1fcb3?showdoc=.jpg)

# 2、镜像层内容很重要
镜像各层的重要性。Docker缓存中的现有层，只有当改镜像层内容没有变更时，才能被使用。在Docker构建期间更改的层越多，Docker需要执行更多的工作来重建镜像。镜像层顺序也很重要。如果某个图层的所有父图层均未更改，则该图层就能被重用。因此，最好把比较频繁更改的图层放在上面，以便对其更改会影响较少的子图层。
镜像层的顺序和内容很重要。当你把应用程序打包为Docker镜像时，最简单的方法是将整个应用程序放置到一个单独的镜像层中。但是，如果该应用程序包含大量静态库依赖，那么即使更改很少的代码，也需要重新构建整个镜像层。这就需要在Docker缓存中，花费大量构建时间和空间。

# 3、镜像层影响部署
部署Docker镜像时，镜像层也很重要。在部署Docker镜像之前，它们会被推送到Docker远程仓库。该仓库是所有部署镜像的源头，并且经常包含同一镜像的许多版本。Docker非常高效，每个层仅存储一次。但是，对于频繁部署且具有不断重建的大体积层的镜像，这就不行了。大体积层的镜像，即使内部只有很少的更改，也必须单独存储在仓库中并在网络中推送。因为需要移动并存储不变的内容，这就会增加部署时间。

# 4、Docker中的Spring Boot应用
使用uber-jar方法的Spring Boot应用程序本身就是独立的部署单元。该模型非常适合在虚拟机或构建包上进行部署，因为该应用程序可带来所需的一切。但是，这对Docker部署是一个缺点：Docker已经提供了打包依赖项的方法。将整个Spring Boot JAR放入Docker镜像是很常见的，但是，这会导致Docker镜像的应用程序层中的不变内容太多。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/18ddc7ed2c3f30d34a2891c59e780b94?showdoc=.jpg)

# 5、单层方法
Docker的Spring Boot指南 列出了单层Dockerfile来构建你的Docker镜像：
```bash
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
它的最终结果是一个正常运行的Docker镜像，其运行方式与你期望Spring Boot应用程序运行的方式完全相同。但是，由于它基于整个应用程序JAR，因此存在分层效率问题。随着应用程序源的更改，整个Spring Boot JAR都会被重建。下次构建Docker镜像时，将重新构建整个应用程序层，包括所有不变的依赖库。

## 5.1 深入地研究单层方法
单层方法使用Open Boot JDK基础镜像之上的Spring Boot JAR作为Docker层构建Docker镜像：
```bash
$ docker images
REPOSITORY                    TAG         IMAGE ID            CREATED             SIZE
springio/spring-petclinic     latest      94b0366d5ba2        16 seconds ago      140MB
```
生成的Docker镜像为140 MB。你可以使用docker history 命令检查图层 。你可以看到Spring Boot应用程序JAR已复制到镜像中，大小为38.3 MB。
`docker history springio/spring-petclinic`
下次构建Docker镜像时，将重新创建整个38 MB的层，因为重新打包了JAR文件。
在此示例中，应用程序的大小相对较小（因为仅基于spring-boot-starter-web和其他依赖项，例如spring-actuator）。在实际开发中，这些大小通常要大得多，因为它们不仅包括Spring Boot库，还包括其他第三方库。根据我的经验，实际的Spring Boot应用程序的大小范围可能在50 MB到250 MB之间（如果不是更大的话）。
仔细观察该应用程序，应用程序JAR中只有372 KB是应用程序代码。其余38 MB是依赖库。这意味着实际上只有0.1％的层在变化。其余99.9％不变。

## 5.2 镜像层生命周期
这是基于镜像层的基本考虑：内容的生命周期。镜像层的内容应具有相同的生命周期。Spring Boot应用程序的内容有两个不同的生命周期：不经常更改的依赖库和经常更改的应用程序类。
每次由于应用程序代码更改而重建该层时，也会包含不变的二进制文件。在快速的应用程序开发环境中，不断更改和重新部署应用程序代码，这种附加成本可能变得非常昂贵。
想象一个应用团队在Pet Clinic上进行迭代。团队每天更改和重新部署应用程序10次。这10个新层的成本为每天383 MB。如果使用更多实际大小，则每天最多可以达到2.5 GB或更多。最终将浪费大量的构建时间，部署时间和Docker仓库空间。
快速迭代的开发和交付是决定我们是继续使用简单的单层方法，还是采用更有效的替代方法。