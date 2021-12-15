[TOC]

在前后端分离的项目中，登录策略也有不少，不过 JWT 算是目前比较流行的一种解决方案了，本文就和大家来分享一下如何将 Spring Security 和 JWT 结合在一起使用，进而实现前后端分离时的登录解决方案。

# 1、无状态登录
## 1.1 什么是有状态？
有状态服务，即服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如Tomcat中的Session。例如登录：用户登录后，我们把用户的信息保存在服务端session中，并且给用户一个cookie值，记录对应的session，然后下次请求，用户携带cookie值来（这一步有浏览器自动完成），我们就能识别到对应session，从而找到用户的信息。

- 服务端保存大量数据，增加服务端压力
- 服务端保存用户状态，不支持集群化部署

## 1.2 什么是无状态
微服务集群中的每个服务，对外提供的都使用RESTful风格的接口。而RESTful风格的一个最重要的规范就是：服务的无状态性，即：

- 服务端不保存任何客户端请求者信息
- 客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份

那么这种无状态性有哪些好处呢？

- 客户端请求不依赖服务端的信息，多次请求不需要必须访问到同一台服务器
- 服务端的集群和状态对客户端透明
- 服务端可以任意的迁移和伸缩（可以方便的进行集群化部署）
- 减小服务端存储压力

## 1.3 如何实现无状态
无状态登录的流程：

- 首先客户端发送账户名/密码到服务端进行认证
- 认证通过后，服务端将用户信息加密并且编码成一个token，返回给客户端
- 以后客户端每次发送请求，都需要携带认证的token
- 服务端对客户端发送来的token进行解密，判断是否有效，并且获取用户登录信息

# 2、JWT
JWT，全称是Json Web Token， 是一种JSON风格的轻量级的授权和身份认证规范，可实现无状态、分布式的Web应用授权：

JWT 作为一种规范，并没有和某一种语言绑定在一起，常用的Java 实现是GitHub 上的开源项目 jjwt，地址如下：https://github.com/jwtk/jjwt

## 2.1 JWT数据格式
JWT包含三部分数据：

- Header：头部，通常头部有两部分信息：
  - 声明类型，这里是JWT
  - 加密算法，自定义
我们会对头部进行Base64Url编码（可解码），得到第一部分数据。

- Payload：载荷，就是有效数据，在官方文档中(RFC7519)，这里给了7个示例信息：
  - iss (issuer)：表示签发人
  - exp (expiration time)：表示token过期时间
  - sub (subject)：主题
  - aud (audience)：受众
  - nbf (Not Before)：生效时间
  - iat (Issued At)：签发时间
  - jti (JWT ID)：编号
这部分也会采用Base64Url编码，得到第二部分数据。

- Signature：签名，是整个数据的认证信息。一般根据前两步的数据，再加上服务的的密钥secret（密钥保存在服务端，不能泄露给客户端），通过Header中配置的加密算法生成。用于验证整个数据完整和可靠性。

