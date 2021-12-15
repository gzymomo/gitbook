**第一部分：目前工作中涉及到的content-type 有三种：**

content-type：在Request Headers里，告诉服务器我们发送的请求信息是哪种格式的。

 

**1 content-type:*****\*application/x-www-form-urlencoded\****

默认的。如果不指定content-type，默认使用此格式。

参数格式：key1=value1&key2=value2

**2 content-type:application/json**

参数为json格式 

{

 "key1":"value1",

 "key2":"value2"

}

**3 content-type:multipart/form-data [[dinghanhua](http://www.cnblogs.com/)]**

上传文件用这种格式

发送的请求示例：

**![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706114929592-1848412764.png)**

 

**第二部分  不同的content-type如何输入参数**

**1 content-type:\**\*\*application/x-www-form-urlencoded\*\**\***

***\**\*参数可以在Parameters或Body D\*\**\******\**\*ata里输入，格式不同，如下图所示。\*\**\***

***\**\*这两个参数输入的tab页只能使用一个，某一个有数据后不能切换到另一个。\*\**\***

***\**\*Parameters：\*\**\***

![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706140631280-2126976730.png)

 

**Body Data：**

![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706140642952-936303422.png)

 

**2  \**content-type:application/json\****

 **2.1 首先添加信息头管理。http请求上点击右键》添加》配置元件》 HTTP信息头管理器**

![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706132117452-1711304211.png)

 

**2.2  信息头编辑页面，点击添加，输入content-type \**application/json\****

![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706132206561-598962028.png)

 

**2.3 在http请求，Body Data中输入json格式的参数**

**![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706141147077-1029950898.png)**

 

**3 content-type:multipart/form-data [[dinghanhua](http://www.cnblogs.com/)]**

**在http请求编辑页面，选中Use multipart/form-data for POST**

**Parameters中输入除了上传的文件以外的参数：参数名和参数值**

**Files Upload中上传文件，参数名和MIME类型**

**![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706141904936-408051295.png)**

**![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706142039608-1397672682.png)**

上传文件如果不成功，修改Implementation为java试一下。

![img](https://images2015.cnblogs.com/blog/987451/201607/987451-20160706153729702-1930714302.png) 