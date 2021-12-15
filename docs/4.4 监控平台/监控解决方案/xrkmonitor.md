###  项目简介

集监控点监控、日志监控、数据可视化及监控告警为一体的分布式开源监控系统。
通过插件方式支持常用监控需求，插件可自由选择且支持一键部署、移除、启用、禁用等操作。
提供丰富的图表和多种数据类型，满足对数据可视化的需要。
在线演示版地址：[http://open.xrkmonitor.com](http://open.xrkmonitor.com/)

**相比其它开源监控系统优势**

1. 支持插件功能, 监控插件无需开发，自由选择监控插件，控制台安装即可使用
2. 集成告警功能, 支持多种告警方式
3. 集成分布式日志系统功能
4. 支持多种部署方式
   a.集中部署（全部服务部署在一台机器，适合个人或者小团队开发者）
   b.分布式部署（分布式部署在多台机器，适合小中型企业大规模监控需求）
5. 支持自动化配置（机器部署agent后自动注册到监控系统无需在控制台配置、视图根据上报自动绑定相关上报机器）
6. 支持多用户访问（子账号由管理员账号在控制台添加）
7. 上报接口支持主流开发语言，数据上报api 提供类似公共库接口的便捷

### 特色功能推荐

1. IP地址库: 支持通过IP地址上报时将IP地址转为物理地址，相同物理地址归并展示，一个监控API 即可轻松生成监控
   数据的物理地址分布图，插件示例：[monitor_apache_log](https://gitee.com/xrkmonitorcom/monitor_apache_log)
   ![img](http://xrkmonitor.com/monitor/images/china_map.png)

   

2. 监控插件市场: 让监控成为可以复用的组件，无需开发一键安装即可使用，更多监控插件持续开发中
   ![img](http://xrkmonitor.com/monitor/images/plugin_show.png)

   

3. 分布式日志系统: 支持大规模系统日志上报，日志上报支持频率限制、日志染色、自定义字段等高级功能，控制台
   日志查看支持按关键字、排除关键字、上报时间、上报机器等方式过滤日志，从茫茫日志中轻松找到您需要的日志
   ![img](http://xrkmonitor.com/monitor/images/web_log.gif)

4. 视图机制: 监控图表支持视图定制模式，视图可按上报服务器、监控点随意组合，轻松定制您需要的监控视图，并
   可在监控图表上直接设置告警值
   ![img](http://xrkmonitor.com/monitor/images/web_attr_git.gif)

5. 告警集成: 集成告警功能, 支持邮件、短信、微信、PC客户端等告警方式，告警功能无需开发直接可用
   ![img](http://xrkmonitor.com/monitor/images/open_warn_git.png)

### docker 方式部署

以下假设宿主机的IP 地址为：192.168.128.210, 以v2.8_2020-10-16 版本作为示例说明 docker 部署方法

1. 拉取 docker 镜像: docker pull registry.cn-hangzhou.aliyuncs.com/xrkmonitor/release:v2.8_2020-10-16
2. 通过镜像启动容器：docker images 查看镜像ID，假设为：93297f01d06b
   启动容器：
   docker run -idt -p27000:27000/udp -p38080:38080/udp -p28080:28080/udp -p80:80 --env xrk_host_ip=192.168.128.210 --env xrk_http_port=8080 -v /data/xrkmonitor/docker_mysql:/var/lib/mysql -v /data/xrkmonitor/docker_slog:/home/mtreport/slog 93297f01d06b
   参数说明：
   -p27000:27000/udp -p38080:38080/udp -p28080:28080/udp -p80:80 日志、监控点、接入服务等的端口映射
   --env xrk_host_ip=192.168.128.210 宿主机的ip地址, 根据实际情况传递
   --env xrk_http_port=80 web 控制台的映射端口（可以使用非80端口映射容器80端口，相关问题可以查看在线文档的 docker 部署部分）
   -v /data/xrkmonitor/docker_mysql:/var/lib/mysql, 挂载宿主机目录
   -v /data/xrkmonitor/docker_slog:/home/mtreport/slog，挂载日志目录
3. docker ps -a 查看运行的容器，docker attach 进入容器，进入目录 /home/mtreport, 执行 ./start_docker.sh 启动监控系统服务
4. 在浏览器端使用宿主机IP即可访问控制台： [http://192.168.128.210，控制台默认账号密码为：sadmin/sadmin](http://192.168.128.xn--210%2C:sadmin-k68ql82b20f9cw63mu81att0f47xdukg7r4h/sadmin)
5. 如需停止服务可执行 : /home/mtreport/stop_docker.sh 脚本

**agent 部署说明：**
容器中 /home/mtreport/slog_mtreport_client.tar.gz 为 agent 部署文件，可以将其拷贝到需要部署的机器上
SERVER_MASTER 配置改为宿主机的IP地址， 如果需要在宿主机上部署 agent，需要指定：AGENT_CLIENT_IP 不能与
容器启动时传递的xrk_host_ip 相同(xrk_host_ip 已被docker 容器使用了)，IP 可以不存在，主要用于机器识别。
关于 agent 的详细说明可以参考文档。

### 使用 docker 镜像编译源码

使用 docker 镜像编译源码方法如下：

1. 拉取镜像： docker pull registry.cn-hangzhou.aliyuncs.com/xrkmonitor/compile:last
2. 执行镜像容器：docker run -idt [镜像id] (docker images 获取)
3. 进入容器：docker attach [容器id] (docker ps -a 获取)
4. 进入容器目录：/home/xrkmonitor/open, 执行 make 即可编译项目全部源码
5. 编译成功后生成集中部署包，进入 tools_sh 目录，执行：./make_all.sh 即可生成完整的集中部署包

关于集中部署请参考文档：[源码编译-集中部署](http://xrkmonitor.com/monitor/showdoc/showdoc/web/#/4?page_id=38)

docker 编译镜像安装了 vim/gcc/git 等工具，如需更新源码，进入目录:/home/xrkmonitor/open 执行 git pull 即可
如您想使用镜像进行二次开发，建议您挂载宿主机目录，将源码下载到挂载的目录进行修改， 不要使用容器中的
/home/xrkmonitor/open 目录，以免工作成果丢失。

### 在线部署

安装脚本: install.sh
从以下链接下载后, 按提示执行即可, 需要系统支持 bash
(wget http://xrkmonitor.com/monitor/download/install.sh; chmod +x install.sh; ./install.sh )
示例（如您安装失败可在评论区留言系统版本或者加入Q群咨询）：![在线安装示例](https://images.gitee.com/uploads/images/2020/1016/154842_d1f6dcda_5075697.gif)

在线部署说明:
安装脚本会先检查当前系统是否支持在线安装, 如不支持您可以下载源码后在系统上编译安装
在线部署目前只支持集中部署方式, 即所有服务部署在一台机器上, 该机器上需要安装 mysql/apache
安装脚本使用中文 utf8 编码, 安装过程请将您的终端设置为 utf8, 以免出现乱码
安装脚本同时支持 root 账号和普通账号操作, 使用普通账号执行安装部署要求如下:

1. 在线部署使用动态链接库, 需要在指定目录下执行安装脚本, 目录为: /home/mtreport
2. 普通账号某些目录可能无权操作, 需要授权才能正常安装

卸载脚本: uninstall_xrkmonitor.sh
在线部署过程中会下载该脚本, 如需卸载可执行该脚本
在线部署详细说明文档：[在线部署](http://xrkmonitor.com/monitor/showdoc/showdoc/web/#/4?page_id=55)

我们强烈建议您先在本地虚拟机上执行在线安装, 熟悉安装流程后在实际部署到您的服务器上。

### 离线部署(自行编译源码)

如果在线安装失败或者需要二次开发, 可以使用源码编译方式安装

操作步骤：

1. git clone https://gitee.com/xrkmonitorcom/open.git 下载源码
2. 进入源码目录，执行 make 完成源码编译
3. 进入 tools_sh 目录，执行 make_all.sh 生成部署包
4. 在安装目录解压部署包，执行 local_install.sh 完成安装

(如遇编译环境问题，可以尝试使用docker 镜像编译)
监控系统卸载脚本: uninstall_xrkmonitor.sh，移除彻底不留丝毫痕迹

安装环境变量同在线安装一样, 具体可以查看说明文档: [源码编译-集中部署](http://xrkmonitor.com/monitor/showdoc/showdoc/web/#/4?page_id=38)
控制台默认账号密码: sadmin/sadmin

### 使用的技术方案

1. apache + mysql(监控点数据、配置信息使用 mysql 存储, 支持分布式部署)
2. 前端 web 控制台采用 [dwz 开源框架](http://jui.org/)
3. 前端监控图表采用开源 [echarts](https://www.echartsjs.com/zh/index.html) 绘制
4. 后台 cgi 使用开源的cgi模板引擎 - [clearsilver](http://www.clearsilver.net/), 所有cgi支持以fastcgi方式部署
5. 后台服务使用了开源的 socket 开发框架 - [C++ Sockets](http://www.alhem.net/Sockets/)

### 当前监控上报API支持的语言如下(更多语言支持在开发中)

1. [c/c++ 开发接口](http://xrkmonitor.com//monitor/showdoc/showdoc/web/#/4?page_id=45)
2. [php 开发接口](http://xrkmonitor.com//monitor/showdoc/showdoc/web/#/4?page_id=51)
3. [linux shell 开发接口](http://xrkmonitor.com//monitor/showdoc/showdoc/web/#/4?page_id=72)
4. [javascript 开发接口](http://xrkmonitor.com//monitor/showdoc/showdoc/web/#/4?page_id=76)

### 插件市场

1. [linux_base](https://gitee.com/xrkmonitorcom/plugin_linux_base) - c/c++语言开发，用于监控linux 系统 cpu/内存/磁盘/网络等资源
2. [monitor_apache_log](https://gitee.com/xrkmonitorcom/monitor_apache_log) - c/c++语言开发，用于监控apache 网站的流量访问量等
3. [linux_file_monitor](https://gitee.com/xrkmonitorcom/linux_file_monitor) - shell 语言开发，用于监控 linux系统文件目录的增删改变动
4. [monitor_website](https://gitee.com/xrkmonitorcom/monitor_website) - javascript 语言开发，用于监控网站访客基本信息和运行异常信息

**项目演示链接：[字符云监控项目演示 http://open.xrkmonitor.com](http://open.xrkmonitor.com/)**

**在线文档：- [在线文档 http://xrkmonitor.com/monitor/dmt_open_doc.html](http://xrkmonitor.com/monitor/dmt_open_doc.html)**