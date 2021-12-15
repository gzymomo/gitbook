# 什么是OSS

全称为**Object Storage Service**，也叫**对象存储服务**，是一种解决和处理离散单元的方法，可提供基于分布式系统之上的对象形式的数据存储服务，具有可拓展、可管理、低成本等特点，支持中心和边缘存储，能够实现存储需求的弹性伸缩，主要应用于海量数据管理的各类场景。

这概念真是够难以理解的。简单说点我知道的吧，平常我们的文件地址都是 `/User/felord/video/xxx.mp4`的目录树结构，系统先要找到`User`，然后一级一级往下找一直到目标为止，这是一种结构化的存储方式。对象存储就不一样了，所有的文件都放在一个特定的池子里，只不过文件的携带有它自己的元信息，通过元信息去检索文件。这里举一个形象的例子：

```json
{"oss":[
    {"file":"xxxxx","meta":{"id":"1111"},"type":""},
    {"content":"xxxxx","meta":{"id":"1211"},"type":"","created":"","name":""}, 
]}
```

上图的`oss`就是一个对象存储，它里面存了携带信息不一样、甚至结构都不一样的东西，我们可以根据其元信息`meta`检索它们。**OSS**具有以下特点：

- 效率更高。不受复杂目录系统对性能的影响。
- 可扩展性更强。分布式架构，更便于进行水平扩展，从而容纳进任意大规模的数据。
- 可用性更强。数据一般都会有多个位于不同机器的复制，确保数据不丢失。
- 平台无关，可以通过**Restful**接口进行操作对象。

> **OSS**通常被用来存储图片、音视频等文件，以及对这些文件的处理。

# 哪些OSS可以使用

通常我们有两种选择，花钱买或者自己搞。

## 充钱才能变得更强

这句话这里也是很实用的，目前几乎所有的云厂商都有自己的对象存储产品，你可以对比一下花钱购买它们，通过配合**CDN**能达到非常好的用户体验，胖哥的**felord.cn**就使用了云厂商的对象存储。购买他们的服务

- 可靠性强，数据丢失可能性低。
- 免维护，不需要自行维护。
- 可配合其它一些特色功能，比如缩略图、CDN等等。

## 自己动手丰衣足食

不想花钱就只能自己动手了，目前我知道的开源方案有两种。

一种是**Ceph**,一个分布式存储系统，高可用，高扩展性。但是一般人玩不转，就连**开源中国**的**红薯**都被坑惨了😆。

![大半年后红薯被Ceph玩坏了](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100851226-1723828134.png)

另一种是**Minio**，用**Golang**写的。我目前还没发现有什么坑，文档居然还有中文文档！我用**Docker**不到三分钟就玩起来了，居然还自带控制台！其它功能也挺齐全，各种客户端**SDK**齐全。

![Minio Logo](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100851443-2101792793.png)

## 整合到Spring Boot

无论你花钱还是自己搞都可以，这两种方式各有各的好处。所以我要把这两种方式整合到**kono Spring Boot**脚手架项目中。这种组件封装成为**Spring Boot Starter**再好不过了。在日常开发中这种基础组件都建议做成**Starter**。参考我的 [最强自定义Spring Boot Starter教程](https://mp.weixin.qq.com/s/ezz1zQ6O4qV4pwqz1UjdTg)里的方式，我将**aliyun**的**OSS SDK**和**Minio SDK**封装成**Starter**了。

**达到了开箱即用。而且非常灵活，你配置哪种使用哪种，可以二选一，也可以全都要，还可以全都不要。**

> 项目地址: https://gitee.com/felord/oss-spring-boot.git。

获取到项目后通过**Maven**命令`mvn install`安装到本地依赖库，或者你发布到你的远程私有**Maven**仓库。然后再引用**Starter**，**切记先后步骤**：

```xml
<!--  一定要先拉取项目通过 mvn install 安装到本地  -->
<dependency>
    <groupId>cn.felord</groupId>
    <artifactId>oss-spring-boot-starter</artifactId>
    <version>1.0.0.RELEASE</version>
</dependency>
```

## Minio配置流程

接着就是使用了，先在你**Minio**的控制台上创建一个**bucket**,可以理解为一个对象池。

![创建 bucket](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100851644-2055345198.png)

然后把策略设置为**可读写**。

![编辑名称为img的bucket的策略](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100851820-121332677.png)

![可读写策略](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100851976-467092548.png)

搞完开始在项目中配置，`application.yaml`中:

```yaml
oss:
  minio:
  # 启用 
    active: true  
    access-key: minio_access_key
    secret-key: felord_cn_sec_key
  # minio 地址  
    endpoint: http://localhost:9000
```

## aliyun OSS 配置流程

额外引入依赖：

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.3.8</version>
</dependency>
```

> 这是必须的步骤。

去**ali OSS控制台申请**跟**Minio**差不多的几样东西用来配置：

```yaml
oss:
  aliyun:
    active: true
    access-key-id: LTAI4GH4EQXtKEbJDrADvWNH
    access-key-secret: XjDpNn5JqHAHPDXGL6xIebyUkyFAZ7
    endpoint: oss-cn-beijing.aliyuncs.com
```

## Starter的使用

以下是我对**OSS**操作的抽象接口：

```java
package cn.felord.oss;

import java.io.InputStream;

/**
 * The interface Storage.
 *
 * @author felord.cn
 * @since 2020 /8/24 19:54
 */
public interface Storage {


    /**
     * 存放对象
     *
     * @param bucketName   bucket  名称
     * @param objectName  自定义对象名称
     * @param inputStream  对象的输入流
     * @param contentType  参考http 的 MimeType 值
     * @throws Exception the exception
     */
    void putObject(String bucketName, String objectName, InputStream inputStream, String contentType) throws Exception;

    /**
     *  获取对象
     *
     * @param bucketName the bucket name
     * @param objectName the object name
     * @return the object
     */
    InputStream getObject(String bucketName, String objectName) throws Exception;

    /**
     *  获取对象的URL
     *
     * @param bucketName the bucket name
     * @param objectName the object name
     * @return the object url
     */
    String getObjectUrl(String bucketName, String objectName) throws Exception;

    /**
     *  删除对象
     *
     * @param bucketName the bucket name
     * @param objectName the object name
     */
    void removeObject(String bucketName, String objectName) throws Exception;

}
```

然后分别使用了以上两种**OSS**进行了实现。

![对应的两种实现](https://img2020.cnblogs.com/other/1739473/202008/1739473-20200825100852140-537663851.png)

并分别以`aliyunStorage`、`minioStorage`为名称将`AliyunStorage`和`MinioStorage`注入**Spring IoC**。

使用起来非常简单：

```java
@Autowired
@Qualifier("minioStorage")
Storage storage;

@Test
public void testOss() throws Exception {
    File file = new File("./456.jpg");

    InputStream inputStream = new FileInputStream(file);

    storage.putObject("img","pic_122",inputStream, MimeTypeUtils.IMAGE_JPEG_VALUE);
```