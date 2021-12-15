- [InfluxDB+Grafana+Jmeter性能监控平台搭建](https://www.cnblogs.com/hong-fithing/p/14488406.html)



# InfluxDB+Grafana+Jmeter性能监控平台搭建

在做性能测试的时候，重点关注点是各项性能指标，用Jmeter工具，查看指标数据，就是借助于聚合报告，但查看时也并不方便。那如何能更直观的查看各项数据呢？可以通过`InfluxDB+Grafana+Jmeter`来实现数据的可视化。

讲到这里，可能会对 InfluxDB+Grafana 陌生些，没关系，后续会详细讲解。

我们先来看一张官网的配图，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305163352172-399941436.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305163352172-399941436.png)

如上所示，这就是接下来一系列教程的最终效果了，数据可视化，可添加各种指标数据，是不是很不错。觉得不错那就一起一探究竟吧。

注意：环境搭建基于`linux+docker`

在介绍这三个工具前，还是来简单说下，三个工具各自的作用：

- Jmeter：想必大家都很熟悉了，就不多说了，它在平台中扮演的角色是：采集数据
- InfluxDB：Go 语言开发的一个开源分布式时序数据库，非常适合存储指标、事件、分析等数据；它在平台中扮演的角色是：数据存储
- Grafana：纯 Javascript 开发的前端工具，用于访问 InfluxDB，自定义报表、显示图表等；它在平台中扮演的角色是：数据展示

看完简单介绍，是不是觉得比聚合报告强大多了。

官方网址：[InfluxDB网址](https://www.influxdata.com/)

官方文档：[InfluxDB文档](https://docs.influxdata.com/influxdb/)

- 特色功能

1）基于时间序列，支持与时间有关的相关函数（如最大，最小，求和等）；

2）可度量性：可以实时对大量数据进行计算；

3）基于事件：支持任意的事件数据。

- 主要特点

1）无结构（无模式）：可以是任意数量的列；

2）可拓展；

3）支持min, max, sum, count, mean, median 等一系列函数，方便统计；

4）原生的HTTP支持，内置HTTP API；

5）强大的类SQL语法；

6）自带管理界面，方便使用。

自从在开发同学建议使用docker后，觉得配置环境真的太方便了，今天的博文内容，环境部署都是使用docker，需要注意下，我使用的不是虚拟机，是去年在阿里囤的个服务器，比虚拟机方便太多。

使用命令 `docker pull influxdb:latest` 即可，简单。

使用命令 `docker run -itd -p 8086:8086 -p 2003:2003 -p 8083:8083 --name my_influxdb influxdb` 即可，容器名字可自定义。

说明：

- 8083端口：InfluxDB的UI界面展示的端口
- 8086端口：Grafana用来从数据库取数据的端口
- 2003端口：Jmeter往数据库发数据的端口

服务是docker启的，所以需要进入到容器内操作，进入容器命令如下所示：
 `docker exec -it influxdb_demo bash`

我们进入到目录：`cd /usr/bin`，先生成配置文件，使用命令`influxd config > /etc/influxdb/influxdb.conf` 即可生成。

容器内没有vim命令，可以将文件拷贝到宿主机上进行编辑，使用命令`docker cp -a 819e902a45f9:/etc/influxdb/influxdb.conf /home`，将配置文件拷贝到home路径下。

我们来修改配置，主要是配置存储jmeter数据的数据库与端口号，使用命令`vim influxdb.conf`,找到graphite并且修改它的库与端口，修改如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305181854958-2104962572.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305181854958-2104962572.png)

找到http，将前面的#号去掉，我在配置时，配置文件的http是默认启用的，所以我这里是没有配置的

网上有帖子说配置`admin`的，我在配置时，也没有这个配置项，应该是版本更新了的原因。

配置修改好后，还得将文件拷贝到容器中，我们可以先将配置文件删除，再拷贝。操作命令`docker cp -a  /home/influxdb.conf  819e902a45f9:/etc/influxdb/influxdb.conf ` 即可。

修改了配置文件，服务重启下，操作命令`docker restart my_influxdb`。

在修改配置文件时，我们配置了一个数据库，这个数据库默认是没有的，需要手动创建下，我们依然进入到目录：`cd /usr/bin`。

- 文件结构



```
influxd            # influxdb服务器
influx             # influxdb命令行客户端
influx_inspect     # 查看工具
influx_stress      # 压力测试工具
influx_tsm         # 数据库转换工具（将数据库从b1或bz1格式转换为tsm1格式）
```

- 启动客户端

使用命令`./influx` 启动客户端，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305182748460-2117118350.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305182748460-2117118350.png)

启动后，就可以输入对应的sql语句操作数据了。

创建数据库，使用命令`create database "jmeter"`，创建后，我们可以用命令`show databases`查看数据库，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305182925472-1687031898.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305182925472-1687031898.png)

创建人员，使用命令`create user "用户名" with password '密码' with all privileges`，创建好后，使用命令`show users`可以查看人员，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305183123869-1011683957.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305183123869-1011683957.png)

- 启动服务端

使用命令`./influxd` 启动服务端，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305202121537-1406599301.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305202121537-1406599301.png)

服务搭建好之后，可能有博友有疑问了，数据如何查询呢？的确，我们配置好后，怎么查看数据库中的数据。

远程连接并进入容器中，启动influx，查询语法跟平时用的sql差不多，直接上图来看一下：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305194549057-526556037.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305194549057-526556037.png)

这是我之前用jmeter跑过数据了，所以可以查询到数据。没跑过，那自然是没有数据的。今天暂时不讲jmeter的使用，先把InfluxDB分享完，目前知道如何查询数据即可。

我们在使用数据库，一般都会使用终端连接，比如：Navicat；我们用linux系统，也会选择用xshell或者SecureCRT连接一样，之前在github上找到个工具，[下载地址](https://github.com/CymaticLabs/InfluxDBStudio/releases/tag/v0.2.0-beta.1)

这个工具是windows上使用，下载下来，免安装，可以直接使用。

打开软件，新建连接，如下所示，界面很简洁：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305195624836-1664518855.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305195624836-1664518855.png)

我们点击新建按钮，就可以填写数据，创建连接了，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305195930006-8321711.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305195930006-8321711.png)

建立连接的字段说明：

- Name 名称 - 连接的名称。这是使用此连接时将看到的标签
- Address 地址 - InfluxDB服务器的IP。端口默认即可
- Database 数据库 - 用于连接的数据库。将其留空以列出所有数据库（需要管理员权限）
- UserName 用户名 - 用于连接的InfluxDB用户名
- Password 密码 - 与连接一起使用的InfluxDB密码
- Security - Use SSL 使用SSL - 连接到InfluxDB时是否使用SSL安全性（HTTPS）

填写好字段数据后，我们可以点Test，测试下连接是否正常，如下界面说明连接成功：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305200310638-1627676882.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305200310638-1627676882.png)

我们连接好后，也就可以通过终端工具来查询数据了，如下所示：
 [![img](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305200544652-1876285370.png)](https://img2020.cnblogs.com/blog/1242227/202103/1242227-20210305200544652-1876285370.png)

效果跟容器内查询是一样的，只是终端方式更加便捷。这里需要注意的是，由于我使用的是阿里服务器，所以端口还需要在安全组中开放端口，这里是需要注意下的。

完成上述配置后，InfluxDB+Grafana+Jmeter性能监控平台的第一步也就完成了，操作起来是不是挺简单的，不动手觉得难，操作起来实际简单。