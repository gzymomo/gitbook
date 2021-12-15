- [全新一代API网关，带可视化管理，文档贼友好！](https://juejin.cn/post/6981613816719081479)



## 摘要

提到API网关，大家比较熟悉的有Spring Cloud体系中的Gateway和Zuul，这些网关在使用的时候基本都要修改配置文件或自己开发功能。今天给大家介绍一款功能强大的API网关`apisix`，自带可视化管理功能，多达三十种插件支持，希望对大家有所帮助！

## 简介

apisix是一款云原生微服务API网关，可以为API提供终极性能、安全性、开源和可扩展的平台。apisix基于Nginx和etcd实现，与传统API网关相比，apisix具有动态路由和插件热加载，特别适合微服务系统下的API管理。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6290719135f74b3ba6bc6f8a0fdca1d0~tplv-k3u1fbpfcp-zoom-1.image)

## 核心概念

> 我们先来了解下apisix的一些核心概念，对我们接下来的使用会很有帮助！

- 上游（Upstream）：可以理解为虚拟主机，对给定的多个目标服务按照配置规则进行负载均衡。
- 路由（Route）：通过定义一些规则来匹配客户端的请求，然后对匹配的请求执行配置的插件，并把请求转发给指定的上游。
- 消费者（Consumer）：作为API网关，有时需要知道API的消费方具体是谁，通常可以用来做身份认证。
- 服务（Service）： 可以理解为一组路由的抽象。它通常与上游是一一对应的，路由与服务之间，通常是多对一的关系。
- 插件（Plugin）：API网关对请求的增强操作，可以对请求增加限流、认证、黑名单等一系列功能。可以配置在消费者、服务和路由之上。

## 安装

> 由于官方提供了Docker Compose部署方案，只需一个脚本即可安装apisix的相关服务，非常方便，这里我们也采用这种方案来部署。

- 首先下载`apisix-docker`项目，其实我们只需要使用其中的`example`目录就行了，下载地址：[github.com/apache/apis…](https://github.com/apache/apisix-docker)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32e912efec014a98aaf26c2a7ba913a3~tplv-k3u1fbpfcp-zoom-1.image)

- 接下来我们把`example`目录上传到Linux服务器上去，来了解下这个目录里面的东西；

```bash
drwxrwxrwx. 2 root root   25 Jun 19 10:12 apisix_conf   # apisix配置文件目录
drwxrwxrwx. 2 root root   71 Jun 24 09:36 apisix_log    # apisix日志文件目录
drwxrwxrwx. 2 root root   23 Jun 23 17:10 dashboard_conf  # 可视化工具apisix-dashboard配置文件目录
-rwxrwxrwx. 1 root root 1304 Jun 19 10:12 docker-compose-alpine.yml # docker-compose 部署脚本（alpine）版本
-rwxrwxrwx. 1 root root 1453 Jun 19 10:12 docker-compose.yml # docker-compose 部署脚本
drwxrwxrwx. 2 root root   27 Jun 19 10:12 etcd_conf # ectd配置文件目录
drwxrwxrwx. 3 root root   31 Jun 23 17:06 etcd_data # ectd数据目录
drwxrwxrwx. 2 root root  107 Jun 19 10:12 mkcert
drwxrwxrwx. 2 root root   40 Jun 19 10:12 upstream # 两个测试用的Nginx服务配置
复制代码
```

- 从`docker-compose.yml`中我们可以发现，该脚本不仅启动了apisix、apisix-dashboard、etcd这三个核心服务，还启动了两个测试用的Nginx服务；

```yaml
version: "3"

services:
  # 可视化管理工具apisix-dashboard
  apisix-dashboard:
    image: apache/apisix-dashboard:2.7
    restart: always
    volumes:
    - ./dashboard_conf/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml
    ports:
    - "9000:9000"
    networks:
      apisix:
  
  # 网关apisix
  apisix:
    image: apache/apisix:2.6-alpine
    restart: always
    volumes:
      - ./apisix_log:/usr/local/apisix/logs
      - ./apisix_conf/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    depends_on:
      - etcd
    ##network_mode: host
    ports:
      - "9080:9080/tcp"
      - "9443:9443/tcp"
    networks:
      apisix:
  
  # apisix配置数据存储etcd
  etcd:
    image: bitnami/etcd:3.4.15
    user: root
    restart: always
    volumes:
      - ./etcd_data:/bitnami/etcd
    environment:
      ETCD_ENABLE_V2: "true"
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ADVERTISE_CLIENT_URLS: "http://0.0.0.0:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
    ports:
      - "2379:2379/tcp"
    networks:
      apisix:

  # 测试用nginx服务web1，调用返回 hello web1
  web1:
    image: nginx:1.19.0-alpine
    restart: always
    volumes:
      - ./upstream/web1.conf:/etc/nginx/nginx.conf
    ports:
      - "9081:80/tcp"
    environment:
      - NGINX_PORT=80
    networks:
      apisix:

  # 测试用nginx服务web2，调用返回 hello web2
  web2:
    image: nginx:1.19.0-alpine
    restart: always
    volumes:
      - ./upstream/web2.conf:/etc/nginx/nginx.conf
    ports:
      - "9082:80/tcp"
    environment:
      - NGINX_PORT=80
    networks:
      apisix:

networks:
  apisix:
    driver: bridge
复制代码
```

- 在`docker-compose.yml`文件所在目录下，使用如下命令可以一次性启动所有服务；

```bash
docker-compose -p apisix-docker up -d
复制代码
```

- 启动成功后，使用如下命令可查看所有服务的运行状态；

```bash
docker-compose -p apisix-docker ps
复制代码
              Name                            Command               State                       Ports                     
--------------------------------------------------------------------------------------------------------------------------
apisix-docker_apisix-dashboard_1   /usr/local/apisix-dashboar ...   Up      0.0.0.0:9000->9000/tcp                        
apisix-docker_apisix_1             sh -c /usr/bin/apisix init ...   Up      0.0.0.0:9080->9080/tcp, 0.0.0.0:9443->9443/tcp
apisix-docker_etcd_1               /opt/bitnami/scripts/etcd/ ...   Up      0.0.0.0:2379->2379/tcp, 2380/tcp              
apisix-docker_web1_1               /docker-entrypoint.sh ngin ...   Up      0.0.0.0:9081->80/tcp                          
apisix-docker_web2_1               /docker-entrypoint.sh ngin ...   Up      0.0.0.0:9082->80/tcp 
复制代码
```

- 接下来就可以通过可视化工具来管理apisix了，登录账号密码为`admin:admin`，访问地址：http://192.168.5.78:9000/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdfe7a0d4a644c75b74b8d3004b72ccc~tplv-k3u1fbpfcp-zoom-1.image)

- 登录之后看下界面，还是挺漂亮的，apisix搭建非常简单，基本无坑；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/537cd098c0e44cfc812c3fe72d540963~tplv-k3u1fbpfcp-zoom-1.image)

