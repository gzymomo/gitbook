# 1、Gradle

Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建开源工具，它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置。



Gradle，这是一个基于 JVM 的富有突破性构建工具。Gradle 正迅速成为许多开源项目和前沿企业构建系统的选择，同时也在挑战遗留的自动化构建项目。

它为您提供了:

- 一个像 ant 一样，通用的灵活的构建工具
- 一种可切换的，像 maven 一样的基于约定约定优于配置的构建框架
- 强大的多工程构建支持
- 强大的依赖管理(基于 ApacheIvy)
- 对已有的 maven 和 ivy 仓库的全面支持
- 支持传递性依赖管理，而不需要远程仓库或者 pom.xml 或者 ivy 配置文件
- ant 式的任务和构建是 gradle 的第一公民
- 基于 groovy，其 build 脚本使用 groovy dsl 编写
- 具有广泛的领域模型支持你的构建



# 2、Gradle特性

### 基于声明的构建和基于约定的构建

Gradle 的核心在于基于 Groovy 的丰富而可扩展的域描述语言(DSL)。 Groovy  通过声明性的语言元素将基于声明的构建推向下层，你可以按你想要的方式进行组合。 这些元素同样也为支持 Java， Groovy，OSGi，Web 和 Scala 项目提供了基于约定的构建。 并且，这种声明性的语言是可以扩展的。你可以添加新的或增强现有的语言元素。  因此，它提供了简明、可维护和易理解的构建。 

### 为以依赖为基础的编程方式提供语言支持

声明性语言优点在于通用任务图，你可以将其充分利用在构建中. 它提供了最大限度的灵活性，以让 Gradle 适应你的特殊需求。

### 构建结构化

Gradle 的灵活和丰富性最终能够支持在你的构建中应用通用的设计模式。  例如，它可以很容易地将你的构建拆分为多个可重用的模块，最后再进行组装，但不要强制地进行模块的拆分。  不要把原本在一起的东西强行分开（比如在你的项目结构里），从而避免让你的构建变成一场噩梦。  最后，你可以创建一个结构良好，易于维护，易于理解的构建。

### 深度 API

Gradle 允许你在构建执行的整个生命周期，对它的核心配置及执行行为进行监视并自定义。

### Gradle 的扩展

Gradle 有非常良好的扩展性。 从简单的单项目构建，到庞大的多项目构建，它都能显著地提升你的效率。 这才是真正的结构化构建。通过最先进的增量构建功能，它可以解决许多大型企业所面临的性能瓶颈问题。

### 多项目构建

Gradle 对多项目构建的支持非常出色。项目依赖是首先需要考虑的问题。 我们允许你在多项目构建当中对项目依赖关系进行建模，因为它们才是你真正的问题域。 Gradle 遵守你的布局。

Gradle 提供了局部构建的功能。 如果你在构建一个单独的子项目，Gradle 也会帮你构建它所依赖的所有子项目。 你也可以选择重新构建依赖于特定子项目的子项目。 这种增量构建将使得在大型构建任务中省下大量时间。

### 多种方式管理依赖

不同的团队喜欢用不同的方式来管理他们的外部依赖。 从 Maven 和 Ivy 的远程仓库的传递依赖管理，到本地文件系统的 jar 包或目录，Gradle 对所有的管理策略都提供了方便的支持。

### Gradle 是第一个构建集成工具

Ant tasks 是最重要的。而更有趣的是，Ant projects 也是最重要的。 Gradle 对任意的 Ant  项目提供了深度导入，并在运行时将 Ant 目标(target)转换为原生的 Gradle 任务(task)。 你可以从 Gradle  上依赖它们(Ant targets)，增强它们，甚至在你的 build.xml 上定义对 Gradle tasks 的依赖。Gradle  为属性、路径等等提供了同样的整合。

Gradle 完全支持用于发布或检索依赖的 Maven 或 Ivy 仓库。 Gradle 同样提供了一个转换器，用于将一个 Maven pom.xml 文件转换为一个 Gradle 脚本。Maven 项目的运行时导入的功能将很快会有。

## 易于移植

Gradle 能适应你已有的任何结构。因此，你总可以在你构建项目的同一个分支当中开发你的 Gradle 构建脚本，并且它们能够并行进行。 我们通常建议编写测试，以保证生成的文件是一样的。 这种移植方式会尽可能的可靠和减少破坏性。这也是重构的最佳做法。

### Groovy

