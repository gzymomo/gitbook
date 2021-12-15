[TOC]

# 1、Shiro简介
## 1.1 基础概念
Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理。作为一款安全框架Shiro的设计相当巧妙。Shiro的应用不依赖任何容器，它不仅可以在JavaEE下使用，还可以应用在JavaSE环境中。

## 1.2 核心角色
1. Subject：认证主体
代表当前系统的使用者，就是用户，在Shiro的认证中，认证主体通常就是userName和passWord，或者其他用户相关的唯一标识。

2. SecurityManager：安全管理器
Shiro架构中最核心的组件，通过它可以协调其他组件完成用户认证和授权。实际上，SecurityManager就是Shiro框架的控制器。

3. Realm：域对象
定义了访问数据的方式，用来连接不同的数据源，如：关系数据库，配置文件等等。

## 1.3 核心理念
Shiro自己不维护用户和权限，通过Subject用户主体和Realm域对象的注入，完成用户的认证和授权。

# 2、SpringBoot整合Shiro
## 2.1 核心依赖
```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```
## 2.2 Shiro核心配置
```java
@Configuration
public class ShiroConfig {
    /**
     * Session Manager：会话管理
     * 即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；
     * 会话可以是普通JavaSE环境的，也可以是如Web环境的；
     */
    @Bean("sessionManager")
    public SessionManager sessionManager(){
        DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
        //设置session过期时间
        sessionManager.setGlobalSessionTimeout(60 * 60 * 1000);
        sessionManager.setSessionValidationSchedulerEnabled(true);
        // 去掉shiro登录时url里的JSESSIONID
        sessionManager.setSessionIdUrlRewritingEnabled(false);
        return sessionManager;
    }

    /**
     * SecurityManager：安全管理器
     */
    @Bean("securityManager")
    public SecurityManager securityManager(UserRealm userRealm, SessionManager sessionManager) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setSessionManager(sessionManager);
        securityManager.setRealm(userRealm);
        return securityManager;
    }
    /**
     * ShiroFilter是整个Shiro的入口点，用于拦截需要安全控制的请求进行处理
     */
    @Bean("shiroFilter")
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);
        shiroFilter.setLoginUrl("/userLogin");
        shiroFilter.setUnauthorizedUrl("/");
        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/userLogin", "anon");
        shiroFilter.setFilterChainDefinitionMap(filterMap);
        return shiroFilter;
    }
    /**
     * 管理Shiro中一些bean的生命周期
     */
    @Bean("lifecycleBeanPostProcessor")
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
    /**
     * 扫描上下文，寻找所有的Advistor(通知器）
     * 将这些Advisor应用到所有符合切入点的Bean中。
     */
    @Bean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator proxyCreator = new DefaultAdvisorAutoProxyCreator();
        proxyCreator.setProxyTargetClass(true);
        return proxyCreator;
    }
    /**
     * 匹配所有加了 Shiro 认证注解的方法
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
}
```
## 2.3 域对象配置
```java
@Component
public class UserRealm extends AuthorizingRealm {
    @Resource
    private SysUserMapper sysUserMapper ;
    @Resource
    private SysMenuMapper sysMenuMapper ;
    /**
     * 授权(验证权限时调用)
     * 获取用户权限集合
     */
    @Override
    public AuthorizationInfo doGetAuthorizationInfo
    (PrincipalCollection principals) {
        SysUserEntity user = (SysUserEntity)principals.getPrimaryPrincipal();
        if(user == null) {
            throw new UnknownAccountException("账号不存在");
        }
        List<String> permsList;
        //默认用户拥有最高权限
        List<SysMenuEntity> menuList = sysMenuMapper.selectList();
        permsList = new ArrayList<>(menuList.size());
        for(SysMenuEntity menu : menuList){
            permsList.add(menu.getPerms());
        }
        //用户权限列表
        Set<String> permsSet = new HashSet<>();
        for(String perms : permsList){
            if(StringUtils.isEmpty(perms)){
                continue;
            }
            permsSet.addAll(Arrays.asList(perms.trim().split(",")));
        }
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.setStringPermissions(permsSet);
        return info;
    }
    /**
     * 认证(登录时调用)
     * 验证用户登录
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken authToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken)authToken;
        //查询用户信息
        SysUserEntity user = sysUserMapper.selectOne(token.getUsername());
        //账号不存在
        if(user == null) {
            throw new UnknownAccountException("账号或密码不正确");
        }
        //账号锁定
        if(user.getStatus() == 0){
            throw new LockedAccountException("账号已被锁定,请联系管理员");
        }
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo
                (user, user.getPassword(),
                        ByteSource.Util.bytes(user.getSalt()),
                        getName());
        return info;
    }
    @Override
    public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
        HashedCredentialsMatcher shaCredentialsMatcher = new HashedCredentialsMatcher();
        shaCredentialsMatcher.setHashAlgorithmName(ShiroUtils.hashAlgorithmName);
        shaCredentialsMatcher.setHashIterations(ShiroUtils.hashIterations);
        super.setCredentialsMatcher(shaCredentialsMatcher);
    }
}
```
## 3.4 核心工具类
```java
public class ShiroUtils {
    /**  加密算法 */
    public final static String hashAlgorithmName = "SHA-256";
    /**  循环次数 */
    public final static int hashIterations = 16;
    public static String sha256(String password, String salt) {
        return new SimpleHash(hashAlgorithmName, password, salt, hashIterations).toString();
    }
    // 获取一个测试账号 admin
    public static void main(String[] args) {
        // 3743a4c09a17e6f2829febd09ca54e627810001cf255ddcae9dabd288a949c4a
        System.out.println(sha256("admin","123")) ;
    }
    /**
     * 获取会话
     */
    public static Session getSession() {
        return SecurityUtils.getSubject().getSession();
    }
    /**
     * Subject：主体，代表了当前“用户”
     */
    public static Subject getSubject() {
        return SecurityUtils.getSubject();
    }
    public static SysUserEntity getUserEntity() {
        return (SysUserEntity)SecurityUtils.getSubject().getPrincipal();
    }
    public static Long getUserId() {
        return getUserEntity().getUserId();
    }
    public static void setSessionAttribute(Object key, Object value) {
        getSession().setAttribute(key, value);
    }
    public static Object getSessionAttribute(Object key) {
        return getSession().getAttribute(key);
    }
    public static boolean isLogin() {
        return SecurityUtils.getSubject().getPrincipal() != null;
    }
    public static void logout() {
        SecurityUtils.getSubject().logout();
    }
}
```
## 3.5 自定义权限异常提示
```java
@RestControllerAdvice
public class ShiroException {
    @ExceptionHandler(AuthorizationException.class)
    public String authorizationException (){
        return "抱歉您没有权限访问该内容!";
    }
    @ExceptionHandler(Exception.class)
    public String handleException(Exception e){
        return "系统异常!";
    }
}
```

