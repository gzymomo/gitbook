[Spring Boot 创建 Docker 镜像](https://www.cnblogs.com/liululee/p/13992852.html)



## 1. 传统Docker构建

使用Spring Boot 构建 Docker 镜像的传统方法是使用 Dockerfile 。下面是一个简单的例子：

```java
FROM openjdk:8-jdk-alpine
EXPOSE 8080
ARG JAR_FILE=target/demo-app-1.0.0.jar
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

然后我们可以使用 *docker build* 命令来创建 `Docker` 映像。这对大多数应用程序都很好，但也有一些缺点。

首先，我们使用的是 `Spring Boot` 创建的 fat jar。**这会影响启动时间，尤其是在集装箱环境中**。我们可以通过添加jar文件的分解内容来节省启动时间。

其次，Docker镜像是分层构建的。Spring Boot fat jar 的特性使得所有的应用程序代码和第三方库都放在一个层中。**这意味着即使只有一行代码更改，也必须重新构建整个层**。

通过在构建之前分解 jar ，应用程序代码和第三方库各自获得自己的层。这样，我们便可以利用Docker的缓存机制。现在，当某一行代码被更改时，只需要重新构建相应的层。

考虑到这一点，让我们看看Spring Boot 如何改进创建Docker镜像的过程。

## 2. Buildpacks

**BuildPacks 是一种提供框架和应用程序依赖性的工具**。

例如，给定一个Spring Boot fat jar，一个buildpack将为我们提供Java运行时。这使我们可以跳过 Dockerfile 并自动获得一个合理的docker 镜像。

Spring Boot 包括对 bulidpacks 的Maven和Gradle支持。例如，使用Maven构建时，我们将运行以下命令：

```shell
./mvnw spring-boot:build-image
```

我们观察下一些相关的输出，看看发生了什么：

```shell
[INFO] Building jar: target/demo-0.0.1-SNAPSHOT.jar
...
[INFO] Building image 'docker.io/library/demo:0.0.1-SNAPSHOT'
...
[INFO]  > Pulling builder image 'gcr.io/paketo-buildpacks/builder:base-platform-api-0.3' 100%
...
[INFO]     [creator]     ===> DETECTING
[INFO]     [creator]     5 of 15 buildpacks participating
[INFO]     [creator]     paketo-buildpacks/bellsoft-liberica 2.8.1
[INFO]     [creator]     paketo-buildpacks/executable-jar    1.2.8
[INFO]     [creator]     paketo-buildpacks/apache-tomcat     1.3.1
[INFO]     [creator]     paketo-buildpacks/dist-zip          1.3.6
[INFO]     [creator]     paketo-buildpacks/spring-boot       1.9.1
...
[INFO] Successfully built image 'docker.io/library/demo:0.0.1-SNAPSHOT'
[INFO] Total time:  44.796 s
```

第一行显示我们构建了标准的 fat jar，与其他典型的maven包一样。

下一行开始Docker映像构建。然后，看到这个 bulid 拉取了 packeto 构建器。

packeto 是基于云原生 bulidpacks 的实现。**它负责分析我们的项目并确定所需的框架和库**。在我们的例子中，它确定我们有一个Spring Boot项目并添加所需的构建包。

最后，我们看到生成的Docker映像和总构建时间。注意，在第一次构建时，花了相当多的时间下载构建包并创建不同的层。

buildpacks 的一大特点是Docker映像是多层的。因此，如果我们只更改应用程序代码，后续构建将更快：

```shell
...
[INFO]     [creator]     Reusing layer 'paketo-buildpacks/executable-jar:class-path'
[INFO]     [creator]     Reusing layer 'paketo-buildpacks/spring-boot:web-application-type'
...
[INFO] Successfully built image 'docker.io/library/demo:0.0.1-SNAPSHOT'
...
[INFO] Total time:  10.591 s
```

## 3. 层级jar包

在某些情况下，我们可能不喜欢使用 bulidpacks ——也许我们的基础架构已经绑定到另一个工具上，或者我们已经有了我们想要重新使用的自定义 Dockerfiles 。

**基于这些原因，Spring Boot 还支持使用分层jars** 构建Docker映像。为了了解它的工作原理，让我们看看一个典型的Spring Boot fat jar 布局：

```bash
org/
  springframework/
    boot/
  loader/
...
BOOT-INF/
  classes/
...
lib/
...
```

fat jar 由3个主要区域组成：

- 启动Spring应用程序所需的引导类
- 应用程序代码
- 第三方库

使用分层jar，结构看起来很相似，但是我们得到了一个新的 *layers.idx* 将 fat jar 中的每个目录映射到一个层的文件：

```yaml
- "dependencies":
  - "BOOT-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "BOOT-INF/classes/"
  - "BOOT-INF/classpath.idx"
  - "BOOT-INF/layers.idx"
  - "META-INF/"
```

Out-of-the-box, Spring Boot provides four layers:

开箱即用，Spring Boot 提供4层：

- *dependencies*: 来自第三方的依赖
- *snapshot-dependencies*: 来自第三方的 snapshot 依赖
- *resources*: 静态资源
- *application*: 应用程序代码和资源（resources）

**我们的目标是将应用程序代码和第三方库放置到层中，以反映它们更改的频率**。

例如，应用程序代码可能是更改最频繁的代码，因此它有自己的层。此外，每一层都可以独立演化，只有当一层发生变化时，才会为它重建 Docker 镜像。

现在我们了解了分层 jar 结构，接下来看看如何利用它来制作 Docker 映像。

### 3.1.创建分层 jar

首先，我们必须建立一个项目来创建一个分层的jar。对于Maven，则需要在POM的 Spring Boot plugin 部分添加一个新的配置：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <layers>
            <enabled>true</enabled>
        </layers>
    </configuration>
</plugin>
```

有了这个配置，Maven *package* 命令（包括它的其他依赖命令）将使用前面提到的四个默认层生成一个新的分层jar。

### 3.2. 查看和提取分层

下一步，我们需要从 jar 中提取层，这样Docker镜像才能拥有正确的层。
要检查分层jar的任何层，可以运行以下命令：

```bash
java -Djarmode=layertools -jar demo-0.0.1.jar list
```

然后提取它们，运行命令：

```bash
java -Djarmode=layertools -jar demo-0.0.1.jar extract
```

### 3.3. 创建Docker映像

将这些层合并到 Docker 映像中的最简单方法是使用 Dockerfile ：

```bash
FROM adoptopenjdk:11-jre-hotspot as builder
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract
 
FROM adoptopenjdk:11-jre-hotspot
COPY --from=builder dependencies/ ./
COPY --from=builder snapshot-dependencies/ ./
COPY --from=builder spring-boot-loader/ ./
COPY --from=builder application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

这个 Dockerfile 从fat jar中提取层，然后将每个层复制到Docker映像中。

**每个COPY指令最终都会在Docker映像**中生成一个新层。

如果我们构建这个Dockerfile，我们可以看到分层jar中的每个层都作为自己的层添加到Docker镜像中：

```bash
...
Step 6/10 : COPY --from=builder dependencies/ ./
 ---> 2c631b8f9993
Step 7/10 : COPY --from=builder snapshot-dependencies/ ./
 ---> 26e8ceb86b7d
Step 8/10 : COPY --from=builder spring-boot-loader/ ./
 ---> 6dd9eaddad7f
Step 9/10 : COPY --from=builder application/ ./
 ---> dc80cc00a655
...
```