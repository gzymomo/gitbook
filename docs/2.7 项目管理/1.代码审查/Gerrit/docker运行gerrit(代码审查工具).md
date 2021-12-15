# 在生产环境中使用Docker创建Gerrit



```csharp
version: '3'
services:
  gerrit:
    image: gerritcodereview/gerrit
    ports:
      - "29418:29418"
      - "8010:8080"
    depends_on:
      - ldap
    volumes:
      - /usr/local/docker/gerrit/etc:/var/gerrit/etc
      - /usr/local/docker/gerrit/git:/var/gerrit/git
      - /usr/local/docker/gerrit/db:/var/gerrit/db
      - /usr/local/docker/gerrit/index:/var/gerrit/index
      - /usr/local/docker/gerrit/cache:/var/gerrit/cache
    environment:
      - CANONICAL_WEB_URL=http://localhost
    #entrypoint: /entrypoint.sh init
  ldap:
    image: osixia/openldap
    ports:
      - "389:389"
      - "636:636"
    environment:
      - LDAP_ADMIN_PASSWORD=secret
    volumes:
      - /usr/local/docker/gerrit/ldap/var:/var/lib/ldap
      - /usr/local/docker/gerrit/ldap/etc:/etc/ldap/slapd.d
  ldap-admin:
    image: osixia/phpldapadmin
    ports:
      - "6443:443"
    environment:
      - PHPLDAPADMIN_LDAP_HOSTS=ldap
```

# /usr/local/docker/gerrit/etc/gerrit.config 创建配置文件



```csharp
[gerrit]
  basePath = git
  canonicalWebUrl = http://localhost

[index]
  type = LUCENE

[auth]
  type = ldap
  gitBasicAuth = true

[ldap]
  server = ldap://ldap
  username=cn=admin,dc=example,dc=org
  accountBase = dc=example,dc=org
  accountPattern = (&(objectClass=person)(uid=${username}))
  accountFullName = displayName
  accountEmailAddress = mail

[sendemail]
  smtpServer = localhost

[sshd]
  listenAddress = *:29418

[httpd]
  listenUrl = http://*:8080/

[cache]
  directory = cache

[container]
  user = root
```

# /usr/local/docker/gerrit/etc/secure.config 创建配置文件



```csharp
[ldap]
  password = secret
```

# 初始化环境

1.修改docker-compose.yml
 把#entrypoint: /entrypoint.sh init注释放开!



![img](https:////upload-images.jianshu.io/upload_images/18557813-e695f5604b5aecdf.png?imageMogr2/auto-orient/strip|imageView2/2/w/786/format/webp)

image.png

2.执行命令: docker-compose up gerrit

# 后台启动

1.把#entrypoint: /entrypoint.sh init注释掉!

2.再输入命令: docker-compose up -d

# 配置Gerrit的管理员账号

1. 访问:https:ip:6443

   ![img](https:////upload-images.jianshu.io/upload_images/18557813-4048041c81945ac5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

   image.png

2.登录
 账号:cn=admin,dc=example,dc=org
 密码:secret

3.创建gerrit账号
 1.点击Create a child entry



![img](https:////upload-images.jianshu.io/upload_images/18557813-b38fb8aa6ec9d6d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

2.选择账号模板



![img](https:////upload-images.jianshu.io/upload_images/18557813-46e1f0f8e638ae80.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

3.填入账号信息
 参数为:
 Given Name: Gerrit
 Last Name: Admin
 Common Name: Gerrit Admin
 User ID: gerritadmin
 Email: gerritadmin@localdomain
 Password: secret

![img](https:////upload-images.jianshu.io/upload_images/18557813-8bc4c80ae8b98af6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

4.提交到LDAP (commit the changes to LDAP)

![img](https:////upload-images.jianshu.io/upload_images/18557813-bb4f664c610b7163.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

# 登录 Gerrit

1.输入地址
 我这边配置的地址是:http:ip:8010

![img](https:////upload-images.jianshu.io/upload_images/18557813-3be7536f51681ac2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

2.登录



![img](https:////upload-images.jianshu.io/upload_images/18557813-3875f813308cc494.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

3.登录成功

![img](https:////upload-images.jianshu.io/upload_images/18557813-d86e54bee78a7489.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png



作者：架构师与哈苏
链接：https://www.jianshu.com/p/2740a3f9e9ba
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。