Gradle 的构建脚本是采用 Groovy 写的，而不是用 XML。  但与其他方法不同，它并不只是展示了由一种动态语言编写的原始脚本的强大。 那样将导致维护构建变得很困难。 Gradle  的整体设计是面向被作为一门语言，而不是一个僵化的框架。 并且 Groovy 是我们允许你通过抽象的 Gradle 描述你个人的 story  的黏合剂。 Gradle 提供了一些标准通用的 story。这是我们相比其他声明性构建系统的主要特点。 我们的 Groovy  支持也不是简单的糖衣层，整个 Gradle 的 API 都是完全 groovy 化的。只有通过 Groovy才能去运用它并对它提高效率。

### The Gradle wrapper

Gradle Wrapper 允许你在没有安装 Gradle 的机器上执行 Gradle 构建。  这一点是非常有用的。比如，对一些持续集成服务来说。 它对一个开源项目保持低门槛构建也是非常有用的。 Wrapper  对企业来说也很有用，它使得对客户端计算机零配置。 它强制使用指定的版本，以减少兼容支持问题。

### 自由和开源

Gradle 是一个开源项目，并遵循 ASL 许可。

# 3、Gradle项目分析

关于 gradle 的项目层次，我们新建一个项目看一下，项目地址在 [EasyGradle](https://github.com/5A59/android-training/tree/master/gradle/EasyGradle)

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105245765-1977617981.jpg)

 

## 2.1 settings.gradle

settings.gradle 是负责配置项目的脚本
 对应 [Settings](https://github.com/gradle/gradle/blob/v4.1.0/subprojects/core/src/main/java/org/gradle/api/initialization/Settings.java) 类，gradle 构建过程中，会根据 settings.gradle 生成 Settings 的对象
 对应的可调用的方法在[文档](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)里可以查找
 其中几个主要的方法有:

- include(projectPaths)
- includeFlat(projectNames)
- project(projectDir)

一般在项目里见到的引用子模块的方法，就是使用 include，这样引用，子模块位于根项目的下一级

```groovy
include ':app'
```

如果想指定子模块的位置，可以使用 project 方法获取 Project 对象，设置其 projectDir 参数

```groovy
include ':app'
project(':app').projectDir = new File('./app')
```

## 2.2 rootproject/build.gradle

build.gradle 负责整体项目的一些配置，对应的是 [Project](https://github.com/gradle/gradle/blob/v4.1.0/subprojects/core/src/main/java/org/gradle/api/Project.java) 类
 gradle 构建的时候，会根据 build.gradle 生成 Project 对象，所以在 build.gradle 里写的 dsl，其实都是 Project 接口的一些方法，Project 其实是一个接口，真正的实现类是 [DefaultProject](https://github.com/gradle/gradle/blob/v4.1.0/subprojects/core/src/main/java/org/gradle/api/internal/project/DefaultProject.java)
 build.gradle 里可以调用的方法在 [Project](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html) 可以查到
 其中几个主要方法有：

- buildscript // 配置脚本的 classpath
- allprojects // 配置项目及其子项目
- respositories // 配置仓库地址，后面的依赖都会去这里配置的地址查找
- dependencies // 配置项目的依赖

以 EasyGradle 项目来看

```groovy
buildscript { // 配置项目的 classpath
    repositories {  // 项目的仓库地址，会按顺序依次查找
        google()
        jcenter()
        mavenLocal()
    }
    dependencies { // 项目的依赖
        classpath 'com.android.tools.build:gradle:3.0.1'
        classpath 'com.zy.plugin:myplugin:0.0.1'
    }
}

allprojects { // 子项目的配置
    repositories {
        google()
        jcenter()
        mavenLocal()
    }
}
```

## 2.3 module/build.gradle

build.gradle 是子项目的配置，对应的也是 Project 类
 子项目和根项目的配置是差不多的，不过在子项目里可以看到有一个明显的区别，就是引用了一个插件 apply plugin  "com.android.application"，后面的 android dsl 就是 application 插件的  extension，关于 android plugin dsl 可以看 [android-gradle-dsl](http://google.github.io/android-gradle-dsl/current/)
 其中几个主要方法有：

- compileSdkVersion // 指定编译需要的 sdk 版本
- defaultConfig // 指定默认的属性，会运用到所有的 variants 上
- buildTypes // 一些编译属性可以在这里配置，可配置的所有属性在 [这里](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html)
- productFlavor // 配置项目的 flavor

以 app 模块的 build.gradle 来看

```groovy
apply plugin: 'com.android.application' // 引入 android gradle 插件

android { // 配置 android gradle plugin 需要的内容
    compileSdkVersion 26
    defaultConfig { // 版本，applicationId 等配置
        applicationId "com.zy.easygradle"
        minSdkVersion 19
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
    }
    buildTypes { 
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions { // 指定 java 版本
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }

    // flavor 相关配置
    flavorDimensions "size", "color"
    productFlavors {
        big {
            dimension "size"
        }
        small {
            dimension "size"
        }
        blue {
            dimension "color"
        }
        red {
            dimension "color"
        }
    }
}

// 项目需要的依赖
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar']) // jar 包依赖
    implementation 'com.android.support:appcompat-v7:26.1.0' // 远程仓库依赖
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    implementation project(':module1') // 项目依赖
}
```



## 2.4 依赖

在 gradle 3.4 里引入了新的依赖配置，如下：

| 新配置         | 弃用配置 | 行为                                                         | 作用                                                         |
| -------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| implementation | compile  | 依赖项在编译时对模块可用，并且仅在运行时对模块的消费者可用。 对于大型多项目构建，使用 implementation 而不是 api/compile 可以显著缩短构建时间，因为它可以减少构建系统需要重新编译的项目量。 大多数应用和测试模块都应使用此配置。 | implementation 只会暴露给直接依赖的模块，使用此配置，在模块修改以后，只会重新编译直接依赖的模块，间接依赖的模块不需要改动 |
| api            | compile  | 依赖项在编译时对模块可用，并且在编译时和运行时还对模块的消费者可用。 此配置的行为类似于 compile（现在已弃用），一般情况下，您应当仅在库模块中使用它。 应用模块应使用 implementation，除非您想要将其 API 公开给单独的测试模块。 | api 会暴露给间接依赖的模块，使用此配置，在模块修改以后，模块的直接依赖和间接依赖的模块都需要重新编译 |
| compileOnly    | provided | 依赖项仅在编译时对模块可用，并且在编译或运行时对其消费者不可用。 此配置的行为类似于 provided（现在已弃用）。 | 只在编译期间依赖模块，打包以后运行时不会依赖，可以用来解决一些库冲突的问题 |
| runtimeOnly    | apk      | 依赖项仅在运行时对模块及其消费者可用。 此配置的行为类似于 apk（现在已弃用）。 | 只在运行时依赖模块，编译时不依赖                             |

还是以 EasyGradle 为例，看一下各个依赖的不同： 项目里有三个模块：app，module1， module2
 模块 app 中有一个类 ModuleApi
 模块 module1 中有一个类 Module1Api
 模块 module2 中有一个类 Module2Api
 其依赖关系如下：

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105523258-1821691335.png)

 

 implementation 依赖
当 module1 使用 implementation 依赖 module2 时，在 app 模块中无法引用到 Module2Api 类

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105550911-1484295450.png)

 

 api 依赖
当 module1 使用 api 依赖 module2 时，在 app 模块中可以正常引用到 Module2Api 类，如下图

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105628665-337331437.png)

 

 compileOnly 依赖
当 module1 使用 compileOnly 依赖 module2 时，在编译阶段 app 模块无法引用到 Module2Api 类，module1 中正常引用，但是在运行时会报错

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105700291-1891790808.png)

 

