[TOC]

# 1、FastDFS简介
FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件上传、文件下载等，解决了大容量存储和负载均衡的问题。

## 1.1 核心角色
FastDFS是由跟踪服务器（trackerserver）、存储服务器（storageserver）和客户端（client）三个部分组成。

1. 跟踪服务器
FastDFS的协调者，负责管理所有的storage server和group，每个storage在启动后会连接Tracker，告知自己所属的group等信息，并保持周期性的心跳，tracker根据storage的心跳信息，建立group到[storage server list]的映射表。

2. 存储服务器
以组（group）为单位，一个group内包含多台storage机器，数据互为备份，存储空间以group内容量最小的storage为准，所以建议group内的多个storage尽量配置相同，以免造成存储空间的浪费。

3. 客户端
业务请求的发起方，通过专有接口，使用TCP/IP协议与跟踪器服务器或存储节点进行数据交互。

## 1.2 运转流程
1. 存储服务定时向跟踪服务上传状态信息；
2. 客户端发起请求；
3. 跟踪器同步存储器状态，返回存储服务端口和IP;
4. 客户端执行文件操作（上传，下载）等。

# 2、SpringBoot整合FastDFS
## 2.1 核心步骤
```java
1)、配置FastDFS执行环境
2)、文件上传配置
3)、整合Swagger2测试接口
```
## 2.2 核心依赖
```xml
<!-- FastDFS依赖 -->
<dependency>
    <groupId>com.github.tobato</groupId>
    <artifactId>fastdfs-client</artifactId>
    <version>1.26.5</version>
</dependency>
<!-- Swagger2 核心依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.1</version>
</dependency>
```
## 2.3 配置FastDFS
0) 核心配置文件
```yml
fdfs:
  # 链接超时
  connect-timeout: 60
  # 读取时间
  so-timeout: 60
  # 生成缩略图参数
  thumb-image:
    width: 150
    height: 150
  tracker-list: 192.168.72.130:22122
```
1) 核心配置类
```java
@Configuration
@Import(FdfsClientConfig.class)
// Jmx重复注册bean的问题
@EnableMBeanExport(registration = RegistrationPolicy.IGNORE_EXISTING)
public class DfsConfig {
}
```
2）文件工具类
```java
@Component
public class FileDfsUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(FileDfsUtil.class);
    @Resource
    private FastFileStorageClient storageClient ;
    /**
     * 上传文件
     */
    public String upload(MultipartFile multipartFile) throws Exception{
        String originalFilename = multipartFile.getOriginalFilename().
                                  substring(multipartFile.getOriginalFilename().
                                  lastIndexOf(".") + 1);
        StorePath storePath = this.storageClient.uploadImageAndCrtThumbImage(
                              multipartFile.getInputStream(),
                              multipartFile.getSize(),originalFilename , null);
        return storePath.getFullPath() ;
    }
    /**
     * 删除文件
     */
    public void deleteFile(String fileUrl) {
        if (StringUtils.isEmpty(fileUrl)) {
            LOGGER.info("fileUrl == >>文件路径为空...");
            return;
        }
        try {
            StorePath storePath = StorePath.parseFromUrl(fileUrl);
            storageClient.deleteFile(storePath.getGroup(), storePath.getPath());
        } catch (Exception e) {
            LOGGER.info(e.getMessage());
        }
    }
}
```
## 2.4 文件上传配置
```yml
spring:
  application:
    name: ware-fast-dfs
  servlet:
    multipart:
      enabled: true
      max-file-size: 10MB
      max-request-size: 20MB
```
## 2.5 配置Swagger2
主要用来生成文件上传的测试界面。

1）配置代码类
```java
@Configuration
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.fast.dfs"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SpringBoot利用Swagger构建API文档")
                .description("使用RestFul风格, 创建人：知了一笑")
                .termsOfServiceUrl("https://github.com/cicadasmile")
                .version("version 1.0")
                .build();
    }
}
```
2）启动类注解
`@EnableSwagger2`

# 3、演示案例
## 3.1 接口代码
```java
@RestController
public class FileController {
    @Resource
    private FileDfsUtil fileDfsUtil ;
    /**
     * 文件上传
     */
    @ApiOperation(value="上传文件", notes="测试FastDFS文件上传")
    @RequestMapping(value = "/uploadFile",headers="content-type=multipart/form-data", method = RequestMethod.POST)
    public ResponseEntity<String> uploadFile (@RequestParam("file") MultipartFile file){
        String result ;
        try{
            String path = fileDfsUtil.upload(file) ;
            if (!StringUtils.isEmpty(path)){
                result = path ;
            } else {
                result = "上传失败" ;
            }
        } catch (Exception e){
            e.printStackTrace() ;
            result = "服务异常" ;
        }
        return ResponseEntity.ok(result);
    }
    /**
     * 文件删除
     */
    @RequestMapping(value = "/deleteByPath", method = RequestMethod.GET)
    public ResponseEntity<String> deleteByPath (){
        String filePathName = "group1/M00/00/00/wKhIgl0n4AKABxQEABhlMYw_3Lo825.png" ;
        fileDfsUtil.deleteFile(filePathName);
        return ResponseEntity.ok("SUCCESS") ;
    }
}
```
## 3.2 执行流程
```java
1、访问http://localhost:7010/swagger-ui.html测试界面
2、调用文件上传接口，拿到文件在FastDFS服务的路径
3、浏览器访问：http://192.168.72.130/group1/M00/00/00/wKhIgl0n4AKABxQEABhlMYw_3Lo825.png
4、调用删除接口，删除服务器上图片
5、清空浏览器缓存，再次访问图片Url,返回404
```