- 还有两个测试服务，`web1`访问地址：http://192.168.5.78:9081/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f1e55cbb56e43678952147836f80948~tplv-k3u1fbpfcp-zoom-1.image)

- 另一个测试服务`web2`访问地址：http://192.168.5.78:9082/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1aa90680ff14419c8240867b1e197949~tplv-k3u1fbpfcp-zoom-1.image)

## 使用

> apisix作为新一代的网关，不仅支持基本的路由功能，还提供了丰富的插件，功能非常强大。

### 基本使用

> 我们先来体验下apisix的基本功能，之前已经启动了两个Nginx测试服务`web1`和`web2`，接下来我们将通过apisix的路由功能来访问它们。

- 首先我们需要创建上游（Upstream），上游相当于虚拟主机的概念，可以对真实的服务提供负载均衡功能；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e7e353cd99348599c95e0fc3fc2a930~tplv-k3u1fbpfcp-zoom-1.image)

- 创建`web1`的上游，设置好名称、负载均衡算法和目标节点信息；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03ca404e2c4a4728b3cabf6cc2bf8af2~tplv-k3u1fbpfcp-zoom-1.image)

- 再按照上述方法创建`web2`的上游，创建完成后上游列表显示如下；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9285d44f69844ad8ad3e42b15e268801~tplv-k3u1fbpfcp-zoom-1.image)

- 再创建`web1`的路由（Route），路由可以用于匹配客户端的请求，然后转发到上游；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34827d29dc3f4758ab7cd2544945e952~tplv-k3u1fbpfcp-zoom-1.image)

- 再选择好路由的上游为`web1`；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1c90c8b6de94d22873ec286d16c5379~tplv-k3u1fbpfcp-zoom-1.image)

- 接下来选择需要应用到路由上的插件，apisix的插件非常丰富，多达三十种，作为基本使用，我们暂时不选插件；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/144c12a591a746e7aeafeb871af40a0a~tplv-k3u1fbpfcp-zoom-1.image)

- 再创建`web2`的路由，创建完成后路由列表显示如下；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17e3003d3fba48a79269c31a791e66ff~tplv-k3u1fbpfcp-zoom-1.image)

- 接下来我们通过apisix网关访问下`web1`服务：http://192.168.5.78:9080/web1/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dc4faa6e921485ab917171d6bc97f5c~tplv-k3u1fbpfcp-zoom-1.image)

