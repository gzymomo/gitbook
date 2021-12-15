> Build a Docker image with the Gerrit code review system

源代码名称:**docker-gerrit**

源代码网址:http://www.github.com/openfrontier/docker-gerrit

[docker-gerrit源代码文档](http://www.github.com/openfrontier/docker-gerrit/wiki)

[docker-gerrit源代码下载](http://www.github.com/openfrontier/docker-gerrit/releases)

Git URL:

复制

```
git://www.github.com/openfrontier/docker-gerrit.git
```

Git Clone代码到本地:

复制

```
git clone http://www.github.com/openfrontier/docker-gerrit
```

Subversion代码到本地:

复制

```
$ svn co --depth empty http://www.github.com/openfrontier/docker-gerrit
Checked out revision 1.
$ cd repo
$ svn up trunk
```

Gerrit Docker镜像

支持PostgreSQL和OpenLDAP集成的Gerrit代码审查系统，此镜像基于openjdk:jre-alpine或openjdk:jre-slim，使此镜像小而快速。

## 版本

基于Alpine

- openfrontier gerrit：最新> 2.14.5.1
- openfrontier/gerrit:2.13.x-> 2.13.9
- openfrontier/gerrit:2.12.x-> 2.12.7
- openfrontier/gerrit:2.11.x-> 2.11.10
- openfrontier/gerrit:2.10.x-> 2.10.6
- 基于Debian
- openfrontier gerrit :jre-slim > 2.14.5.1

## 容器快速开始

- 初始化，并启动gerrit

复制

```
docker run  --name gerrit -d -p 8081:8080 -p 29418:29418 openfrontier/gerrit
```

- 打开浏览器到http://:8080

## 使用HTTP身份验证类型

复制

```
docker run -d -p 8080:8080 -p 29418:29418 -e AUTH_TYPE=HTTP openfrontier/gerrit
```

## 使用另一个容器作为gerrit站点存储，

- 创建卷容器

复制

```
docker run --name gerrit_volume openfrontier/gerrit echo"Gerrit volume container."
```

- 使用上面创建的卷初始化，并启动gerrit

复制

```
docker run -d --volumes-from gerrit_volume -p 8080:8080 -p 29418:29418 openfrontier/gerrit
```

## 使用本地目录作为gerrit站点存储，

- 为gerrit站点创建一个站点目录

复制

```
mkdir ~/gerrit_volume
```

- 使用上面创建的本地目录初始化，并启动gerrit

复制

```
docker run -d -v ~/gerrit_volume:/var/gerrit/review_site -p 8080:8080 -p 29418:29418 openfrontier/gerrit
```

## 启动时安装插件

当调用gerrit init--batch时，可以列出要使用--install plugin=<plugin uname>安装的插件，这可以使用GERRIT_INIT_ARGS环境变量来完成，有关更多信息，请参见[gerrit Documentation](https://gerrit-review.googlesource.com/Documentation/pgm-init.html)。

复制

```
#Install download-commands plugin on start up


docker run -d -p 8080:8080 -p 29418:29418 -e GERRIT_INIT_ARGS='--install-plugin=download-commands' openfrontier/gerrit
```

## 扩展此图像。

与Postgres映像类似，如果您想进行其他配置中间脚本，请在/docker-entrypoint-init.d下添加一个或多个* .sh或* .nohup脚本。默认情况下创建此目录，`/docker-entrypoint-init.d`中的脚本在初始化gerrit之后运行，但在自定义gerrit配置之前，允许你以编程方式重写环境变量项，使用nohup命令将`*.nohup`脚本运行到后台。

还可以使用简单的`Dockerfile`扩展镜像，下面的示例将添加一些脚本，以便在启动时初始化容器。

复制

```
FROM openfrontier/gerrit:latestCOPY gerrit-create-user.sh /docker-entrypoint-init.d/gerrit-create-user.shCOPY gerrit-upload-ssh-key.sh /docker-entrypoint-init.d/gerrit-upload-ssh-key.shCOPY gerrit-init.nohup /docker-entrypoint-init.d/gerrit-init.nohupRUN chmod +x /docker-entrypoint-init.d/*.sh /docker-entrypoint-init.d/*.nohup
```

## 使用docke rized PostgreSQL和OpenLDAP运行dockerized gerrit，

支持[gerrit.config ldap节](https://gerrit-review.googlesource.com/Documentation/config-gerrit.html#ldap)中的所有属性，

复制

```
# Start postgres docker 
docker run 
 --name pg-gerrit 
 -p 5432:5432 
 -e POSTGRES_USER=gerrit2 
 -e POSTGRES_PASSWORD=gerrit 
 -e POSTGRES_DB=reviewdb 
 -d postgres
 #Start gerrit docker ( AUTH_TYPE=HTTP_LDAP is also supported ) 
 docker run 
 --name gerrit 
 --link pg-gerrit:db 
 -p 8080:8080 
 -p 29418:29418 
 -e WEBURL=http://your.site.domain:8080 
 -e DATABASE_TYPE=postgresql 
 -e AUTH_TYPE=LDAP 
 -e LDAP_SERVER=ldap://ldap.server.address 
 -e LDAP_ACCOUNTBASE=<ldap-basedn> 
 -d openfrontier/gerrit
```

## 设置sendemail选项，

支持[gerrit.config sendmail部分](https://gerrit-review.googlesource.com/Documentation/config-gerrit.html#sendemail)中的一些基本属性，

复制

```
#Start gerrit docker with sendemail supported.#All SMTP_* attributes are optional.#Sendemail function will be disabled if SMTP_SERVER is not specified. 
docker run 
 --name gerrit 
 -p 8080:8080 
 -p 29418:29418 
 -e WEBURL=http://your.site.domain:8080 
 -e SMTP_SERVER=smtp.server.address 
 -e SMTP_SERVER_PORT=25 
 -e SMTP_ENCRYPTION=tls 
 -e SMTP_USER=<smtp user> 
 -e SMTP_PASS=<smtp password> 
 -e SMTP_CONNECT_TIMEOUT=10sec 
 -e SMTP_FROM=USER 
 -d openfrontier/gerrit
```

## 设置用户选项

支持[gerrit.config用户部分](https://gerrit-review.googlesource.com/Documentation/config-gerrit.html#user)中的所有属性，

复制

```
#Start gerrit docker with user info provided.#All USER_* attributes are optional. 
docker run 
 --name gerrit 
 -p 8080:8080 
 -p 29418:29418 
 -e WEBURL=http://your.site.domain:8080 
 -e USER_NAME=gerrit 
 -e USER_EMAIL=gerrit@your.site.domain 
 -d openfrontier/gerrit
```

## 设置OAUTH选项

复制

```
 docker run 
 --name gerrit 
 -p 8080:8080 
 -p 29418:29418 
 -e AUTH_TYPE=OAUTH 
 # Don't forget to set Gerrit FQDN for correct OAuth -e WEB_URL=http://my-gerrit.example.com/
 -e OAUTH_ALLOW_EDIT_FULL_NAME=true 
 -e OAUTH_ALLOW_REGISTER_NEW_EMAIL=true 
 # Google OAuth -e OAUTH_GOOGLE_RESTRICT_DOMAIN=your.site.domain 
 -e OAUTH_GOOGLE_CLIENT_ID=1234567890 
 -e OAUTH_GOOGLE_CLIENT_SECRET=dakjhsknksbvskewu-googlesecret 
 -e OAUTH_GOOGLE_LINK_OPENID=true 
 # Github OAuth -e OAUTH_GITHUB_CLIENT_ID=abcdefg 
 -e OAUTH_GITHUB_CLIENT_SECRET=secret123 
 # GitLab OAuth# How to obtain secrets: https://docs.gitlab.com/ee/integration/oauth_provider.html -e OAUTH_GITLAB_ROOT_URL=http://my-gitlab.example.com/ 
 -e OAUTH_GITLAB_CLIENT_ID=abcdefg 
 -e OAUTH_GITLAB_CLIENT_SECRET=secret123 
 # Bitbucket OAuth -e OAUTH_BITBUCKET_CLIENT_ID=abcdefg 
 -e OAUTH_BITBUCKET_CLIENT_SECRET=secret123 
 -e OAUTH_BITBUCKET_FIX_LEGACY_USER_ID=true 
 -d openfrontier/gerrit
```

## 使用gitiles而不是gitweb

复制

```
 docker run 
 --name gerrit 
 -p 8080:8080 
 -p 29418:29418 
 -e GITWEB_TYPE=gitiles 
 -d openfrontier/gerrit
```