# Docker安装Kong API Gateway并使用

来源：微信公众号：南瓜慢说

## 1 简介

Kong不是一个简单的产品，本文讲的Kong主要指的是Kong API Gateway，即API网关。这次主要是简单体验一把，通过Docker安装，然后使用它的Route功能。

![图片](https://mmbiz.qpic.cn/mmbiz_png/6cDkia4YUdoOZEibZMyQWLagt7d9F3rCpnHV1Y8x53WXPTUdDBJrtR3B8mufsyOZicwib6aWibV3hibfd9Dam3ibh8GEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2 安装

创建Docker的Network：

```
# 创建
$ docker network create kong-net
# 检查
$ docker network list
```

Kong可以使用无数据库模式，为了窥探一下它的配置，我们还是使用数据库，启动如下：

```
$ docker run -itd --network=kong-net \
    --name kong-database \
    -e POSTGRES_DB=kong \
    -e POSTGRES_USER=pkslow \
    -e POSTGRES_PASSWORD=pkslow-kong \
    -p 5432:5432 \
    postgres:13
```

接着进行migrations操作，可以理解为是准备数据库：

```
$ docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=pkslow" \
     -e "KONG_PG_PASSWORD=pkslow-kong" \
     kong:2.5.0-ubuntu kong migrations bootstrap
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/6cDkia4YUdoOZEibZMyQWLagt7d9F3rCpnDt8GNCUiacTHOgE58lhEZLhleQZZMwyzbrq8S7FOQjt6gB5nkEP9CkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

准备就绪后，就可以启动Kong了：

```
$ docker run -itd --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_PG_USER=pkslow" \
     -e "KONG_PG_PASSWORD=pkslow-kong" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 127.0.0.1:8001:8001 \
     -p 127.0.0.1:8444:8444 \
     kong:2.5.0-ubuntu
```

它的Admin端口为8001，通过下面命令验证：

```
$ curl -i http://localhost:8001/
```

## 3 测试Route功能

先创建一个服务，可以理解为注册一个服务，服务名为pkslow，地址为( www.pkslow.com )：

```
$ curl -X POST --url http://localhost:8001/services/ --data 'name=pkslow' --data 'url=https://www.pkslow.com'
```

创建路由规则，路径为/pkslow，对应的服务为pkslow：

```
$ curl -X POST --url http://localhost:8001/services/pkslow/routes --data 'paths[]=/pkslow'
```

这样，当我们访问路径/pkslow时，其它访问的就是服务pkslow的内容。

访问测试，注意端口为8000了：

```
$ curl -i -X GET --url http://localhost:8000/pkslow
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/6cDkia4YUdoOZEibZMyQWLagt7d9F3rCpn4QVc8rbpVTUrnIvExiaTibKDb4stJpNVGdhBU1PZl9s6js3vD8ahFNyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

到此，我们就成功安装并使用了Kong Gateway的Route功能了。

## 4 总结

Kong的强大在于它可以安装许多的插件来实现各种功能，如验证、限流、缓存等。它的强大，等你来挖掘。

# Kong入门简介

- [Kong 入门简介](https://www.cnblogs.com/bhfdz/p/14303309.html)

## 一、Kong安装

### 1.创建网络

```shell
sudo docker network create kong-net
```

### 2.创建数据库（采用postgres）

```shell
sudo docker run -d --name kong-database \
--network=kong-net \
-p 5432:5432 \
-e "POSTGRES_USER=kong" \
-e "POSTGRES_DB=kong" \
-e "POSTGRES_PASSWORD=kong" \
postgres:9.6
```

### 3.数据库管理工具（非必须）

```shell
sudo docker run -d -p 5433:80 --name pgadmin4 \
--network=kong-net \
-e PGADMIN_DEFAULT_EMAIL=admin@123.com \
-e PGADMIN_DEFAULT_PASSWORD=123456 \
dpage/pgadmin4
```

### 4.初始化Kong数据库

```shell
sudo docker run --rm \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kong" \
-e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
kong:latest kong migrations bootstrap
```

### 5.创建Kong

```shell
sudo docker run -d --name kong \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kong" \
-e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
kong:latest
```

### 6.初始化Konga数据库（推荐的Kong管理后台，*实力允许可以进行二开！*）

```shell
sudo docker run --rm --network=kong-net pantsel/konga -c prepare -a postgres -u postgresql://kong:kong@kong-database:5432/konga_db
```

### 7.创建Konga

```shell
sudo docker run -p 1337:1337 \
 --network=kong-net \
 -e "DB_ADAPTER=postgres" \
 -e "DB_HOST=kong-database" \
 -e "DB_USER=kong" \
 -e "DB_PASSWORD=kong" \
 -e "DB_DATABASE=konga_db" \
 -e "KONGA_HOOK_TIMEOUT=120000" \
 -e "NODE_ENV=production" \
 --name konga \
 pantsel/konga
```

### 8.万事大吉

登录http://localhost:1337进入Konga，配置 Kong's admin API的连接，然后激活此链接，炫酷的面板，大吉大利！！！
 ![img](https://img2020.cnblogs.com/blog/1116614/202101/1116614-20210120155225838-2092328951.png)

![img](https://img2020.cnblogs.com/blog/1116614/202101/1116614-20210120155235131-999953100.png)

## 二、服务初体验

### 1.添加服务

```shell
curl -i -X POST http://localhost:8001/services \
     --data name=example_service \
     --data url='http://mockbin.org'
```

http://mockbin.org ，点进入你就知道是干啥的了！以下验证此服务

```shell
curl -i http://localhost:8001/services/example_service
```

### 2.添加路由

```shell
curl -i -X POST http://localhost:8001/services/example_service/routes \
--data 'paths[]=/mock' \
 --data 'name=mocking'
```

以下验证此服务

```shell
curl -i -X GET http://localhost:8000/mock
```

### 3.添加限流

```shell
curl -i -X POST http://localhost:8001/plugins \
--data "name=rate-limiting" \
--data "config.minute=5" \
--data "config.policy=local"
```

以下验证此服务，每分钟只能请求5次

```shell
curl -i -X GET http://localhost:8000/mock/request
```

超过5次，

```json
{
  "message":"API rate limit exceeded"
}
```

如下图

![img](https://img2020.cnblogs.com/blog/1116614/202101/1116614-20210120155247742-2094481912.png)

### 4.代理缓存

```shell
curl -i -X POST http://localhost:8001/plugins \
--data name=proxy-cache \
--data config.content_type="application/json" \
--data config.cache_ttl=30 \
--data config.strategy=memory
```

注意抓取请求头的变化（与教程有点出入，不太好使！）

```shell
curl -i -X GET http://localhost:8000/mock/request
```

删除缓存

```shell
curl -i -X DELETE http://localhost:8001/proxy-cache
```

### 5.身份验证

```shell
curl -X POST http://localhost:8001/routes/mocking/plugins \
--data name=key-auth
```

再次访问服务，HTTP/1.1 401 Unauthorized

![img](https://img2020.cnblogs.com/blog/1116614/202101/1116614-20210120155256312-407486662.png)

设置使用者

```shell
curl -i -X POST -d "username=consumer&custom_id=consumer" http://localhost:8001/consumers/
```

创建凭据，对于此示例，将密钥设置为apikey。如果未输入任何密钥，则Kong将自动生成密钥。

```shell
curl -i -X POST http://localhost:8001/consumers/consumer/key-auth -d 'key=apikey'
```

返回结果如下：

```json
{
	"created_at": 1611110699,
	"id": "6695bd72-16e6-490d-b983-c141c39b5da8",
	"tags": null,
	"ttl": null,
	"key": "apikey",
	"consumer": {
		"id": "cbdec9e6-70aa-4166-9289-e1fe5737ab6e"
	}
}
```

再次访问服务，返回正常

```shell
curl -i http://localhost:8000/mock/request -H 'apikey:apikey'
```

### 6.禁用插件（可选）

```shell
curl -X GET http://localhost:8001/routes/mocking/plugins/
```

以下是返回的数据

```json
{
	"next": null,
	"data": [{
		"created_at": 1611109944,
		"id": "e488b6e6-6183-499c-b430-0aa676245ee5",
		"tags": null,
		"enabled": true,
		"protocols": ["grpc", "grpcs", "http", "https"],
		"name": "key-auth",
		"consumer": null,
		"service": null,
		"route": {
			"id": "ed6baf5a-5d32-4550-91d8-661fc3539e44"
		},
		"config": {
			"key_in_query": true,
			"key_names": ["apikey"],
			"key_in_header": true,
			"run_on_preflight": true,
			"anonymous": null,
			"hide_credentials": false,
			"key_in_body": false
		}
	}]
}
```

禁用此插件

```shell
curl -X PATCH http://localhost:8001/routes/mocking/plugins/e488b6e6-6183-499c-b430-0aa676245ee5 \
 --data "enabled=false"
```

### 7.负载均衡

配置上游服务

```shell
curl -X POST http://localhost:8001/upstreams \
 --data name=upstream
```

以前配置的服务指向上游

```shell
curl -X PATCH http://localhost:8001/services/example_service \
--data host='upstream'
```

向上游添加目标，***此处玩脱了容器卡住多次也不能添加成功***

```shell
curl -X POST http://localhost:8001/upstreams/upstream/targets \
--data target='mockbin.org:80'

curl -X POST http://localhost:8001/upstreams/upstream/targets \
--data target='httpbin.org:80'
```

浏览器中访问 http://localhost:8000/mock 进行验证

特别声明文中参考：
 https://www.cnblogs.com/jerryqm/p/13030425.html