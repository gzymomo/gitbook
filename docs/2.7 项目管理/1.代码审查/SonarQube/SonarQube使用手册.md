- [SonarQube](https://www.cnblogs.com/ioufev/tag/SonarQube/)

- [使用SonarQube扫描项目代码检测覆盖率    ](https://www.cnblogs.com/longronglang/p/14226765.html)



# 配置本地 maven 配置文件 conf/settings.xml

添加如下内容：

```xml
  <pluginGroups>
	<!-- SonarQube 插件 -->
	<pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
  </pluginGroups>
  
  <profiles>
	<!-- SonarQube 插件 -->
	<profile>
		<id>sonar</id>
		<activation>
			<activeByDefault>true</activeByDefault>
		</activation>
		<properties>
			<!-- Optional URL to server. Default value is http://localhost:9000 -->
			<sonar.host.url>
			  http://192.168.1.106:9000
			</sonar.host.url>
		</properties>
	</profile>
  </profiles>
```

# 在pom.xml中引入JaCoCo插件

添加如下内容：

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.2</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <!-- change phase from verify to test -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

# 配置扫描数据

## 1、添加文件

在项目下：添加 sonar-project.properties 文件

## 2、复制以下文件内容

```yaml
# 指定SonarQube instance必须是唯一的
sonar.projectKey=springboot_demo
# 设置SonarQube UI显示的名称 
# PS：有人会问这里的名称是否可以是中文名称，我在网上搜索了好多资料都说是不可以的（至少我看到的资
#料都是）后来自己尝试了一下，答案是可以写成中文的，但是要换一种方式，比如你想把项目名称命名为“测
#试”，那么在这里就要写成“\u6d4b\u8bd5”，那么下面这个参数就应该这样写“sonar.projectName= 
#\u6d4b\u8bd5”，说白了就是将中文转成Unicode
sonar.projectName=springboot_demo
sonar.projectVersion=1.0
sonar.language=java
# 指定src和classes文件夹位置，当然也可以是全路径，如果是当前工程根目录下用“.”表示也可以，比如“sonar.sources=.”
sonar.sources=src/main
sonar.test=src/test
sonar.java.binaries=target
# 下面的这两个参数作用是相同的，因为有时我们需要指定某个文件夹或者忽略某个文件夹
# sonar.inclusions=src1/**,src3/**
# sonar.exclusions=src2/**,src4/**
# 源码编码，默认是系统编码
sonar.sourceEncoding=UTF-8
# Set jacoco Configuration
# 指定代码覆盖率工具
sonar.core.codeCoveragePlugin=jacoco
# 指定exec二进制文件存放路径
#sonar.jacoco.reportPaths=[yourPath/]jacoco.exec
#本demo之前设置的exec文件是在工程根目录下的target/coverage-reports下：

sonar.jacoco.reportPaths=target/jacoco.exec
# 以下属性可选择性加，当然也可以不加
sonar.dynamicAnalysis=reuseReports
sonar.jacoco.reportMissing.force.zero=false
```

## 3、运行

在项目根目录下，运行`mvn package`。

[![img](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195537018-1892645442.png)](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195537018-1892645442.png)

当build成功的时候Jacoco的结果就会产生在target/site/jacoco文件夹下。

[![img](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195651617-1798781698.png)](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195651617-1798781698.png)

接着，在项目根目录下运行命令 sonar-scanner，如果看到以下结果证明已经覆盖率已经可以在SonarQube上查阅。

[![img](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195926685-85367944.png)](https://img2020.cnblogs.com/blog/718867/202101/718867-20210103195926685-85367944.png)

# 在SonarQube查看扫描结果