反编译打包好的 apk，可以看到 Module2Api 是没有被打包到 apk 里的 

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105733310-1073772551.png)

 

 runtimeOnly 依赖
当 module1 使用 runtimeOnly 依赖 module2 时，在编译阶段，module1 也无法引用到 Module2Api

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105814363-1096021393.png)

 

 

## 2.5 flavor

在介绍下面的流程之前，先明确几个概念，flavor，dimension，variant
 在 android gradle plugin 3.x 之后，每个 flavor 必须对应一个 dimension，可以理解为 flavor 的分组，然后不同 dimension 里的 flavor 两两组合形成一个 variant
 举个例子 如下配置:



```groovy
flavorDimensions "size", "color"

productFlavors {
    big {
        dimension "size"
    }
    small {
        dimension "size"
    }
    blue {
        dimension "color"
    }
    red {
        dimension "color"
    }
}
```

那么生成的 variant 对应的就是 bigBlue，bigRed，smallBlue，smallRed
 每个 variant 可以对应的使用 variantImplementation 来引入特定的依赖，比如：bigBlueImplementation，只有在 编译 bigBlue variant的时候才会引入

# 4、gradle wrapper

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028105916054-103581223.jpg)



**gradlew / gradlew.bat** 这个文件用来下载特定版本的 gradle  然后执行的，就不需要开发者在本地再安装 gradle 了。这样做有什么好处呢？开发者在本地安装  gradle，会碰到的问题是不同项目使用不同版本的 gradle 怎么处理，用 wrapper  就很好的解决了这个问题，可以在不同项目里使用不同的 gradle 版本。gradle wrapper 一般下载在  GRADLE_CACHE/wrapper/dists 目录下

