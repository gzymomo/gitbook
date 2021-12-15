- [SpringBoot 之自动化部署至服务器](http://blog.bhusk.com/articles/2020/07/07/1594111798627)
- [maven wagon-maven-plugin 实现远程部署](https://blog.csdn.net/xiaojin21cen/article/details/103901325)
- [使用wagon-maven-plugin部署Java项目到远程服务器](https://www.cnblogs.com/codechangeword/articles/11100301.html)



# 一、wagon-maven-plugin概述

Maven 插件 wagon-maven-plugin 来自动完成部署。

开发springboot 项目，（测试）部署项目时，要先打包成 jar 文件，再 SCP 上传的linux 服务器上，用shell 命令停止原有的服务，删除原有的代码，再运行刚刚上传的jar …，这是一系列的重复繁琐操作。而 `wagon-maven-plugin` 可以简化这些繁琐操作。



# 二、代码实例

## 2.1 配置 Linux 服务器用户名和密码

```xml
<properties>
        <!--服务器项目运行的地址-->
        <service-path>/project/</service-path>
        <pack-name>${project.artifactId}-${project.version}.jar</pack-name>
        <!--ssh登录服务器的ip和端口 端口一般默认22-->
        <remote-addr>ip:port</remote-addr>
        <remote-username>服务器用户名</remote-username>
        <remote-passwd>服务器密码</remote-passwd>
</properties>
```

## 2.2 maven 依赖 jar

```xml
<!-- https://mvnrepository.com/artifact/org.codehaus.mojo/wagon-maven-plugin -->
<dependency>
   <groupId>org.codehaus.mojo</groupId>
   <artifactId>wagon-maven-plugin</artifactId>
   <version>2.0.0</version>
</dependency>
```

## 2.3 更改pom.xml的build

```xml
<build>
<extensions>
    <extension>
        <groupId>org.apache.maven.wagon</groupId>
        <artifactId>wagon-ssh</artifactId>
        <version>2.8</version>
    </extension>
</extensions>
<plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>wagon-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
            <fromFile>target/${pack-name}</fromFile>
            <url><![CDATA[scp://${remote-username}:${remote-passwd}@${remote-addr}${service-path}]]></url>
            <!-- 在服务器执行的命令集合 -->
            <commands>
                <!-- 杀死原来的jar进程 -->
                <command>pkill -f ${pack-name}</command>
                <!-- 重新启动jar进程，程序的输出结果写到log文件中 -->
                <command><![CDATA[nohup java -jar ${service-path}/${pack-name} --spring.profiles.active=dev > ${service-path}/bd.log 2>&1 & ]]></command>
                <command><![CDATA[netstat -nptl]]></command>
                <command><![CDATA[ps -ef | grep java | grep -v grep]]></command>
            </commands>
            <!-- 显示运行命令的输出结果 -->
            <displayCommandOutputs>true</displayCommandOutputs>
        </configuration>

    </plugin>
</plugins>
</build>
```

## 2.4 执行命令

在 pom.xml 文件相同目录下终端执行以下 mvn 命令

```none
mvn clean package wagon:upload-single wagon:sshexec
```

# 三、2、配置

## 3.1 wagon-maven-plugin 的基础配置

```xml
<build>
	<finalName>assets</finalName>
	<extensions>
		<extension>
			<groupId>org.apache.maven.wagon</groupId>
			<artifactId>wagon-ssh</artifactId>
			<version>2.10</version>
		</extension>
	</extensions>

	<plugins>
		<plugin>
			<groupId>org.codehaus.mojo</groupId>
			<artifactId>wagon-maven-plugin</artifactId>
			<version>1.0</version>			
			<configuration>				
				<!-- 需要部署的文件 -->
				<fromFile>target/assets.jar</fromFile>
				<!-- 部署目录  用户：密码@ip+部署地址：端口 -->
				<url><![CDATA[ scp://root:密码@192.168.1.100:28/usr/tomcat_assets/ ]]></url>
				
				<!--shell 执行脚本 -->
				<commands>
					<!-- 停止服务-->
					<command>sh /usr/tomcat_assets/stop.sh</command>
					<!-- 启动服务 -->
					<command>sh /usr/tomcat_assets/start.sh</command>
				</commands>
				<displayCommandOutputs>true</displayCommandOutputs>
			</configuration>
		</plugin>
	</plugins>
</build>
```

**使用步骤：**

1. 先打包，执行 `mvn clean package`。我们熟知的maven 打包命令，这里对 springboot 项目进行打包 ；
2. 再上传打包的文件，执行：`wagon:upload-single` ；
3. 执行shell 命令（脚本），主要是停止、删除原来的服务，启动新的服务，执行：`wagon:shexec`。

如果想上传文件、执行shell 命令 一起执行，将两个命令合并到一块，即： `mvn wagon:upload-single wagon:sshexec` 。（注意： 命令要有先后顺序）

如果想打包、上传文件、执行shell 命令 三者一起执行，则执行： `mvn clean package wagon:upload-single wagon:sshexec` 。

## 3.2 将 `wagon:upload-single wagon:sshexec` 合并到 `package` 命令中（优化、可选步骤）

如果觉得上面的命令复杂，每次都是 打包、上传文件、执行shell 脚本三者一起执行，可以将 上传文件、执行shell 脚本 合并到 打包命令中，即将 `wagon:upload-single wagon:sshexec` 合并到 `package` 命令中，当执行打包命令 `package` 时，就会执行 上传文件、 执行shell 命令。

```xml
<build>
	<finalName>assets</finalName>
	<extensions>
		<extension>
			<groupId>org.apache.maven.wagon</groupId>
			<artifactId>wagon-ssh</artifactId>
			<version>2.10</version>
		</extension>
	</extensions>

	<plugins>
		<plugin>
			<groupId>org.codehaus.mojo</groupId>
			<artifactId>wagon-maven-plugin</artifactId>
			<version>1.0</version>

			<executions>
				<execution>				
					<id>upload-deploy</id>					
					<!-- 运行package打包的同时运行upload-single和sshexec -->
					<phase>package</phase>
					<goals>
						<goal>upload-single</goal>
						<goal>sshexec</goal>
					</goals>
					<configuration>						
						<!-- 需要部署的文件 -->
						<fromFile>target/assets.jar</fromFile>
						<!-- 部署目录  用户：密码@ip+部署地址：端口 -->
						<url><![CDATA[ scp://root:密码@192.168.1.100:28/usr/tomcat_assets/ ]]> </url>

						<!--shell 执行脚本 -->
						<commands>
							<!-- 停止服务-->
							<command>sh /usr/tomcat_assets/stop.sh</command>
							<!-- 启动服务 -->
							<command>sh /usr/tomcat_assets/start.sh</command>
						</commands>
						<displayCommandOutputs>true</displayCommandOutputs>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>     
```

**使用方法：** 执行 `mvn clean package` 即可。

此处的 `mvn clean package` 相当于执行 `mvn clean package wagon:upload-single wagon:sshexec` 。

**说明：**

此步骤仅仅优化打包、上传、 执行shell 命令的执行，不是必须的配置。

## 3.3 密码 在 maven 的 settings.xml （优化配置）

文件上传会涉及到密码，密码可以放到pom.xml （上面已演示）， 也可以放到 maven 的 settings.xml 。

### 3.3.1 在 settings.xml 中配置密码

```xml
<servers>
	<server>  
		<id>assets</id>  
		<username>root</username>  
		<password><![CDATA[密码]]></password>  
	</server>
</servers>
```

**注意：** `<id>`的值 要保证唯一。

### 3.3.2 pom.xml 的配置

`<configuration>` 标签增加 `<serverId>`，其值与settings.xml 中 `<server>` 标签中的 `<id>`的值一样。

```xml
<configuration>
	<serverId>assets</serverId>
</configuration>
```

如图所示：
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200401080508550.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9qaW4yMWNlbg==,size_16,color_FFFFFF,t_70)

## 3.4  停止、启动服务相关的脚本

`stop.sh` 停止服务的脚本：

```shell
#!/bin/bash

APP_NAME='assets.jar'

tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`
if [ ${tpid} ];then
    echo 'Stop Process...'
    kill -9 $tpid
fi

sleep 1

tpid=`ps -ef|grep $APP_NAME|grep -v grep|grep -v kill|awk '{print $2}'`

if [ ${tpid} ];then
    echo 'Kill Process!'
    kill -9 $tpid
else
    echo 'Stop Success!'
fi
```

`start.sh` 启动脚本：

```shell
#!/bin/bash

fileDir=/usr/tomcat_assets
fileName=assets.jar

nohup  /usr/java/jdk1.8.0_201/bin/java -jar  ${fileDir}/${fileName} > ${fileDir}/assets.log   2>&1 &

echo $?

echo 'Start Success! '
```

## 3.5 与 wagon-maven-plugin 无关，但是项目可能用的配置（可选配置）

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-compiler-plugin</artifactId>
	<configuration>
		<source>1.8</source>
		<target>1.8</target>
		<encoding>UTF-8</encoding>
	</configuration>
</plugin>
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<!-- 修改相应的SpringBootApplication.java -->
		<mainClass>xxx.yyy.zzz.Application</mainClass>
	</configuration>
	<executions>
		<execution>
			<goals>
				<goal>repackage</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```