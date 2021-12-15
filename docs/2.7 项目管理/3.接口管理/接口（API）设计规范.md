- [接口设计规范](https://blog.csdn.net/weixin_38229356/article/details/83155559)
- [接口规范](https://www.jianshu.com/p/2c60c9f5f668)
- [Eolinker接口规范-教程系列](https://www.cnblogs.com/dc20181010/tag/API/)



优秀的设计是产品变得卓越的原因。设计API意味着提供有效的接口，可以帮助API使用者更好地了解、使用和集成，同时帮助人们有效地维护它。每个产品都需要使用手册，API也不例外。
 在API领域，可以将设计视为服务器和客户端之间的协议进行建模。API协议可以帮助内部和外部的利益相关者理解应该做什么，以及如何更好地协同工作来构建一个出色的API。

- **什么是接口?**

接口全称是应用程序编程接口，是应用程序重要的组成部分。接口可以是一个功能，例如天气查询，短信群发等，接口也可以是一个模块，例如登录验证。接口通过发送请求参数至接口url，经过后台代码处理后，返回所需的结果。

- **为什么需要编写接口文档?**

由于接口所包含的内容比较细，在项目中常常需要使用接口文档。研发人员可以根据接口文档进行开发、协作，测试人员可以根据接口文档进行测试，系统也需要参照接口文档进行维护等。



![image-20210702094009859](https://gitee.com/AiShiYuShiJiePingXing/img/raw/master/img/image-20210702094009859.png)

# 一、接口（API）设计规范

## 1.1 基本规范

### 1.1.1 公共参数

公共参数是每个接口都要携带的参数，描述每个接口的基本信息，用于统计或其他用途，放在header或url参数中。例如：

| 字段名称 | 说明                     |
| -------- | ------------------------ |
| version  | 客户端版本。1.0.0        |
| token    | 登录令牌                 |
| os       | 手机系统版本。12         |
| from     | 请求来源。android/ios/h5 |
| screen   | 手机尺寸。1080*1920      |
| model    | 机型。IPhone7            |
| net      | 网络状态。wifi           |

### 1.1.2 响应数据

为了方便给客户端响应，响应数据会包含三个属性，状态码（code）,信息描述（message）,响应数据（data）。客户端根据状态码及信息描述可快速知道接口，如果状态码返回成功，再开始处理数据。

array类型数据。通过list字段，保证data的Object结构。

分页类型数据。返回总条数，用于判断是否可以加载更多。

```json
// object类型数据
{
    "code":1,
    "msg":"成功",
    "data":{}
}
// array类型数据。
{
    "code":1,
    "msg":"成功",
    "data":{
        "list":[]
    }
}
// 分页类型数据。
{
    "code":1,
    "msg":"成功",
    "data":{
        "list":[]
        "total":"10"
    }
}
```

列表类数据接口，无论是否要求分页，最好支持分页，pageSize=Integer.Max即可。

响应结果定义及常用方法：

```java
public class R implements Serializable {

    private static final long serialVersionUID = 793034041048451317L;

    private int code;
    private String message;
    private Object data = null;

    public int getCode() {
        return code;
    }
    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }

    public Object getData() {
        return data;
    }

    /**
     * 放入响应枚举
     */
    public R fillCode(CodeEnum codeEnum){
        this.setCode(codeEnum.getCode());
        this.setMessage(codeEnum.getMessage());
        return this;
    }

    /**
     * 放入响应码及信息
     */
    public R fillCode(int code, String message){
        this.setCode(code);
        this.setMessage(message);
        return this;
    }

    /**
     * 处理成功，放入自定义业务数据集合
     */
    public R fillData(Object data) {
        this.setCode(CodeEnum.SUCCESS.getCode());
        this.setMessage(CodeEnum.SUCCESS.getMessage());
        this.data = data;
        return this;
    }
}
```

### 1.1.3 字段类型规范

统一使用String类型。某些情况，统一使用String可以防止解析失败，减少类型转化操作。

Boolean类型，1是0否。客户端处理时，非1都是false。

```java
if("1".equals(isVip)){
    
}else{
    
}
```

status类型字段，从1+开始，区别Boolean的0和1。“0”有两种含义，（1）Boolean类型的false，（2）默认的status

### 1.1.4 上传/下载

上传/下载，参数增加文件md5，用于完整性校验（传输过程可能丢失数据）。

### 1.1.5 避免精度丢失

缩小单位保存数据，如：钱以分为单位、距离以米为单位。

## 1.2 调用接口的先决条件-token

获取token一般会涉及到几个参数`appid`，`appkey`，`timestamp`，`nonce`，`sign`。我们通过以上几个参数来获取调用系统的凭证。

`appid`和`appkey`可以直接通过平台线上申请，也可以线下直接颁发。`appid`是全局唯一的，每个`appid`将对应一个客户，`appkey`需要高度保密。

`timestamp`是时间戳，使用系统当前的unix时间戳。时间戳的目的就是为了减轻DOS攻击。防止请求被拦截后一直尝试请求接口。服务器端设置时间戳阀值，如果请求时间戳和服务器时间超过阀值，则响应失败。

`nonce`是随机值。随机值主要是为了增加`sign`的多变性，也可以保护接口的幂等性，相邻的两次请求`nonce`不允许重复，如果重复则认为是重复提交，响应失败。

`sign`是参数签名，将`appkey`，`timestamp`，`nonce`拼接起来进行md5加密（当然使用其他方式进行不可逆加密也没问题）。

`token`，使用参数`appid`，`timestamp`，`nonce`，`sign`来获取token，作为系统调用的唯一凭证。`token`可以设置一次有效（这样安全性更高），也可以设置时效性，这里推荐设置时效性。如果一次有效的话这个接口的请求频率可能会很高。`token`推荐加到请求头上，这样可以跟业务参数完全区分开来。

## 1.3 使用POST作为接口请求方式

一般调用接口最常用的两种方式就是GET和POST。两者的区别也很明显，GET请求会将参数暴露在浏览器URL中，而且对长度也有限制。为了更高的安全性，所有接口都采用POST方式请求。

### 1.3.1 GET、POST、PUT、DELETE对比

#### 1. GET

- 安全且幂等
- 获取表示
- 变更时获取表示（缓存）
  ==适合查询类的接口使用==

#### 2. POST

- 不安全且不幂等
- 使用服务端管理的（自动产生）的实例号创建资源
- 创建子资源
- 部分更新资源
- 如果没有被修改，则不过更新资源（乐观锁）
  ==适合数据提交类的接口使用==

#### 3. PUT

- 不安全但幂等
- 用客户端管理的实例号创建一个资源
- 通过替换的方式更新资源
- 如果未被修改，则更新资源（乐观锁）
  ==适合更新数据的接口使用==

#### 4. DELETE

- 不安全但幂等
- 删除资源
  ==适合删除数据的接口使用==



## 1.4 客户端IP白名单

ip白名单是指将接口的访问权限对部分ip进行开放。这样就能避免其他ip进行访问攻击，设置ip白名单比较麻烦的一点就是当你的客户端进行迁移后，就需要重新联系服务提供者添加新的ip白名单。设置ip白名单的方式很多，除了传统的防火墙之外，spring cloud alibaba提供的组件sentinel也支持白名单设置。为了降低api的复杂度，推荐使用防火墙规则进行白名单设置。

## 1.5 单个接口针对ip限流

限流是为了更好的维护系统稳定性。使用redis进行接口调用次数统计，ip+接口地址作为key，访问次数作为value，每次请求value+1，设置过期时长来限制接口的调用频率。

## 1.6 记录接口请求日志

使用aop全局记录请求日志，快速定位异常请求位置，排查问题原因。

## 1.7 敏感数据脱敏

在接口调用过程中，可能会涉及到订单号等敏感数据，这类数据通常需要脱敏处理，最常用的方式就是加密。加密方式使用安全性比较高的`RSA`非对称加密。非对称加密算法有两个密钥，这两个密钥完全不同但又完全匹配。只有使用匹配的一对公钥和私钥，才能完成对明文的加密和解密过程。



## 1.8 瘦客户端

客户端尽量不处理逻辑

客户端不处理金额

客户端参数校验规则可以通过接口返回，同时提供默认规则，接口不通则使用默认规则。

## 1.9 拓展性

图片文案等，与校验规则类似，通过接口返回，并提供默认。

列表界面

```json
// 静态列表
{
    "name": "张三",
    "sex": "男",
    "age": "20岁",
    "nickName": "小张"
}
// 动态列表
{
    "userInfos":[
    {
        "key":"姓名",
        "value":"张三"
    },{
        "key":"性别",
        "value":"男"
    },{
        "key":"年龄",
        "value":"20岁"
    },{
        "key":"昵称",
        "value":"小张"
    }]
}
```

多个boolean可以flag替换

```json
{
    "flag":"7" // 二进制：111，三位分别表示三个boolean字段
}

long flag = 7;
System.out.println("bit="+Long.toBinaryString(flag));
System.out.println("第一位="+((flag&1)==1));
System.out.println("第二位="+((flag&2)==1));
System.out.println("第三位="+((flag&4)==1));
```

# 二、接口的幂等性

<font color='red'>幂等性：不论你请求多少次，资源的状态是一样的。</font>

幂等性是指任意多次请求的执行结果和一次请求的执行结果所产生的影响相同。说的直白一点就是查询操作无论查询多少次都不会影响数据本身，因此查询操作本身就是幂等的。但是新增操作，每执行一次数据库就会发生变化，所以它是非幂等的。

## 2.1 如何设计接口的幂等性

幂等问题的解决有很多思路，这里讲一种比较严谨的。提供一个生成随机数的接口，随机数全局唯一。调用接口的时候带入随机数。第一次调用，业务处理成功后，将随机数作为key，操作结果作为value，存入redis，同时设置过期时长。第二次调用，查询redis，如果key存在，则证明是重复提交，直接返回错误。

### 2.1.1 insert

>  1.全局唯一的id
>  2.先查询一下然后再决定是插入还是更新
>  3.创建一个去重表（redis）
>  4.token -> redis

### 2.1.2 update

>  1.多版本控制
>  2.数据库乐观锁/悲观锁
>  3.使用状态码 status=0



## 2.2 分布式ID生成器

不能使用数据库本身的自增功能来产生主键值，原因是生产环境为分片部署的。会导致有重复的id值
使用snowflake （雪花）算法（twitter出品）生成唯一的主键值
![img](https:////upload-images.jianshu.io/upload_images/20411018-bfb077a93d0a5944.png?imageMogr2/auto-orient/strip|imageView2/2/w/1076)

- 41bit的时间戳可以支持该算法使用到2082年
- 10bit的工作机器id可以支持1024台机器
- 序列号支持1毫秒产生4096个自增序列id
- 整体上按照时间自增排序
- 整个分布式系统内不会产生ID碰撞
- 每秒能够产生26万ID左右

# 三、响应状态码

采用http的状态码进行数据封装，例如200表示请求成功，4xx表示客户端错误，5xx表示服务器内部发生错误。状态码设计参考如下：

| 分类 | 描述                                         |
| ---- | -------------------------------------------- |
| 1xx  | 信息，服务器收到请求，需要请求者继续执行操作 |
| 2xx  | 成功                                         |
| 3xx  | 重定向，需要进一步的操作以完成请求           |
| 4xx  | 客户端错误，请求包含语法错误或无法完成请求   |
| 5xx  | 服务端错误                                   |

状态码枚举类：

```java
public enum CodeEnum {

    // 根据业务需求进行添加
    SUCCESS(200,"处理成功"),
    ERROR_PATH(404,"请求地址错误"),
    ERROR_SERVER(505,"服务器内部发生错误");
    
    private int code;
    private String message;
    
    CodeEnum(int code, String message) {
        this.code = code;
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

## 3.1 正常响应

响应状态码2xx

- 200：常规请求
- 201：创建成功

## 3.2 重定向响应

响应状态码3xx

- 301：永久重定向
- 302：暂时重定向

## 3.3 客户端异常

响应状态码4xx

- 403：请求无权限
- 404：请求路径不存在
- 405：请求方法不存在

## 3.4 服务器异常

响应状态码5xx

- 500：服务器异常

