[Spring Cloud Alibaba(6)---Nacos持久化Mysql8.0版本](https://www.cnblogs.com/qdhxhz/p/14701077.html)

##  一、背景

我们服务的信息、配置的信息都放在哪的？官网有说过

```
当我们使用默认配置启动Nacos时，所有配置文件都被Nacos保存在了内置的数据库中。
```

这里所指的内置数据库其实就是内存中,既然是配置在内存中，那么每当我们重启Nacos的时候,所有配置好的信息都会丢失，这显然是我们不能够接受的,所以我们就需要去配置，

让配置数据存在Mysql中，这样当我们重启服务器的时候，配置数据依旧在。

```
架构信息
Nacos 1.4.2 + MYSQL 8.0.22
```



## 二、Nacos客户端部署

客户端部署其实官方给了两种方式 一种是 **从 Github 上下载源码**。另一种是 **直接下载压缩包**。[官方地址](https://nacos.io/zh-cn/docs/quick-start.html)

之前在写**Nacos概述**的时候，是直接去官网下载 nacos-server-1.3.2.tar.gz 的压缩包,下载解压后 通过命令运行。这里我们需要用第二种方式，我们直接去拉nacos源码后，

自己打成nacos-server-1.4.2-SNAPSHOT.tar.gz 压缩包。

为什么要不用官方直接提供的压缩包，而需要我们自己下源码在打成压缩包，这样多次一举呢？

是因为,我们是 MYSQL 8.0.22 版本的，所以我们要下载源码，把pom文件中对应的连接数据库的驱动jar包改成8.0.22版本的,然后重新打成压缩包。

#### 1、拉取源码

这个当前拉取的是1.4.2版本的Nacos,你也可以指定拉取版本

```
git clone https://github.com/alibaba/nacos.git
```

#### 2、修改Mysql连接驱动

1.4.2版本的Nacos所用的mysql-connector-java版本是8.0.21,这里修改成 8.0.22。

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.22</version>
</dependency>
```

`注意` 其实这里可以不用修改,因为8.0.21和8.0.22，差异很小，所以我们不用修改都是可以的。就是说我们都不用取拉源码在通过命令打包，而是直接下载1.4.2版本的

压缩包就可以了。当然如果你的是1.3.1以下版本的Nacos，那么Mysql驱动是5.7版本的那么这个是需要修改成8以上版本的。

#### 3、打包

进入nacos目录

```
cd nacos
```

maven打包

```
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U 
```

如果直接成功后，在`distribution/target/`目录中就已经有相应的解压包了。

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210425165942509-1317315050.jpg)

#### 4、解压

```
tar -xvf nacos-server-1.4.2-SNAPSHOT.tar.gz
```

接下来我们先不启动服务器，因为我们还需要修改一些配置文件



## 三、修改配置文件 

#### 1、初始化数据库

Nacos的数据库脚本文件在我们下载Nacos-server时的压缩包中就有

```
进入nacos\conf目录，初始化文件：nacos-mysql.sql
```

此处我创建一个名为 `nacos` 的数据库，然后执行初始化脚本，成功后会生成 12 张表

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210425170003966-2078895594.jpg)

#### 2、修改配置文件

它的配置文件也在 `nacos\conf`目录下，名为 `application.properties`，在文件底部添加数据源配置：

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

#### 3、启动服务器

现在数据库和配置都已经修改了，那么就启动服务器

```
//先进入bin目录
 cd nacos/bin
```

启动命令(standalone代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行:

```
bash startup.sh -m standalone
```

`注意` 执行命令地方在上面压缩的 **distribution/target/nacos-server-1.4.2-SNAPSHOT/nacos/bin** 位置下，而不是在distribution的bin下。

启动后，能够正常访问下面地址，那就说明已经配置成功，

```
http://127.0.0.1:8848/nacos
```

如果上面访问失败，那就去 **nacos\log** 目录下，名为 `nacos.log` 看下有没有错误日志



## 四 测试 

#### 1、在客户端新添加一条配置数据

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210425170018354-51590324.jpg)

看列表 也显示这条配置集已经创建成功

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210425170027635-1369477320.jpg)

#### 2、查看数据库

既然客户端都生成成功了，那就来看下数据库中有没有持久化这条数据

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210425170037106-2141209251.jpg)

很明显 数据中也有这条数据，这样就算nacos重新启动这条数据也还是会有了。



###   参考

[1、nacos官网快速启动](https://nacos.io/zh-cn/docs/quick-start.html)

[2、nacos数据持久化MySQL8.0以上版本](https://www.jianshu.com/p/990146986e30)

[3、Spring Cloud Alibaba基础教程：Nacos的数据持久化](https://www.cnblogs.com/larscheng/p/11422909.html)