# 3、案例演示代码
## 3.1 测试接口
```java
@RestController
public class ShiroController {
    private static Logger LOGGER = LoggerFactory.getLogger(ShiroController.class) ;
    @Resource
    private SysMenuMapper sysMenuMapper ;
    /**
     * 登录测试
     * http://localhost:7011/userLogin?userName=admin&passWord=admin
     */
    @RequestMapping("/userLogin")
    public void userLogin (
            @RequestParam(value = "userName") String userName,
            @RequestParam(value = "passWord") String passWord){
        try{
            Subject subject = ShiroUtils.getSubject();
            UsernamePasswordToken token = new UsernamePasswordToken(userName, passWord);
            subject.login(token);
            LOGGER.info("登录成功");
        }catch (Exception e) {
            e.printStackTrace();
        }
    }
    /**
     * 服务器每次重启请求该接口之前必须先请求上面登录接口
     * http://localhost:7011/menu/list 获取所有菜单列表
     * 权限要求：sys:user:shiro
     */
    @RequestMapping("/menu/list")
    @RequiresPermissions("sys:user:shiro")
    public List list(){
        return sysMenuMapper.selectList() ;
    }
    /**
     * 用户没有该权限，无法访问
     * 权限要求：ccc:ddd:bbb
     */
    @RequestMapping("/menu/list2")
    @RequiresPermissions("ccc:ddd:bbb")
    public List list2(){
        return sysMenuMapper.selectList() ;
    }
    /**
     * 退出测试，退出后没有任何权限
     */
    @RequestMapping("/userLogOut")
    public String logout (){
        ShiroUtils.logout();
        return "success" ;
    }
}
```
## 3.2 测试流程
1)、登录后取得权限
http://localhost:7011/userLogin?userName=admin&passWord=admin
2)、访问有权限接口
http://localhost:7011/menu/list
3)、访问无权限接口
http://localhost:7011/menu/list2
4)、退出登录
http://localhost:7011/userLogOut