# 背景

用maven构建的父子工程。父工程下有多个子工程。现在要实现的功能是将某个项目打包制作为docker镜像然后推送到一个Docker镜像仓库（Harbor镜像仓库）。

# dockerfile-maven-plugin插件

在父pom中，引入插件：

```xml
<pluginManagement>
     <plugins>
         <plugin>
             <groupId>com.spotify</groupId>
             <artifactId>dockerfile-maven-plugin</artifactId>
             <version>${docker.maven.plugin.version}</version>
             <executions>
                 <execution>
                     <id>default</id>
                     <goals>
                         <!--如果package时不想用docker打包,就注释掉这个goal-->
                         <goal>build</goal>
                         <goal>push</goal>
                     </goals>
                 </execution>
             </executions>
             <configuration>
                 <contextDirectory>${project.basedir}</contextDirectory>
                 <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
                 <repository>${docker.repostory}/${docker.registry.name}/${project.artifactId}</repository>
                 <buildArgs>
                     <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                 </buildArgs>
             </configuration>
         </plugin>
     </plugins>
 </pluginManagement>
```

其中，插件的属性配置如下：

```xml
<properties>
    <!--Harbor仓库的地址，ip:port-->
    <docker.repostory>192.168.1.6:9001</docker.repostory>
    <!--上传的Docker镜像前缀，此前缀一定要和Harbor中的项目名称一致，表示打包后的Docker镜像会上传到Harbor的哪个项目中-->
    <docker.registry.name>fyk_project</docker.registry.name>
    <docker.maven.plugin.version>1.4.10</docker.maven.plugin.version>
</properties>
```

这样，在每个子工程中，只要进行一些其他的个性化配置就好了，例如配置镜像的标签，如下：

```xml
<finalName>fyk-config</finalName>
<plugins>
    <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>dockerfile-maven-plugin</artifactId>
        <configuration>
            <tag>${docker.image.tag}</tag>
        </configuration>
    </plugin>
</plugins>
```

这里的配置属性为：

```xml
<properties>
    <!--Docker镜像的标签，也就是版本-->
    <docker.image.tag>v1.0.0</docker.image.tag>
</properties>
```

到这里，IDEA中，dockerfile-maven-plugin的maven配置就算完成了。但是，此时还不能制作镜像和推送镜像，还需镜像其他两项配置。

## 制作镜像配置

完成IDEA的maven插件配置之后，现在我们开始制作镜像。
首先要明白：**镜像的制作以及推送操作，都是由Docker来完成的，所以，必须要安装Docker环境。**简单的说dockerfile-maven-plugin只是简化了直接操作Docker的复杂度，该是Docker完成的事情，还得由Docker来完成。
所以，明白这一点之后，就知道了并不是一定要在本地安装Docker环境，只要有Docker环境就可以了。
既然是只要有Docker环境就可以了，那么这里我使用服务器上的Docker环境（当然，也可以使用本地的Docker环境，使用本地Docker环境无需额外的配置，按照教程在本地安装上Docker就可以了）。要使用服务器的Docker环境，需要**配置一个环境变量**（默认情况下，插件是连接本地的Docker环境，即127.0.0.1），环境变量的配置如下，改成你自己的服务器Docker环境即可（**可能需要重启电脑**）：
DOCKER_HOST tcp://192.168.1.6:2375
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200808130616648.png)
此时，在进行Docker镜像制作的操作还需一步，编写Dockerfile文件：在子工程的目录下（与pom文件同级）创建一个Dockerfile文件，内容如下：

```shell
FROM frolvlad/alpine-oraclejdk8
EXPOSE 10000
ADD target/fyk-config.jar /fyk-config.jar
ENTRYPOINT exec java -jar /fyk-config.jar
```

好了，现在执行mvn clean package打包命令，即可制作镜像，当该命令执行完之后，就可以在Docker环境中，查看镜像了，如下，就是我才制作的镜像：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200808131333238.png)

## 推送镜像到仓库

由于我要推送的仓库是私有的，需要用户名密码，所以，要在maven的配置文件（setting文件）中，添加如下配置：

```xml
<servers>
	<server>
		<id>192.168.1.6:9001</id>
		<username>fyk</username>
		<password>Fyk123456</password>
		<configuration>
			<email>844645164@qq.com</email>
		</configuration>
	</server>
</servers>
```

这里的id，是仓库的IP和端口。
配置好之后，执行命令：

```shell
mvn dockerfile:push
```

该命令的意思是，推送镜像到镜像仓库。推送的镜像就是刚才在Docker中生成的镜像（实例上，该命令是将Docker中的镜像推送到仓库中，至于这个镜像是通过什么方式创建的，并不管）。执行完该命令，查看Harbor仓库：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200808134038813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Z5azg0NDY0NTE2NA==,size_16,color_FFFFFF,t_70)至此，IDEA中使用maven插件dockerfile-maven-plugin制作并推送Docker镜像到私有仓库(Harbor)就完成了。

# 问题

**不能连接远程Docker服务？**
该问题，可能是防火墙的原因（这个简单，关闭防火墙或者开双边墙，如果不能开墙，就在本地安装Docker）；如果墙是通的，那可能是Docker没有开启远程访问：
开启docker的远程访问：
进入到/lib/systemd/system/docker.service

```shell
vim /lib/systemd/system/docker.service
```

找到ExecStart行，修改成下边这样

```shell
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

重启守护进程

```shell
systemctl daemon-reload
```

重启docker

```shell
systemctl restart docker
```

用浏览器访问验证
http://ip:2375/images/json
至此，docker的远程访问就打开了。

------

**在操作过程供，某一步操作失败，例如，失败提示需要docker login等等。**
其实这个问题，和dockerfile-maven-plugin插件没有关系。首先明白一点，该插件只是简化操作而已，该是Docker做的事情，还是要有Docker来做。因此，如果出现太多这种问题，可以自己在docker环境中手动上传一个镜像到仓库试试，如果能上传成功，那么按照本文所述方式，应该都可以成功。如果手动上传失败了，那可能是环境中，某个地方错了。比如，提示docker login，那么就登录就是了。具体的问题，可以重新查阅下其他资料。

------

**报错：Get https://192.168.1.6:9001/v2/: http: server gave HTTP response to HTTPS client**
在Docker环境中（服务器上）的文件：/etc/docker/daemon.json中，添加：
“insecure-registries”:[“192.168.1.6:9001”]
如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200808135731265.png)