[CORS跨源资源共享概念及配置（Kubernetes Ingress和Spring Cloud Gateway）](https://www.cnblogs.com/larrydpk/p/14950935.html)

# 1 跨源资源共享CORS

**跨源资源共享** ([CORS](https://developer.mozilla.org/en-US/docs/Glossary/CORS)) （或通俗地译为跨域资源共享）是一种基于[HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP) 头的机制，该机制通过允许服务器标示除了它自己以外的其它[origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin)（域，协议和端口），这样浏览器可以访问加载这些资源。

首先要明确的是，浏览器访问资源才会有`CORS`的存在，如果通过其它`HTTP Client`等代码，就不会出现。`CORS`简单一点讲就是当在浏览器地址栏的`源Origin`与所访问的资源的地址的源不同，就是跨源了。比如在前后端分离的开发中，`UI`的地址为`http://localhost:3000`，而服务的地址为`http://localhost:8080`，通过`JavaScript`获取服务的数据，就需要跨源。

## 1.1 预检preflight

对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET) 以外的 HTTP 请求，或者搭配某些 MIME 类型的 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST) 请求），浏览器必须首先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨源请求。服务器确认允许之后，才发起实际的 HTTP 请求。

所以，`CORS`是需要**服务器**端打开的一个特性，而不是**客户端**。

对于简单请求，不需要预检：

![img](https://img2020.cnblogs.com/other/946674/202106/946674-20210629170023379-1507306714.png)

预检一般是通过`OPTION`方法来进行：

![img](https://img2020.cnblogs.com/other/946674/202106/946674-20210629170024837-89844633.png)

需要注意：

请求的首部中携带了 `Cookie` 信息（`credentials:include`），如果 `Access-Control-Allow-Origin` 的值为“`*`”，请求将会失败。

# 2 kubernetes ingress打开CORS

可以在`ingress`层面打开CORS，而不用在应用层面。配置如下：

```yaml
annotations:
kubernetes.io/ingress.class: "nginx"
nginx.ingress.kubernetes.io/enable-cors: "true"
nginx.ingress.kubernetes.io/cors-allow-origin: "*"
nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS, DELETE"
nginx.ingress.kubernetes.io/cors-allow-headers: "DNT,X-CustomHeader,X-LANG,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,X-Api-Key,X-Device-Id,Access-Control-Allow-Origin"
```

当考虑到某些场景不能使`allow-origin`为`*`，所以可以按下面这样配置：

```yaml
nginx.ingress.kubernetes.io/enable-cors: "true"
nginx.ingress.kubernetes.io/cors-allow-methods: "PUT, GET, POST, OPTIONS"
nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
nginx.ingress.kubernetes.io/configuration-snippet: |
	more_set_headers "Access-Control-Allow-Origin: $http_origin";
```

# 3 spring cloud gateway打开CORS

可以通过配置properties来实现，也可以通过`Java`配置`WebFilter`来实现。

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://www.pkslow.com"
            allowedMethods:
            - GET
```

Java的方式大致如下：

```java
@Bean
CorsWebFilter corsWebFilter() {
    CorsConfiguration corsConfig = new CorsConfiguration();
    corsConfig.setAllowedOrigins(Arrays.asList("https://www.pkslow.com"));
    corsConfig.setMaxAge(8000L);
    corsConfig.addAllowedMethod("PUT");

    UrlBasedCorsConfigurationSource source =
      new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", corsConfig);

    return new CorsWebFilter(source);
}
```