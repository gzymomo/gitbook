[TOC]

博客园：字母哥：[MinIO很强-让我放弃FastDFS拥抱MinIO的8个理由](https://www.cnblogs.com/zimug/p/13444086.html)
简书：黄宝玲_1003：[Minio 文件服务（1）—— Minio部署使用及存储机制分析](https://www.jianshu.com/p/3e81b87d5b0b)
[Minio 文件服务（2）—— Minio用Nginx做负载均衡](https://www.jianshu.com/p/a30fee4a33e9)

MinIO官网地址：docs.min.io/cn/ 

# 一、Minio简介

> Minio 是一个基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。
> Minio是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL。

# 二、Minio优势
## 2.1 UI界面
fastDFS默认是不带UI界面的，看看MinIO的界面吧。这个界面不需要你单独的部署，和服务端一并安装。

![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/e6af76aaa02c7f57e50afa1a4bb3e3c3?showdoc=.jpg)

## 2.2 性能
MinIO号称是世界上速度最快的对象存储服务器。在标准硬件上，对象存储的读/写速度最高可以达到183 GB/s和171 GB/s。关于fastDFS我曾经单线程测试写了20万个文件，总共200G，大约用时10个小时。总体上是很难达到MinIO“号称的”以G为单位的每秒读写速度。

![](https://img2020.cnblogs.com/other/1815316/202008/1815316-20200806082638831-1607794313.png)

## 2.3 容器化支持
MinIO提供了与k8s、etcd、docker等容器化技术深度集成方案，可以说就是为了云环境而生的。这点是FastDFS不具备的。

![](https://img2020.cnblogs.com/other/1815316/202008/1815316-20200806082639330-1212346584.png)

## 2.4 丰富的SDK支持
fastDFS目前提供了 C 和 Java SDK ，以及 PHP 扩展 SDK。下图是MinIO提供的SDK支持，MinIO几乎提供了所有主流开发语言的SDK以及文档。

![](https://img2020.cnblogs.com/other/1815316/202008/1815316-20200806082639552-1716004316.png)

# 三、存储机制
Minio使用纠删码erasure code和校验和checksum来保护数据免受硬件故障和无声数据损坏。 即便丢失一半数量（N/2）的硬盘，仍然可以恢复数据。

## 3.1 纠删码
>纠删码是一种恢复丢失和损坏数据的数学算法，目前，纠删码技术在分布式存储系统中的应用主要有三类，阵列纠删码（Array Code: RAID5、RAID6等）、RS(Reed-Solomon)里德-所罗门类纠删码和LDPC(LowDensity Parity Check Code)低密度奇偶校验纠删码。Erasure Code是一种编码技术，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。

Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。因此下面主要讲解RS类纠删码。

## 3.2 RS code编码数据恢复原理：
> RS编码以word为编码和解码单位，大的数据块拆分到字长为w（取值一般为8或者16位）的word，然后对word进行编解码。 数据块的编码原理与word编码原理相同，后文中以word为例说明，变量Di, Ci将代表一个word。
> 把输入数据视为向量D=(D1，D2，..., Dn）, 编码后数据视为向量（D1, D2,..., Dn, C1, C2,.., Cm)，RS编码可视为如下（图1）所示矩阵运算。
> 图1最左边是编码矩阵（或称为生成矩阵、分布矩阵，Distribution Matrix），编码矩阵需要满足任意n*n子矩阵可逆。为方便数据存储，编码矩阵上部是单位阵（n行n列），下部是m行n列矩阵。下部矩阵可以选择范德蒙德矩阵或柯西矩阵。

![](https://upload-images.jianshu.io/upload_images/15296782-683a2bf10cf61f78.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

RS最多能容忍m个数据块被删除。 数据恢复的过程如下：
（1）假设D1、D4、C2丢失，从编码矩阵中删掉丢失的数据块/编码块对应的行。（图2、3）
（2）由于B' 是可逆的，记B'的逆矩阵为 (B'^-1)，则B' * (B'^-1) = I 单位矩阵。两边左乘B' 逆矩阵。 （图4、5）
（3）得到如下原始数据D的计算公式 。

![](https://upload-images.jianshu.io/upload_images/15296782-ae5b18d3832ec889.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
（4）对D重新编码，可得到丢失的编码码

## 3.3 实践
Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。 这就意味着如果是12块盘，一个对象会被分成6个数据块、6个奇偶校验块，可以丢失任意6块盘（不管其是存放的数据块还是奇偶校验块），仍可以从剩下的盘中的数据进行恢复。

# 四、部署
## 4.1 单节点

**容器部署：**
```bash
docker pull minio/minio
 
#在Docker中运行Minio单点模式
docker run -p 9000:9000 -e MINIO_ACCESS_KEY=sunseaiot -e MINIO_SECRET_KEY=sunseaiot minio/minio server /data
 
#要创建具有永久存储的Minio容器，您需要将本地持久目录从主机操作系统映射到虚拟配置~/.minio 并导出/data目录
#建立外挂文件夹 /Users/hbl/dockersp/volume/minio/
docker run -d -p 9000:9000 -e MINIO_ACCESS_KEY=sunseaiot -e MINIO_SECRET_KEY=sunseaiot -v /Users/hbl/dockersp/volume/minio/data:/data -v /Users/hbl/dockersp/volume/minio/config:/root/.minio minio/minio server /data
```

## 4.2 多节点
> 分布式搭建的流程和单节点基本一样，Minio服务基于命令行传入的参数自动切换成单机模式还是分布式模式。

> 分布式Minio单租户存在最少4个盘最多16个盘的限制（受限于纠删码）。这种限制确保了Minio的简洁，同时仍拥有伸缩性。如果你需要搭建一个多租户环境，你可以轻松的使用编排工具（Kubernetes）来管理多个Minio实例。

# 五、Java SDK访问Minio服务
```java
package com.minio.client;

import io.minio.MinioClient;
import io.minio.errors.MinioException;
import lombok.extern.slf4j.Slf4j;
import org.xmlpull.v1.XmlPullParserException;

import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

@Slf4j
public class FileUploader {
    public static void main(String[] args) throws NoSuchAlgorithmException, IOException, InvalidKeyException, XmlPullParserException {
        try {
            MinioClient minioClient = new MinioClient("https://minio.sunseaiot.com", "sunseaiot", "sunseaiot",true);

            // 检查存储桶是否已经存在
            if(minioClient.bucketExists("ota")) {
                log.info("Bucket already exists.");
            } else {
                // 创建一个名为ota的存储桶
                minioClient.makeBucket("ota");
                log.info("create a new bucket.");
            }

            //获取下载文件的url，直接点击该url即可在浏览器中下载文件
            String url = minioClient.presignedGetObject("ota","hello.txt");
            log.info(url);

            //获取上传文件的url，这个url可以用Postman工具测试，在body里放入需要上传的文件即可
            String url2 = minioClient.presignedPutObject("ota","ubuntu.tar");
            log.info(url2);

            // 下载文件到本地
            minioClient.getObject("ota","hello.txt", "/Users/hbl/hello2.txt");
            log.info("get");

            // 使用putObject上传一个本地文件到存储桶中。
            minioClient.putObject("ota","tenant2/hello.txt", "/Users/hbl/hello.txt");
            log.info("/Users/hbl/hello.txt is successfully uploaded as hello.txt to `task1` bucket.");
        } catch(MinioException e) {
            log.error("Error occurred: " + e);
        }
    }
}
```