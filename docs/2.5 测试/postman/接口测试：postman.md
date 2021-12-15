# [接口测试：postman](https://www.cnblogs.com/uncleyong/p/11268846.html)

下面主要介绍postman测试http协议接口的用法，包含get，post（form-data，json，上传文件，cookie）。

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman--get请求

参数拼接在url后面

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730141301464-2028903323.png)

下面分别表示http响应状态码、请求耗时，响应大小，而上面的code=9630是程序内部定义的状态码

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190731090137249-865389851.png)

右上角code

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730142140892-678708972.png) 

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman--post请求：form-data

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123140020-1725318843.png)

cookies，response.set_cookie(username,token)

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123215479-1767240519.png)

headers

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123238640-1909910145.png)

右上角cookies

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122403961-1487289012.png)

可以删除cookie

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122335742-1017252404.png)

右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

请求体里不同的input之间用一段叫boundary的字符串分割，每个input都有了自己一个小header，其后空行接着是数据

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122455779-541436215.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### post请求--postman：x-www-form-urlencoded

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123349957-1123311913.png)

自动添加上了请求头 

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123454108-2031280338.png)

cookies

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123625450-58600034.png)

headers 

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123645456-1069287171.png)

 右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

将input的name、value用‘=’连接，不同的input之间用‘&’连接

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122728704-2055639091.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman：上传文件

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730123945607-1622648797.png)

右上角code

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124016945-994928750.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman：发json

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124141940-1213865599.png)

自动加入了请求头信息

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124206283-698862028.png)

 右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124232956-2145557090.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman：cookie中传token

token是登录返回的，add_user3这个功能必须先要登录

**特别说明**：实际测试过程中，如果token失效时间很长，可以像下面获取到token后写死；但是，最好是通过关联，动态获取

postman动态获取参考：https://www.cnblogs.com/uncleyong/p/10991383.html

此篇重点不是关联，所以token写死

请求头信息

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124746604-2057350732.png)

 请求

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124636290-351973933.png)

右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

可以看到，token在cookie中

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124656531-1367051562.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman：form-data，body中传token

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124855185-328884184.png)

右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

请求内容

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730124915815-55793357.png)

[回到顶部](https://www.cnblogs.com/uncleyong/p/11268846.html#_labelTop)

### postman：json，body中传token

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730125348298-2015038044.png)

自动加上了请求头信息

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730125401507-1776581129.png)

右上角code

 ![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730122423437-951414872.png)

![img](https://img2018.cnblogs.com/blog/1024732/201907/1024732-20190730125423548-1117742446.png)

 

至此，postman测试http协议接口的主要使用方法介绍完了。

postman更多功能，参考：https://www.cnblogs.com/UncleYong/p/10991383.html