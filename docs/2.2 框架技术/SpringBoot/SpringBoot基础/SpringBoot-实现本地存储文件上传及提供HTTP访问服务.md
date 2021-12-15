服务端接收上传的目的是提供文件的访问服务，有哪些可以提供文件访问的静态资源目录呢？

- `classpath:/META-INF/resources/` ,
- `classpath:/static/` ,
- `classpath:/public/` ,
- `classpath:/resources/`

# 文件上传目录自定义配置

spring boot 为我们提供了**使用`spring.resources.static-locations`配置自定义静态文件的位置。**

```yml
web:
  upload-path: D:/data/

spring:
  resources:
    static-locations: classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,file:${web.upload-path}
```

- 配置`web.upload-path`为与项目代码分离的静态资源路径，即：文件上传保存根路径
- 配置`spring.resources.static-locations`，除了带上Spring Boot默认的静态资源路径之外，加上file:${web.upload-path}指向外部的文件资源上传路径。该路径下的静态资源可以直接对外提供HTTP访问服务。

# 文件上传的Controller实现

```java
@RestController
public class FileUploadController {

    //绑定文件上传路径到uploadPath
    @Value("${web.upload-path}")
    private String uploadPath;
 
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd/");
 
    @PostMapping("/upload")
    public String upload(MultipartFile uploadFile,
                         HttpServletRequest request) {

        // 在 uploadPath 文件夹中通过日期对上传的文件归类保存
        // 比如：/2019/06/06/cf13891e-4b95-4000-81eb-b6d70ae44930.png
        String format = sdf.format(new Date());
        File folder = new File(uploadPath + format);
        if (!folder.isDirectory()) {
            folder.mkdirs();
        }
 
        // 对上传的文件重命名，避免文件重名
        String oldName = uploadFile.getOriginalFilename();
        String newName = UUID.randomUUID().toString()
                + oldName.substring(oldName.lastIndexOf("."), oldName.length());
        try {
            // 文件保存
            uploadFile.transferTo(new File(folder, newName));
 
            // 返回上传文件的访问路径
            String filePath = request.getScheme() + "://" + request.getServerName()
                    + ":" + request.getServerPort()  + format + newName;
            return filePath;
        } catch (IOException e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR);
        }

    }
}
```

# 写一个模拟的文件上传页面，进行测试

把该upload.html文件放到classpath:public目录下，对外提供访问。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="uploadFile" value="请选择上传文件">
    <input type="submit" value="保存">
</form>
</body>
</html>
复制代码
```

访问测试、点击“选择文件”，之后保存

文件被保存到服务端的`web.upload-path`指定的资源目录下



![img](http://cdn.zimug.com/springboot-upload-2.png)



浏览器端响应结果如下，返回一个文件HTTP访问路径：



![img](http://cdn.zimug.com/springboot-upload-3.png)



使用该HTTP访问路径，在浏览器端访问效果如下。证明我们的文件已经成功上传到服务端，以后需要访问该图片就通过这个HTTP URL就可以了。