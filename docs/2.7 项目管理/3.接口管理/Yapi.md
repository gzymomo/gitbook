# 一、Yapi介绍



![img](https://mmbiz.qpic.cn/mmbiz_jpg/R5ic1icyNBNd7upvdpIIj6VNHz36cbahX4CcssCAhbb81zhJQRJSY9Y7YqeLzE4AYTGVmqZ5FIWmicrYlIRP14UHQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

YApi 是高效、易用、功能强大的 api 管理平台，旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API，YApi 还为用户提供了优秀的交互体验，开发人员只需利用平台提供的接口数据写入工具以及简单的点击操作就可以实现接口的管理。

Yapi是去哪儿网开源的一款接口管理工具。Yapi旨意将接口作为一个公共的可视化的方式打通前端、后台、测试环节，整合在一块，共同使用维护，提高接口的维护成本。Yapi是一款免费开源的Api接口文档工具，需要下载部署在自己的服务器上。Yapi也是我们现在正在使用的接口文档工具。

## 特点

主要特点如下：

- 权限管理  YApi 成熟的团队管理扁平化项目权限配置满足各类企业的需求；
- 可视化接口管理 基于 websocket 的多人协作接口编辑功能和类 postman 测试工具，让多人协作成倍提升开发效率；
- Mock Server 易用的 Mock Server，再也不用担心 mock 数据的生成了；
- 自动化测试 完善的接口自动化测试,保证数据的正确性；
- 数据导入 支持导入 swagger, postman, har 数据格式，方便迁移旧项目；
- 插件机制  强大的插件机制，满足各类业务需求；

# 二、Yapi安装部署

## 2.1 Docker构建Yapi

1、启动 MongoDB

```
docker run -d --name mongo-yapi mongo
```

2、获取 Yapi 镜像，版本信息可在 阿里云镜像仓库 查看

```
docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
```

3、初始化 Yapi 数据库索引及管理员账号

```
docker run -it --rm  --link mongo-yapi:mongo --entrypoint npm  --workdir /api/vendors registry.cn-hangzhou.aliyuncs.com/anoy/yapi  run install-server
```

> 自定义配置文件挂载到目录 /api/config.json，官方自定义配置文件 -> 传送门

4、启动 Yapi 服务

```
docker run -d  --name yapi  --link mongo-yapi:mongo  --workdir /api/vendors  -p 3000:3000  registry.cn-hangzhou.aliyuncs.com/anoy/yapi  server/app.js
```

## 2.2 手动构建 yapi 镜像

1、下载 YAPI 到本地

```
wget -o yapi.tar.gz https://github.com/YMFE/yapi/archive/v1.8.0.tar.gz
下载地址：https://github.com/YMFE/yapi/releases
```

2、编辑 Dockerfile

```
FROM node:12-alpine as builder
RUN apk add --no-cache git python make openssl tar gcc
COPY yapi.tar.gz /home
RUN cd /home && tar zxvf yapi.tar.gz && mkdir /api && mv /home/yapi-1.8.0 /api/vendors
RUN cd /api/vendors && \    npm install --production --registry https://registry.npm.taobao.org
FROM node:12-alpine
MAINTAINER 545544032@qq.com
ENV TZ="Asia/Shanghai" HOME="/"
WORKDIR ${HOME}
COPY --from=builder /api/vendors /api/vendors
COPY config.json /api/
EXPOSE 3000
ENTRYPOINT ["node"]
```

3、构建镜像

```
docker build -t yapi .
```

# 三、使用 Yapi

访问 http://localhost:3000  登录账号 admin@admin.com，密码 ymfe.org



![img](https://mmbiz.qpic.cn/mmbiz/R5ic1icyNBNd4MhibPoX2urKYAQeNMInnVjkoBRtZibLSmIiaGd2rtah7Z6I8ZQ0dv6icziagZo7Ck0Z853j7ib7r0WWcg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz/R5ic1icyNBNd4MhibPoX2urKYAQeNMInnVjHkRjzU4aicMvpKZMmTYCA1YopaQWfbVLK5ibqRj93m0QskawbDm9jj1g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

至此，帅气的 Yapi 就可以轻松使用啦！



## 3.1 其他相关操作

关闭 Yapi

```
docker stop yapi
```

启动 Yapi

```
docker start yapi
```

升级 Yapi

```
# 1、停止并删除旧版容器
docker rm -f yapi
# 2、获取最新镜像d
ocker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
# 3、启动新容器
docker run -d \  --name yapi \  --link mongo-yapi:mongo \  --workdir /api/vendors \  -p 3000:3000 \  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \  server/app.js
```

