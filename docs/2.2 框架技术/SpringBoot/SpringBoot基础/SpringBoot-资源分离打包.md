# SpringBoot资源分离打包

# 前言

打包Spring Boot应用时候，为了方便修改配置文件或者不进行重复打依赖包节省资源和部署时间，经常会使用资源分离打包的方式，下面分享一个资源分离打包 Maven 配置

# 配置

项目 Maven POM 文件配置如下：



```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>

            <!-- 排除文件配置 -->
            <!-- <excludes> -->
            <!-- <exclude>*.**</exclude> -->
            <!-- <exclude>*/**.xml</exclude> -->
            <!-- </excludes> -->

            <!-- 包含文件配置，现在只打包 com 文件夹 -->
            <includes>
                <include>
                    **/com/**
                </include>
            </includes>

            <archive>
                <manifest>
                    <!-- 配置加入依赖包 -->
                    <addClasspath>true</addClasspath>
                    <classpathPrefix>lib/</classpathPrefix>
                    <useUniqueVersions>false</useUniqueVersions>
                    <!-- Spring Boot 启动类(自行修改) -->
                    <mainClass>com.sevenbillion.Application</mainClass>
                </manifest>
                <manifestEntries>
                    <!-- 外部资源路径加入 manifest.mf 的 Class-Path -->
                    <Class-Path>resources/</Class-Path>
                </manifestEntries>
            </archive>
            <!-- jar 输出目录 -->
            <outputDirectory>${project.build.directory}/pack/</outputDirectory>
        </configuration>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <!-- 复制依赖 -->
        <executions>
            <execution>
                <id>copy-dependencies</id>
                <phase>package</phase>
                <goals>
                    <goal>copy-dependencies</goal>
                </goals>
                <configuration>
                    <!-- 依赖包 输出目录 -->
                    <outputDirectory>${project.build.directory}/pack/lib</outputDirectory>
                </configuration>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <artifactId>maven-resources-plugin</artifactId>
        <!-- 复制资源 -->
        <executions>
            <execution>
                <id>copy-resources</id>
                <phase>package</phase>
                <goals>
                    <goal>copy-resources</goal>
                </goals>
                <configuration>
                    <resources>
                        <resource>
                            <directory>src/main/resources</directory>
                        </resource>
                    </resources>
                    <!-- 资源文件 输出目录 -->
                    <outputDirectory>${project.build.directory}/pack/resources</outputDirectory>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
```

完成上面的配置以后，直接`mvn package`
 打包完成以后，效果图如下:

![img](https:////upload-images.jianshu.io/upload_images/7250727-ed61fa9bc713ef69.png?imageMogr2/auto-orient/strip|imageView2/2/w/342/format/webp)



![img](https:////upload-images.jianshu.io/upload_images/7250727-9d13e6c270217cdd.png?imageMogr2/auto-orient/strip|imageView2/2/w/646/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/7250727-4b531e8375bcc282.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

`lib`存放的是相关依赖包

`resources`存放的是相关配置文件和资源文件

`***.jar`就是只打包源文件编译后的class文件

然后直接在当前目录执行 `java -jar ***.jar`启动应用就ok了。



作者：hdfg159
链接：https://www.jianshu.com/p/f8abd76b6e7c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。