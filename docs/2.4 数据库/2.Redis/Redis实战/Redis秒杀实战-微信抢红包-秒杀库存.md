[Redis秒杀实战-微信抢红包-秒杀库存](https://www.cnblogs.com/chenyanbin/p/13587508.html)



# 微信抢红包实现原理[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#微信抢红包实现原理)

## 业务流程分析 [![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200823231516978-1835079871.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200823231516978-1835079871.png)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#业务流程分析 )

## 功能拆解[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#功能拆解)

### 新建红包[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#新建红包)

　　**在DB**、**Redis**分别**新增一条记录**

### 抢红包(并发)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#抢红包(并发))

　　**请求Redis**，**红包剩余个数**，**大于0**才可以**拆**，**等会0**时，提示用户，**红包已抢完**

### 拆红包(并发)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#拆红包(并发))

#### 用到技术

　　**Redis**中数据类型的**String特性**的**原子递减**（**DECR key**）和**减少指定值**（**DECRBY key decrement**）

#### 业务

1. **请求Redis**，当**剩余红包个数大于0**，**红包个数**原子**递减**，随机**获取红包**
2. **计算金额**，当最后一个红包时，最后一个红包金额=总金额-总已抢红包金额
3. **更新数据库**

### 查看红包记录[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#查看红包记录)

　　**查询DB**即可

## 数据库表设计[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#数据库表设计)

红包流水表

```
CREATE TABLE `red_packet_info` (
 `id` int(11) NOT NULL AUTO_INCREMENT, 
 `red_packet_id` bigint(11) NOT NULL DEFAULT 0 COMMENT '红包id，采⽤
timestamp+5位随机数', 
 `total_amount` int(11) NOT NULL DEFAULT 0 COMMENT '红包总⾦额，单位分',
 `total_packet` int(11) NOT NULL DEFAULT 0 COMMENT '红包总个数',
 `remaining_amount` int(11) NOT NULL DEFAULT 0 COMMENT '剩余红包⾦额，单位
分',
 `remaining_packet` int(11) NOT NULL DEFAULT 0 COMMENT '剩余红包个数',
 `uid` int(20) NOT NULL DEFAULT 0 COMMENT '新建红包⽤户的⽤户标识',
 `create_time` timestamp COMMENT '创建时间',
 `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE
CURRENT_TIMESTAMP COMMENT '更新时间',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='红包信息
表，新建⼀个红包插⼊⼀条记录';
```

红包记录表

```
CREATE TABLE `red_packet_record` (
 `id` int(11) NOT NULL AUTO_INCREMENT, 
 `amount` int(11) NOT NULL DEFAULT '0' COMMENT '抢到红包的⾦额',
 `nick_name` varchar(32) NOT NULL DEFAULT '0' COMMENT '抢到红包的⽤户的⽤户
名',
 `img_url` varchar(255) NOT NULL DEFAULT '0' COMMENT '抢到红包的⽤户的头像',
 `uid` int(20) NOT NULL DEFAULT '0' COMMENT '抢到红包⽤户的⽤户标识',
 `red_packet_id` bigint(11) NOT NULL DEFAULT '0' COMMENT '红包id，采⽤
timestamp+5位随机数', 
 `create_time` timestamp COMMENT '创建时间',
 `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE
CURRENT_TIMESTAMP COMMENT '更新时间',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='抢红包记
录表，抢⼀个红包插⼊⼀条记录';
```

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200824215520831-273722339.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200824215520831-273722339.png)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200824215539597-2123635438.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200824215539597-2123635438.png)

## 发红包API[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#发红包api)

### 发红包接口开发[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#发红包接口开发)

- 新增一条红包记录
- **往mysql**里面添**加一条红包记录**
- **往redis**里面添**加一条红包数量记录**
- **往redis**里面添**加一条红包金额记录**

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200825221041111-1539712425.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200825221041111-1539712425.png)

　　注意，往db中就单纯存入一条记录，Service层和Mapper层，就简单的一条sql语句，主要是提供思路，下面会附案例源码，不要慌

## 抢红包API[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#抢红包api)

- **抢红包**功能属于**原子减**操作
- 当大小小于0时原子减失败
- 当**红包个数为0**时，**后**面**进来**的用户**全部抢红包失败**，并不会进入拆红包环节
- 抢红包功能设计
  - 将红包ID的请求放入请求队列中，如果发现超过红包的个数，直接返回
- 注意事项
  - **抢到红包不一定能拆成功**

####  抢红包算法拆解

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200825225651119-2111039266.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200825225651119-2111039266.png)

　　通过**上图算法得出**，**靠前**面的人，**手气最佳几率小**，**手气最佳，往往在后面**

1. 发100元，共10个红包，那么平均值是10元一个，那么发出来的红包金额在0.01~20元之间波动
2. 当前面4个红包总共被领了30元时，剩下70元，总共6个红包，那么这6个红包的金额在0.01~23.3元之间波动

