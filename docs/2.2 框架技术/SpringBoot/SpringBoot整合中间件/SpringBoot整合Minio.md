[TOC]



博客园：字母哥：[MinIO很强-让我放弃FastDFS拥抱MinIO的8个理由](https://www.cnblogs.com/zimug/p/13444086.html)
简书：黄宝玲_1003：[Minio 文件服务（1）—— Minio部署使用及存储机制分析](https://www.jianshu.com/p/3e81b87d5b0b)
[Minio 文件服务（2）—— Minio用Nginx做负载均衡](https://www.jianshu.com/p/a30fee4a33e9)

CSDN：[Spring Boot + 对象存储服务MinIO API + Vue 实现图片上传以及展示](https://blog.csdn.net/master_02/article/details/105191445)

CSDN：[分布式对象存储服务Minio的可观测性方案](https://blog.csdn.net/weichuangxxb/article/details/105476200)

MinIO官网地址：docs.min.io/cn/

# 一、Minio简介

> Minio 是一个基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。
> Minio是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL。

# 二、Minio优势

## 2.1 UI界面

fastDFS默认是不带UI界面的，看看MinIO的界面吧。这个界面不需要你单独的部署，和服务端一并安装。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93d3cuc2hvd2RvYy5jYy9zZXJ2ZXIvYXBpL2F0dGFjaG1lbnQvdmlzaXRmaWxlL3NpZ24vZTZhZjc2YWFhMDJjN2Y1N2U1MGFmYTFhNGJiM2UzYzM?x-oss-process=image/format,png)

## 2.2 性能

MinIO号称是世界上速度最快的对象存储服务器。在标准硬件上，对象存储的读/写速度最高可以达到183 GB/s和171 GB/s。关于fastDFS我曾经单线程测试写了20万个文件，总共200G，大约用时10个小时。总体上是很难达到MinIO“号称的”以G为单位的每秒读写速度。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDIwLmNuYmxvZ3MuY29tL290aGVyLzE4MTUzMTYvMjAyMDA4LzE4MTUzMTYtMjAyMDA4MDYwODI2Mzg4MzEtMTYwNzc5NDMxMy5wbmc?x-oss-process=image/format,png)

## 2.3 容器化支持

MinIO提供了与k8s、etcd、docker等容器化技术深度集成方案，可以说就是为了云环境而生的。这点是FastDFS不具备的。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDIwLmNuYmxvZ3MuY29tL290aGVyLzE4MTUzMTYvMjAyMDA4LzE4MTUzMTYtMjAyMDA4MDYwODI2MzkzMzAtMTIxMjM0NjU4NC5wbmc?x-oss-process=image/format,png)

## 2.4 丰富的SDK支持

fastDFS目前提供了 C 和 Java SDK ，以及 PHP 扩展 SDK。下图是MinIO提供的SDK支持，MinIO几乎提供了所有主流开发语言的SDK以及文档。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9pbWcyMDIwLmNuYmxvZ3MuY29tL290aGVyLzE4MTUzMTYvMjAyMDA4LzE4MTUzMTYtMjAyMDA4MDYwODI2Mzk1NTItMTcxNjAwNDMxNi5wbmc?x-oss-process=image/format,png)

# 三、存储机制

Minio使用纠删码erasure code和校验和checksum来保护数据免受硬件故障和无声数据损坏。 即便丢失一半数量（N/2）的硬盘，仍然可以恢复数据。

## 3.1 纠删码

> 纠删码是一种恢复丢失和损坏数据的数学算法，目前，纠删码技术在分布式存储系统中的应用主要有三类，阵列纠删码（Array Code: RAID5、RAID6等）、RS(Reed-Solomon)里德-所罗门类纠删码和LDPC(LowDensity Parity Check Code)低密度奇偶校验纠删码。Erasure Code是一种编码技术，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。

Minio采用Reed-Solomon code将对象拆分成N/2数据和N/2 奇偶校验块。因此下面主要讲解RS类纠删码。

## 3.2 RS code编码数据恢复原理：

> RS编码以word为编码和解码单位，大的数据块拆分到字长为w（取值一般为8或者16位）的word，然后对word进行编解码。 数据块的编码原理与word编码原理相同，后文中以word为例说明，变量Di, Ci将代表一个word。
> 把输入数据视为向量D=(D1，D2，…, Dn）, 编码后数据视为向量（D1, D2,…, Dn, C1, C2,…, Cm)，RS编码可视为如下（图1）所示矩阵运算。
> 图1最左边是编码矩阵（或称为生成矩阵、分布矩阵，Distribution Matrix），编码矩阵需要满足任意n*n子矩阵可逆。为方便数据存储，编码矩阵上部是单位阵（n行n列），下部是m行n列矩阵。下部矩阵可以选择范德蒙德矩阵或柯西矩阵。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTI5Njc4Mi02ODNhMmJmMTBjZjYxZjc4LnBuZw?x-oss-process=image/format,png)

