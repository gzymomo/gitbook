## 二. 配置服务器

### 1. Docker安装

服务器需要安装Docker，如未安装参考这篇文章安装即可 [Docker实战 | 第一篇：Linux 安装 Docker](https://www.cnblogs.com/haoxianrui/p/14067423.html)

### 2. Docker开启远程访问



```
vim /usr/lib/systemd/system/docker.service
# 在ExecStart=/usr/bin/dockerd追加
-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
```

[![img](https://i.loli.net/2020/12/02/zjmryVLKf4U2sCv.png)](https://i.loli.net/2020/12/02/zjmryVLKf4U2sCv.png)



```
# 重新启动
systemctl daemon-reload
systemctl restart docker
```



```
# 开放2375端口
firewall-cmd --zone=public --add-port=2375/tcp --permanent
firewall-cmd --reload
```

### 3. 远程访问测试



```
# 查看端口监听是否开启
netstat -nlpt
# curl测试是否生效
curl http://127.0.0.1:2375/info
```

## 二. 配置IDEA

IDEA安装Docker插件,打开插件市场（File->Settings->Plugins）

[![img](https://i.loli.net/2020/12/02/twmD13ReHbFTiA6.png)](https://i.loli.net/2020/12/02/twmD13ReHbFTiA6.png)

安装Docker插件后，配置Docker远程链接

[![img](https://i.loli.net/2020/12/02/otZXbuYAlciwUPQ.png)](https://i.loli.net/2020/12/02/otZXbuYAlciwUPQ.png)

## 三. Maven插件构建Docker镜像

### 1. Maven构建Docker镜像方式

maven构建docker镜像有两种方式，分别docker-maven-plugin和dockerfile-maven，都是出自Spotify公司之手。

进入项目 https://github.com/spotify/docker-maven-plugin

其中有个很显眼的提示：

[![img](https://i.loli.net/2020/12/02/m8vkO7Kqn2dlYZz.png)](https://i.loli.net/2020/12/02/m8vkO7Kqn2dlYZz.png)

[![img](https://i.loli.net/2020/12/02/kUORVwigomFhtIx.png)](https://i.loli.net/2020/12/02/kUORVwigomFhtIx.png)

**docker-maven-plugin**可以不用Dockerfile,纯粹通过pom.xml的配置自动生成Dockerfile来构建Docker镜像。

**dockerfile-maven**依赖Dockerfile文件，需放到项目根目录下，也就是和pom.xml同级。

显然官方推荐的是 dockerfile-maven 这种依赖Dockerfile的方式,但是在部署 [youlai-mall](https://github.com/hxrui/youlai-mall.git) 项目使用 docker-maven-plugin 只要配置好 pom.xml 便无需修改外置配置了，所以更为方便省心，下面就这两种方式如何实现镜像构造进行逐一说明。其中统一以 [youlai-mall](https://github.com/hxrui/youlai-mall.git) 的 youlai-gateway 网关模块进行构建。

### 2. docker-maven-plugin方式构造镜像

**(1). 配置pom.xml**



```
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>
    <executions>
        <!--执行mvn package,即执行 mvn clean package docker:build-->
        <execution>
            <id>build-image</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!-- 镜像名称 -->
        <imageName>${project.artifactId}</imageName>
        <!-- 指定标签 -->
        <imageTags>
            <imageTag>latest</imageTag>
        </imageTags>
        <!-- 基础镜像-->
        <baseImage>openjdk:8-jdk-alpine</baseImage>

        <!-- 切换到容器工作目录-->
        <workdir>/ROOT</workdir>

        <entryPoint>["java","-jar","${project.build.finalName}.jar"]</entryPoint>

        <!-- 指定远程 Docker API地址  -->
        <dockerHost>http://192.168.1.111:2375</dockerHost>

        <!-- 复制 jar包到docker容器指定目录-->
        <resources>
            <resource>
                <targetPath>/ROOT</targetPath>
                <!-- 用于指定需要复制的根目录，${project.build.directory}表示target目录 -->
                <directory>${project.build.directory}</directory>
                <!-- 用于指定需要复制的文件，${project.build.finalName}.jar就是打包后的target目录下的jar包名称　-->
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

**(2). maven打包制作镜像**

项目是聚合工程，先全局执行 `mvn install -DskipTests=true` 完成安装模块jar包到本地仓库，不然模块之间的依赖会报错。

切到文件夹 `youlai-gateway` 执行项目的打包 `mvn package -DskipTests = true`, 在 `package` 生命周期完成镜像的生成。

[![img](https://i.loli.net/2020/12/05/VOkm4SJ6LMWl5oa.png)](https://i.loli.net/2020/12/05/VOkm4SJ6LMWl5oa.png)

**(3). idea创建和启动容器**

[![img](https://i.loli.net/2020/12/05/hQ7KBXTAGf6FMpl.png)](https://i.loli.net/2020/12/05/hQ7KBXTAGf6FMpl.png)

[![img](https://i.loli.net/2020/12/05/8SsoM3VpTJnjIOv.png)](https://i.loli.net/2020/12/05/8SsoM3VpTJnjIOv.png)

**(4). 容器启动测试**

[![img](https://i.loli.net/2020/12/05/7v1xZC9FHfAncUl.png)](https://i.loli.net/2020/12/05/7v1xZC9FHfAncUl.png)

### 3. dockerfile-maven方式构造镜像

**(1). 创建Dockerfile**

按照dockerfile-maven插件的使用说明，创建Dockerfile放置到项目根目录下(pom.xml同级)

[![img](https://i.loli.net/2020/12/04/CjVMrqefbFi2G5v.png)](https://i.loli.net/2020/12/04/CjVMrqefbFi2G5v.png)



```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
ADD target/${JAR_FILE} /app.jar
EXPOSE 9999
ENTRYPOINT ["java","-jar","/app.jar"]
```

Dockerfile参考Spring官方，参考链接 https://spring.io/guides/gs/spring-boot-docker/

| 指令                                  | 说明                                                         |
| :------------------------------------ | :----------------------------------------------------------- |
| FROM openjdk:8-jdk-alpine             | 基础镜像JDK，无则自动拉取镜像                                |
| VOLUME /tmp                           | 挂载容器/tmp目录至宿主机，SpringBoot使用内置Tomcat，默认工作目录/tmp；VOLUME不能指定挂载目录，默认挂载到宿主机/var/lib/docker目录。 |
| ARG JAR_FILE                          | 变量声明，对应pom.xml的JAR_FILE标签的变量                    |
| ADD target/${JAR_FILE} /app.jar       | 复制jar包至容器并重命名为app.jar                             |
| EXPOSE 9999                           | 声明容器暴露端口，仅仅声明无实际作用                         |
| ENTRYPOINT ["java","-jar","/app.jar"] | 设定容器启动时第一个运行的命令及其参数                       |

**(2). 配置pom.xml**



```
<build>
    <plugins>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.13</version>
            <executions>
                <execution>
                    <id>default</id>
                    <goals>
                        <goal>build</goal>
                        <goal>push</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <repository>${project.artifactId}</repository>
                <tag>latest</tag>
                <buildArgs>
                    <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**(3). 设置环境变量**

dockerfile-maven插件默认的DOCKER_HOST=localhost:2375,如果需要构建镜像到服务器，需要修改DOCKER_HOST系统环境变量

[![img](https://i.loli.net/2020/12/04/sLUBEPHbvgqxlAV.png)](https://i.loli.net/2020/12/04/sLUBEPHbvgqxlAV.png)

如果DOCKER_HOST不固定的也可以设置临时变量方便灵活切换



```
set DOCKER_HOST=tcp://192.168.1.111:2375
```

**(4). idea创建和启动容器**

和 `docker-maven-plugin` 一致，请参考上文。