**gradle/wrapper/gradle-wrapper.properties** 是一些 gradlewrapper 的配置，其中用的比较多的就是 distributionUrl，可以执行 gradle 的下载地址和版本
 **gradle/wrapper/gradle-wrapper.jar** 是 gradlewrapper 运行需要的依赖包

### 5、gradle init.gradle

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028110245902-1947179752.png)

 

在 gradle 里，有一种 init.gradle 比较特殊，这种脚本会在每个项目 build 之前先被调用，可以在其中做一些整体的初始化操作，比如配置 log 输出等等
 使用 init.gradle 的方法：

1. 通过 --init-script 指定 init.gradle 位置 eg: gradlew --init-script initdir/init.gradle
2. init.gradle 文件放在 USER_HOME/.gradle/ 目录下
3. .gradle 脚本放在 USER_HOME/.gradle/init.d/ 目录下
4. .gradle  脚本放在 GRDALE_HOME/init.d/ 目录下

# 5、gradle 生命周期及回调

gradle 构建分为三个阶段
 **初始化阶段**
 初始化阶段主要做的事情是有哪些项目需要被构建，然后为对应的项目创建 Project 对象

**配置阶段**
 配置阶段主要做的事情是对上一步创建的项目进行配置，这时候会执行 build.gradle 脚本，并且会生成要执行的 task

**执行阶段**
 执行阶段主要做的事情就是执行 task，进行主要的构建工作

gradle 在构建过程中，会提供一些列回调接口，方便在不同的阶段做一些事情，主要的接口有下面几个

```groovy
gradle.addBuildListener(new BuildListener() {
    @Override
    void buildStarted(Gradle gradle) {
        println('构建开始')
        // 这个回调一般不会调用，因为我们注册的时机太晚，注册的时候构建已经开始了，是 gradle 内部使用的
    }

    @Override
    void settingsEvaluated(Settings settings) {
        println('settings 文件解析完成')
    }

    @Override
    void projectsLoaded(Gradle gradle) {
        println('项目加载完成')
        gradle.rootProject.subprojects.each { pro ->
            pro.beforeEvaluate {
                println("${pro.name} 项目配置之前调用")
            }
            pro.afterEvaluate{
                println("${pro.name} 项目配置之后调用")
            }
        }
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
        println('项目解析完成')
    }

    @Override
    void buildFinished(BuildResult result) {
        println('构建完成')
    }
})

gradle.taskGraph.whenReady {
    println("task 图构建完成")
}
gradle.taskGraph.beforeTask {
    println("每个 task 执行前会调这个接口")
}
gradle.taskGraph.afterTask {
    println("每个 task 执行完成会调这个接口")
}
```



# 6、自定义 task

![img](https://img2018.cnblogs.com/blog/363274/201910/363274-20191028110430964-586439175.png)

默认创建的 task 继承自 DefaultTask 如何声明一个 task



```groovy
task myTask {
    println 'myTask in configuration'
    doLast {
        println 'myTask in run'
    }
}

class MyTask extends DefaultTask {
    @Input Boolean myInputs
    @Output 
    @TaskAction
    void start() {
    }
}

tasks.create("mytask").doLast {
}
```



Task 的一些重要方法分类如下：

- Task 行为
   Task.doFirst
   Task.doLast
- Task 依赖顺序
   Task.dependsOn
   Task.mustRunAfter
   Task.shouldRunAfter
   Task.finalizedBy
- Task 的分组描述
   Task.group
   Task.description
- Task 是否可用
   Task.enabled
- Task 输入输出
   gradle 会比较 task 的 inputs 和 outputs 来决定 task 是否是最新的，如果 inputs 和 outputs 没有变化，则认为 task 是最新的，task 就会跳过不执行
   Task.inputs
   Task.outputs
- Task 是否执行
   可以通过指定 Task.upToDateWhen = false 来强制 task 执行 Task.upToDateWhen

比如要指定 Task 之间的依赖顺序，写法如下：



```groovy
task task1 {
    doLast {
        println('task2')
    }
}
task task2 {
    doLast {
        println('task2')
    }
}
task1.finalizedBy(task2)
task1.dependsOn(task2)
task1.mustRunAfter(task2)
task1.shouldRunAfter(task2)
task1.finalizedBy(task2)
```