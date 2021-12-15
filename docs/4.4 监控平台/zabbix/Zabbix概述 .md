[TOC]

Zabbix 是由 Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。可用于监视各种网络服务、服务器和网络机器等状态。

zabbix 是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。zabbix 能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。

Zabbix 作为企业级分布式监控系统，具有很多优点，如：分布式监控，支持 node 和 proxy 分布式模式；自动化注册，根据规则，自动注册主机到监控平台，自动添加监控模板；支持 agentd、snmp、ipmi 和 jmx 等很多通信方式。

zabbix 分三个部分，agent 和 server 、web 三部分

zabbix-agent 部署在被监控机上面,zabbix-server (建议部署在另外一台主机上),zabbix-agent 会发送数据到zabbix-server 或者zabbix-server 主动索取数据，zabbix-server 将获取的数据存在mysql 数据库中（或者其他的数据库）. (zabbix.com 官网的manual 上面有安装配置) 。web 从server上获取数据，然后展示给用户。


# 一、Zabbix组件
## 1.1 Zabbix Server
负责接收 agent 发送的报告信息的核心组件，所有配置，统计数据及操作数据均由其组织进行.

## 1.2 Database Storage
专用于存储所有配置信息，以及由zabbix收集的数据。

## 1.3 Web interface
Zabbix的GUI接口，通常与Server运行在同一台主机上。

## 1.4 Proxy
可选组件，常用于分布式监控环境中，代理Server收集部分监控端的监控数据，并统一发往Server端。

## 1.5 Agent
部署在被监控主机上，负责收集本地数据并发往Server端或Proxy端。

# 二、进程
默认情况下，Zabbix包含5个程序：zabbix_agentd、zabbix_get、zabbix_proxy、zabbix_sender、zabbix_server，zabbix_java_gateway可寻，需另外安装。

# 三、组件的功能及作用
## 3.1 zabbix_agentd
客户端守护进程，此进程收集客户端数据，例如 cpu 负载、内存、硬盘使用情况等。

## 3.2 zabbix_get
zabbix 工具，单独使用的命令，通常在 server 或者proxy端执行获取远程客户端信息的命令。 通常用户排错。 例如在server端获取不到客户端的内存数据， 我们可以使用zabbix_get获取客户端的内容的方式来做故障排查。

## 3.3 zabbix_sender
zabbix 工具，用于发送数据给 server 或者proxy，通常用于耗时比较长的检查。很多检查非常耗时间，导致 zabbix 超时。于是我们在脚本执行完毕之后，使用 sender 主动提交数据。

## 3.4 zabbix_server
zabbix 服务端守护进程。zabbix_agentd、zabbix_get、zabbix_sender、zabbix_proxy、zabbix_java_gateway 的数据最终都是提交到 server。
备注：当然不是数据都是主动提交给 zabbix_server,也有的是 server 主动去取数据。

## 3.5 zabbix_proxy
zabbix 代理守护进程。功能类似server，唯一不同的是它只是一个中转站，它需要把收集到的数据提交/被提交到 server 里。

## 3.6 zabbix_java_gateway
zabbix2.0 之后引入的一个功能。顾名思义：Java 网关，类似 agentd，但是只用于 Java方面。需要特别注意的是，它只能主动去获取数据，而不能被动获取数据。 它的数据最终会给到server或者proxy。

# 四、Zabbix监控环境中的术语
 - 主机（host）：要监控的网络设备，可由IP或DNS名称指定。
 - 主机组（host group）：主机的逻辑容器，可以包含主机和模板，但同一个组织内的主机和模板不能互相链接；主机组通常在给用户或用户组指派监控权限时使用；
 - 监控项（item）：一个特定监控指标的相关的数据，这些数据来自于被监控对象；item是zabbix进行数据收集的核心，相对某个监控对象，每个item都由"key"标识。
 - 触发器（trigger）：一个表达式，用于评估某监控对象的特定item内接收到的数据是否在合理范围内，也就是阈值；接收的数据量大于阈值时，触发器状态将从“OK”转变为“problem”，当数据再次恢复到合理范围，又抓变为“OK”。
 - 事件（event）：触发一个值得关注的事情，比如触发器状态转变，新的agent或重新上线的agent的自动注册等。
 - 动作（action）：指对于特定事件先定义的处理方法，如发送通知，何时执行操作。
 - 报警媒介类型（media）：发送通知的手段或通道，如Email、SMS、Jabber等。
 - 模板（template）：用于快速定义被监控主机的预设条目集合，通常包含了item、trigger、graph、screen、application以及low-level discovery rule；模板可以直接连接至某个主机；
 - 前端（frontend）：Zabbix的Web端口。

# 五、Zabbix监听端口（socker进程）
 - zabbix_server---->监控10051
 - zabbix_agentd---->监听10050
