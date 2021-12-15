- [超详细Maven技术应用指南](https://www.cnblogs.com/ziph/p/13157977.html)





# 什么是maven？

Maven是一个项目管理工具，它包含了一个对象模型。一组标准集合，一个依赖管理系统。和用来运行定义在生命周期阶段中插件目标和逻辑。
**核心功能**
Maven的核心功能是合理叙述项目间的依赖关系，通俗点 就是通过pom.xml文件的配置获取jar包不用手动的去添加jar包，，这个pom.xml包我后面会叙述，不过已经学习过maven的 人应该对这个很熟悉。其本质就是通过配置pom.xml来获取jar包，当然这是在该项目必须是maven项目的前提下。那么什么是maven项目
**maven项目是啥？**
我们这样来理解maven项目，就是在java项目和web项目上裹了一层maven，本质上java项目还是java项目，web项目还是web项目，但是包裹了maven之后，就可以使用maven提供的一些功能，即通过pom.xml添加jar包
就像在蜜汁鸡外面裹了一层面粉油炸一下变成了炸鸡，但是他还是一只鸡



## Maven能够解决什么问题

**在想Maven可以解决什么问题之前我们先来想想我们开发过程中经常遇到什么问题**
1、我们需要引用各种 jar 包，尤其是比较大的工程，引用的 jar 包往往有几十个乃至上百个， 每用到一种 jar 包，都需要手动引入工程目录，而且经常遇到各种让人抓狂的 jar 包冲突，版本冲突。
2、我们辛辛苦苦写好了 Java 文件，可是只懂 0 和 1 的白痴电脑却完全读不懂，需要将它编译成二进制字节码。好歹现在这项工作可以由各种集成开发工具帮我们完成，Eclipse、IDEA 等都可以将代码即时编译。当然，如果你嫌生命漫长，何不铺张，也可以用记事本来敲代码，然后用 javac 命令一个个地去编译，逗电脑玩。
3、世界上没有不存在 bug 的代码，计算机喜欢 bug 就和人们总是喜欢美女帅哥一样。为了追求美为了减少 bug，因此写完了代码，我们还要写一些单元测试，然后一个个的运行来检验代码质量。
4、再优雅的代码也是要出来卖的。我们后面还需要把代码与各种配置文件、资源整合到一起，定型打包，如果是 web 项目，还需要将之发布到服务器，供人蹂躏。
**maven所帮助我们解决的问题**
**以上的这些问题maven都把我们解决了，没错maven可以帮我们
构建工程，
2管理jar，
3.编译代码，
4.自动运行单元测试，
5.打包
6.生成报表，
7.部署项目，生成web站点。**



#  接下来我就举个例子让大家先见识见识maven的功能

​	前面我们通过web阶段的项目，要能够将项目运行起来，就必须将该项目所依赖的一些jar包添加到工程中，否则项目就不可以运行了，如果相同架构的项目有十几个，那么我们就需要将这一份jar包复制到十个不同的工程中我们一起来看看CRM工程的大小
**使用传统的CRM项目**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910192312415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910192312415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

**使用maven构建**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910192351739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910192351739.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

# Maven的依赖管理

为什么使用maven之后文件夹就如此之小了呢？其实这我们在前面就提到过了即通过配置pom.xml的文件来配置依赖，而Maven的一个核心特征就是依赖管理，当我们涉及到多模块的项目（包含成百个模块或者子项目），管理依赖就变成了一个极为困难的任务Maven展示出了他对处理这种情形的高度控制
传统的web项目中，我们必须将工程所依赖的jar包复制到工程中，导致工程变的很大，那么maven是如何通过操作使工程变少的呢

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910194354918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910194354918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)通过图解可以发现maven工程不直接将jar包导入到工程中，而是通过再pom.xml中添加所需的jar包的坐标，这样就避免了jar直接引入进来，在需要用到jar包的时候，只要查找pom.xml文件，再通过pom.xml中的坐标，到一个专门用于存放jar包的仓库中根据坐标从而找到这些jar包，再把这些jar包拿去运行
看到这读者们可能会有疑问
**1.存放jar包的仓库长什么样？**
这个我们会在后面一一讲解，仓库也分为许多种类
**2.通过读取pom.xml坐标，来找jar的方式会不会导致速度很慢。从而导致这些方案不可行**
通过 pom.xml 文件配置要引入的 jar 包的坐标，再读取坐标并到仓库中加载 jar 包，这
样我们就可以直接使用 jar 包了，为了解决这个过程中速度慢的问题，maven 中也有索引的概念，通过建立索引，可以大大提高加载 jar 包的速度，使得我们认为 jar 包基本跟放在本地的工程文件中再读取出来的速度是一样的。这个过程就好比我们查阅字典时，为了能够加快查找到内容，书前面的目录就好比是索引，有了这个目录我们就可以方便找到内容了，一样的在 maven 仓库中有了索引我们就可以认为可以快速找到 jar 包。

# 仓库的概念

仓库就是存放jar包的地方，即我们前面说的通过pom.xml中通过设置索引来到仓库中寻找jar包
仓库分为：本地仓库，第三方仓库，中央仓库

