## 1、maven依赖

本开发教程适用于sonarqube5.x、6.x。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>wxtx</groupId>
    <artifactId>TestSonar</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>sonar-plugin</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <sonar.version>6.3</sonar.version>
        <!-- this 6.3 is only required to be compliant with SonarLint and it is 
            required even if you just want to be compliant with SonarQube 5.6 -->
        <java.plugin.version>4.7.1.9272</java.plugin.version>
        <sslr.version>1.21</sslr.version>
        <gson.version>2.6.2</gson.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.sonarsource.sonarqube</groupId>
            <artifactId>sonar-plugin-api</artifactId>
            <version>${sonar.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.sonarsource.java</groupId>
            <artifactId>sonar-java-plugin</artifactId>
            <type>sonar-plugin</type>
            <version>${java.plugin.version}</version>
        </dependency>

        <dependency>
            <groupId>org.sonarsource.java</groupId>
            <artifactId>java-frontend</artifactId>
            <version>${java.plugin.version}</version>
        </dependency>

        <dependency>
            <groupId>org.sonarsource.sslr-squid-bridge</groupId>
            <artifactId>sslr-squid-bridge</artifactId>
            <version>2.6.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.codehaus.sonar.sslr</groupId>
                    <artifactId>sslr-core</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.codehaus.sonar</groupId>
                    <artifactId>sonar-plugin-api</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.codehaus.sonar.sslr</groupId>
                    <artifactId>sslr-xpath</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>jcl-over-slf4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.sonarsource.java</groupId>
            <artifactId>java-checks-testkit</artifactId>
            <version>${java.plugin.version}</version>
        </dependency>

        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>${gson.version}</version>
        </dependency>


        <dependency>
            <groupId>org.sonarsource.sslr</groupId>
            <artifactId>sslr-testing-harness</artifactId>
            <version>${sslr.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.6.1</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>0.9.30</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.sonarsource.sonar-packaging-maven-plugin</groupId>
                <artifactId>sonar-packaging-maven-plugin</artifactId>
                <version>1.17</version>
                <extensions>true</extensions>
                <configuration>
                    <pluginDescription>test</pluginDescription>
                    <pluginKey>java-custom</pluginKey>
                    <pluginName>Java Custom Rules</pluginName>
                    <pluginClass>wxtx.com.MySonarPlugin</pluginClass>
                    <sonarLintSupported>true</sonarLintSupported>
                    <sonarQubeMinVersion>5.6</sonarQubeMinVersion> <!-- allow to depend on API 6.x but run on LTS -->
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 2、制定规则

```java
/**
 * @author 孙泽明
 * 抽象类命名使用TXAbstract或TXBase开头
 */
@Rule(
    // 规则id
    key = "TXAbstractClassNameCheck",
    // 规则名称
    name = "抽象类命名使用TXAbstract或TXBase开头",
    // 规则介绍
    description = "抽象类命名使用TXAbstract或TXBase开头",
    // 规则标签
    tags = {"wxtx-java"},
    // 规则级别
    priority = Priority.MINOR)
@SqaleSubCharacteristic(RulesDefinition.SubCharacteristics.ARCHITECTURE_CHANGEABILITY)
// 纠正错误所需时间
@SqaleConstantRemediation("10min")
public class TXAbstractClassNameCheck extends BaseTreeVisitor implements JavaFileScanner {
    private JavaFileScannerContext context;
    public void scanFile(JavaFileScannerContext context) {
        this.context = context;
        scan(context.getTree());
    }

    @Override
    public void visitClass(ClassTree tree) {
        if(tree == null || tree.simpleName() == null){
            super.visitClass(tree);
            return;
        }

        String className = tree.simpleName().name();
        boolean isAbstract = tree.symbol().isAbstract();
        if(isAbstract && isNameIll(className)){
            context.reportIssue(this, tree, "抽象类命名使用TXAbstract或TXBase开头");
        }
        super.visitClass(tree);
    }

    private boolean isNameIll(String className){
        return !className.startsWith("TXAbstract") && !className.startsWith("TXBase");
    }
}
```

## 3、注册规则

3.1、自定义插件入口

```java
public class TXSonarPlugin implements Plugin  {
    public void define(Context context) {
        context.addExtension(TXJavaRulesDefinition.class);
        context.addExtension(TXJavaFileCheckRegistrar.class);
    }
}
```

3.2、server extensions
在sonarqube server启动时实例化，实现org.sonar.api.server.rule.RulesDefinition接口

```java
public class TXJavaRulesDefinition implements RulesDefinition {
    public static final String REPOSITORY_KEY = "TXRepo";
    public void define(Context context) {
        NewRepository repository = context.createRepository(REPOSITORY_KEY,Java.KEY);
        repository.setName("Java编码规范");
        AnnotationBasedRulesDefinition.load(repository, "java",TXRulesList.getChecks());
        repository.done();
    }
}
```

3.3、batch extensions
在分析代码的时候实例化，实现org.sonar.plugins.java.api.CheckRegistrar接口

```java
public class TXJavaFileCheckRegistrar implements CheckRegistrar {
     public void register(RegistrarContext registrarContext) {
            registrarContext.registerClassesForRepository(TXJavaRulesDefinition.REPOSITORY_KEY,
                    Arrays.asList(checkClasses()), Arrays.asList(testCheckClasses()));
        }

        public static Class<? extends JavaCheck>[] checkClasses() {
            new Class[] {TXAbstractClassNameCheck.class};
        }
        @SuppressWarnings("unchecked")
        public static Class<? extends JavaCheck>[] testCheckClasses() {
            return new Class[] {};
        }
}1234567891011121314
public class TXRulesList {

    private TXRulesList() {
    }
    public static List<Class> getChecks() {
        return ImmutableList.<Class>builder().addAll(getJavaChecks()).addAll(getJavaTestChecks()).build();
    }
    public static List<Class<? extends JavaCheck>> getJavaChecks() {
        return ImmutableList.<Class<? extends JavaCheck>>builder()
            .add(TXAbstractClassNameCheck.class).build();
    }
    public static List<Class<? extends JavaCheck>> getJavaTestChecks() {
        return ImmutableList.<Class<? extends JavaCheck>>builder().build();
    }
}
```

## 4、输出

maven执行命令mvn clean install输出jar包，其中修改MANIFEST.MF的Plugin-Dependencies，例如：

```
Manifest-Version: 1.0
Plugin-Dependencies: META-INF/lib/*
Plugin-Description: test
Plugin-BuildDate: 2017-11-08T12:12:27+0800
Archiver-Version: Plexus Archiver
SonarLint-Supported: true
Built-By: Administrator
Plugin-License: 
Plugin-Version: 0.0.1-SNAPSHOT
Sonar-Version: 5.6
Plugin-Developers: 
Plugin-ChildFirstClassLoader: false
Plugin-Key: javacustom
Plugin-Class: wxtx.com.MySonarPlugin
Created-By: Apache Maven 3.3.9
Build-Jdk: 1.8.0_144
Plugin-Name: Java Custom Rules
```

并将依赖的jar包复制进META-INF/lib/目录下(例如guava、sonar-plugin-api)

## 5、测试

5.1、将自定义插件jar放到sonar下的extensions/plugins路径下
5.2、启动SonarQube
5.3、激活规则：登录admin账号后，点击activate按钮激活规则
![这里写图片描述](https://img-blog.csdn.net/20171003100522092)