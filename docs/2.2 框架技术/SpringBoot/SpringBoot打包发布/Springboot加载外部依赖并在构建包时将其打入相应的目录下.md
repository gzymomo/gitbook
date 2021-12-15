- [springboot加载外部依赖并在构建包时将其打入相应的目录下](https://www.cnblogs.com/kingsonfu/p/12693271.html)



当我们在maven仓库中无法找到需要的依赖时，需要将相应的依赖jar包下载下来放到项目的某个目录下，然后通过配置文件配置将其引入项目中使用。如下引入sigar依赖:[具体下载地址](http://sourceforge.net/projects/sigar/files/latest/download?source=files)

1、依赖具体目录如下：

![img](https://img2020.cnblogs.com/blog/761230/202004/761230-20200413185609816-656857275.png)

2、pom.xml配置：

```xml
<!-- 此处使用外部引用 -->
<dependency>
    <groupId>org.hyperic</groupId>
    <artifactId>sigar</artifactId>
    <scope>system</scope>
    <version>1.6.4</version>	
    <systemPath>${project.basedir}/libs/sigar.jar</systemPath>
</dependency>
```

maven构建打包插件配置将Scope为system的依赖加入构建包中：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- 将SystemScope配置的依赖打包到构建包中，或者通过配置resource来处理，见下resources配置 -->
        <includeSystemScope>true</includeSystemScope>
    </configuration>
</plugin>
```

或者配置resources进行配置将外部依赖jar放入BOOT-INF/lib/目录下

```xml
<resources>
    <!-- 将当前libs目录下的所有jar包含到BOOT-INF/lib/目录下 -->
    <resource>
        <directory>libs</directory>
        <targetPath>BOOT-INF/lib/</targetPath>
        <includes>
            <include>**/*.jar</include>
        </includes>
    </resource>
    <!-- 将src/main/resources目录下的文件包含到BOOT-INF/classes/目录下 -->
    <resource>
        <directory>src/main/resources</directory>
        <targetPath>BOOT-INF/classes/</targetPath>
    </resource>
</resources>
```

这样在构建好的包中就能将sigar.jar打入构建包中：

![img](https://img2020.cnblogs.com/blog/761230/202004/761230-20200413190236084-750291690.png)

 　这个场景对于少数本地jar依赖添加处理还行，如果lib下有很多jar需要添加处理，显然不是很方便，此时可以在插件maven-compiler-plugin中进行配置编译参数<compilerArguments>，添加extdirs将jar包相对路径添加到配置中，如下： 

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>UTF-8</encoding>
        <compilerArguments>
            <verbose />
            <!-- 将jdk的依赖jar打入项目中，这样项目中使用的jdk的依赖就尅正常使用 -->
            <bootclasspath>${java.home}/lib/rt.jar;${java.home}/lib/jce.jar;${java.home}/lib/jsse.jar</bootclasspath>
            <extdirs>${project.basedir}/libs</extdirs>
        </compilerArguments>
    </configuration>
</plugin>
```

