[Spring Cloud Alibaba(2)---RestTemplate微服务项目](https://www.cnblogs.com/qdhxhz/p/14647250.html)

```
前言
```

因为要运用 Spring Cloud Alibaba 开源组件到分布式项目中，所以这里先搭建一个不通过 Spring Cloud只通过 `RestTemplate` 来让SpringBoot和Mybatis进行模块之间

通讯，之后会在此基础上再添加Spring Cloud Alibaba框架及各个组件。

## 一、项目整体概述 

这里整理了一张图，代表接下来这个项目的模块划分，同时用到的一些组件

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210412203225410-19307354.jpg)

从这幅图可以看出整个项目所需要用到的组件有:

```
 GateWay(网关),
 Feign(服务调用),
 Nacos(注册中心+配置中心),
 Zipkin(链路追踪组件),
 Sentinel(流量控制组件)
```

一共创建了三个服务

```
商品微服务
订单微服务
用户微服务
```

这篇博客的目的就是搭建上面三个服务，而不添加任何微服务的组件。



## 二、项目环境和数据库设计 

上面已经说过，一共有三个微服务(**商品微服务,订单微服务,用户微服务**),所以这里也一共有三个数据库。

#### 1、商品微服务

**1).创建商品微服务的数据库**

```sql
CREATE DATABASE mall_goods
```

**2).创建商品表**

```sql
CREATE TABLE `goods` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `goods_name` varchar(524) DEFAULT NULL COMMENT '商品名称',
  `price` int DEFAULT NULL COMMENT '商品价格(分)',
  `goods_img` varchar(524) DEFAULT NULL COMMENT '商品封⾯图',
  `summary` varchar(1026) DEFAULT NULL COMMENT '概述',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

**3).表里插入几条模拟商品数据**

```sql
INSERT INTO `goods` (`id`, `goods_name`, `price`, `goods_img`, `summary`, `create_time`)
VALUES
	(1,'男士纯棉短袖T恤',1800,'www.txun.com','很休闲的一款T恤','2021-04-03 11:48:46'),
	(2,'2021秋季风衣女装',7200,'www.fy.com','很好的一件风衣','2021-04-03 11:48:46'),
	(3,'2021春装新款简约显瘦圆领连衣裙',3600,'www.lyq.com','很好一件连衣裙','2021-04-03 11:48:46');
```

#### 2、订单微服务

**1).创建订单微服务的数据库**

```sql
CREATE DATABASE mall_orders
```

**2).创建订单表**

```sql
CREATE TABLE `goods_order` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `order_no` varchar(64) DEFAULT NULL COMMENT '订单号',
  `total_fee` int DEFAULT NULL COMMENT '⽀付⾦额，单位分',
  `goods_id` int DEFAULT NULL COMMENT '商品ID',
  `goods_title` varchar(256) DEFAULT NULL COMMENT '商品标题',
  `goods_img` varchar(256) DEFAULT NULL COMMENT '商品图⽚',
  `user_id` int DEFAULT NULL COMMENT '⽤户id',
  `state` int DEFAULT NULL COMMENT '0表示未⽀付，1表示已⽀付',
  `create_time` datetime DEFAULT NULL COMMENT '订单⽣成时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

**3).表里插入几条模拟订单数据**

```sql
INSERT INTO `goods_order` (`id`, `order_no`, `total_fee`, `goods_id`, `goods_title`, `goods_img`, `user_id`, `state`, `create_time`)
VALUES
	(1,'2021033000001-1',1800,1,'男士纯棉短袖T恤','www.txun.com',1,1,'2021-04-03 11:48:46'),
	(2,'2021033000001-2',7200,2,'2021秋季风衣女装','www.fy.com',2,1,'2021-04-03 11:48:46');
```

#### 3、用户微服务

**1).创建用户微服务的数据库**

```sql
CREATE DATABASE mall_user
```

**2).创建用户表**

```sql
CREATE TABLE `user` (
	`id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
	`phone` varchar(32) DEFAULT NULL  COMMENT '手机号',
	`pwd` varchar(128) DEFAULT NULL  COMMENT '密码',
	`sex` int(2) DEFAULT NULL  COMMENT '性别',
	`img` varchar(128) DEFAULT NULL  COMMENT '头像',
	`username` varchar(128) DEFAULT NULL  COMMENT '用户名',
    `create_time` datetime DEFAULT NULL   COMMENT '创建时间',
	PRIMARY KEY (`id`)
) ENGINE = InnoDB AUTO_INCREMENT = 1 CHARSET = utf8mb4;
```

**3).表里插入几条用户数据**

```sql
INSERT INTO `user` (`id`, `phone`, `pwd`, `sex`, `img`, `username`, `create_time`)
VALUES
	(1,'18812345678','123456',1,'www.touxiang.com','小小','2021-04-03 11:48:46'),
	(2,'18887654321','654321',2,'www.touxiang.com','张三','2021-04-03 11:48:46');
```



## 三、项目搭建 

#### 1、技术架构

项目总体技术选型

```
SpringBoot2.3.3 + Maven3.5.4 +JDK8 
```

#### 2、项目整体结构

```
mall-parent  #父工程
 |
  ---mall-common #公共模块
 |
  ---mall-goods  #商品服务(端口:6001)
 |
  ---mall-order  #订单服务(端口:7001)
 |
  ---mall-user   #用户服务(端口:8001)
```

`说明` 有关项目具体的代码我这边就不放上去了,文章最下面会附上GitHub地址。



## 四、测试

这里主要测试两点

```
1）各模块连接数据库是否成功
2）订单服务调商品服务接口能否成功
```

#### 1、订单接口代码

```java
@RestController
@RequestMapping("api/v1/goods_order")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("getGoods")
    public Object getGoods(int goodsId) {
        Goods goods = restTemplate.getForObject("http://localhost:6001/api/v1/goods/findByGoodsId?goodsId=" + goodsId, Goods.class);
        return goods;
    }
}
```

#### 2、测试

通过postMan测试

![img](https://img2020.cnblogs.com/blog/1090617/202104/1090617-20210412203927820-274885588.jpg)

通过图片可以说明

```
1）商品服务连接数据库成功
2）订单服务调商品服务成功
```

`说明` 这只是一个学习的项目，所以比较简单,下面就是一步一步添加spring cloud alibaba 框架及相应组件。

`github地址`：[RestTemplateStudy](https://github.com/zhangzhizhong520/RestTemplateStudy)