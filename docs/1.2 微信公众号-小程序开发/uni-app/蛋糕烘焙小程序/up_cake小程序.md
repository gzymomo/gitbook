# 微信小程序头部

![image-20210221083428820](http://lovebetterworld.com/image-20210221083428820.png)



# 一、首页

小程序进入后，默认进去到首页界面。





# 二、点餐

## 2.1 接口一：获取店铺信息

进入到点餐界面后，接口一获取店铺信息：

`/upCakeInterfaceAnonController/anon/queryStoreByStoreId?storeId=1`

![image-20210221083642081](http://lovebetterworld.com/image-20210221083642081.png)

获取photo（照片），isOpen（是否营业），mobile（联系方式），street（所在位置），storeName（店名），openTime（营业时间）

![image-20210221083932272](http://lovebetterworld.com/image-20210221083932272.png)



![image-20210221084224153](http://lovebetterworld.com/image-20210221084224153.png)



## 2.2 接口二：获取类别种类

进入到点餐界面获取的第二个接口：获取该店铺的类别种类

`/upCakeInterfaceAnonController/anon/queryTypesByStoreId?storeId=1`

这个最后的参数默认传storedId=1这个就行，因为只有这一个店铺。

![image-20210221084313133](http://lovebetterworld.com/image-20210221084313133.png)



## 2.3 接口三：获取具体商品

点击左侧类别，右侧显示对应的商品

`/upCakeInterfaceAnonController/anon/queryGoodsByTypesId?typesId=1`

商品的那个图片， 我会返回图片的uri。



## 2.4 接口四：添加至购物车

(订单这个我在想想)







# 三、取餐

## 3.1 接口一：查询未取的订单

进入到取餐界面，默认查询的时候，已经下的订单，但是取送状态为未取的。



## 3.2 接口二：查看历史订单

查看历史已经取了的订单。





# 四、我的

我的界面修改内容：

1. 去掉上方的绿色图片。

2. 去除多余功能

![image-20210221082951090](http://lovebetterworld.com/image-20210221082951090.png)



## 4.1 接口一：查看我的资料

（这个地方得先看看获取微信授权，能不能将那些信息存储，存储后在可以查询，得先调试获取微信信息接口）



## 4.2 接口二：查看收货地址

（根据会员Id查询收货地址列表）