### 抢红包接口开发[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#抢红包接口开发)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826003430520-814368941.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826003430520-814368941.png)

## 测试[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#测试)

### 发红包[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#发红包)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826003712834-1793224326.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826003712834-1793224326.png)

### 模拟高并发抢红包(Jmeter压测工具)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#模拟高并发抢红包(jmeter压测工具))

　　因为我发了**10个红包**，**金额是20000**，使用压测工具，**模拟50个请求**，**只允许前10个请求能抢到红包**，**并且金额等于20000**。

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826004251579-174033303.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826004251579-174033303.png)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826010229396-1071712151.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826010229396-1071712151.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826010257741-1322162855.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200826010257741-1322162855.gif)

# 布隆过滤器(重要)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#布隆过滤器(重要))

## 介绍[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#介绍)

　　布隆过滤器是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

### 优点[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#优点)

　　相比于其他的数据结构，布隆过滤器在空间和时间方面都有巨大的优势。布隆过滤器存储空间和插入/查询时间都是常数。另外三列函数相互之间没有关系，方便由硬件并行实现。布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势。

### 缺点[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#缺点)

　　但是布隆过滤器的缺点和有点一样明显。误算率是其中之一。随着存入的元素数量增加，误算率随之增加。但是如果元素数量太少，则使用散列表足矣。

## 布隆过滤器有什么用？[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#布隆过滤器有什么用？)

1. 黑客流量攻击：故意访问不存在的数据，导致查程序不断访问DB的数据
2. 黑客安全阻截：当黑客访问不存在的缓存时迅速返回避免缓存及DB挂掉
3. 网页爬虫对URL的去重，避免爬取相同的URL地址
4. 反垃圾邮件，从数十亿个垃圾邮件列表中判断某邮件是否垃圾邮件(同理，垃圾短信)
5. 缓存击穿，将已存在的缓存放到布隆中，当黑客访问不存在的缓存时迅速返回避免缓存及DB挂掉

# 布隆过滤器实现会员转盘抽奖[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#布隆过滤器实现会员转盘抽奖)

## 需求[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#需求)

　　一个抽奖程序，只针对会员用户有效

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829220858193-750214193.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829220858193-750214193.png)

 

## 通过google布隆过滤器存储会员数据[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#通过google布隆过滤器存储会员数据)

1. 程序启动时将数据放入内存中
2. google自动创建布隆过滤器
3. 用户ID进来之后判断是否是会员

## 代码实现[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#代码实现)

### 引入依赖[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#引入依赖)

```
        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>29.0-jre</version>
        </dependency>
```

### 数据库会员表[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#数据库会员表)

