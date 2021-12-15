# gradle配置及依赖方式说明

1. setting.gradle

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
    }
}
rootProject.name = 'demo' 
//项目名
```

2. build.gradle

```groovy
plugins {
    id 'org.springframework.boot' version '2.1.6.RELEASE'
    id 'java'
}

apply plugin: 'io.spring.dependency-management'  //应用的插件

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {  //远程仓库，根据先后顺序，决定优先级
	maven { url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    mavenCentral() 
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

3. 依赖版本号处理

```groovy
compile ‘com.google.code.gson:gson:2.8.0’ 
```

4. 在Gradle中可以不指定版本号，比如：

```groovy
compile ‘com.google.code.gson:gson:2.+’ 引入gson 大版本为2的包 
compile ‘com.google.code.gson:gson:latest.release’引入gson 最新的包
```

5. 统一管理版本号

```groovy
def dpc = rootProject.ext.testVersion
ext{
    //dependencies
    testVersion ='xx.xx.xx'
}

//使用
compile test dpc
```