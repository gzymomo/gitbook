[TOC]

SpringSecurity 在角色继承上有两种不同的写法，在 Spring Boot2.0.8（对应 Spring Security 也是 5.0.11）上面是一种写法，从 Spring Boot2.1.0（对应 Spring Security5.1.1）又是另外一种写法。

# 以前的写法
这里说的以前写法，就是指 SpringBoot2.0.8（含）之前的写法，在之前的写法中，角色继承只需要开发者提供一个 RoleHierarchy 接口的实例即可，例如下面这样：
```java
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    String hierarchy = "ROLE_dba > ROLE_admin ROLE_admin > ROLE_user";
    roleHierarchy.setHierarchy(hierarchy);
    return roleHierarchy;
}
```
在这里我们提供了一个 RoleHierarchy 接口的实例，使用字符串来描述了角色之间的继承关系， ROLE_dba 具备 ROLE_admin 的所有权限，而 ROLE_admin 则具备 ROLE_user 的所有权限，继承与继承之间用一个空格隔开。提供了这个 Bean 之后，以后所有具备 ROLE_user 角色才能访问的资源， ROLE_dba 和 ROLE_admin 也都能访问，具备 ROLE_amdin 角色才能访问的资源， ROLE_dba 也能访问。

# 现在的写法
但是上面这种写法仅限于 Spring Boot2.0.8（含）之前的版本，在之后的版本中，这种写法则不被支持，新版的写法是下面这样：
```java
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
    String hierarchy = "ROLE_dba > ROLE_admin \n ROLE_admin > ROLE_user";
    roleHierarchy.setHierarchy(hierarchy);
    return roleHierarchy;
}
```
变化主要就是分隔符，将原来用空格隔开的地方，现在用换行符了。

上面两种不同写法都是配置角色的继承关系，配置完成后，接下来指定角色和资源的对应关系即可，如下：
```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests().antMatchers("/admin/**")
            .hasRole("admin")
            .antMatchers("/db/**")
            .hasRole("dba")
            .antMatchers("/user/**")
            .hasRole("user")
            .and()
            .formLogin()
            .loginProcessingUrl("/doLogin")
            .permitAll()
            .and()
            .csrf().disable();
}
```
这个表示` /db/**` 格式的路径需要具备 dba 角色才能访问，` /admin/** `格式的路径则需要具备 admin 角色才能访问， `/user/** ` 格式的路径，则需要具备 user 角色才能访问，此时提供相关接口，会发现，dba 除了访问` /db/**` ，也能访问 `/admin/**` 和 `/user/**` ，admin 角色除了访问` /admin/** `，也能访问 `/user/**` ，user 角色则只能访问 ` /user/** `。

# 源码分析
这样两种不同的写法，其实也对应了两种不同的解析策略，角色继承关系的解析在 RoleHierarchyImpl 类的 buildRolesReachableInOneStepMap 方法中，Spring Boot2.0.8（含）之前该方法的源码如下：
```java
private void buildRolesReachableInOneStepMap() {
	Pattern pattern = Pattern.compile("(\\s*([^\\s>]+)\\s*>\\s*([^\\s>]+))");
	Matcher roleHierarchyMatcher = pattern
			.matcher(this.roleHierarchyStringRepresentation);
	this.rolesReachableInOneStepMap = new HashMap<GrantedAuthority, Set<GrantedAuthority>>();
	while (roleHierarchyMatcher.find()) {
		GrantedAuthority higherRole = new SimpleGrantedAuthority(
				roleHierarchyMatcher.group(2));
		GrantedAuthority lowerRole = new SimpleGrantedAuthority(
				roleHierarchyMatcher.group(3));
		Set<GrantedAuthority> rolesReachableInOneStepSet;
		if (!this.rolesReachableInOneStepMap.containsKey(higherRole)) {
			rolesReachableInOneStepSet = new HashSet<>();
			this.rolesReachableInOneStepMap.put(higherRole,
					rolesReachableInOneStepSet);
		}
		else {
			rolesReachableInOneStepSet = this.rolesReachableInOneStepMap
					.get(higherRole);
		}
		addReachableRoles(rolesReachableInOneStepSet, lowerRole);
		logger.debug("buildRolesReachableInOneStepMap() - From role " + higherRole
				+ " one can reach role " + lowerRole + " in one step.");
	}
}
```
从这段源码中我们可以看到，角色的继承关系是通过正则表达式进行解析，通过空格进行切分，然后构建相应的 map 出来。

Spring Boot2.1.0（含）之后该方法的源码如下：
```java
private void buildRolesReachableInOneStepMap() {
	this.rolesReachableInOneStepMap = new HashMap<GrantedAuthority, Set<GrantedAuthority>>();
	try (BufferedReader bufferedReader = new BufferedReader(
			new StringReader(this.roleHierarchyStringRepresentation))) {
		for (String readLine; (readLine = bufferedReader.readLine()) != null;) {
			String[] roles = readLine.split(" > ");
			for (int i = 1; i < roles.length; i++) {
				GrantedAuthority higherRole = new SimpleGrantedAuthority(
						roles[i - 1].replaceAll("^\\s+|\\s+$", ""));
				GrantedAuthority lowerRole = new SimpleGrantedAuthority(roles[i].replaceAll("^\\s+|\\s+$
				Set<GrantedAuthority> rolesReachableInOneStepSet;
				if (!this.rolesReachableInOneStepMap.containsKey(higherRole)) {
					rolesReachableInOneStepSet = new HashSet<GrantedAuthority>();
					this.rolesReachableInOneStepMap.put(higherRole, rolesReachableInOneStepSet);
				} else {
					rolesReachableInOneStepSet = this.rolesReachableInOneStepMap.get(higherRole);
				}
				addReachableRoles(rolesReachableInOneStepSet, lowerRole);
				if (logger.isDebugEnabled()) {
					logger.debug("buildRolesReachableInOneStepMap() - From role " + higherRole
							+ " one can reach role " + lowerRole + " in one step.");
				}
			}
		}
	} catch (IOException e) {
		throw new IllegalStateException(e);
	}
}
```
从这里我们可以看到，这里并没有一上来就是用正则表达式，而是先将角色继承字符串转为一个 BufferedReader ，然后一行一行的读出来，再进行解析，最后再构建相应的 map。