```
CREATE TABLE `sys_user` (
 `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
 `user_name` varchar(11) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '⽤户名',
 `image` varchar(11) CHARACTER SET utf8mb4 DEFAULT NULL COMMENT '⽤户头像',
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8;
```

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829224927351-1211103818.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829224927351-1211103818.png)

### 初始化布隆过滤器[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#初始化布隆过滤器)

　　dao层和dao映射文件，就单纯的一个sql查询，看核心方法，下面会附源码滴，不要慌好嘛

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829230609455-250799074.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829230609455-250799074.png)

### 控制层[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#控制层)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829230349665-1488553948.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829230349665-1488553948.png)

### 测试 [#](https://www.cnblogs.com/chenyanbin/p/13587508.html#测试 )

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829231202897-1711665258.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829231202897-1711665258.gif)

## 缺点[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#缺点)

1. 内存级别产部
2. 重启即失效
3. 本地内存无法用在分布式场景
4. 不支持大数据量存储

# Redis布隆过滤器[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#redis布隆过滤器)

## 优点[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#优点)

1. 可扩展性Bloom过滤器
2. 不存在重启即失效或定时任务维护的成本

## 缺点[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#缺点)

1. 需要网络IO，性能比基于内存的过滤器低

## 布隆过滤器安装[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#布隆过滤器安装)

### 下载[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#下载)

github：https://github.com/RedisBloom/RedisBloom

```
链接: https://pan.baidu.com/s/16DlKLm8WGFzGkoPpy8y4Aw  密码: 25w1
```

### 编译[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#编译)

make

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829233530850-1719644224.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200829233530850-1719644224.png)

### 将Rebloom加载到Redis中[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#将rebloom加载到redis中)

　　**先把Redis给停掉**！！！在redis.conf里面添加一行命令->加载模块

```
loadmodule /usr/soft/RedisBloom-2.2.4/redisbloom.so
```

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830001318055-1077987772.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830001318055-1077987772.png)

### 测试布隆过滤器[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#测试布隆过滤器)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830003122751-732001989.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830003122751-732001989.png)

# SpringBoot整合Redis布隆过滤器(重点)[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#springboot整合redis布隆过滤器(重点))

## 编写两个lua脚本[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#编写两个lua脚本)

1. 添加数据到指定名称的布隆过滤器
2. 从指定名称的布隆过滤器获取key是否存在的脚本

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010432997-1454760482.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010432997-1454760482.png)

```
local bloomName = KEYS[1]
local value = KEYS[2]
--bloomFilter
local result_1 = redis.call('BF.ADD',bloomName,value)
return result_1
```

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010504965-1476109069.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010504965-1476109069.png)

```
local bloomName = KEYS[1]
local value = KEYS[2]
--bloomFilter
local result_1 = redis.call('BF.EXISTS',bloomName,value)
return result_1
```

## 在RedisService.java中添加2个方法[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#在redisservice.java中添加2个方法)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010402640-1468793536.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830010402640-1468793536.png)

## 验证[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#验证)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830011447143-619868788.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830011447143-619868788.gif)

# 秒杀系统设计[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#秒杀系统设计)

## 秒杀业务流程图[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#秒杀业务流程图)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830020241284-1860097792.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830020241284-1860097792.png)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830020401786-2031329226.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200830020401786-2031329226.png)

## 数据落地存储方案[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#数据落地存储方案)

1. 通过分布式redis减库存
2. DB存最终订单信息数据

## API性能调优[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#api性能调优)

1. 性能瓶颈在高并发秒杀
2. 技术难题在于超卖问题

## 实现步骤[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#实现步骤)

1、**提前将秒杀数据缓存到redis**

```
set skuId_start_1 0_1554045087 --秒杀标识
set skuId_access_1 12000 --允许抢购数
set skuId_count_1 0 --抢购计数
set skuId_booked_1 0 --真实秒杀数
```

1. 秒杀开始前，skuId_start为0，代表活动未开始
2. 当skuId_start改为1时，活动开始，开始秒杀叭
3. 当接受下单数达到sku_count*1.2后，继续拦截所有请求，商品剩余数量为0(为啥接受抢购数为1万2呢，看业务流程图，涉及到“校验订单信息”，一般设置的值要比总数多一点，多多少自己定)

2、**利用Redis缓存加速增库存数**

```
"skuId_booked":10000 //从0开始累加，秒杀的个数只能加到1万
```

3、将**用户订单数据写入MQ**(异步方式)，可以看我另外一篇博客：[点我直达](https://www.cnblogs.com/chenyanbin/p/12841966.html)

4、**另外一台服务器监听mq，将订单信息写入到DB**

好了，以上就是完整的开发步骤，下面我们开始编写代码

## 代码实战[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#代码实战)

### 网关浏览拦截层[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#网关浏览拦截层)

1、先判断秒杀是否已经开始

2、利用Redis缓存incr拦截流量

- 用incr方法原子加
- 通过原子加判断当前skuId_access是否达到最大值

### 订单信息校验层[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#订单信息校验层)

1、校验当前用户是否已经买过这个商品

- 需要存储用户的uid
- 存数据库效率太低
- 存Redis value方式数据太大
- 存布隆过滤器性能高且数据量小(**推荐**)

2、校验通过直接返回抢购成功

### 开发lua脚本实现库存扣除[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#开发lua脚本实现库存扣除)

1、库存扣除成功，获取当前最新库存

2、如果库存大于0，即马上进行库存扣除，并且访问抢购成功给用户

3、考虑原子性问题

- 保证原子性的方式，采用lua脚本
- 采用lua脚本方式保证原子性带来缺点，性能有所下降
- 不保证原子性缺点，放入请求量可能大于预期
- **当前扣除库存场景必须保证原子性，否则会导致超卖**

4、返回抢购结果

- 抢购成功
- 库存没了，抢购失败

#### 控制层

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831001728354-502448448.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831001728354-502448448.png)

 

#### Service层

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831005825463-315453517.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831005825463-315453517.gif)

#### 布隆过滤器

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831002230398-1009185033.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831002230398-1009185033.png)

## 初始化redis缓存[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#初始化redis缓存)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831004622522-2143383176.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831004622522-2143383176.png)

```
set skuId_start_1 0_1554045087 --秒杀标识
set skuId_access_1 12000 --允许抢购数
set skuId_count_1 0 --抢购计数
set skuId_booked_1 0 --真实秒杀数
```

## 秒杀验证[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#秒杀验证)

jmeter配置

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831002356412-2109783163.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831002356412-2109783163.png)

压测秒杀验证原子性

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831010939971-298803856.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831010939971-298803856.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011010919-1139645498.gif)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011010919-1139645498.gif)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011332832-1207871942.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011332832-1207871942.png)

# 项目下载[#](https://www.cnblogs.com/chenyanbin/p/13587508.html#项目下载)

[![img](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011727140-1231393118.png)](https://img2020.cnblogs.com/blog/1504448/202008/1504448-20200831011727140-1231393118.png)

```
链接: https://pan.baidu.com/s/1hZUPRAljkqO05fYluqJBhQ  密码: 1iwr
```