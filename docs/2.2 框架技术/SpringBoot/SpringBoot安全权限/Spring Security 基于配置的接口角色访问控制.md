[Spring Security 实战干货：基于配置的接口角色访问控制](https://www.cnblogs.com/felordcn/p/12142505.html)



## 1.前言

对于受限的访问资源，并不是对所有认证通过的用户开放的。比如 **A** 用户的角色是会计，那么他就可以访问财务相关的资源。**B** 用户是人事，那么他只能访问人事相关的资源。我们在 一文中也对基于角色的访问控制的相关概念进行了探讨。在实际开发中我们如何对资源进行角色粒度的管控呢？今天我来告诉你 **Spring Security** 是如何来解决这个问题的。



## 2. 将角色写入 UserDetails

我们使用 `UserDetailsService` 加载 `UserDetails` 时也会把用户的 `GrantedAuthority` 权限集写入其中。你可以将角色持久化并在这个点进行注入然后配置访问策略，后续的问题交给 **Spring Security** 。

## 3. 在 HttpSecurity 中进行配置角色访问控制

我们可以通过配置 `WebSecurityConfigurerAdapter` 中的 `HttpSecurity` 来控制接口的角色访问。

### 3.1 通过判断用户是否持有角色来进行访问控制

> httpSecurity.authorizeRequests().antMatchers("/foo/test").hasRole(“ADMIN”)

表示 持有 `ROLE_ADMIN` 角色的用户才能访问 `/foo/test` 接口。注意：`hasRole(String role)` 方法入参不能携带前缀 `ROLE_` 。我们来查看 `SecurityExpressionRoot` 中相关源码:

```java
public final boolean hasRole(String role) {
    return hasAnyRole(role);
}
```

很明显 `hasRole` 方法源于 `hasAnyRole` (持有任何其中角色之一，就能满足访问条件，用于一个接口开放给多个角色访问时) :

```java
public final boolean hasAnyRole(String... roles) {
    return hasAnyAuthorityName(defaultRolePrefix, roles);
}
```

如果一个接口开放给多个角色，比如 `/foo/test` 开放给了 `ROLE_APP` 和 `ROLE_ADMIN` 可以这么写：

> httpSecurity.authorizeRequests().antMatchers("/foo/test").hasAnyRole(“APP”,“ADMIN”)

`hasAnyRole` 方法最终的实现为 `hasAnyAuthorityName(String prefix, String... roles)`:

```java
private boolean hasAnyAuthorityName(String prefix, String... roles) {
    Set<String> roleSet = getAuthoritySet();

    for (String role : roles) {
        String defaultedRole = getRoleWithDefaultPrefix(prefix, role);
        if (roleSet.contains(defaultedRole)) {
            return true;
        }
    }
    return false;
}
```

上面才是根本的实现, 需要一个 `prefix` 和每一个 `role` 进行拼接，然后用户的角色集合 `roleSet` 中包含了就返回`true` 放行，否则就 `false` 拒绝。默认的 `prefix` 为 `defaultRolePrefix= ROLE_` 。

### 3.2 通过判断用户的 GrantedAuthority 来进行访问控制

我们也可以通过 `hasAuthority` 和 `hasAnyAuthority` 来判定。 其实底层实现和 `hasAnyRole` 方法一样，只不过 `prefix` 为 `null` 。也就是你写入的 `GrantedAuthority` 是什么样子的，这里传入参数的就是什么样子的，不再受 `ROLE_` 前缀的制约。

**2.1** 章节的写法等同如下的写法：

> httpSecurity.authorizeRequests().antMatchers("/foo/test").hasAuthority(“ROLE_ADMIN”)

> httpSecurity.authorizeRequests().antMatchers("/foo/test").hasAnyAuthority(“ROLE_APP”,“ROLE_ADMIN”)

## 4. 匿名访问

匿名身份验证的用户和未经身份验证的用户之间没有真正的概念差异。**Spring Security** 的匿名身份验证只是为您提供了一种更方便的方式来配置访问控制属性。所有的匿名用户都持有角色 `ROLE_ANONYMOUS` 。所以你可以使用 **2.1** 和 **2.2** 章节的方法来配置匿名访问:

> httpSecurity.authorizeRequests().antMatchers("/foo/test").hasAuthority(“ROLE_ANONYMOUS”)

你也可以通过以下方式进行配置：

> httpSecurity.authorizeRequests().antMatchers("/foo/test").anonymous()

## 5. 开放请求

开放请求可以这么配置：

> httpSecurity.authorizeRequests().antMatchers("/foo/test").permitAll()

## 6. permitAll 与 anonymous 的一些探讨

**开放请求** 其实通常情况下跟 **匿名请求** 有交叉。它们的主要区别在于： 当前的 `Authentication` 为 `null` 时 `permitAll` 是放行的，而 `anonymous` 需要 `Authentication` 为 `AnonymousAuthenticationToken` 。这里是比较难以理解的，下面是来自 Spring 文档中的一些信息：

> 通常，采用“默认拒绝”的做法被认为是一种良好的安全做法，在该方法中，您明确指定允许的内容，并禁止其他所有内容。定义未经身份验证的用户可以访问的内容的情况与此类似，尤其是对于Web应用程序。许多站点要求用户必须通过身份验证才能使用少数几个URL（例如，主页和登录页面）。在这种情况下，最简单的是为这些特定的URL定义访问配置属性，而不是为每个受保护的资源定义访问配置属性。换句话说，有时很高兴地说默认情况下需要ROLE_SOMETHING，并且只允许该规则的某些例外，例如应用程序的登录，注销和主页。您还可以从过滤器链中完全忽略这些页面，从而绕过访问控制检查，
> 这就是我们所说的匿名身份验证。

使用 `permitAll()` 将配置授权，以便在该特定路径上允许所有请求（来自匿名用户和已登录用户）,`anonymous()` 主要是指用户的状态（是否登录）。基本上，直到用户被“认证”为止，它就是“匿名用户”。就像每个人都有“默认角色”一样。

