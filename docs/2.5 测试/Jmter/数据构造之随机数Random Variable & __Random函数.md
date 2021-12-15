 **接口测试有时参数使用随机数构造。jmeter添加随机数两种方式**

**1 添加配置 》 Random Variable** 

**2 __Random函数  ${__Random(1000,9999)}**

 

 

**方式一 R\**andom Variable\**** 

 

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021011542213-89113153.png)

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021012028532-362523900.png)

 

 

**方式二 __Random()函数**

 

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021011726423-1678691025.png)

 

 

添加http请求，2个参数：订单号，用户分别是两种方式生成的。

订单号 = 日期+__Random函数生成随机数

用户名= 随机变量输出的固定格式随机数

```
random_function    orderid_${__time(yyyyMMddHHmmss)}${__Random(1000,9999)}
random_variable    ${rand}
```

 

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021012714674-1448364711.png)

 

添加结果树，多运行几次，查看生成的订单号和用户名

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021013035592-960482329.png)

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021013046252-1572712222.png)

![img](https://img2018.cnblogs.com/blog/987451/201810/987451-20181021013112738-855398088.png)