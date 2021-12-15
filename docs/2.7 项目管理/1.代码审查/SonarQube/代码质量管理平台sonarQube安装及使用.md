- [SonarQube手册](https://www.cnblogs.com/ioufev/articles/12696498.html)
-  [Jenkins持续集成git、gitlab、sonarqube(7.0)、nexus，自动化部署实战，附安装包](https://www.cnblogs.com/chenyanbin/p/qq543210188.html)             



## SonarQube如何保障代码质量

它是从 Architecture Design, Coding Rule, Potential Bugs, Duplications, Comments, Unit Tests,Complexity 7个维度检查代码质量的。
![img](https://img2020.cnblogs.com/blog/861554/202012/861554-20201228104352855-1626456173.png)



### 前置条件

- mysql 5.6 | 5.7
- jdk1.8

 我使用的是7.0，版本要求：[点我直达](https://docs.sonarqube.org/7.0/Requirements.html)[
](https://docs.sonarqube.org/7.9/requirements/requirements/)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200916222747488-145410555.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200916222747488-145410555.gif)

下载地址：[点我直达](https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.0.zip)

### 官网

[点我直达](https://www.sonarqube.org/)

### 安装

```bash
1、依赖：yum install unzip -y
2、解压：unzip sonarqube-7.0.zip 
3、移动：mv sonarqube-7.0 /usr/local/
4、切换： cd /usr/local/
5、登录mysql：mysql -u root -p
6、创建库：CREATE DATABASE sonar DEFAULT CHARACTER SET utf8;
7、退出mysql：exit
8、进入sonarqube：cd sonarqube-7.0/conf/
```

### 修改配置

```yaml
vim sonar.properties
主要配置以下内容
sonar.jdbc.username=root
sonar.jdbc.password=root
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.web.host=0.0.0.0
sonar.web.context=/sonar
sonar.web.port=9000
```

 

### 启动

```bash
useradd sonar
chown -R sonar:sonar /usr/local/sonarqube-7.0
su sonar
cd /usr/local/sonarqube-7.0/bin/linux-x86-64/
./sonar.sh start
```

### 访问

　　**记得开放防火墙端口：9000，账号/密码：admin/admin**

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919001951915-1087621668.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919001951915-1087621668.gif)

### 汉化

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919003452658-1423757856.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919003452658-1423757856.gif)

　　安装完，重启服务再次打开网页即可

```
sonar7.0的中文包，网页在线安装不上，我是去github上下载，手动安装的，按照以下几步即可，下面我也会提供，我直接下载的是jar包，源码包还得编码(因为我懒)
1. 在https://github.com/SonarCommunity/sonar-l10n-zh，下载汉化包源码；
2. 本地打包，cmd里面，在解压包里面运行： mvn install 
3. 将打好的jar包，放到： sonarqube/extensions/plugins  目录先；
4. 重启sonar，即可
```

　　具体操作如下，github地址：[点我直达](https://github.com/SonarQubeCommunity/sonar-l10n-zh)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919012354888-474918393.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919012354888-474918393.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919012430004-1732477344.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919012430004-1732477344.gif)

### 使用

```xml
settings.xml
============================
<?xml version="1.0" encoding="UTF-8"?>
<settings
    xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <pluginGroups></pluginGroups>
    <proxies></proxies>
    <servers></servers>
    <mirrors>
        <!--maven代理开始-->
        <mirror>
            <id>huaweicloud</id>
            <mirrorOf>*,!HuaweiCloudSDK</mirrorOf>
            <url>https://mirrors.huaweicloud.com/repository/maven/</url>
        </mirror>
        <mirror>
            <id>aliyun</id>
            <name>aliyun Maven</name>
            <mirrorOf>*,!HuaweiCloudSDK</mirrorOf>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </mirror>
        <!--maven代理结束-->
    </mirrors>
    <profiles>
        <!--sonar配置开始-->
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.jdbc.url>jdbc:mysql://192.168.199.199:3306/sonar?useUnicode=true&amp;characterEncoding=utf8</sonar.jdbc.url>
                <sonar.jdbc.username>root</sonar.jdbc.username>
                <sonar.jdbc.password>root</sonar.jdbc.password>
                <sonar.host.url>http://192.168.199.199:9000/sonar</sonar.host.url>
            </properties>
        </profile>
        <!--sonar配置结束-->
    </profiles>
</settings>
git init
mvn clean install
mvn sonar:sonar
```

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919152803604-53961896.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919152803604-53961896.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919152838847-516007499.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919152838847-516007499.gif)

```xml
忽略某一条规则，pom.xml下面追加

<properties>
    <sonar.exclusions>
        src/main/java/com/.../domain/model/**/*,
        src/main/java/com/.../exchange/**/*
</sonar.exclusions>
</properties>
```



# Jenkins整合Sonar

刚开始的时候sonarqube里面没有项目随着代码的重新发布，会将项目也提交到sonarqube中

```
#projectKey项目的唯一标识，不能重复
sonar.projectKey=yb
sonar.projectName=springboot-test
sonar.projectVersion=1.0
sonar.sourceEncoding=UTF-8
sonar.modules=java-module
# Java module 
java-module.sonar.projectName=test 
java-module.sonar.language=java 
# .表示projectBaseDir指定的目录 
java-module.sonar.sources=src 
java-module.sonar.projectBaseDir=. 
java-module.sonar.java.binaries=target/
```

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919230059479-775588360.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919230059479-775588360.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919231127604-429068352.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919231127604-429068352.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919231625508-118511158.gif)](https://img2020.cnblogs.com/blog/1504448/202009/1504448-20200919231625508-118511158.gif)

