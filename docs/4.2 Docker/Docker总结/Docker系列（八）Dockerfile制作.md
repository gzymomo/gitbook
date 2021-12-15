## DockerFile解析

### 是什么

#### 概念

- DockerFile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

#### 构建三步骤

- 编写Dockerfile文件
- docker build
- docker run

#### Dockerfile示例

- 以centos为例，[官方地址](https://github.com/CentOS/sig-cloud-instance-images/blob/7c2e214edced0b2f22e663ab4175a80fc93acaa9/docker/Dockerfile)![image-20201010135256455](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/608f9829852f457484aa622e9ea71a10~tplv-k3u1fbpfcp-zoom-1.image)

### DockerFile构建过程解析

#### Dockerfile内容基础知识

- 1：每条保留字指令都必须为大写且后面要跟随至少一个参数
- 2：指令按照从上到下，顺序执行
- 3：# 表示注释
- 4：每条指令都会创建一个新的镜像层，并对镜像进行提交

#### Docker执行Dockerfile的大致流程

- 1：docker从基础镜像运行一个容器
- 2：执行一条指令并对容器做出修改
- 3：执行类似docker commit的操作提交一个新的镜像层
- 4：docker再基于刚提交的镜像运行一个容器
- 5：执行dockerfile中的下一条指令知道所有指令都执行完成

#### 小总结

- 从应用软件的角度来看，Dockerfile 、Docker镜像与Docker容器分别代表软件的三个不同阶段

  - Dockerfile是软件的原材料
  - Docker镜像是软件的交付品
  - Docker容器则可以认为是软件的运行态

- Dockerfile面向开发，Docker镜像称为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石

  ![image-20201010144549125](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7debac0465a946c79c52ff82b665ca3b~tplv-k3u1fbpfcp-zoom-1.image)

  - 1：Dockerfile,需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程（当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制）等等；
  - 2：Docker镜像，在用Dockerfile定义一个文件之后，docker builder时会产生一个Docker镜像，当运行Docker镜像时，会真正开始提供服务；
  - 3：Docker容器，容器是直接提供服务的

### DockerFile体系结构（保留字指令）

- FROM ：基础镜像，当前新镜像是基于哪个镜像的
- MAINTAINER：镜像维护者的姓名和邮箱地址
- RUN：容器构建时需要运行的命令
- EXPOSE：当前容器对外暴露的端口号
- WORKDIR：指定在创建容器后，终端默认登录进来的工作目录，一个落脚点
- ENV：用来在构建镜像过程中设置环境变量
  - ENV MY_PATH /usr/mytest 这个环境变量可以在后续的任务RUN指令中使用，这就如同在命令面前制定了环境变量前缀一样；也可以在其它指令中直接使用这些环境变量
  - 比如：WORKDIR $MY_PATH
- ADD：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
- COPY：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件或目录复制到新的一层的镜像内的<目标路径>位置
  - COPY src dest
  - COPY ["src","dest"]
- VOLUME：容器数据卷，用于数据保存和持久化工作
- CMD：
  - 指定一个容器启动时要运行的命令![image-20201010161345112](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b1100a314b3342ef8c9a5b2dc7fcae19~tplv-k3u1fbpfcp-zoom-1.image)
  - Dockerfile中可以有多个CMD命令，但只有最后一个生效，CMD会被docker run 之后的参数替换
- ENTRYPOINT：
  - 指定一个容器启动时要运行的命令
  - ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数
- ONBUILD：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

#### 小总结

- ![image-20201010161855123](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2101844423c744a8882d0317faacf1cc~tplv-k3u1fbpfcp-zoom-1.image)

### 案例

#### Base镜像（scratch）

- Docker Hub 中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的

#### 自定义镜像mycentos

- Docker Hub默认的centos镜像![image-20201010163103164](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/216cd5f4631a4d10bfeaeecf0626b716~tplv-k3u1fbpfcp-zoom-1.image)

- 自定义mycentos目的使我们自己的镜像具备如下：

  - 登录后的默认路径
  - 支持vim编辑器
  - 支持查看网络配置 ifconfig

- 第一步：编写Dockerfile

  ```shell
  FROM centos
  MAINTAINER touchair<touchair@163.com>
  ENV MYPATH /usr/local
  WORKDIR $MYPATH
  
  RUN yum -y install vim
  RUN yum -y install net-tools
  
  EXPOSE 80
  
  CMD echo $MYPATH
  CMD echo "success---------------OK"
  CMD /bin/bash
  复制代码
  ```

- 第二步：构建

  ```shell
  # docker build -t 新镜像的名字：TAG .
   docker build -f /var/mydocker/Dockerfile2 -t mycentos:1.4 .
  复制代码
  ```

  ![image-20201010164723079](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04fcb127cacd416b9ece33f245dd30ab~tplv-k3u1fbpfcp-zoom-1.image)

  ![image-20201010164754838](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e39a6a1bea34f3fb279a52750f6e7bd~tplv-k3u1fbpfcp-zoom-1.image)

- 第三步：运行自定义的镜像

  ```shell
  docker run -it mycentos:1.4
  复制代码
  ```

  ![image-20201010165022279](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2746995a09784234b947266a5e3a6e3a~tplv-k3u1fbpfcp-zoom-1.image)

- 列出镜像的变更历史

  ```shell
  #docker history 镜像名
  docker history mycentos:1.4
  复制代码
  ```

  - ![image-20201010165254684](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ebd138fa67a44959813a5626eb513f5~tplv-k3u1fbpfcp-zoom-1.image)

  #### 

#### CMD/ENTRYPOINT（区别） 镜像案例

- **CMD：命令替换/覆盖   ENTRYPOINT：命令追加**
- 都是指定一个容器启动时要运行的命令
- CMD
  - Dockerfile中可以有多个CMD命令，但只要最后一个生效，CMD会被 docker run之后的参数替换
  - Case：tomcat的讲解演示
    - CMD ["catalina.sh","run"]
    - CMD ls -l
    - 结果 ls -l **命令会覆盖** Catalina.sh run
- ENTRYPOINT
  - docker run 之后的参数会被当做参数传递给ENTRYPOINT，之后形成新的命令组合

#### 自定义镜像Tomcat9

- 1：在 /var/mydocker/目录下，创建 tomcat9 目录

- 2：将 jdk 和 tomcat 安装的压缩包拷贝进上一步目录

- 3：在 tomcat9 目录下新建 Dockerfile 文件

  ```shell
  #自定义的tomcat9 Dockerfile
  
  FROM         centos
  MAINTAINER    touchair<touchair@163.com>
  #把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
  COPY c.txt /usr/local/cincontainer.txt
  #把java与tomcat添加到容器中
  ADD jdk-8u161-linux-x64.tar.gz /usr/local/
  ADD apache-tomcat-9.0.38.tar.gz /usr/local/
  #安装vim编辑器
  RUN yum -y install vim
  #设置工作访问时候的WORKDIR路径，登录落脚点
  ENV MYPATH /usr/local
  WORKDIR $MYPATH
  #配置java与tomcat环境变量
  ENV JAVA_HOME /usr/local/jdk1.8.0_161
  ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.38
  ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.38
  ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
  #容器运行时监听的端口
  EXPOSE  8080
  #启动时运行tomcat
  # ENTRYPOINT ["/usr/local/apache-tomcat-9.0.38/bin/startup.sh" ]
  # CMD ["/usr/local/apache-tomcat-9.0.38/bin/catalina.sh","run"]
  CMD /usr/local/apache-tomcat-9.0.38/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.38/bin/logs/catalina.out
  复制代码
  ```

- 4：构建

  - tomcat9 目录下

  - ```shell
    docker build -t touchairtomcat9 .
    复制代码
    ```

    - ![image-20201012104347596](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f29f94f8fc224b0895398b958e98e5a5~tplv-k3u1fbpfcp-zoom-1.image)

      ![image-20201012104428633](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a22793a0e7f4291bdd701301f619eba~tplv-k3u1fbpfcp-zoom-1.image)

- 5：run

  - ```shell
    docker images touchairtomcat9
    复制代码
    ```

    ![image-20201012104544205](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9c10ff527c2c46d39789564e9243da8b~tplv-k3u1fbpfcp-zoom-1.image)

    

  - ```shell
    # 指定容器端口 名称 并挂载到宿主机
    docker run -d -p 9080:8080 --name touchairtt9 -v /var/mydocker/tomcat9/test:/usr/local/apache-tomcat-9.0.38/webapps/test -v /var/mydocker/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.38/logs --privileged=true touchairtomcat9
    复制代码
    ```

    - ![image-20201012114420840](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d980a62a0f02465093027618e18a2878~tplv-k3u1fbpfcp-zoom-1.image)

- 6：验证

  - 浏览器访问：192.168.83.133:9080

    ![image-20201012131947021](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94e376b98aa74bd9822c3c5a5e4a01a9~tplv-k3u1fbpfcp-zoom-1.image)

  - 进入容器查看：

    - ![image-20201012132137482](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df1933073fb741bbb4d64ac12de2693d~tplv-k3u1fbpfcp-zoom-1.image)

- 7：结合前述的容器卷将测试的web服务test发布

  - 新建test工程

    - WEB-INF 目录下 web.xml

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://java.sun.com/xml/ns/javaee"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
        id="WebApp_ID" version="2.5">
        
        <display-name>test</display-name>
       
      </web-app>
      复制代码
      ```

    - a.jsp

      ```jsp
      <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
      <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
      <html>
        <head>
          <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
          <title>Insert title here</title>
        </head>
        <body>
          -----------welcome------------
          <%="i am in docker tomcat self "%>
          <br>
          <br>
          <% System.out.println("=============docker tomcat self");%>
        </body>
      </html>
      复制代码
      ```

    ![image-20201012133149075](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/023dd8d8f55f491ea193cd9061f3db5b~tplv-k3u1fbpfcp-zoom-1.image)

  - 浏览器访问：192.168.83.133:9080/test/jsp

    ![image-20201012133444807](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/02ec87882f644e6992950654a481d2d1~tplv-k3u1fbpfcp-zoom-1.image)

  - 查看容器内的日志打印![image-20201012133905120](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2f41721cefb48e1999158fd29d438b6~tplv-k3u1fbpfcp-zoom-1.image)

