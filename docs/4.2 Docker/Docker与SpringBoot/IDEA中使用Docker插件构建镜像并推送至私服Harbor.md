- [Centos8.3、docker部署springboot项目实战记录](https://www.cnblogs.com/jishuzhaichen/p/14943084.html)

- [最强菜鸟](https://blog.csdn.net/qq_40298902)：[菜鸟的IDEA使用Docker插件](https://blog.csdn.net/qq_40298902/article/details/106543208)
- [【Docker】Maven打包SpringBoot项目成Docker镜像并上传到Harbor仓库（Eclipse、STS、IDEA、Maven通用）](https://www.cnblogs.com/binghe001/p/12810675.html)
- [使用Dockerfile Maven插件](https://www.cnblogs.com/zyon/p/11266952.html)
- [Idea远程一键部署springboot到Docker](https://juejin.cn/post/6844903865192562696)
- [Docker教程(九)部署Spring Boot项目](https://cloud.tencent.com/developer/article/1667567)



# 一、开启Docker服务器的远程访问

## 1.1 开启2375远程访问

默认的dokcer是不支持远程访问的，需要加点配置，开启Docker的远程访问

```bash
# 首先查看docker配置文件所在位置
systemctl status docker

# 会输出如下内容：
● docker.service - Docker Application Container Engine
   Loaded: loaded (/etc/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-12-17 14:22:23 CST; 18min ago
     Docs: http://docs.docker.com
 Main PID: 25113 (dockerd)
```

确定docker配置文件位置在：/etc/systemd/system/docker.service

然后编辑修改docker配置文件：

```bash
vi /lib/systemd/system/docker.service
```

找到包含ExecStart的这行，添加如下内容：

```bash
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock \
```

其中，2375端口为docker远程服务端口，包含了docker可视化工具portainer，以及远程上传镜像的功能。



## 1.2 添加harbor镜像配置

编辑docker的配置文件：

```bash
vi /etc/docker/daemon.json
# 添加harbor镜像地址
{
 "insecure-registries": ["192.168.0.20:81"]
}
```



## 1.3 重启docker服务

```bash
# 后台配置重新加载
systemctl daemon-reload 
# 重启docker服务
systemctl restart docker.service
# 此处可能会出现docker无法启动情况，可能是由于docker.service配置文件修改错误，重新修改一次然后重新执行上述命令即可

#查看配置的端口号（2375）是否开启（非必要）
netstat -nlpt
```

# 二、通过IDEA操作Docker

## 2.1 下载docker插件

使用idea的docker插件连接docker，idea默认已经下载过docker插件了，如果没有的话，需要在idea下载docker插件
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604112112666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)

## 2.2 配置远程docker

点击idea的设置选项（file --> setting -> docker）,新建连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604112328618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604112921867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
连接成功之后就可以使用服务器(虚拟机)上的docker了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604113352380.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)

## 2.3 拉取镜像

idea可以通过可视化的方式拉取镜像，不用自己去敲命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604155353918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020060415552085.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604155630138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
有时候会出现拉取的时间超时的情况，可以配置一下国内的镜像获取阿里云的加速器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604155749151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)

## 2.4 创建容器并运行

创建并且运行docker容器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604153157588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604153857944.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
创建成功之后可以看到新创建的容器，也可以在服务器(虚拟机)上用docker命令查看
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604155045834.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)

重启容器、停止容器和删除容器等操作
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604152032718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)

# 三、IDEA-Maven打包镜像

简书：尹楷楷：[maven 插件docker-maven-plugin之推送镜像到harbor私有仓库（二）](https://www.jianshu.com/p/2481d9843e86)
CSDN：喝醉的咕咕鸟：[docker--私服搭建和推送于Springboot项目打成镜像](https://blog.csdn.net/weixin_43549578/article/details/87864614)

将构建好的镜像通过docker-maven-plugin插件上传到harbor私服



## 3.1 修改maven的配置文件settings.xml

在maven的配置文件中，添加harbor私服的用户名及密码：

```xml
  <servers>
    <server>
    <id>dockerharbor</id>
    <username>harbor</username>
    <password>123456</password>
    <configuration>
      <email>123456@aliyun.com</email>
    </configuration>
  </server>
  </servers>
```

## 3.2 修改SpringBoot项目中的pom.xml

添加属性配置，属性配置，在后面的插件配置里有引用这个：

- docker.repostory 是docker私服地址，harbor配置完默认端口就是80，可以不带端口号。但是我将之改成81了
- docker.registry.name 即是在harbor中配置的镜像仓库名，必须一致！这里我配的是test，因为harbor中配置的镜像仓库名也是test。

![](https://upload-images.jianshu.io/upload_images/13965490-e28983fe59e365f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

```xml
<properties>
	<!--docker插件-->
	<!-- docker私服地址,Harbor配置完默认地址就是80,默认不带端口号。但是我这里是81 -->
	<docker.repostory>192.168.10.11:81</docker.repostory>
	<!--项目名,需要和Harbor中的项目名称保持一致 -->
	<docker.registry.name>test</docker.registry.name>
</properties>
```

## 3.3 docker-maven-plugin插件配置

- serverId 指定之前在maven的settings.xml中配置的server节点，这样maven会去找其中配置的用户名密码和邮箱
- registryUrl 指定上面配置的properties属性，即是harbor私服的访问url，注意我设置的使用81端口，默认是80端口
- imageName 指定上传harbor私服的镜像名，必须和harbor上的url、镜像仓库名保持一致。其中的docker.registry.name就是上面配置的properties属性

### 方式一：

```bash
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <executions>
                    <execution>
                        <id>build-image</id>
                        <!--用户只需执行mvn package ，就会自动执行mvn docker:build-->
                        <phase>package</phase>
                        <goals>
                            <goal>build</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <serverId>harbor</serverId>
                    <registryUrl>http://${docker.repostory}</registryUrl>
                    <!--Building image 192.168.0.20/demo1-->
                    <imageName>${docker.repostory}/${docker.registry.name}/${project.artifactId}:${project.version}
                    </imageName>
                    <!--指定标签 这里指定的是镜像的版本，我们默认版本是latest-->
                    <!--<imageTags>
                        <imageTag>latest</imageTag>
                    </imageTags>-->
                    <!--指定基础镜像jdk1.8-->
                    <baseImage>jdk:1.8</baseImage>
                    <!-- 指定 Dockerfile 路径-->
                    <!--镜像制作人本人信息 -->
                    <!--<maintainer>1090239782@qq.com</maintainer>-->
                    <!--切换到工作目录-->
                    <workdir>/opt</workdir>
                    <!--${project.build.finalName}.jar是打包后生成的jar包的名字-->
                    <entryPoint>["java", "-jar","-Xms256m","-Xmx512m","/${project.build.finalName}.jar"]
                    </entryPoint>
                    <!--必须配置dockerHost标签（除非配置系统环境变量DOCKER_HOST）-->
                    <dockerHost>http://192.168.0.20:2375</dockerHost>
                    <!-- jar包位置-->
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <!-- target目录下-->
                            <directory>${project.build.directory}</directory>
                            <!--通过jar包名找到jar包-->
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

接下来只要先点击clean清除之前的所有打包的文件，然后再点击package打包文件即可完成镜像的构建，真正的一键部署
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604170831799.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604173321992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
到此镜像构建成功，接下来只要创建容器跑起来即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604173459373.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200604173602215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjk4OTAy,size_16,color_FFFFFF,t_70)
通过ip访问



此种方式，直接通过Maven的package命令，即可实现镜像的制作，但是要推送镜像到harbor私服，还需执行docker:push，即：

点击push，将镜像推送到harbor私服中

![](https://upload-images.jianshu.io/upload_images/13965490-5bbc95af04353c2d.png?imageMogr2/auto-orient/strip|imageView2/2/w/233/format/webp)



### 方式二：

```xml
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>docker-maven-plugin</artifactId>
  <version>1.0.0</version>
  <configuration>
	  <serverId>my-hub</serverId>
	  <registryUrl>http://${docker.repostory}</registryUrl>
	  <!--必须配置dockerHost标签（除非配置系统环境变量DOCKER_HOST）-->
	  <dockerHost>http://192.168.10.11:2375</dockerHost>
	  <!--Building image 192.168.10.11/demo1-->
	  <imageName>${docker.repostory}/${docker.registry.name}/${project.artifactId}:${project.version}</imageName>
	  <!-- 指定 Dockerfile 路径-->
	  <dockerDirectory>${basedir}/</dockerDirectory>
	  <!-- jar包位置-->
	  <resources>
		  <resource>
		  <targetPath>/ROOT</targetPath>
		  <!-- target目录下-->
		  <directory>${project.build.directory}</directory>
		  <!--通过jar包名找到jar包-->
		  <include>${pack-name}</include>
		  </resource>
	  </resources>
  </configuration>
</plugin>
```

那么Dockerfile文件中的jar包名相应需要修改：

```bash
FROM java:8
WORKDIR /ROOT
ADD /ROOT/demo1-2.jar /ROOT/
ENTRYPOINT ["java", "-jar", "demo1-2.jar"]
```

点击pakage打包，target 上生成了springboot工程的jar包



完了之后，点击docker bulid 构建工程镜像
![](https://upload-images.jianshu.io/upload_images/13965490-bf11c9dcb4a265fb.png?imageMogr2/auto-orient/strip|imageView2/2/w/370/format/webp)

然后点击push，将镜像推送到harbor私服中

![](https://upload-images.jianshu.io/upload_images/13965490-5bbc95af04353c2d.png?imageMogr2/auto-orient/strip|imageView2/2/w/233/format/webp)

## 3.4 docker-maven-plugin操作容器

此部分内容参考：

- 掘金：MacroZheng：https://juejin.im/post/6868060821927723021





- `docker-maven-plugin`不仅可以操作镜像，还可以操作容器，比如我们以前需要使用如下Docker命令来运行容器；

```bash
docker run -p 8080:8080 --name mall-tiny-fabric \
--link mysql:db \
-v /etc/localtime:/etc/localtime \
-v /mydata/app/mall-tiny-fabric/logs:/var/logs \
-d 192.168.3.101:5000/mall-tiny/mall-tiny-fabric:0.0.1-SNAPSHOT
```

- 现在我们只需在插件中配置即可，在`<image>`节点下添加`<run>`节点可以定义容器启动的行为：

```xml
<!--定义容器启动行为-->
<run>
    <!--设置容器名，可采用通配符-->
    <containerNamePattern>${project.artifactId}</containerNamePattern>
    <!--设置端口映射-->
    <ports>
        <port>8080:8080</port>
    </ports>
    <!--设置容器间连接-->
    <links>
        <link>mysql:db</link>
    </links>
    <!--设置容器和宿主机目录挂载-->
    <volumes>
        <bind>
            <volume>/etc/localtime:/etc/localtime</volume>
            <volume>/mydata/app/${project.artifactId}/logs:/var/logs</volume>
        </bind>
    </volumes>
</run>
```

- 之后直接使用`docker:start`命令即可启动了；

```bash
mvn docker:start

[root@linux-local mydata]# docker ps
CONTAINER ID        IMAGE                                                         COMMAND                  CREATED             STATUS              PORTS                                            NAMES
95ce77c0394b        192.168.3.101:5000/mall-tiny/mall-tiny-fabric:0.0.1-SNAPSHOT   "java -jar /mall-tin…"   32 seconds ago      Up 31 seconds       0.0.0.0:8080->8080/tcp                           mall-tiny-fabric
```

- 停止容器使用`docker:stop`命令即可；

```bash
mvn docker:stop
```

- 删除容器使用`docker:remove`命令，是不是很方便！

```bash
mvn docker:remove
```

# 四、Java API上传镜像到Harbor仓库

[**Lesise**](javascript:void(0);):[docker---java上传镜像到Harbor仓库](https://www.it610.com/article/1304427940234170368.htm)



SpringBoot项目上传镜像到Harbor

## 4.1 pom.xml添加依赖

```xml
<!--java操作docker -->
<dependency>
    <groupId>com.github.docker-java</groupId>
    <artifactId>docker-java</artifactId>
    <version>3.1.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jersey</artifactId>
</dependency>
```

## 4.2 yml配置文件

```yaml
#docker访问路径
docker:
  url: tcp://ip:2375
#私服仓库配置
harbor:
  login_address: Harbor登录地址
  username: xx
  password: xxx
```

## 4.3 Java工具类

```java
import com.github.dockerjava.api.DockerClient;
import com.github.dockerjava.api.model.AuthConfig;
import com.github.dockerjava.api.model.Image;
import com.github.dockerjava.api.model.PushResponseItem;
import com.github.dockerjava.core.DockerClientBuilder;
import com.github.dockerjava.core.command.PushImageResultCallback;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.util.List;

/**
 * docker工具类
 */
@Component
public class DockerUtils {
     

    //docker服务器
    @Value("${docker.url}")
    private  String DOCKER_URL ;

    //Harbor登录用户名
    @Value("${harbor.username}")
    private   String HARBOR_USERNAME  ;

    //Harbor登录密码
    @Value("${harbor.password}")
    private   String HARBOR_PASSWORD  ;

    //Harbor的登录地址
    @Value("${harbor.login_address}")
    private   String HARBOR_LOGIN_ADDRESS ;


    /**
     * 获取docker链接
     *
     * @return
     */
    public  DockerClient getDockerClient() {
        DockerClient dockerClient = DockerClientBuilder.getInstance(DOCKER_URL).build();
        return dockerClient;
    }


    /**
     * 参考之前文章《docker---镜像的加载》，镜像如何保存到本地
     * 上传镜像
     * @param file  镜像文件
     * @param imageName
     * @throws Exception
     */
    public  void uploadImage(File file, String imageName) throws Exception {
     
        DockerClient dockerClient = getDockerClient();
        InputStream inputStream = new FileInputStream(file);
        //Harbor登录信息
        AuthConfig autoConfig = new AuthConfig().withRegistryAddress(HARBOR_LOGIN_ADDRESS).withUsername(HARBOR_USERNAME).withPassword(HARBOR_PASSWORD);
        //加载镜像
        dockerClient.loadImageCmd(inputStream).exec();
        //获取加载镜像的名称
        String uploadImageName = "";
        String imageFile = file.getName().substring(0, file.getName().lastIndexOf("."));
        String imageId = imageFile.substring(imageFile.lastIndexOf("_") + 1);
        List<Image> list = dockerClient.listImagesCmd().exec();
        for(Image image : list){
     
            if(image.getId().contains(imageId)){
     
                uploadImageName= image.getRepoTags()[0] ;
            }
        }
        //镜像打tag
        dockerClient.tagImageCmd(uploadImageName, imageName, imageName.split(":")[1]).exec();
        //push至镜像仓库
        PushImageResultCallback pushImageResultCallback = new PushImageResultCallback() {
     
            @Override
            public void onNext(PushResponseItem item) {
                super.onNext(item);
            }
            @Override
            public void onComplete() {
                super.onComplete();
            }
        };
        dockerClient.pushImageCmd(imageName).withAuthConfig(autoConfig).exec(pushImageResultCallback).awaitSuccess();
        //push成功后，删除本地加载的镜像
        dockerClient.removeImageCmd(imageName).exec();
        dockerClient.removeImageCmd(uploadImageName).exec();
        //关闭文件流
        if (inputStream != null) {
            inputStream.close();
        }
    }
}
```

## 4.4 API测试

```java
@RestController
public class HarborController {
     

    @Autowired
    private DockerUtils dockerUtils;
    
    @PostMapping("/uploadImages")
    public ResultObject uploadImages(String projectName, String imageName, String tag, String filePath) throws Exception {
        String imageNames = HOST_HOST + "/" + projectName + "/" + imageName + ":" + tag;
        dockerUtils.uploadImage(new File(filePath), imageNames);
        return ResultObject.ok();
    }
}
```

