- [腾讯上线零点巡航，用Java手撕一个人脸识别系统](https://blog.51cto.com/u_15157097/2989718)



# 人脸识别的十个关键技术

那开干之前，首先我们肯定要知道人脸识别实现需要攻破哪些难题，查了一些资料后总结了以下十个需要解决的关键

- **人脸检测（Face Detection）：是检测出图像中人脸所在位置的一项技术**
- **人脸配准（Face Alignment）：是定位出人脸上五官关键点坐标的一项技术**
- **人脸属性识别（Face Attribute）：是识别出人脸的性别、年龄、姿态、表情等属性值的一项技术**
- **人脸提特征（FaceFeatureExtraction）：是将一张人脸图像转化为一串固定长度的数值的过程**
- **人脸比对（Face Compare）：是衡量两个人脸之间相似度的算法**
- **人脸验证（Face Verification）：是判定两个人脸图是否为同一人的算法**
- **人脸识别（Face Recognition）：是识别出输入人脸图对应身份的算法**
- **人脸检索：是查找和输入人脸相似的人脸序列的算法**
- **人脸聚类（Face Cluster）：是将一个集合内的人脸根据身份进行分组的算法**
- **人脸活体（Face Liveness）：是判断人脸图像是来自真人还是来自攻。击假体(照片、视频等)的方法**

免费的人脸识别SDK： ArcSoft，地址：https://ai.arcsoft.com.cn

这个平台可以一键生成APPID、SDK KEY后续会用到，根据需要选择不同的环境（本文基于windows环境），然后下载SDK是一个压缩包。

**1、下载demo项目**

github地址：https://github.com/xinzhfiu/ArcSoftFaceDemo， 本地搭建数据库，创建表:user_face_info。这个表主要用来存人像特征，其中主要的字段 face_feature 用二进制类型 blob 存放人脸特征。

```html
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user_face_info
-- ----------------------------
DROP TABLE IF EXISTS `user_face_info`;
CREATE TABLE `user_face_info` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `group_id` int(11) DEFAULT NULL COMMENT '分组id',
  `face_id` varchar(31) DEFAULT NULL COMMENT '人脸唯一Id',
  `name` varchar(63) DEFAULT NULL COMMENT '名字',
  `age` int(3) DEFAULT NULL COMMENT '年纪',
  `email` varchar(255) DEFAULT NULL COMMENT '邮箱地址',
  `gender` smallint(1) DEFAULT NULL COMMENT '性别，1=男，2=女',
  `phone_number` varchar(11) DEFAULT NULL COMMENT '电话号码',
  `face_feature` blob COMMENT '人脸特征',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `fpath` varchar(255) COMMENT '照片路径',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `GROUP_ID` (`group_id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;
SET FOREIGN_KEY_CHECKS = 1;
1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.
```

**2、修改application.properties文件**

整个项目还是比较完整的，只需改一些配置即可启动，但有几点注意的地方，后边会重点说明。

config.arcface-sdk.sdk-lib-path： 存放SDK压缩包中的三个.dll文件的路径

config.arcface-sdk.app-id ： 开发者中心的APPID

config.arcface-sdk.sdk-key ：开发者中心的SDK Key

```html
config.arcface-sdk.sdk-lib-path=d:/arcsoft_lib
config.arcface-sdk.app-id=8XMHMu71Dmb5UtAEBpPTB1E9ZPNTw2nrvQ5bXxBobUA8
config.arcface-sdk.sdk-key=BA8TLA9vVwK7G6btJh2A2FCa8ZrC6VWZLNbBBFctCz5R

# druid  本地的数据库地址
spring.datasource.druid.url=jdbc:mysql://127.0.0.1:3306/xin-master?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC
spring.datasource.druid.username=junkang
spring.datasource.druid.password=junkang
1.2.3.4.5.6.7.8.
```

**3、根目录创建lib文件夹**

在项目根目录创建文件夹 lib,将下载的SDK压缩包中的arcsoft-sdk-face-2.2.0.1.jar放入项目根目录

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d4f7a9e60db4547bbf4ac40235b9f69~tplv-k3u1fbpfcp-zoom-1.image)

**4、引入arcsoft依赖包**

```html
 <dependency>
      <groupId>com.arcsoft.face</groupId>
      <artifactId>arcsoft-sdk-face</artifactId>
      <version>2.2.0.1</version>
      <scope>system</scope>
      <systemPath>${basedir}/lib/arcsoft-sdk-face-2.2.0.1.jar</systemPath>
</dependency>
1.2.3.4.5.6.7.
```

pom.xml文件要配置includeSystemScope属性，否则可能会导致arcsoft-sdk-face-2.2.0.1.jar引用不到

```html
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <includeSystemScope>true</includeSystemScope>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
1.2.3.4.5.6.7.8.9.10.11.12.
```

**5、启动项目**

到此为止配置完成，run Application文件启动

测试一下demo，如下页面即启动成功

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fc2e46aa6bb4880b597b7b8132d5972~tplv-k3u1fbpfcp-watermark.image)

# 开始操作

**1、录入人脸图像**

页面输入名称，点击摄像头注册调起本地摄像头，提交后将当前图像传入后台，识别提取当前人脸体征，保存至数据库。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d3c65ecefe247fd8707a33cef9edb7d~tplv-k3u1fbpfcp-zoom-1.image)

**2、人脸对比**

录入完人脸图像后测试一下能否识别成功，提交当前的图像，发现识别成功相似度92%。但是作为程序员对什么事情都要持怀疑的态度，这结果不是老铁在页面写死的吧？

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff5896b8a4cd402089b7135dea0a3f71~tplv-k3u1fbpfcp-watermark.image)
 为了进一步验证，这回把脸挡住再试一下，发现提示“人脸不匹配”，证明真的有进行比对。

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0122053401914e67a02de98782063d47~tplv-k3u1fbpfcp-zoom-1.image)

# 源码分析

简单看了一下项目源码，分析一下实现的过程：

页面和JS一看就是后端程序员写的，不要问我问为什么？懂的都懂，哈哈哈 ~ ，源码这里就不贴了，太累赘，感兴趣的可以着重去看看下面这三个部分。

**1、JS调起本地摄像头拍照，上传图片文件字符串**

**2、后台解析图片，提取人像特征**

后台解析前端传过来的图片，提取人像特征存入数据库，人像特征的提取主要是靠FaceEngine引擎，顺着源码一路看下去，自己才疏学浅实在是没懂具体是个什么样的算法。

**3、人像特征对比**

人脸识别：将前端传入的图像经过人像特征提取后，和库中已存在的人像信息对比分析

后台解析前端传过来的图片，提取人像特征存入数据库，人像特征的提取主要是靠FaceEngine引擎，顺着源码一路看下去，自己才疏学浅实在是没懂具体是个什么样的算法。

**整个人脸识别功能的大致流程图如下：**

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b67d4cd6feca4f3cadd36c96e5948805~tplv-k3u1fbpfcp-zoom-1.image)