**5.1本地仓库**
用来存储从远程仓库或者中央仓库下载的插件和jar包，项目使用一些插件或jar包
优先从本地仓库查找
默认本地仓库的位置在${user.dir}/.m2/repository，${user.dir}表示 windows 用户目录。
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910200729205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910200729205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
**5.2第三方仓库**
d第三方仓库，又称为内部中心仓库，又称为私服
私服：一般由公司自己设立，只为本公司内部共享使用，它既可以作为公司内部构建协作和存档，也可作为公用类库镜像缓存，减少在外部访问和下载的频率使用私服为了减少对中央仓库的访问私服可以使用的是局域网，中央仓库必须使用外网。也就是一般公司都会创建这种第三方仓库，保证项目开发时，项目所需用的jar都从该仓库中拿，每个人的版本就都一样。
**注意：连接私服，需要单独配置。如果没有配置私服，默认不使用**

**5.3中央仓库**
在 maven 软件中内置一个远程仓库地址 http://repo1.maven.org/maven2 ，它是中央仓库，服务于整个互联网，它是由 Maven 团队自己维护，里面存储了非常全的 jar 包，它含了世界上大部分流行的开源项目构件。

**获取jar包的过程**
优先从本地仓库查找，如果本地仓库没有该jar包，如果配置了私服，就从私服中查找，私服中没有就从中央仓库中查找，然后下载到本地仓库，下次使用就可以直接从本地仓库中查找，没有配置私服则，直接从中央仓库中查找
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910203705820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910203705820.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

# Maven java项目结构

Maven工程目录结构

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910204812247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910204812247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)图中有一个target目录，是因为将该java项目进行了编译，src/main/java下的源代码就会编译成.class文件放入target目录中，target就是输出目录。
作为一个 maven 工程，它的 src 目录和 pom.xml 是必备的。
进入 src 目录后，我们发现它里面的目录结构如下：
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910204957488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200910204957488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

项目名称
--pom.xml 核心配置，项目根下
--src
--main
--java java源码目录
--resources java配置文件目录
--test
--java 源码测试目录
--resource 测试配置目录

# maven的常用命令

## **7.1 compile**

compile是maven工程的编译命令，作用是将 src/main/java 下的文件编译为 class 文件输出到 target
目录下。

## **7.2 test**

test是maven工程的测试命令，会执行 src/test/java 下的单元测试类。
cmd 执行 mvn test 执行 src/test/java 下单元测试类，下图为测试结果，运行 1 个测试用例，全部成功。

## **7.3 clean**

clean是maven工程的清理命令，执行clean会删除target目录及其内容

## **7.4 package**

package是maven工程的打包命令，对于java工程执行 package 打成 jar 包，对于 web 工程打成 war
包。

## **7.5 install**

install 是 maven 工程的安装命令，执行 install 将 maven 打成 jar 包或 war 包发布到本地仓库。
从运行结果中，可以看出：当后面的命令执行时，前面的操作过程也都会自动执行

# maven的生命周期

maven对项目构建过程分为三套相互独立的生命周期，这里说的三套而且是相互独立，
这三套分别是：
Clean Lifecycle ：在进行真正的构建之前进行一些清理工作。
Default Lifecycle ：构建的核心部分，编译，测试，打包，部署等等。
Site Lifecycle ：生成项目报告，站点，发布站点。

# maven的概念模型

Maven 包含了一个项目对象模型 (Project Object Model)，一组标准集合，一个项目生命周期(Project
Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段
(phase)中插件(plugin)目标(goal)的逻辑。

[![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091114242717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/2020091114242717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

**9.1项目对象模型**：
一个maven工程都有一个pom.xml文件。通过pom.xml文件定义项目的坐标，项目的依赖，项目的信息
插件目标等

**9.2依赖管理系统**：
通过 maven 的依赖管理对项目所依赖的 jar 包进行统一管理。
比如：项目依赖 junit4.9，通过在 pom.xml 中定义 junit4.9 的依赖即使用 junit4.9，如下所示是 junit4.9
的依赖定义：



```
<!-- 依赖关系 -->
<dependencies>
<!-- 此项目运行使用 junit，所以此项目依赖 junit -->
<dependency>
<!-- junit 的项目名称 -->
<groupId>junit</groupId>
<!-- junit 的模块名称 -->
<artifactId>junit</artifactId>
<!-- junit 版本 -->
<version>4.9</version>
<!-- 依赖范围：单元测试时使用 junit -->
<scope>test</scope>
</dependency>
```

**9.3 一个项目的生命周期**
使用maven完成项目的构建，项目构建包括：清理，编译，部署等过程，maven将这些过程规范为一个生命周期，如下所示是生命周期的各阶段

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200910222221107.png#pic_center)](https://img-blog.csdnimg.cn/20200910222221107.png#pic_center)
maven 通过执行一些简单命令即可实现上边生命周期的各各过程，比如执行 mvn compile 执行编译、
执行 mvn clean 执行清理。

**9.4 一组标准集合**
maven将整个项目管理过程定义为一组标准集合，比如通过maven构建工程有标准的目录结构，有标准的生命周期阶段，依赖管理有标准的坐标定义

**9.5 插件目标**

maven管理项目生命周期都是基于插件完成的

# 使用idea开发meven项目

这个就是几个简单的参数配置一下没什么东西讲，我就放个流程图好了

**1.**
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150111939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150111939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)**2.**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150140601.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150140601.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
**3.**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150204727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150204727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

1. 

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150222573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150222573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)

1. 

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150241628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150241628.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
6.
[![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091115030519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/2020091115030519.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
7.
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150329666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150329666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
**8.**
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150433911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150433911.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)**9.**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150500619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150500619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)**10.**

[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150617889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150617889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)
**11.**
[![在这里插入图片描述](https://img-blog.csdnimg.cn/20200911150636432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)](https://img-blog.csdnimg.cn/20200911150636432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BqaDg4,size_16,color_FFFFFF,t_70#pic_center)