生成的数据格式如下图：
![](http://www.javaboy.org/images/sb/20-2.png)

注意，这里的数据通过 . 隔开成了三部分，分别对应前面提到的三部分，另外，这里数据是不换行的，图片换行只是为了展示方便而已。

## 2.2 JWT交互流程
流程图：
![](http://www.javaboy.org/images/sb/20-3.png)

步骤翻译：

1. 应用程序或客户端向授权服务器请求授权
2. 获取到授权后，授权服务器会向应用程序返回访问令牌
3. 应用程序使用访问令牌来访问受保护资源（如API）
因为JWT签发的token中已经包含了用户的身份信息，并且每次请求都会携带，这样服务的就无需保存用户信息，甚至无需去数据库查询，这样就完全符合了RESTful的无状态规范。

## 2.3 JWT 存在的问题
JWT 也不是天衣无缝，由客户端维护登录状态带来的一些问题在这里依然存在，举例如下：

1. 续签问题，这是被很多人诟病的问题之一，传统的cookie+session的方案天然的支持续签，但是jwt由于服务端不保存用户状态，因此很难完美解决续签问题，如果引入redis，虽然可以解决问题，但是jwt也变得不伦不类了。
2. 注销问题，由于服务端不再保存用户信息，所以一般可以通过修改secret来实现注销，服务端secret修改后，已经颁发的未过期的token就会认证失败，进而实现注销，不过毕竟没有传统的注销方便。
3. 密码重置，密码重置后，原本的token依然可以访问系统，这时候也需要强制修改secret。
4. 基于第2点和第3点，一般建议不同用户取不同secret。

# 3、实践
## 3.1 环境搭建
创建一个Spring Boot项目，创建时需要添加Spring Security依赖，创建完成后，添加 jjwt 依赖，完整的pom.xml文件如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```
然后在项目中创建一个简单的 User 对象实现 UserDetails 接口，如下：
```java
public class User implements UserDetails {
    private String username;
    private String password;
    private List<GrantedAuthority> authorities;
    public String getUsername() {
        return username;
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }
    //省略getter/setter
}
```
这个就是我们的用户对象，先放着备用，再创建一个HelloController，内容如下：
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello jwt !";
    }
    @GetMapping("/admin")
    public String admin() {
        return "hello admin !";
    }
}
```
HelloController 很简单，这里有两个接口，设计是 /hello 接口可以被具有 user 角色的用户访问，而 /admin 接口则可以被具有 admin 角色的用户访问。

## 3.2 JWT 过滤器配置
接下来提供两个和 JWT 相关的过滤器配置：

一个是用户登录的过滤器，在用户的登录的过滤器中校验用户是否登录成功，如果登录成功，则生成一个token返回给客户端，登录失败则给前端一个登录失败的提示。
第二个过滤器则是当其他请求发送来，校验token的过滤器，如果校验成功，就让请求继续执行。
这两个过滤器，我们分别来看，先看第一个：
```java
public class JwtLoginFilter extends AbstractAuthenticationProcessingFilter {
    protected JwtLoginFilter(String defaultFilterProcessesUrl, AuthenticationManager authenticationManager) {
        super(new AntPathRequestMatcher(defaultFilterProcessesUrl));
        setAuthenticationManager(authenticationManager);
    }
    @Override
    public Authentication attemptAuthentication(HttpServletRequest req, HttpServletResponse resp) throws AuthenticationException, IOException, ServletException {
        User user = new ObjectMapper().readValue(req.getInputStream(), User.class);
        return getAuthenticationManager().authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword()));
    }
    @Override
    protected void successfulAuthentication(HttpServletRequest req, HttpServletResponse resp, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        Collection<? extends GrantedAuthority> authorities = authResult.getAuthorities();
        StringBuffer as = new StringBuffer();
        for (GrantedAuthority authority : authorities) {
            as.append(authority.getAuthority())
                    .append(",");
        }
        String jwt = Jwts.builder()
                .claim("authorities", as)//配置用户角色
                .setSubject(authResult.getName())
                .setExpiration(new Date(System.currentTimeMillis() + 10 * 60 * 1000))
                .signWith(SignatureAlgorithm.HS512,"sang@123")
                .compact();
        resp.setContentType("application/json;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.write(new ObjectMapper().writeValueAsString(jwt));
        out.flush();
        out.close();
    }
    protected void unsuccessfulAuthentication(HttpServletRequest req, HttpServletResponse resp, AuthenticationException failed) throws IOException, ServletException {
        resp.setContentType("application/json;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.write("登录失败!");
        out.flush();
        out.close();
    }
}
```
关于这个类，我说如下几点：

1. 自定义 JwtLoginFilter 继承自 AbstractAuthenticationProcessingFilter，并实现其中的三个默认方法。
2. attemptAuthentication方法中，我们从登录参数中提取出用户名密码，然后调用AuthenticationManager.authenticate()方法去进行自动校验。
3. 第二步如果校验成功，就会来到successfulAuthentication回调中，在successfulAuthentication方法中，将用户角色遍历然后用一个 , 连接起来，然后再利用Jwts去生成token，按照代码的顺序，生成过程一共配置了四个参数，分别是用户角色、主题、过期时间以及加密算法和密钥，然后将生成的token写出到客户端。
4. 第二步如果校验失败就会来到unsuccessfulAuthentication方法中，在这个方法中返回一个错误提示给客户端即可。

来看第二个token校验的过滤器：
```java
public class JwtFilter extends GenericFilterBean {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) servletRequest;
        String jwtToken = req.getHeader("authorization");
        System.out.println(jwtToken);
        Claims claims = Jwts.parser().setSigningKey("sang@123").parseClaimsJws(jwtToken.replace("Bearer",""))
                .getBody();
        String username = claims.getSubject();//获取当前登录用户名
        List<GrantedAuthority> authorities = AuthorityUtils.commaSeparatedStringToAuthorityList((String) claims.get("authorities"));
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(username, null, authorities);
        SecurityContextHolder.getContext().setAuthentication(token);
        filterChain.doFilter(req,servletResponse);
    }
}
```
关于这个过滤器，我说如下几点：

1. 首先从请求头中提取出 authorization 字段，这个字段对应的value就是用户的token。
2. 将提取出来的token字符串转换为一个Claims对象，再从Claims对象中提取出当前用户名和用户角色，创建一个UsernamePasswordAuthenticationToken放到当前的Context中，然后执行过滤链使请求继续执行下去。

如此之后，两个和JWT相关的过滤器就算配置好了。

## 3.3 Spring Security 配置
接下来我们来配置 Spring Security,如下：
```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("admin")
                .password("123").roles("admin")
                .and()
                .withUser("sang")
                .password("456")
                .roles("user");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/hello").hasRole("user")
                .antMatchers("/admin").hasRole("admin")
                .antMatchers(HttpMethod.POST, "/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .addFilterBefore(new JwtLoginFilter("/login",authenticationManager()),UsernamePasswordAuthenticationFilter.class)
                .addFilterBefore(new JwtFilter(),UsernamePasswordAuthenticationFilter.class)
                .csrf().disable();
    }
}
```

1. 简单起见，这里我并未对密码进行加密，因此配置了NoOpPasswordEncoder的实例。
2. 简单起见，这里并未连接数据库，我直接在内存中配置了两个用户，两个用户具备不同的角色。
3. 配置路径规则时， /hello 接口必须要具备 user 角色才能访问， /admin 接口必须要具备 admin 角色才能访问，POST 请求并且是 /login 接口则可以直接通过，其他接口必须认证后才能访问。
4. 最后配置上两个自定义的过滤器并且关闭掉csrf保护。

## 3.4 测试
做完这些之后，我们的环境就算完全搭建起来了，接下来启动项目然后在 POSTMAN 中进行测试，如下：
![](http://www.javaboy.org/images/sb/20-4.png)

登录成功后返回的字符串就是经过 base64url 转码的token，一共有三部分，通过一个 . 隔开，我们可以对第一个 . 之前的字符串进行解码，即Header，如下：
![](http://www.javaboy.org/images/sb/20-6.png)

再对两个 . 之间的字符解码，即 payload：
![](http://www.javaboy.org/images/sb/20-6.png)

可以看到，我们设置信息，由于base64并不是加密方案，只是一种编码方案，因此，不建议将敏感的用户信息放到token中。

接下来再去访问 /hello 接口，注意认证方式选择 Bearer Token，Token值为刚刚获取到的值，如下：
![](http://www.javaboy.org/images/sb/20-7.png)
