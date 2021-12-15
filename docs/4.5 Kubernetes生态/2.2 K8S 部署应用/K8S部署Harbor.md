- [Kubernetes-5-2：Harbor仓库的几种高可用方案与搭建](https://www.cnblogs.com/v-fan/p/14385057.html)
- [Kubernetes 之 Harbor 仓库](https://www.escapelife.site/posts/74f6395f.html)

# 一、Kubernetes之Harbor仓库

## 1. Harbor 基本组件

企业级环境中基于 Harbor 搭建自己的安全认证仓库。

Docker 容器应用的开发和运行离不开可靠的镜像管理，虽然 Docker 官方也提供了公共的镜像仓库，但是从安全和效率等方面考虑，部署我们私有环境内的  Registry 也是非常必要的。Harbor 是由 VMware 公司开源的企业级的 Docker Registry  管理项目，它包括权限管理(RBAC)、AD/LDAP集成、日志审核、管理界面、自我注册、镜像复制和中文支持等功能，在新版本中还添加了Helm仓库托管的支持。



![Kuternetes私有仓库Harbor](https://www.escapelife.site/images/kuternetes-harbor-tool-1.png)

### 1.1 Kuternetes私有仓库Harbor

`Harbor` 的每个组件都是以 `Docker` 容器的形式构建的，使用 `Docker Compose` 来对它进行部署。用于部署 `Harbor` 的 `Docker Compose` 模板位于 `/Deployer/docker-compose.yml` 中，其由 `5` 个容器组成，这几个容器通过 `Docker link` 的形式连接在一起，在容器之间通过容器名字互相访问。对终端用户而言，只需要暴露 `Proxy`(即`Nginx`) 的服务端口即可。

- Proxy
  - 由`Nginx`服务器构成的反向代理
- Registry
  - 由`Docker`官方的开源官方的开源`Registry`镜像构成的容器实例
- UI
  - 即架构中的`core services`服务，构成此容器的代码是`Harbor`项目的主体
- MySQL
  - 由官方`MySQL`镜像构成的数据库容器
- Log
  - 运行着`rsyslogd`的容器，通过`log-driver`的形式收集其他容器的日志

## 2. Harbor 特性解释

> **主要介绍 Harbor 工具的特性和优点**

![Kuternetes私有仓库Harbor](https://www.escapelife.site/images/kuternetes-harbor-tool-2.png)

### 2.1 Kuternetes私有仓库Harbor

- [1] 基于角色控制
  - 用户和仓库都是基于项目进行组织的，而用户基于项目可以拥有不同的权限
- [2] 基于镜像的复制策略
  - 镜像可以在多个`Harbor`实例之间进行复制实例之间进行复制
- [3] 支持 LDAP
  - `Harbor`的用户授权可以使用已经存在`LDAP`用户
- [4] 镜像删除&垃圾回收
  - `Image`可以被删除并且回收`Image`占用的空间
- [5] 用户 UI
  - 用户可以轻松的浏览、搜索镜像仓库以及对项目进行管理
- [6] 轻松的部署功能
  - `Harbor`提供了提供了`online`/`offline`/`virtualappliance`安装
- [7] Harbor 和 docker registry 关系
  - `Harbor`实质上是对`docker registry`做了封装，扩展了自己的业务模块

## 3. Harbor 认证过程

> **认证过程是理解其精髓的核心知识点**

`Harbor` 最核心的功能就是给 `docker registry` 添加上一层权限保护的功能，要实现这个功能，就需要我们在使用 `docker login`、`pull`、`push` 等命令的时候进行拦截，先进行一些权限相关的校验再进行操作，其实这一系列的操作 `docker registry v2` 就已经为我们提供了支持，`v2` 版本集成了一个安全认证的功能，将安全认证暴露给外部服务，让外部服务去实现。

![Harbor整体架构](https://www.escapelife.site/images/kuternetes-harbor-tool-3.jpg)

### 3.1 Harbor整体架构

下述就是完整的授权过程，当用户完成下述过程以后便可以执行相关的`pull`/`push`操作，认证信息会每次都带在请求头中。

- **[a]** `docker daemon`从`docker registry`拉取镜像。
- **[b]** 如果`docker registry`需要进行授权时，`registry`将会返回`401`响应，同时在响应中包含了`docker client`如何进行认证的信息。
- **[c]** `docker client`根据`registry`返回的信息，向`auth server`发送请求获取认证`token`信息。
- **[d]** `auth server`则根据自己的业务实现去验证提交的用户信息是否存符合业务要求。
- **[e]** 用户数据仓库返回用户的相关信息。
- **[f]** `auth server`将会根据查询的用户信息，生成`token`令牌，以及当前用户所具有的相关权限信息。
- **[g]** `docker client`端接收到返回的`200`状态码说明操作成功，在控制台上打印`Login Succeeded`的信息。

![整个登录过程](https://www.escapelife.site/images/kuternetes-harbor-tool-4.jpg)

**整个登录过程**





![Harbor认证流程](https://www.escapelife.site/images/kuternetes-harbor-tool-5.png)

**Harbor认证流程**

## 4. Harbor 配置要求

> **配置要求主要受限于硬件要求**

- **[1] Hardware**

| Resource | Minimum | Recommended |
| -------- | ------- | ----------- |
| **CPU**  | 2 CPU   | 4 CPU       |
| **Mem**  | 4 GB    | 8 GB        |
| **Disk** | 40 GB   | 160 GB      |

- **[2] Software**

| Software           | Version                       | Description                                                  |
| ------------------ | ----------------------------- | ------------------------------------------------------------ |
| **Docker engine**  | version 17.06.0-ce+ or higher | For installation instructions, see [docker engine doc](https://docs.docker.com/engine/installation/) |
| **Docker Compose** | version 1.18.0 or higher      | For installation instructions, see [docker compose doc](https://docs.docker.com/compose/install/) |
| **Openssl**        | latest is preferred           | Used to generate certificate and keys for Harbor             |

- **[3] Network ports**

| Port     | Protocol  | Description                                                  |
| -------- | --------- | ------------------------------------------------------------ |
| **443**  | **HTTPS** | Harbor portal and core API accept HTTPS requests on this port. You can change this port in the configuration file. |
| **4443** | **HTTPS** | Connections to the Docker Content Trust service for Harbor. Only  required if Notary is enabled. You can change this port in the  configuration file. |
| **80**   | **HTTP**  | Harbor portal and core API accept HTTP requests on this port. You can change this port in the configuration file. |

- **[4] Harbor Components**

| Number | Component                   | Version      |
| ------ | --------------------------- | ------------ |
| 1      | **Postgresql**              | 9.6.10-1.ph2 |
| 2      | **Redis**                   | 4.0.10-1.ph2 |
| 3      | **Clair**                   | 2.0.8        |
| 4      | **Beego**                   | 1.9.0        |
| 5      | **Chartmuseum**             | 0.9.0        |
| 6      | **Docker**/**distribution** | 2.7.1        |
| 7      | **Docker**/**notary**       | 0.6.1        |
| 8      | **Helm**                    | 2.9.1        |
| 9      | **Swagger-ui**              | 3.22.1       |

------

## 5. Harbor 安装步骤

> **[Harbor 的官方用户指南手册](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md)**
>
> - [**配置 HTTPS 设置步骤**](https://github.com/goharbor/harbor/blob/master/docs/configure_https.md)
> - [**服务的升级和降级操作**](https://github.com/goharbor/harbor/blob/master/docs/migration_guide.md)

- **[1] 官网主页**

```bash
https://github.com/goharbor/harbor/releases
```

- **[2] 选择合适的资源包**

```bash
# Online installer
harbor-offline-installer-v1.9.2.tgz       605 MB
harbor-offline-installer-v1.9.2.tgz.asc   833 Bytes

# Online installer
harbor-online-installer-v1.9.2.tgz        8.2 KB
harbor-online-installer-v1.9.2.tgz.asc    833 Bytes
```

- **[3] 解压对应安装包**

```bash
# Online installer
$ tar xvf harbor-online-installer-version.tgz

# Offline installer
$ tar xvf harbor-offline-installer-version.tgz

# Mv to /usr/local dir
$ sudo mv harbor /usr/local
```

- **[4] 修改 harbor.cfg 配置文件**

```bash
# Harbor版本
_version = 1.5.0

# 设置访问地址; 可以使用IP/域名(必须)
hostname = docker.escaplife.site

# 默认服务是走http协议，当然也可以设置为https协议(必须)
# 如果设置https协议的话，则nginx ssl选项也是需要设置on的
ui_url_protocol = https

# Job最大进程数; 默认值为3
max_job_workers = 50

# 是否创建证书,创建证书将会在下面的路径下生成(必须)
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key

# 私钥存储路径
secretkey_path = /data

# 设置日志大小
admiral_url = NA
log_rotate_count = 50
log_rotate_size = 200M

# 是否使用代理
http_proxy = xxx
https_proxy = xxx
no_proxy = 127.0.0.1,localhost,ui

# 邮箱设置，发送重置密码邮件时使用
email_identity = xxx
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

# 安装Harbor后管理员UI登陆的密码，默认是Harbor12345
harbor_admin_password = Harbor12345

# 认证方式；比如LDAP、数据库认证，默认是db_auth方式
auth_mode = db_auth

# LDAP认证时配置项
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid
ldap_scope = 2
ldap_timeout = 5
ldap_verify_cert = true
ldap_group_basedn = ou=group,dc=mydomain,dc=com
ldap_group_filter = objectclass=group
ldap_group_gid = cn
ldap_group_scope = 2

#是否开启自动注册
self_registration = on

# Token有效时间，默认30分钟
token_expiration = 30

# 用户创建项目权限控制，默认是everyone(所有人)，也可以设置为adminonly(管理员)
project_creation_restriction = everyone

# Mysql数据库root用户默认密码root123，根据实际时使用来进行修改
db_host = mysql
db_password = root123
db_port = 3306
db_user = root

# Redis配置
redis_url = redis:6379
clair_db_host = postgres
clair_db_password = password
clair_db_port = 5432
clair_db_username = postgres
clair_db = postgres
uaa_endpoint = uaa.mydomain.org
uaa_clientid = id
uaa_clientsecret = secret
uaa_verify_cert = true
uaa_ca_cert = /path/to/ca.pem
registry_storage_provider_name = filesystem
registry_storage_provider_config = xxx
```

- **[5] 创建 https 证书以及配置相关目录权限**

```bash
# 创建证书目录
$ sudo mkdir /data/cert

# 生成私钥
$ sudo openssl genrsa -des3 -out server.key 2048

# 创建证书请求
$ sudo openssl req -new -key server.key -out server.csr

# 备份私钥
$ sudo cp server.key server.key.org

# 私钥去除密码
$ sudo openssl rsa -in server.key.org -out server.key

# 签名证书
$ sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# 赋值权限
$ sudo chmod -R 777 /data/cert
```

- **[6] 运行脚本进行安装**

```bash
$ sudo cd /usr/local/harbor
$ sudo ./install.sh
```

- **[7] 访问测试**

```bash
# 各节点配置/etc/hosts文件
192.168.3.23    reg.escaplife.com

# 访问自己配置的域名地址
https://reg.escaplife.com

# 默认管理员用户密码
admin/Harbor12345
```

- **[8] 上传和下载镜像进行测试**

```bash
# 指定Docker信任我们搭建的私有镜像仓库地址
$ sudo vim /etc/docker/daemon.json
{
    ......
    "insecure-registries": ['https://reg.escaplife.com'],
    ......
}

# 下载镜像测试
$ docker pull escape/nginx-test:v1

# 推送镜像测试
$ docker tag escape/nginx-test:v1 reg.escaplife.com/library/nginx-test:v1
$ docker pull reg.escaplife.com/library/nginx-test:v1

# 用户登录测试
$ docker login https://reg.escaplife.com

# 启动容器测试
$ kubectl run nginx-deployment \
    --image=reg.escaplife.com/library/nginx-test:v1 \
    --port=80 --replicas=1

# 通过IPVS查看对应规则
$ sudo ipvsadm -Ln
```



![Harbor认证流程](https://www.escapelife.site/images/kuternetes-harbor-tool-6.png)

**Harbor认证流程**

![Harbor认证流程](https://www.escapelife.site/images/kuternetes-harbor-tool-7.png)

**Harbor认证流程**

![Harbor认证流程](https://www.escapelife.site/images/kuternetes-harbor-tool-8.png)

**Harbor认证流程**

# 二、Harbor仓库的几种高可用方案与搭建

Harbor官方有推出主从架构和双主架构来实现Harbor的高可用及数据备份。

## 1. 主从架构

 说白了，就是往一台Harbor仓库中push镜像，然后再通过这台Harbor分散下发至所有的从Harbor，类似下图：

![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207114959007-352036911.png)

这个方法保证了数据的冗余性，但是仍然解决不了Harbor主节点的单点问题，当业务量足够大时，甚至会引起主节点崩溃。

## 2. 双主架构

双主复制就是两套Harbor的数据互相同步，来保证数据的一致性，然后两套Harbor的前端再挂一层负载，来达到分均流量、减轻某一台压力过大的现象，也避免了单点故障，类似于下图：



![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207143000985-1085610289.png)

这个方案有一个问题：假设有实例A和实例B互为主备，当A挂掉后，所有的业务流量就会流向B，当A修复后，B不会自动同步A新push的镜像，还要手动将B的同步策略关闭，重新开启才能开始同步。

## 3. 共享存储和共享数据库

实现服务的高可用性和数据的冗余，我们也是推荐用这种高可用方式：



![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207143146463-1797420997.png)

思路如下：

1. 将PostgreSQL服务单独部署出来，并将Harbor中默认创建在PostgreSQL的所有表的结构、初始数据等导入进单独部署的PostgreSQL服务中，PostgreSQL数据的冗余就完全可以使用PostgreSQL的同步策略来实现；
2. Redis服务单独部署出来，其他特殊操作无需执行，Redis的高可用也可直接使用其集群解决方案；
3. 最后存储后端要使用共享存储，来实现数据的统一；

### 3.1 具体操作

**环境介绍**

Harbor基于Docker环境运行，所使用的机器必须有docker及docker-compose

| ip                    | 系统版本 | 服务                      |
| --------------------- | -------- | ------------------------- |
| 192.168.24.253(post1) | Centos7  | PostgreSQL、Redis、Harbor |
| 192.168.24.252(post2) | Centos8  | NFS、Harbor               |

> 在此所有其他服务均搭建在post1中，并且是搭建单机，仅为演示使用，生产中数据的备份等再自行研究。
>
> 我这里post1 和 post2的域名分别为：
>
> hub.vfancloud1.com；hub.vfancloud2.com，记得先要写入hosts文件中以防解析不到。

 

1、先部署一套Harbor，用于将其所有表结构导出，部署过程上文已给出：

```
https://www.cnblogs.com/v-fan/p/13034272.html
```

2、进入导出PostgreSQL表结构

```bash
## 进入PostgreSQL容器
docker exec -it xxxx bash

## 执行 psql 进入数据库
postgres [ / ]$ psql
psql (9.6.14)
Type "help" for help.

## 查看当前所有的数据库，postgres、template0、template1为默认数据库
postgres=# \l
                                   List of databases
     Name     |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
--------------+----------+----------+-------------+-------------+-----------------------
 notaryserver | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
              |          |          |             |             | postgres=CTc/postgres+
              |          |          |             |             | server=CTc/postgres
 notarysigner | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =Tc/postgres         +
              |          |          |             |             | postgres=CTc/postgres+
              |          |          |             |             | signer=CTc/postgres
 postgres     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 registry     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
              |          |          |             |             | postgres=CTc/postgres
 template1    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
              |          |          |             |             | postgres=CTc/postgres
(6 rows)

## 导出表结构及数据
postgres [ / ]$ pg_dump -U postgres registry > /tmp/registry.sql
postgres [ / ]$ pg_dump -U postgres notaryserver > /tmp/notaryserver.sql
postgres [ / ]$ pg_dump -U postgres notarysigner > /tmp/notarysigner.sql
    -U 数据库用户
    -p 访问端口
    -f 指定文件，和 > 功能一样
    -h 指定数据库地址
    -s 表示只导出表结构，不导数据

## 导出到宿主机
docker cp [容器id]:/tmp/registry.sql ./
docker cp [容器id]:/tmp/notaryserver.sql ./
docker cp [容器id]:/tmp/notarysigner.sql ./
```

3、单独部署一套PostgreSQL服务

```bash
# Install RPM
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Install server
sudo yum install -y postgresql13-server

# init db
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb

# 修改远程访问配置
vim /var/lib/pgsql/13/data/postgresql.conf
...
将 listen_addresses = 'localhost' 修改为
listen_addresses = '*'
...

# 添加信任的远程连接,生产中不要添加0.0.0.0
vim /var/lib/pgsql/13/data/pg_hba.conf
...
host    all             all             0.0.0.0/0               trust
# host    all             all             0.0.0.0/0               md5
# 最后一列如果是trust，则登录pg不需要密码，若为md5，则需要密码
...


# start and enable server
sudo systemctl enable postgresql-13
sudo systemctl start postgresql-13

# 检查服务是否启动成功
ps看进程 或 ss看端口号
```

4、给postgresql设置密码，增强安全性

```
## 直接写入新密码
postgres=# \password
输入新的密码：
再次输入：
```

5、将备份的数据，导入进单独部署的postgresql中

```bash
## 创建数据库
postgres=# CREATE DATABASE registry;
postgres=# CREATE DATABASE notaryserver;
postgres=# CREATE DATABASE notarysigner;

## 导入数据
psql -h post1 -U postgres -p 5432 -d registry -f registry.sql 
psql -h post1 -U postgres -p 5432 -d notaryserver -f notaryserver.sql 
psql -h post1 -U postgres -p 5432 -d notarysigner -f notarysigner.sql 
    -U 数据库用户
    -p 访问端口
    -f 指定文件，和 < 功能一样
    -h 指定数据库地址
    -d 指定数据库名
```

6、搭建共享存储(在此以nas为例)

```bash
## 安装nfs工具
yum -y install nfs-utils rpcbind

## 编辑共享目录
vim /etc/exports
...
/alibaba *(rw,no_root_squash,no_all_squash,sync)
...

## 挂载
mkdir /alibaba
mount -t nfs post2:/alibaba/ /alibaba/
```

7、搭建Redis

```bash
## 安装 || 或源码安装自行选择
yum -y install redis

## 
vim /etc/redis
...
bind 0.0.0.0 # 设置所有主机可以连接
requirepass redis # 设置客户端连接密码
daemonize yes # 打开守护进程模式
...

## 启动redis
systemctl start redis

## 查看端口状态
[root@centos7 src]# ss -tnlp | grep 6379 
LISTEN     0      128    127.0.0.1:6379                     *:*                   users:(("redis-server",pid=3405,fd=6))
```

8、配置Harbor

```bash
## 自行下载源码包
https://github.com/goharbor/harbor/releases

## 解压进入
tar zxvf harbor-offline-installer-v2.1.2.tgz && cd harbor/

## 导入镜像
docker load -i harbor.v2.1.2.tar.gz

## 编辑配置文件，需要更改的主要有以下几点：
    1.hostname 改为主机ip或完全限定域名，不要使用127.0.0.1或localhost
    2.https选项，如需要，指定crt和key的路径，若不需要，直接注释掉
    3.harbor_admin_password，默认密码，可以更改
    4.data_volume，数据默认存储位置，设计为共享路径
    5.注释掉database模块 及 Clair模块
    6.开启external_database 和 external_redis模块及正确配置其中参数
    7.集群内所有harbor配置均一样，改一下hostname值即可
```

下边进行实际修改，以下为我的配置：

vim harbor.yml.tmpl

```yaml
hostname: hub.vfancloud1.com

https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /data/cert/server.crt
  private_key: /data/cert/server.key

harbor_admin_password: Harbor12345

data_volume: /alibaba

external_database:
  harbor:
    host: 192.168.24.253
    port: 5432
    db_name: external_redis
    username: postgres
    password: 123456
    ssl_mode: disable
    max_idle_conns: 2
    max_open_conns: 0
  clair:
    host: 192.168.24.253
    port: 5432
    db_name: clair
    username: postgres
    password: 123456
    ssl_mode: disable
  notary_signer:
    host: 192.168.24.253
    port: 5432
    db_name: notarysigner
    username: postgres
    password: 123456
    ssl_mode: disable
  notary_server:
    host: 192.168.24.253
    port: 5432
    db_name: notaryserver
    username: postgres
    password: 123456
    ssl_mode: disable

external_redis:
  # support redis, redis+sentinel
  # host for redis: <host_redis>:<port_redis>
  # host for redis+sentinel:
  #  <host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>
  host: 192.168.24.253:6379
  password: redis
  # sentinel_master_set must be set to support redis+sentinel
  #sentinel_master_set:
  # db_index 0 is for core, it's unchangeable
  registry_db_index: 1
  jobservice_db_index: 2
  chartmuseum_db_index: 3
  clair_db_index: 4
  trivy_db_index: 5
  idle_timeout_seconds: 30
```

开始安装：

```bash
./install.sh
✔ ----Harbor has been installed and started successfully.----

[root@kubenode2 harbor]# docker ps 
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS                   PORTS                                         NAMES
0fffdbdc1efd        goharbor/harbor-jobservice:v2.1.2    "/harbor/entrypoint.…"   3 minutes ago       Up 3 minutes (healthy)                                                 harbor-jobservice
330d73923321        goharbor/nginx-photon:v2.1.2         "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes (healthy)   0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp   nginx
f6511d387e7f        goharbor/harbor-core:v2.1.2          "/harbor/entrypoint.…"   3 minutes ago       Up 3 minutes (healthy)                                                 harbor-core
2fe648a128da        goharbor/harbor-registryctl:v2.1.2   "/home/harbor/start.…"   3 minutes ago       Up 3 minutes (healthy)                                                 registryctl
62c6de742d9b        goharbor/registry-photon:v2.1.2      "/home/harbor/entryp…"   3 minutes ago       Up 3 minutes (healthy)                                                 registry
f5e6b82363bb        goharbor/harbor-portal:v2.1.2        "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes (healthy)                                                 harbor-portal
bb0fb84251f1        goharbor/harbor-log:v2.1.2           "/bin/sh -c /usr/loc…"   3 minutes ago       Up 3 minutes (healthy)   127.0.0.1:1514->10514/tcp                     harbor-log
```

第一台机器安装完毕，可以简单测试一下各项功能，正常后继续部署其他服务器

![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207144449671-1243962150.png)

 开始部署第二台，相同的配置，改一下hostname即可：

```
hostname: hub.vfancloud2.com
```

###  3.2 测试

9、在vfancloud1机器push镜像，在vfancloud2查看是否可以同步并pull

```
## 在192.168.24.253中push镜像
## 先在/etc/docker/daemon.json文件添加以下内容
vim /etc/docker/daemon.json
...
{
        "insecure-registries": ["https://hub.vfancloud1.com","https://hub.vfancloud2.com"]
}
...

## 重新加载docker服务    
systemctl daemon-reload 
systemctl restart docker 

## 登录harbor
[root@centos7 harbor]# docker login hub.vfancloud1.com 
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

## push镜像
[root@centos7 alibaba]# docker tag hub.vfancloud.com/test/myapp:v1 hub.vfancloud1.com/library/myapp:v1
[root@centos7 alibaba]# docker push hub.vfancloud1.com/library/myapp:v1
The push refers to repository [hub.vfancloud1.com/library/myapp]
a0d2c4392b06: Pushed 
05a9e65e2d53: Pushed 
68695a6cfd7d: Pushed 
c1dc81a64903: Pushed 
8460a579ab63: Pushed 
d39d92664027: Pushed 
v1: digest: sha256:9eeca44ba2d410e54fccc54cbe9c021802aa8b9836a0bcf3d3229354e4c8870e size: 1569
```

push成功，仓库1中已有此镜像

![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207144714597-694949122.png)

查看仓库2：



![img](https://img2020.cnblogs.com/blog/1715041/202102/1715041-20210207144749778-692157225.png)

> 注意看域名不同 

在post2导入仓库2的镜像：

```
[root@kubenode2 harbor]# docker pull hub.vfancloud2.com/library/myapp:v1
v1: Pulling from library/myapp
550fe1bea624: Pull complete 
af3988949040: Pull complete 
d6642feac728: Pull complete 
c20f0a205eaa: Pull complete 
438668b6babd: Pull complete 
bf778e8612d0: Pull complete 
Digest: sha256:9eeca44ba2d410e54fccc54cbe9c021802aa8b9836a0bcf3d3229354e4c8870e
Status: Downloaded newer image for hub.vfancloud2.com/library/myapp:v1
hub.vfancloud2.com/library/myapp:v1
```

至此Harbor高可用完成！实际使用中将两台Harbor前边加一层负载即可！