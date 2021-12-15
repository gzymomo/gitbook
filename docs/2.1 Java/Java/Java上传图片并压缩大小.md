[java 上传图片 并压缩图片大小](https://www.cnblogs.com/miskis/p/5500822.html)



Thumbnailator 是一个优秀的图片处理的Google开源Java类库。处理效果远比Java API的好。从API提供现有的图像文件和图像对象的类中简化了处理过程，两三行代码就能够从现有图片生成处理后的图片，且允许微调图片的生成方式，同时保持了需要写入的最低限度的代码量。还支持对一个目录的所有图片进行批量处理操作。
支持的处理操作：图片缩放，区域裁剪，水印，旋转，保持比例。



Thumbnailator官网：http://code.google.com/p/thumbnailator/
下面我们介绍下如何使用Thumbnailator



### 缩略图压缩文件jar包

```xml
<!-- 图片缩略图 -->
<dependency>
    <groupId>net.coobird</groupId>
    <artifactId>thumbnailator</artifactId>
    <version>0.4.8</version>
</dependency>
```

 

## 按指定大小把图片进行缩放（会遵循原图高宽比例）

```java
//按指定大小把图片进行缩和放（会遵循原图高宽比例） 
//此处把图片压成400×500的缩略图
Thumbnails.of(fromPic).size(400,500).toFile(toPic);//变为400*300,遵循原图比例缩或放到400*某个高度
```

## 按照指定比例进行缩小和放大

```java
//按照比例进行缩小和放大
Thumbnails.of(fromPic).scale(0.2f).toFile(toPic);//按比例缩小
Thumbnails.of(fromPic).scale(2f);//按比例放大
```

图片尺寸不变，压缩图片文件大小

```java
//图片尺寸不变，压缩图片文件大小outputQuality实现,参数1为最高质量
Thumbnails.of(fromPic).scale(1f).outputQuality(0.25f).toFile(toPic);
```

我这里只使用了 图片尺寸不变，压缩文件大小 源码

```java
/**
     * 
     * @Description:保存图片并且生成缩略图
     * @param imageFile 图片文件
     * @param request 请求对象
     * @param uploadPath 上传目录
     * @return
     */
    public static BaseResult uploadFileAndCreateThumbnail(MultipartFile imageFile,HttpServletRequest request,String uploadPath) {
        if(imageFile == null ){
            return new BaseResult(false, "imageFile不能为空");
        }
        
        if (imageFile.getSize() >= 10*1024*1024)
        {
            return new BaseResult(false, "文件不能大于10M");
        }
        String uuid = UUID.randomUUID().toString();
        
        String fileDirectory = CommonDateUtils.date2string(new Date(), CommonDateUtils.YYYY_MM_DD);
        
        //拼接后台文件名称
        String pathName = fileDirectory + File.separator + uuid + "."
                            + FilenameUtils.getExtension(imageFile.getOriginalFilename());
        //构建保存文件路劲
        //2016-5-6 yangkang 修改上传路径为服务器上 
        String realPath = request.getServletContext().getRealPath("uploadPath");
        //获取服务器绝对路径 linux 服务器地址  获取当前使用的配置文件配置
        //String urlString=PropertiesUtil.getInstance().getSysPro("uploadPath");
        //拼接文件路劲
        String filePathName = realPath + File.separator + pathName;
        log.info("图片上传路径："+filePathName);
        //判断文件保存是否存在
        File file = new File(filePathName);
        if (file.getParentFile() != null || !file.getParentFile().isDirectory()) {
            //创建文件
            file.getParentFile().mkdirs();
        }
        
        InputStream inputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            inputStream = imageFile.getInputStream();
            fileOutputStream = new FileOutputStream(file);
            //写出文件
            //2016-05-12 yangkang 改为增加缓存
//            IOUtils.copy(inputStream, fileOutputStream);
            byte[] buffer = new byte[2048];
            IOUtils.copyLarge(inputStream, fileOutputStream, buffer);
            buffer = null;

        } catch (IOException e) {
            filePathName = null;
            return new BaseResult(false, "操作失败", e.getMessage());
        } finally {
            try {
                if (inputStream != null) {
                    inputStream.close();
                }
                if (fileOutputStream != null) {
                    fileOutputStream.flush();
                    fileOutputStream.close();
                }
            } catch (IOException e) {
                filePathName = null;
                return new BaseResult(false, "操作失败", e.getMessage());
            } 
         }
    
        
        //        String fileId = FastDFSClient.uploadFile(file, filePathName);
        
        /**
         * 缩略图begin
         */
        
        //拼接后台文件名称
        String thumbnailPathName = fileDirectory + File.separator + uuid + "small."
                                    + FilenameUtils.getExtension(imageFile.getOriginalFilename());
        //added by yangkang 2016-3-30 去掉后缀中包含的.png字符串 
        if(thumbnailPathName.contains(".png")){
            thumbnailPathName = thumbnailPathName.replace(".png", ".jpg");
        }
        long size = imageFile.getSize();
        double scale = 1.0d ;
        if(size >= 200*1024){
            if(size > 0){
                scale = (200*1024f) / size  ;
            }
        }
        
        
        //拼接文件路劲
        String thumbnailFilePathName = realPath + File.separator + thumbnailPathName;
        try {
            //added by chenshun 2016-3-22 注释掉之前长宽的方式，改用大小
//            Thumbnails.of(filePathName).size(width, height).toFile(thumbnailFilePathName);
            if(size < 200*1024){
                Thumbnails.of(filePathName).scale(1f).outputFormat("jpg").toFile(thumbnailFilePathName);
            }else{
                Thumbnails.of(filePathName).scale(1f).outputQuality(scale).outputFormat("jpg").toFile(thumbnailFilePathName);
            }
            
        } catch (Exception e1) {
            return new BaseResult(false, "操作失败", e1.getMessage());
        }
        /**
         * 缩略图end
         */
        
        Map<String, Object> map = new HashMap<String, Object>();
        //原图地址
        map.put("originalUrl", pathName);
        //缩略图地址
        map.put("thumbnailUrl", thumbnailPathName);
        return new BaseResult(true, "操作成功", map);
    }
```

 

获取当前使用的配置文件信息 

```java
  /**
     * 根据key从gzt.properties配置文件获取配置信息  
     * @param key 键值
     * @return
     */
    public String getSysPro(String key){
        return getSysPro(key, null);
    }
    /**
     * 根据key从gzt.properties配置文件获取配置信息  
     * @param key 键值
     * @param defaultValue 默认值
     * @return
     */
    public String getSysPro(String key,String defaultValue){
        return getValue("spring/imageserver-"+System.getProperty("spring.profiles.active")+".properties", key, defaultValue);
    }
```

例：

```java
//获取服务器绝对路径 linux 服务器地址   
String urlString=PropertiesUtil.getInstance().getSysPro("uploadPath");
```

###  PropertiesUtil 类

```java
package com.xyz.imageserver.common.properties;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 
 * @ClassName PropertiesUtil.java 
 * @Description 系统配置工具类
 * @author caijy
 * @date 2015年6月9日 上午10:50:38
 * @version 1.0.0
 */
public class PropertiesUtil {
    private Logger logger = LoggerFactory.getLogger(PropertiesUtil.class);
    private ConcurrentHashMap<String, Properties> proMap;
    private PropertiesUtil() {
        proMap = new ConcurrentHashMap<String, Properties>();
    }
    private static PropertiesUtil instance = new PropertiesUtil();

    /**
     * 获取单例对象
     * @return
     */
    public static PropertiesUtil getInstance()
    {
        return instance;
    }
   
    /**
     * 根据key从gzt.properties配置文件获取配置信息  
     * @param key 键值
     * @return
     */
    public String getSysPro(String key){
        return getSysPro(key, null);
    }
    /**
     * 根据key从gzt.properties配置文件获取配置信息  
     * @param key 键值
     * @param defaultValue 默认值
     * @return
     */
    public String getSysPro(String key,String defaultValue){
        return getValue("spring/imageserver-"+System.getProperty("spring.profiles.active")+".properties", key, defaultValue);
    }
    /**
     * 从配置文件中获取对应key值
     * @param fileName 配置文件名
     * @param key   key值
     * @param defaultValue 默认值
     * @return
     */
    public String getValue(String fileName,String key,String defaultValue){
        String val = null;
        Properties properties = proMap.get(fileName);
        if(properties == null){
            InputStream inputStream = PropertiesUtil.class.getClassLoader().getResourceAsStream(fileName);
              try {
                 properties = new Properties();
                properties.load(new InputStreamReader(inputStream,"UTF-8"));
                proMap.put(fileName, properties);
                val = properties.getProperty(key,defaultValue);
            } catch (IOException e) {
                logger.error("getValue",e);
            }finally{
                try {
                    if (inputStream != null) {                        
                        inputStream.close();
                    }
                } catch (IOException e1) {
                    logger.error(e1.toString());
                }
            }
        }else{
            val = properties.getProperty(key,defaultValue);
        }
        return val;
    }
}
```