RS最多能容忍m个数据块被删除。 数据恢复的过程如下：
（1）假设D1、D4、C2丢失，从编码矩阵中删掉丢失的数据块/编码块对应的行。（图2、3）
（2）由于B’ 是可逆的，记B’的逆矩阵为 (B’^-1)，则B’ * (B’^-1) = I 单位矩阵。两边左乘B’ 逆矩阵。 （图4、5）
（3）得到如下原始数据D的计算公式 。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy8xNTI5Njc4Mi1hZTViMThkMzgzMmVjODg5LnBuZw?x-oss-process=image/format,png)
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
12345678
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

# 六、对象存储Minio 客户端工具类，实现文件上传、图像压缩、图像添加水印

[对象存储Minio 客户端工具类，实现文件上传、图像压缩、图像添加水印](https://blog.csdn.net/lihuajian1024/article/details/100772482)

**MioioUtil**负责处理文件的上传及minio的相关初始化，**ImageUtil**负责对图像文件进行压缩及添加水印的操作。

需要在pom.xml中引入两个jar包：

MInio Client sdk包和谷歌的thumbnailator图像处理工具：

```html
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>3.0.10</version>
</dependency>
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.8</version>
</dependency>
```

MinioUtil.class如下：

```java
import io.minio.MinioClient;
import io.minio.errors.InvalidEndpointException;
import io.minio.errors.InvalidPortException;
import io.minio.policy.PolicyType;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.multipart.MultipartFile;
import javax.imageio.ImageIO;
import java.io.InputStream;
import java.time.LocalDate;
import java.util.Random;
 
/**
 * 对象存储工具类
 * @author 10552
 */
public class MinioUtil {
    private static final Logger log = LoggerFactory.getLogger(MinioUtil.class);
    //Mino服务器的AccessKey
    private final transient static String ACCESS_KEY="填写你的Mino服务器AccessKey";
    //Mino服务器的SecretKey
    private final transient static String SECRET_KEY="填写你的Mino服务器SecretKey";
    //桶名称
    private final transient static String BUCKET_NAME="delivery";
    //读写分离-上传服务器
    private final transient static String OSS_URL_WRITE="http://你的服务器上传地址";
    //读写分离-下载服务器
    private final transient static String OOS_URL_READ="http://你的服务器下载地址";
    //minio服务端口，默认是9000
    private final transient static int OSS_PORT=9000;
    private transient static boolean BUCKET_EXISTS=false;
    //单例模式-内部类实现
    private static class MinioClientHolder {
        private static  MinioClient minioClient;
        static {
            try {
                minioClient = new MinioClient(OSS_URL_WRITE, OSS_PORT,ACCESS_KEY, SECRET_KEY);
            } catch (InvalidEndpointException e) {
                e.printStackTrace();
            } catch (InvalidPortException e) {
                e.printStackTrace();
            }
        }
    }
    /**
     * 获取minio客户端实例
     * @return
     */
    private static MinioClient getMinioClient(){
        return MinioClientHolder.minioClient;
    }
 
    /**
     * 上传文件
     * 支持单文件，多文件
     * 返回文件访问路径，多文件以分号‘；’分隔
     * @param muFiles
     * @return
     */
    public static String uploadFiles(MultipartFile... muFiles) {
        if (muFiles.length<1){
            throw new RuntimeException("上传文件为空！");
        }
        StringBuilder str=new StringBuilder();
        for (MultipartFile muFile : muFiles) {
            str.append(uploadFile(muFile));
            str.append(";");
        }
        return str.deleteCharAt(str.length()-1).toString();
 
    }
 
    /**
     * 内部方法
     * 上传文件
     * @param muFile
     * @return
     */
    private static String uploadFile(MultipartFile muFile){
        String fileName = getFilePathName(muFile,false);
        try {
            MinioClient minioClient = getMinioClient();
            if(!BUCKET_EXISTS&&!minioClient.bucketExists(BUCKET_NAME)){
                minioClient.makeBucket(BUCKET_NAME);
                minioClient.setBucketPolicy(BUCKET_NAME, "", PolicyType.READ_ONLY);
                BUCKET_EXISTS=true;
            }
            InputStream inputStream=muFile.getInputStream();
            //如果是图片文件就进行压缩
            if (ImageUtil.isImage(muFile.getOriginalFilename())){
                inputStream=ImageUtil.getInputStream(
                        ImageUtil.setWatermark(
                                ImageUtil.compress(
                                        ImageIO.read(inputStream))),
                        ImageUtil.getFileExtention(muFile.getOriginalFilename()));
            }
            minioClient.putObject(BUCKET_NAME, fileName , inputStream,muFile.getContentType());
        }  catch (Exception e) {
            log.error("文件上传失败",e);
            throw new RuntimeException("文件上传失败");
        }
        return OOS_URL_READ+BUCKET_NAME+fileName;
    }
 
    /**
     * 	 获取文件名
     * @param muFile 文件
     * @param isRetain 是否保留源文件名
     * @return 返回文件名，以当前年月日作为前缀路径
     */
    private static String getFilePathName(MultipartFile muFile,boolean isRetain){
        String fileName = muFile.getOriginalFilename();
        String name=fileName;
        String prefix="";
        if(fileName.indexOf('.')!=-1) {
            name=fileName.substring(0,fileName.indexOf('.'));
            prefix=fileName.substring(fileName.lastIndexOf("."));
        }
 
        LocalDate date = LocalDate.now();
        StringBuilder filePathName=new StringBuilder("/upload/");
        filePathName.append(date.getYear());
        filePathName.append("/");
        filePathName.append(date.getMonthValue());
        filePathName.append("/");
        filePathName.append(date.getDayOfMonth());
        filePathName.append("/");
        //添加随机后缀
        Random r = new Random();
        int pix=r.ints(1, (100 + 1)).findFirst().getAsInt();
        filePathName.append(System.currentTimeMillis());
        filePathName.append(""+pix);
        //文件名超过32字符则截取
        if(isRetain){
            filePathName.append("_");
            if(name.length()>=32){
                name=name.substring(0,32);
            }
            filePathName.append(name);
        }
        filePathName.append(prefix);
        return filePathName.toString();
    }
```





ImageUtil.class工具类如下：

```java
import net.coobird.thumbnailator.Thumbnails;
import net.coobird.thumbnailator.geometry.Positions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.font.FontRenderContext;
import java.awt.font.TextAttribute;
import java.awt.geom.AffineTransform;
import java.awt.geom.Rectangle2D;
import java.awt.image.BufferedImage;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.text.AttributedCharacterIterator;
import java.text.AttributedString;

/**
 * 图像工具类
 */
public class ImageUtil {

    private static final Logger log = LoggerFactory.getLogger(ImageUtil.class);
    //压缩率
    private static final transient  float IMAGE_RATIO=0.1f;
    //压缩最大宽度
    private static final transient  int IMAGE_WIDTH=800;
    // 水印透明度
    private static float alpha = 0.3f;
    // 水印文字字体
    private static Font font = new Font("PingFang SC Regular", Font.PLAIN, 36);
    // 水印文字颜色
    private static Color color = new Color(111, 111, 111);
    //水印文字内容
    private static final String text="这是一个水印文本";
    // 水印之间的间隔
    private static final int XMOVE = 80;
    // 水印之间的间隔
    private static final int YMOVE = 80;

    /**
     * 压缩图像
     * @param image
     * @return
     * @throws IOException
     */
    public static BufferedImage compress(BufferedImage image) throws IOException {
        Thumbnails.Builder<BufferedImage> imageBuilder= Thumbnails.of(image).outputQuality(IMAGE_RATIO);
        if(image.getWidth()>IMAGE_WIDTH){
            return imageBuilder.width(IMAGE_WIDTH).asBufferedImage();
        }
        else {
            return imageBuilder.scale(1).asBufferedImage();
        }
    }

    /**
     * 图像添加水印
     * @param
     * @return
     */
    public static BufferedImage setWatermark(BufferedImage image)throws IOException {
        return Thumbnails.of(image)
                .outputQuality(IMAGE_RATIO)
                .scale(1)
                .watermark(Positions.BOTTOM_RIGHT
                        ,createWatermark(text
                                ,image.getWidth()
                                ,image.getHeight()
                        )
                        ,alpha)
                .asBufferedImage();
    }

    /**
     * 根据文件扩展名判断文件是否图片格式
     * @return
     */
    public static boolean isImage(String fileName) {
        String[] imageExtension = new String[]{"jpeg", "jpg", "gif", "bmp", "png"};
        for (String e : imageExtension) if (getFileExtention(fileName).toLowerCase().equals(e)) return true;
        return false;
    }

    /**
     * 获取文件后缀名称
     * @param fileName
     * @return
     */
    public static String getFileExtention(String fileName) {
        String extension = fileName.substring(fileName.lastIndexOf(".") + 1);
        return extension;
    }

    /**
     * 根据图片对象获取对应InputStream
     *
     * @param image
     * @param readImageFormat
     * @return
     * @throws IOException
     */
    public static InputStream getInputStream(BufferedImage image, String readImageFormat) throws IOException
    {
        ByteArrayOutputStream os = new ByteArrayOutputStream();
        ImageIO.write(image, readImageFormat, os);
        InputStream is = new ByteArrayInputStream(os.toByteArray());
        os.close();
        return is;
    }

    /**
     * 创建水印图片
     * @param text 水印文字
     * @param width 图片宽
     * @param height 图片高
     * @return
     */
    public static BufferedImage createWatermark(String text,int width,int height)  {
        BufferedImage image = new BufferedImage(width
                , height, BufferedImage.TYPE_INT_RGB);
        // 2.获取图片画笔
        Graphics2D g = image.createGraphics();
        // ----------  增加下面的代码使得背景透明  -----------------
        image = g.getDeviceConfiguration().createCompatibleImage(width, height, Transparency.TRANSLUCENT);
        g.dispose();
        g = image.createGraphics();
        // ----------  背景透明代码结束  -----------------
        // 6、处理文字
        AttributedString ats = new AttributedString(text);
        ats.addAttribute(TextAttribute.FONT, font, 0, text.length());
        AttributedCharacterIterator iter = ats.getIterator();
        // 7、设置对线段的锯齿状边缘处理
        g.setRenderingHint(RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BILINEAR);
        g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
        // 8、设置水印旋转
        g.rotate(Math.toRadians(-30));
        // 9、设置水印文字颜色
        g.setColor(color);
        // 10、设置水印文字Font
        g.setFont(font);
        // 11、设置水印文字透明度
        g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, alpha));
        g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER));
        /**
         * 水印铺满图片
         * 计算水印位置
         */
        int x = -width / 2;
        int y = -height / 2;
        int[] arr = getWidthAndHeight(text, font);
        int markWidth = arr[0];// 字体长度
        int markHeight = arr[1];// 字体高度
        // 循环添加水印
        while (x < width * 1.5) {
            y = -height / 2;
            while (y < height * 1.5) {
                g.drawString (text, x, y);
                y += markHeight + YMOVE;
            }
            x += markWidth + XMOVE;
        }
        // 13、释放资源
        g.dispose();
        return image;
    }
    /**
     * 计算字体宽度及高度
     * @param text
     * @param font
     * @return
     */
    private static int[] getWidthAndHeight(String text, Font font) {
        Rectangle2D r = font.getStringBounds(text, new FontRenderContext(
                AffineTransform.getScaleInstance(1, 1), false, false));
        int unitHeight = (int) Math.floor(r.getHeight());//
        // 获取整个str用了font样式的宽度这里用四舍五入后+1保证宽度绝对能容纳这个字符串作为图片的宽度
        int width = (int) Math.round(r.getWidth()) + 1;
        // 把单个字符的高度+3保证高度绝对能容纳字符串作为图片的高度
        int height = unitHeight + 3;
        return new int[]{width, height};
    }
```

# 七、minIO如何设置直接通过访问链接在浏览器中打开文件

[minIO如何设置直接通过访问链接在浏览器中打开文件](https://blog.csdn.net/iKaChu)

## **问题描述：**

在创建文件bucket后，上传html文件，发现访问文件链接，浏览器并不会如预期那样像访问静态网页一样打开测试报告，而是跳转到minIO系统，并且定位到该文件路径

## **解决方案：**

 

原因是因为minIO没有配置bucket策略，默认情况下，minIO没有配置匿名读写的权限

![img](https://img-blog.csdnimg.cn/20200428113908980.png)

 

 

![img](https://img-blog.csdnimg.cn/20200428113823897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lLYUNodQ==,size_16,color_FFFFFF,t_70)

如上图所示，在bucket菜单栏中点击Edit policy，新增Read权限，（Read Only 或 Read and Write均可），即可通过链接的方式直接访问该文件



## **备注：**

由于浏览器的限制，当上传文件时，设置header为application/octet-stream时，浏览器打开链接会默认进行下载而不是在浏览器中加载文件，所以如果想要文件时直接打开，上传时则不要设置application/octet-stream

application/octet-stream ： 二进制流数据（如常见的文件下载）

 

常见的媒体格式类型如下：

- text/html ： HTML格式
- text/plain ：纯文本格式
- text/xml ： XML格式
- image/gif ：gif图片格式
- image/jpeg ：jpg图片格式
- image/png：png图片格式

以application开头的媒体格式类型：

- application/xhtml+xml ：XHTML格式
- application/xml： XML数据格式
- application/atom+xml ：Atom XML聚合格式
- application/json： JSON数据格式
- application/pdf：pdf格式
- application/msword ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： &lt;form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）



