[TOC]

shiro是apache的一个开源框架，是一个权限管理的框架，实现 用户认证、用户授权。
spring中有spring security (原名Acegi)，是一个权限框架，它和spring依赖过于紧密，没有shiro使用简单。
shiro不依赖于spring，shiro不仅可以实现 web应用的权限管理，还可以实现c/s系统，分布式系统权限管理。

# 1、运行流程

1. 首先调用 Subject.login(token) 进行登录，其会自动委托给 Security Manager，调用之前必须通过 SecurityUtils.setSecurityManager() 设置；
2. SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3. Authenticator 才是真正的身份验证者，Shiro API 中核心的身份认证入口点，此处可以自定义插入自己的实现；
4. Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认 ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证；
5. Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。

# 2、绑定线程
第一步，Subject.login(token)方法。
```java
UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);
Subject subject = SecurityUtils.getSubject();
subject.login(token);
```

出现了一个UsernamePasswordToken对象，它在这里会调用它的一个构造函数。
```java
public UsernamePasswordToken(final String username, final String password, final boolean rememberMe) {
    this(username, password != null ? password.toCharArray() : null, rememberMe, null);
}
```

这是shiro的一个验证对象，只是用来存储用户名密码，以及一个记住我属性的。

之后会调用shiro的一个工具类得到一个subject对象。
```java
public static Subject getSubject() {
    Subject subject = ThreadContext.getSubject();
    if (subject == null) {
        subject = (new Subject.Builder()).buildSubject();
        ThreadContext.bind(subject);
    }
    return subject;
}
```

通过getSubject方法来得到一个Subject对象。

这里不得不提到shiro的内置线程类ThreadContext，通过bind方法会将subject对象绑定在线程上。
```java
public static void bind(Subject subject) {
    if (subject != null) {
        put(SUBJECT_KEY, subject);
    }
}
```

```java
public static void put(Object key, Object value) {
    if (key == null) {
        throw new IllegalArgumentException("key cannot be null");
    }

    if (value == null) {
        remove(key);
        return;
    }

    ensureResourcesInitialized();
    resources.get().put(key, value);

    if (log.isTraceEnabled()) {
        String msg = "Bound value of type [" + value.getClass().getName() + "] for key [" +
                key + "] to thread " + "[" + Thread.currentThread().getName() + "]";
        log.trace(msg);
    }
}
```

且shiro的key都是遵循一个固定的格式。
```java
public static final String SUBJECT_KEY = ThreadContext.class.getName() + "_SUBJECT_KEY";
```

经过非空判断后会将值以KV的形式put进去。
当你想拿到subject对象时，也可以通过getSubject方法得到subject对象。
在绑定subject对象时，也会将securityManager对象进行一个绑定。
而绑定securityManager对象的地方是在Subject类的一个静态内部类里（可让我好一顿找）。
在getSubject方法中的一句代码调用了内部类的buildSubject方法。

```java
subject = (new Subject.Builder()).buildSubject();
```

首先调用无参构造，在无参构造里调用有参构造函数。
```java
public Builder() {
    this(SecurityUtils.getSecurityManager());
}

public Builder(SecurityManager securityManager) {
    if (securityManager == null) {
        throw new NullPointerException("SecurityManager method argument cannot be null.");
    }
    this.securityManager = securityManager;
    this.subjectContext = newSubjectContextInstance();
    if (this.subjectContext == null) {
        throw new IllegalStateException("Subject instance returned from 'newSubjectContextInstance' " +
                "cannot be null.");
    }
    this.subjectContext.setSecurityManager(securityManager);
}
```

在此处绑定了securityManager对象。
当然，他也对securityManager对象的空状况进行了处理，在getSecurityManager方法里。
```java
public static SecurityManager getSecurityManager() throws UnavailableSecurityManagerException {
    SecurityManager securityManager = ThreadContext.getSecurityManager();
    if (securityManager == null) {
        securityManager = SecurityUtils.securityManager;
    }
    if (securityManager == null) {
        String msg = "No SecurityManager accessible to the calling code, either bound to the " +
                ThreadContext.class.getName() + " or as a vm static singleton.  This is an invalid application " +
                "configuration.";
        throw new UnavailableSecurityManagerException(msg);
    }
    return securityManager;
}
```

真正的核心就在于securityManager这个对象。

# 3、SecurityManager
SecurityManager是一个接口，他继承了步骤里所谈到的Authenticator，Authorizer类以及用于Session管理的SessionManager。
```java
public interface SecurityManager extends Authenticator, Authorizer, SessionManager {

    Subject login(Subject subject, AuthenticationToken authenticationToken) throws AuthenticationException;   

    void logout(Subject subject);

    Subject createSubject(SubjectContext context);
}
```
看一下它的实现。
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/865f4f7df65a41e3aa6bd1a2c6ab1223~tplv-k3u1fbpfcp-zoom-1.image)

