# 一、mica mqtt 组件

基于 `t-io` 实现的**低延迟**、**高性能**的 `mqtt` 物联网组件。更多使用方式详见： **mica-mqtt-example** 模块。

- Gitee地址：https://gitee.com/596392912/mica-mqtt/
- [mica-mqtt-spring-boot-starter 使用文档](https://gitee.com/596392912/mica-mqtt/blob/master/mica-mqtt-spring-boot-starter/README.md)
- [mica-mqtt 使用文档](https://gitee.com/596392912/mica-mqtt/blob/master/mica-mqtt-core/README.md)
- [mica-mqtt http api 文档详见](https://gitee.com/596392912/mica-mqtt/blob/master/docs/http-api.md)
- [mica-mqtt 发行版本](https://gitee.com/596392912/mica-mqtt/blob/master/CHANGELOG.md)
- [t-io 官方文档](https://www.tiocloud.com/doc/tio/85)
- [mqtt 协议文档](https://github.com/mcxiaoke/mqtt)

# 二、功能

-  支持 MQTT v3.1、v3.1.1 以及 v5.0 协议。
-  支持 websocket mqtt 子协议（支持 mqtt.js）。
-  支持 http rest api，[http api 文档详见](https://gitee.com/596392912/mica-mqtt/blob/master/docs/http-api.md)。
-  支持 MQTT client 客户端。
-  支持 MQTT server 服务端。
-  支持 MQTT 遗嘱消息。
-  支持 MQTT 保留消息。
-  支持自定义消息（mq）处理转发实现集群。
-  MQTT 客户端 阿里云 mqtt 连接 demo。
-  支持 GraalVM 编译成本机可执行程序。
-  支持 Spring boot 项目快速接入（mica-mqtt-spring-boot-starter）。
-  mica-mqtt-spring-boot-starter 支持对接 Prometheus + Grafana。