- 接下来我们通过apisix网关访问下`web2`服务：http://192.168.5.78:9080/web2/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e6d0a44c94c4b68bbc4a38216410fb7~tplv-k3u1fbpfcp-zoom-1.image)

### 进阶使用

> apisix通过启用插件，可以实现一系列丰富的功能，下面我们来介绍几个实用的功能。

#### 身份认证

> 使用JWT来进行身份认证是一种非常流行的方式，这种方式在apisix中也是支持的，可以通过启用`jwt-auth`插件来实现。

- 首先我们需要创建一个消费者对象（Consumer）；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/551f8ab4ee28405fbb399fc40a0ad404~tplv-k3u1fbpfcp-zoom-1.image)

- 然后在插件配置中启用`jwt-auth`插件；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87db7687256f420a8c96bb0f72c7d047~tplv-k3u1fbpfcp-zoom-1.image)

- 启用插件时配置好插件的`key`和`secret`；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4107b59d41734fd881fbc937bc2a2a98~tplv-k3u1fbpfcp-zoom-1.image)

- 创建成功后消费者列表时显示如下；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83dbc0f4c599415abc2c14a96516d838~tplv-k3u1fbpfcp-zoom-1.image)

- 之后再创建一个路由，路由访问路径匹配`/auth/*`，只需启用`jwt-auth`插件即可；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14afaec76c6747769794a494fd4bf4bb~tplv-k3u1fbpfcp-zoom-1.image)

- 访问接口获取生成好的JWT Token，需要添加两个参数，`key`为JWT插件中配置的key，`payload`为JWT中存储的自定义负载数据，JWT Token生成地址：http://192.168.5.78:9080/apisix/plugin/jwt/sign

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d60ba76976bc42a18fadae2fc785e704~tplv-k3u1fbpfcp-zoom-1.image)

- 不添加JWT Token访问路由接口，会返回401，接口地址：http://192.168.5.78:9080/auth/

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/feb4f6c9df5d42d9babf4c69857c3fe8~tplv-k3u1fbpfcp-zoom-1.image)

- 在请求头`Authorization`中添加JWT Token后即可正常访问；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1bd5f941770418f996f5f05f85d46ed~tplv-k3u1fbpfcp-zoom-1.image)

- 当然apisix支持的身份认证并不只这一种，还有下面几种。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88b422d2b67b431c8a2e5afb2b03334c~tplv-k3u1fbpfcp-zoom-1.image)

#### 限流功能

> 有时候我们需要对网关进行限流操作，比如每个客户端IP在30秒内只能访问2次接口，可以通过启用`limit-count`插件来实现。

- 我们在创建路由的时候可以选择配置`limit-count`插件；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f8d64e0fc6654463b7cba46ac6dd7e68~tplv-k3u1fbpfcp-zoom-1.image)

- 然后对`limit-count`插件进行配置，根据`remote_addr`进行限流；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1977b4b185874d78b9add6f0d1626871~tplv-k3u1fbpfcp-zoom-1.image)

- 当我们在30秒内第3次调用接口时，apisix会返回503来限制我们的调用。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6be02a0d1c754bddab471c29a1d01540~tplv-k3u1fbpfcp-zoom-1.image)

#### 跨域支持

> 如果你想让网关支持跨域访问的话，可以通过启用`cors`插件来实现。

- 我们在创建路由的时候可以选择配置`cors`插件；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16ccd84db15843228a6fbfb5802d6b52~tplv-k3u1fbpfcp-zoom-1.image)

- 然后对`cors`插件进行配置，配置好跨域访问策略；

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d0b9bc2508043779bbc6ec59e0d267d~tplv-k3u1fbpfcp-zoom-1.image)

- 调用接口测试可以发现接口已经返回了CORS相关的请求头。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5821704dbea048648443a3fd32622373~tplv-k3u1fbpfcp-zoom-1.image)

## 总结

体验了一把apisix这个全新一代的API网关，有可视化管理的网关果然不一样，简单易用，功能强大！如果你的微服务是云原生的话，可以试着用它来做网关。

其实apisix并不是个小众框架，很多国内外大厂都在使用了，如果你想知道哪些公司在使用，可以参考下面的连接。

[github.com/apache/apis…](https://github.com/apache/apisix/blob/master/powered-by.md)

## 参考资料

apisix的官方文档非常友好，支持中文，简直是业界良心！过一遍官方文档基本就能掌握apisix了。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17a59c6d8623400ea7ae7caaf1dbb20e~tplv-k3u1fbpfcp-zoom-1.image)

官方文档：[apisix.apache.org/zh/docs/api…](https://apisix.apache.org/zh/docs/apisix/getting-started)

## 项目源码地址

[github.com/apache/apis…](https://github.com/apache/apisix-docker)


作者：MacroZheng
链接：https://juejin.cn/post/6981613816719081479
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。