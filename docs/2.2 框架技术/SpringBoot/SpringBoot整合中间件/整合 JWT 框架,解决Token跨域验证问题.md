[TOC]

# 1、传统Session认证
## 1.1 认证过程
1. 用户向服务器发送用户名和密码。
2. 服务器验证后在当前对话（session）保存相关数据。
3. 服务器向返回sessionId，写入客户端 Cookie。
4. 客户端每次请求，需要通过 Cookie，将 sessionId 回传服务器。
5. 服务器收到 sessionId，验证客户端。
## 1.2 存在问题
1. session保存在服务端，客户端访问高并发时，服务端压力大。
2. 扩展性差，服务器集群，就需要 session 数据共享。

# 2、JWT简介
JWT(全称：JSON Web Token)，在基于HTTP通信过程中，进行身份认证。

## 2.1 认证流程
1. 客户端通过用户名和密码登录服务器；
2. 服务端对客户端身份进行验证；
3. 服务器认证以后，生成一个 JSON 对象，发回客户端；
4. 客户端与服务端通信的时候，都要发回这个 JSON 对象；
5. 服务端解析该JSON对象，获取用户身份；
6. 服务端可以不必存储该JSON（Token）对象，身份信息都可以解析出来。
## 2.2 JWT结构说明
抓一只鲜活的Token过来。
```json
{
    "msg": "验证成功",
    "code": 200,
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.
              eyJzdWIiOiJhZG1pbiIsImlhdCI6iZEIj3fQ.
              uEJSJagJf1j7A55Wwr1bGsB5YQoAyz5rbFtF"
}
```
上面的Token被手动格式化了，实际上是用"."分隔的一个完整的长字符串。

**JWT结构**

1. 头部（header) 声明类型以及加密算法；
2. 负载（payload) 携带一些用户身份信息；
3. 签名（signature) 签名信息。

## 2.3 JWT使用方式
通常推荐的做法是客户端在 HTTP 请求的头信息Authorization字段里面。
`Authorization: Bearer <token>`
服务端获取JWT方式
`String token = request.getHeader("token");`

# 3、SpringBoot2整合JWT
## 3.1 核心依赖文件
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```
## 3.2 配置文件
```yml
server:
  port: 7009
spring:
  application:
    name: ware-jwt-token
config:
  jwt:
    # 加密密钥
    secret: iwqjhda8232bjgh432[cicada-smile]
    # token有效时长
    expire: 3600
    # header 名称
    header: token
```
## 3.3 JWT配置代码块
```java
@ConfigurationProperties(prefix = "config.jwt")
@Component
public class JwtConfig {
    /*
     * 根据身份ID标识，生成Token
     */
    public String getToken (String identityId){
        Date nowDate = new Date();
        //过期时间
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);
        return Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setSubject(identityId)
                .setIssuedAt(nowDate)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }
    /*
     * 获取 Token 中注册信息
     */
    public Claims getTokenClaim (String token) {
        try {
            return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
    /*
     * Token 是否过期验证
     */
    public boolean isTokenExpired (Date expirationTime) {
        return expirationTime.before(new Date());
    }
    private String secret;
    private long expire;
    private String header;
    // 省略 GET 和 SET
}
```
# 4、Token拦截案例
## 4.1 配置Token拦截器
```java
@Component
public class TokenInterceptor extends HandlerInterceptorAdapter {
    @Resource
    private JwtConfig jwtConfig ;
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        // 地址过滤
        String uri = request.getRequestURI() ;
        if (uri.contains("/login")){
            return true ;
        }
        // Token 验证
        String token = request.getHeader(jwtConfig.getHeader());
        if(StringUtils.isEmpty(token)){
            token = request.getParameter(jwtConfig.getHeader());
        }
        if(StringUtils.isEmpty(token)){
            throw new Exception(jwtConfig.getHeader()+ "不能为空");
        }
        Claims claims = jwtConfig.getTokenClaim(token);
        if(claims == null || jwtConfig.isTokenExpired(claims.getExpiration())){
            throw new Exception(jwtConfig.getHeader() + "失效，请重新登录");
        }
        //设置 identityId 用户身份ID
        request.setAttribute("identityId", claims.getSubject());
        return true;
    }
}
```
## 4.2 拦截器注册
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Resource
    private TokenInterceptor tokenInterceptor ;
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(tokenInterceptor).addPathPatterns("/**");
    }
}
```
## 4.3 测试接口代码
```java
@RestController
public class TokenController {
    @Resource
    private JwtConfig jwtConfig ;
    // 拦截器直接放行，返回Token
    @PostMapping("/login")
    public Map<String,String> login (@RequestParam("userName") String userName,
                                     @RequestParam("passWord") String passWord){
        Map<String,String> result = new HashMap<>() ;
        // 省略数据源校验
        String token = jwtConfig.getToken(userName+passWord) ;
        if (!StringUtils.isEmpty(token)) {
            result.put("token",token) ;
        }
        result.put("userName",userName) ;
        return result ;
    }
    // 需要 Token 验证的接口
    @PostMapping("/info")
    public String info (){
        return "info" ;
    }
}
```