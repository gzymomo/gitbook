[TOC]

# 1、Security简介
## 1.1 基础概念
Spring Security是一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring的IOC，DI，AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为安全控制编写大量重复代码的工作。

## 1.2 核心API解读
1)、SecurityContextHolder
最基本的对象，保存着当前会话用户认证，权限，鉴权等核心数据。SecurityContextHolder默认使用ThreadLocal策略来存储认证信息，与线程绑定的策略。用户退出时，自动清除当前线程的认证信息。

初始化源码：明显使用ThreadLocal线程。
```java
private static void initialize() {
    if (!StringUtils.hasText(strategyName)) {
        strategyName = "MODE_THREADLOCAL";
    }
    if (strategyName.equals("MODE_THREADLOCAL")) {
        strategy = new ThreadLocalSecurityContextHolderStrategy();
    } else if (strategyName.equals("MODE_INHERITABLETHREADLOCAL")) {
        strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
    } else if (strategyName.equals("MODE_GLOBAL")) {
        strategy = new GlobalSecurityContextHolderStrategy();
    } else {
        try {
            Class<?> clazz = Class.forName(strategyName);
            Constructor<?> customStrategy = clazz.getConstructor();
            strategy = (SecurityContextHolderStrategy)customStrategy.newInstance();
        } catch (Exception var2) {
            ReflectionUtils.handleReflectionException(var2);
        }
    }
    ++initializeCount;
}
```
2)、Authentication
```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    Object getCredentials();
    Object getDetails();
    Object getPrincipal();
    boolean isAuthenticated();
    void setAuthenticated(boolean var1) throws IllegalArgumentException;
}
```
源码分析

1. getAuthorities，权限列表,通常是代表权限的字符串集合;
2. getCredentials，密码,认证之后会移出，来保证安全性;
3. getDetails，请求的细节参数;
4. getPrincipal, 核心身份信息，一般返回UserDetails的实现类。

3)、UserDetails
封装了用户的详细的信息。
```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```
4)、UserDetailsService
实现该接口，自定义用户认证流程，通常读取数据库，对比用户的登录信息，完成认证，授权。
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```
5)、AuthenticationManager
认证流程顶级接口。可以通过实现AuthenticationManager接口来自定义自己的认证方式，Spring提供了一个默认的实现，ProviderManager。
```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication var1) throws AuthenticationException;
}
```

# 2、SpringBoot整合SpringSecurity
## 2.1 流程描述
```java
1)、三个页面分类，page1、page2、page3
2)、未登录授权都不可以访问
3)、登录后根据用户权限，访问指定页面
4)、对于未授权页面，访问返回403：资源不可用
```
## 2.2 核心依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
## 2.3 核心配置
```java
/**
 * EnableWebSecurity注解使得SpringMVC集成了Spring Security的web安全支持
 */
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 权限配置
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 配置拦截规则
        http.authorizeRequests().antMatchers("/").permitAll()
                 .antMatchers("/page1/**").hasRole("LEVEL1")
                 .antMatchers("/page2/**").hasRole("LEVEL2")
                 .antMatchers("/page3/**").hasRole("LEVEL3");
        // 配置登录功能
        http.formLogin().usernameParameter("user")
                .passwordParameter("pwd")
                .loginPage("/userLogin");
        // 注销成功跳转首页
        http.logout().logoutSuccessUrl("/");
        //开启记住我功能
        http.rememberMe().rememberMeParameter("remeber");
    }
    /**
     * 自定义认证数据源
     */
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception{
        builder.userDetailsService(userDetailService())
                .passwordEncoder(passwordEncoder());
    }
    @Bean
    public UserDetailServiceImpl userDetailService (){
        return new UserDetailServiceImpl () ;
    }
    /**
     * 密码加密
     */
    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
    /*
     * 硬编码几个用户
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("spring").password("123456").roles("LEVEL1","LEVEL2")
                .and()
                .withUser("summer").password("123456").roles("LEVEL2","LEVEL3")
                .and()
                .withUser("autumn").password("123456").roles("LEVEL1","LEVEL3");
    }
    */
}
```
## 2.4 认证流程
```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {
    @Resource
    private UserRoleMapper userRoleMapper ;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 这里可以捕获异常，使用异常映射，抛出指定的提示信息
        // 用户校验的操作
        // 假设密码是数据库查询的 123
        String password = "$2a$10$XcigeMfToGQ2bqRToFtUi.sG1V.HhrJV6RBjji1yncXReSNNIPl1K";
        // 假设角色是数据库查询的
        List<String> roleList = userRoleMapper.selectByUserName(username) ;
        List<GrantedAuthority> grantedAuthorityList = new ArrayList<>() ;
        /*
         * Spring Boot 2.0 版本踩坑
         * 必须要 ROLE_ 前缀， 因为 hasRole("LEVEL1")判断时会自动加上ROLE_前缀变成 ROLE_LEVEL1 ,
         * 如果不加前缀一般就会出现403错误
         * 在给用户赋权限时,数据库存储必须是完整的权限标识ROLE_LEVEL1
         */
        if (roleList != null && roleList.size()>0){
            for (String role : roleList){
                grantedAuthorityList.add(new SimpleGrantedAuthority(role)) ;
            }
        }
        return new User(username,password,grantedAuthorityList);
    }
}
```
## 2.5 测试接口
```java
@Controller
public class PageController {
    /**
     * 首页
     */
    @RequestMapping("/")
    public String index (){
        return "home" ;
    }
    /**
     * 登录页
     */
    @RequestMapping("/userLogin")
    public String loginPage (){
        return "pages/login" ;
    }
    /**
     * page1 下页面
     */
    @PreAuthorize("hasAuthority('LEVEL1')")
    @RequestMapping("/page1/{pageName}")
    public String onePage (@PathVariable("pageName") String pageName){
        return "pages/page1/"+pageName ;
    }
    /**
     * page2 下页面
     */
    @PreAuthorize("hasAuthority('LEVEL2')")
    @RequestMapping("/page2/{pageName}")
    public String twoPage (@PathVariable("pageName") String pageName){
        return "pages/page2/"+pageName ;
    }
    /**
     * page3 下页面
     */
    @PreAuthorize("hasAuthority('LEVEL3')")
    @RequestMapping("/page3/{pageName}")
    public String threePage (@PathVariable("pageName") String pageName){
        return "pages/page3/"+pageName ;
    }
}
```
## 2.6 登录界面
这里要和Security的配置文件相对应。
```html
<div align="center">
    <form th:action="@{/userLogin}" method="post">
        用户名:<input name="user"/><br>
        密&nbsp;&nbsp;&nbsp;码:<input name="pwd"><br/>
        <input type="checkbox" name="remeber"> 记住我<br/>
        <input type="submit" value="Login">
    </form>
</div>
```