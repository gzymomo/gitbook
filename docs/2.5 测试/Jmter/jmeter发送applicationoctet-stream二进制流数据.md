http接口请求header里面 content-type: application/octet-stream (二进制流数据)，如何用jmeter发送请求？

 

1 添加http请求头

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181020194632366-1235806424.png)

 

2 http请求 files upload里面写上文件的绝对地址

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181020194758990-975707322.png)

 

发送的文件内容：1 由开发提供的文件 2 有的是通过fiddler抓包获取的二进制流拷贝到文件里保存