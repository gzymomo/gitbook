官方文档：https://www.getpostman.com/docs/v6/

 

**一、简介及安装**

postman是一款功能强大的网页调试和模拟发送HTTP请求的Chrome插件，支持几乎所有类型的HTTP请求，操作简单且方便。

下载地址：https://www.getpostman.com/

![img](https://images2018.cnblogs.com/blog/983980/201802/983980-20180227144840013-480244254.png)

下载成功后，默认安装即可。

 

**二、功能介绍**

启动后界面如下：

![img](https://images2018.cnblogs.com/blog/983980/201802/983980-20180227150832083-937188592.png)

左侧功能栏：History为近期的测试脚本历史记录；Collections为以postman官网API为例的脚本实例，也可以新建文件夹，用于放置不同测试脚本的文件集合；

主界面：可以选择HTTP请求的方法，填写URL、参数，cookie管理、脚本保存&另存为等功能。

 

**三、请求实例**

![img](https://images2018.cnblogs.com/blog/983980/201802/983980-20180227160911346-1018285393.png)

关于不同请求方法的字段说明：

**Authorization**：身份验证，主要用来填写用户名密码，以及一些验签字段；

**form-data**：对应信息头-multipart/form-data,它将表单数据处理为一条消息，以标签为单元用分隔符分开。既可以上传键值对，也可以上传文件（当上传字段是文件时，会有Content-Type来说明文件类型）；

**x-www-form-urlencoded**：对应信息头-application/x-www-from-urlencoded，会将表单内的数据转换为键值对，比如name=zhangsan；

**raw**：可以上传任意类型的文本，比如text、json、xml等；

**binary**：对应信息头-Content-Type:application/octet-stream，只能上传二进制文件，且没有键值对，一次只能上传一个文件；