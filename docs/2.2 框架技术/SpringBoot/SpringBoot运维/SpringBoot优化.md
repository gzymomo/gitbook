- [Spring Boot 这样优化，让你的项目飞起来！](https://mp.weixin.qq.com/s/Kb8MdRXiuXfPIKtI7rq8nw)

# 1 介绍

在SpringBoot的Web项目中，默认采用的是内置Tomcat，当然也可以配置支持内置的jetty，内置有什么好处呢？

- 方便微服务部署。
- 方便项目启动，不需要下载Tomcat或者Jetty

针对目前的容器优化，目前来说没有太多地方，需要考虑如下几个点

- 线程数
- 超时时间
- jvm优化

针对上述的优化点来说，首先线程数是一个重点，初始线程数和最大线程数，初始线程数保障启动的时候，如果有大量用户访问，能够很稳定的接受请求。

而最大线程数量用来保证系统的稳定性，而超时时间用来保障连接数不容易被压垮，如果大批量的请求过来，延迟比较高，不容易把线程打满。这种情况在生产中是比较常见的 ，一旦网络不稳定，宁愿丢包也不愿意把机器压垮。

jvm优化一般来说没有太多场景，无非就是加大初始的堆，和最大限制堆,当然也不是无限增大，根据的情况进快速开始

在spring boot配置文件中application.yml，添加以下配置

```yaml
server:
  tomcat:
    min-spare-threads: 20
    max-threads: 100
  connection-timeout: 5000
```

这块对tomcat进行了一个优化配置，最大线程数是100，初始化线程是20,超时时间是5000ms

#  2 Jvm优化

 这块主要不是谈如何优化，jvm优化是一个需要场景化的，没有什么太多特定参数，一般来说在server端运行都会指定如下参数

初始内存和最大内存基本会设置成一样的，具体大小根据场景设置，-server是一个必须要用的参数，至于收集器这些使用默认的就可以了，除非有特定需求。

## 2.1 使用-server模式

设置JVM使用server模式。64位JDK默认启动该模式

```
java -server -jar springboot-1.0.jar
```

## 2.2 指定堆参数

这个根据服务器的内存大小，来设置堆参数。

- -Xms :设置Java堆栈的初始化大小
- -Xmx :设置最大的java堆大小

```
java -server -Xms512m -Xmx768m  -jar springboot-1.0.jar
```

设置初始化堆内存为512MB，最大为768MB。

## 2.3 远程Debug

在服务器上将启动参数修改为：

```
java -Djavax.net.debug=
ssl -Xdebug -Xnoagent -Djava.compiler=
NONE -Xrunjdwp:transport=
dt_socket,server=y,suspend=
n,address=8888 -jar springboot-1.0.jar
```

这个时候服务端远程Debug模式开启，端口号为8888。

在IDEA中，点击Edit Configuration按钮。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZLoKxNnrOh66ofSBpwNudlRbXP18iaqXSL1kdFmafB8IOYjjPl9n2qUQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

出现弹窗，点击+按钮，找到Remote选项。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZrY3fQ5iaTY0EcLVqdJrdcZzCKZTA5xFBUDmicqVSOo9DCeMKPSfItnkQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



在【1】中填入Remote项目名称，在【2】中填IP地址和端口号，在【3】选择远程调试的项目module，配置完成后点击OK即可

如果碰到连接超时的情况，很有可能服务器的防火墙的问题，举例CentOs7,关闭防火墙

```bash
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```

点击debug按钮，IDEA控制台打印信息：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZ7WiaNgQwWAWiavStgc2faSGa1KNPnBuKLia98GEYgGCLOxNkT1WnIKGKQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



说明远程调试成功。

# 3 JVM工具远程连接

**jconsole与Jvisualvm远程连接**

通常我们的web服务都输部署在服务器上的，在window使用jconsole是很方便的，相对于Linux就有一些麻烦了，需要进行一些设置。

## 3.1 查看hostname,首先使用

```
hostname -i
```

查看，服务器的hostname为127.0.0.1，这个是不对的，需要进行修改

## 3.2 修改hostname

修改/etc/hosts文件，将其第一行的“127.0.0.1 localhost.localdomain localhost”，修改为：“192.168.44.128  localhost.localdomain localhost”.“192.168.44.128”为实际的服务器的IP地

## 3.3 重启Linux，在服务器上输入hostname -i，查看实际设置的IP地址是否为你设置的

## 3.4 启动服务，参数为：

```
java -jar -Djava.rmi.server.hostname=192.168.44.128 -
Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=911 -
Dcom.sun.management.jmxremote.ssl=false -
Dcom.sun.management.jmxremote.authenticate=false jantent-1.0-SNAPSHOT.jar
```

ip为192.168.44.128，端口为911 。

## 3.5 打开Jconsole，进行远程连接,输入IP和端口即可

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZxMO5I6a0D1G5rWfWaQIJMicpAyQgsqmUnHadqadADGeytm6BfvSIEeQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



点击连接，经过稍稍等待之后，即可完成连接，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZrshHpoKm7Rg9ynPrnDia6iahARicQQgbVYDUnz0wzJicJN9ibTA9HMzGp4Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同理，JvisualVm的远程连接是同样的，启动参数也是一样。

然后在本机JvisualVm输入IP：PORT，即可进行远程连接：如下图所示：



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/YscRq4LT5WuOydZGRPn9Ct3xBmJvIFlZ1Q82IM8OecfQdxib7UgX0lD5LwWMQXc144gHKLtv2BdiajNRbias7wkXw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

相比较Jvisualvm功能更加强大一下，界面也更美观~

