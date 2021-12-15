[SpringCloud(8)---zuul权限校验、接口限流](https://www.cnblogs.com/qdhxhz/p/9601170.html)

**项目代码GitHub地址**：https://github.com/yudiandemingzi/spring-cloud-study

## 一、权限校验搭建

正常项目开发时,权限校验可以考虑JWT和springSecurity结合进行权限校验，这个后期会总结，这里做个基于ZuulFilter过滤器进行一个简单的权限校验过滤。

对于组件zuul中，其实带有权限认证的功能，那就是ZuulFilter过滤器。ZuulFilter是Zuul中核心组件，通过继承该抽象类，覆写几个关键方法达到自定义调度请求的作用

使用到的组件包括：Eureka、Feign、Zuul，包括以下四个项目：

 （1）Eureka-server： 7001 注册中心

 （2）product-server ： 8001 商品微服务

 （3）order-server ： 9001 订单微服务

 （4）zuul-gateway ： 6001 Zuul网关

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180906212616101-215602007.png)

有关四个服务的基本配置我这里就不写了，具体可以看之前几篇博客，这里只写LoginFilter权限校验类

####   1、LoginFilter类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 登录过滤器 *记得类上加Component注解
 */
@Component
public class LoginFilter extends ZuulFilter {

    /**
     * 过滤器类型，前置过滤器
     */
    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    /**
     * 过滤器顺序，越小越先执行
     */
    @Override
    public int filterOrder() {
        return 4;
    }

    /**
     * 过滤器是否生效
     * 返回true代表需要权限校验，false代表不需要用户校验即可访问
     */
    @Override
    public boolean shouldFilter() {

        //共享RequestContext，上下文对象
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();

        System.out.println(request.getRequestURI());
        //需要权限校验URL
        if ("/apigateway/order/api/v1/order/save".equalsIgnoreCase(request.getRequestURI())) {
            return true;
        } else if ("/apigateway/order/api/v1/order/list".equalsIgnoreCase(request.getRequestURI())) {
            return true;
        } else if ("/apigateway/order/api/v1/order/find".equalsIgnoreCase(request.getRequestURI())) {
            return true;
        }
        return false;
    }

    /**
     * 业务逻辑
     * 只有上面返回true的时候，才会进入到该方法
     */
    @Override
    public Object run() throws ZuulException {

        //JWT
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();

        //token对象,有可能在请求头传递过来，也有可能是通过参数传过来，实际开发一般都是请求头方式
        String token = request.getHeader("token");

        if (StringUtils.isBlank((token))) {
            token = request.getParameter("token");
        }
        System.out.println("页面传来的token值为：" + token);
        //登录校验逻辑  如果token为null，则直接返回客户端，而不进行下一步接口调用
        if (StringUtils.isBlank(token)) {
            // 过滤该请求，不对其进行路由
            requestContext.setSendZuulResponse(false);
            //返回错误代码
            requestContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

####    2、关键说明

**（1）方法说明**

  filterType : filter类型,分为pre、error、post、 route

  filterOrder: filter执行顺序，通过数字指定，数字越小，执行顺序越先

 shouldFilter: filter是否需要执行 true执行 false 不执行

​      run : filter具体逻辑（上面为true那么这里就是具体执行逻辑）

**（2）filter类型说明**

​      pre: 请求执行之前filter

​     route: 处理请求，进行路由

​     post: 请求处理完成后执行的filter

​     error: 出现错误时执行的filter

####     3、测试

先在请求头和传参都不传token，校验失败：返回401状态码

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180906213252399-855508337.png)

在参数的时候传入token值

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180906213329013-1927956637.png)

 看后台输出
![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180906213453880-1292234025.png)

   说明模拟校验通过，返回用户信息。

 

## 二、接口限流搭建

接口限流可以在nginx层面做限流，也可以在网关层面做限流，这里在网关层面做限流，基于guava框架来做网关限流。

先对guava框架限流的概念进行讲解下：

![img](https://images2018.cnblogs.com/blog/1090617/201809/1090617-20180906213610239-526459252.png)

它的大致意思就是每一个请求进来先到桶里去拿令牌，拿到令牌的请求放行，假设你设置了1000个令牌，如果拿完了，那么后面来调接口的请求就需要排队等有新的令牌才能调用该接口。

#### OrderRateLimiterFilter限流过滤类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 订单限流
 *其它和上面都一样，只是run()中逻辑不一样
 */
@Component
public class OrderRateLimiterFilter extends ZuulFilter {


    //每秒产生1000个令牌
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(1000);

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return -4;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext requestContext = RequestContext.getCurrentContext();
        HttpServletRequest request = requestContext.getRequest();

        //只对订单接口限流
        if ("/apigateway/order/api/v1/order/save".equalsIgnoreCase(request.getRequestURI())) {
            return true;
        }
        return false;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext requestContext = RequestContext.getCurrentContext();

        //就相当于每调用一次tryAcquire()方法，令牌数量减1，当1000个用完后，那么后面进来的用户无法访问上面接口
        //当然这里只写类上面一个接口，可以这么写，实际可以在这里要加一层接口判断。
        if (!RATE_LIMITER.tryAcquire()) {
            requestContext.setSendZuulResponse(false);
            //HttpStatus.TOO_MANY_REQUESTS.value()里面有静态代码常量
            requestContext.setResponseStatusCode(HttpStatus.TOO_MANY_REQUESTS.value());
        }
        return null;
    }
}
```