且这些类和接口都有依次继承的关系。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b33bd40381e74f3aae202db5112c414c~tplv-k3u1fbpfcp-zoom-1.image)

# 4、Relam
Realm充当了Shiro与应用安全数据间的“桥梁”或者“连接器”。也就是说，当与像用户帐户这类安全相关数据进行交互，执行认证（登录）和授权（访问控制）时，Shiro会从应用配置的Realm中查找很多内容。

从这个意义上讲，Realm实质上是一个安全相关的DAO：它封装了数据源的连接细节，并在需要时将相关数据提供给Shiro。当配置Shiro时，你必须至少指定一个Realm，用于认证和（或）授权。配置多个Realm是可以的，但是至少需要一个。

Shiro内置了可以连接大量安全数据源（又名目录）的Realm，如LDAP、关系数据库（JDBC）、类似INI的文本配置资源以及属性文件 等。如果缺省的Realm不能满足需求，你还可以插入代表自定义数据源的自己的Realm实现。

一般情况下，都会自定义Relam来使用。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69adeb7f1dfa419fa5c300aa92c710c6~tplv-k3u1fbpfcp-zoom-1.image)

以及自定义的一个UserRelam。

看一下类图。
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89379fd1b4364a4cad06bfd1d5e6b86e~tplv-k3u1fbpfcp-zoom-1.image)

每个抽象类继承后所需要实现的方法都不一样。
```java
public class UserRealm extends AuthorizingRealm
```

这里继承AuthorizingRealm，需要实现它的两个方法。
```java
//给登录用户授权
protected abstract AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals);

//这个抽象方法属于AuthorizingRealm抽象类的父类AuthenticatingRealm类   登录认证，也是登录的DAO操作所在的方法
protected abstract AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;
```

# 5、Authenticator
uthenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返回 / 抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。

再回到之前登录方法上来看看。

subject.login(token)在第一步中调用了Subject的login方法，找到它的最终实现DelegatingSubject类。
里面有调用了securityManager的login方法，而最终实现就在DefaultSecurityManager这个类里。

```java
Subject subject = securityManager.login(this, token);

public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
    AuthenticationInfo info;
    try {
        info = authenticate(token);
    } catch (AuthenticationException ae) {
        try {
            onFailedLogin(token, ae, subject);
        } catch (Exception e) {
            if (log.isInfoEnabled()) {
                log.info("onFailedLogin method threw an " +
                        "exception.  Logging and propagating original AuthenticationException.", e);
            }
        }
        throw ae; //propagate
    }
```
之后就是验证流程，这里我们会看到第四步，点进去会到抽象类AuthenticatingSecurityManager。再看看它的仔细调用。
```java
public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
    return this.authenticator.authenticate(token);
}
```

真正的调用Relam进行验证并不在这，而是在ModularRealmAuthenticator。

他们之间是一个从左到右的过程。
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cdac438cc90405c845dc84a6926edac~tplv-k3u1fbpfcp-zoom-1.image)

```java
protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
    assertRealmsConfigured();
    Collection<Realm> realms = getRealms();
    if (realms.size() == 1) {
        return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
    } else {
        return doMultiRealmAuthentication(realms, authenticationToken);
    }
}
```

看这个doSingleRealmAuthentication方法。

单Relam验证。

```java
protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
    if (!realm.supports(token)) {
        String msg = "Realm [" + realm + "] does not support authentication token [" +
                token + "].  Please ensure that the appropriate Realm implementation is " +
                "configured correctly or that the realm accepts AuthenticationTokens of this type.";
        throw new UnsupportedTokenException(msg);
    }
    //在此处调用你自定义的Relam的方法来验证。
    AuthenticationInfo info = realm.getAuthenticationInfo(token);
    if (info == null) {
        String msg = "Realm [" + realm + "] was unable to find account data for the " +
                "submitted AuthenticationToken [" + token + "].";
        throw new UnknownAccountException(msg);
    }
    return info;
}
```

多Relam的。
```java
protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {

    AuthenticationStrategy strategy = getAuthenticationStrategy();

    AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);

    for (Realm realm : realms) {

        aggregate = strategy.beforeAttempt(realm, token, aggregate);

        if (realm.supports(token)) {

            AuthenticationInfo info = null;
            Throwable t = null;
            try {
                //调用自定义的Relam的方法来验证。
                info = realm.getAuthenticationInfo(token);
            } catch (Throwable throwable) {
                t = throwable;
            }

            aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);

        } else {
            log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
        }
    }

    aggregate = strategy.afterAllAttempts(token, aggregate);

    return aggregate;
}
```
会发现调用的都是Relam的getAuthenticationInfo方法。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62499e2cb7124785bce4acbbdd2156a4~tplv-k3u1fbpfcp-zoom-1.image)

看到了熟悉的UserRelam，此致，闭环了。