Nginx不仅可以做Web服务器、做反向代理、负载均衡，还可以做限流系统。

此处我们就Nginx为例，介绍一下如何配置一个限流系统。

Nginx使用的限流算法是漏桶算法。

（1）找到Nginx所使用的配置文件所在的位置。在Ubuntu和Debian是在如下位置：

```bash
$ cd /etc/nginx/sites-available/
```

而CentOS则是在如下位置：

```bash
$ cd /etc/nginx/conf.d/
```

（2）在http块中，配置基础的限流配置：

```yaml
http
{     limit_req_zone$binary_remote_addr zone=mylimit:10m rate=10r/s;

     server {
         location /test/ {
             limit_reqzone=mylimit;

             proxy_passhttp://backend;
         }
     }
 }
```

其中4到8行定义的是一个服务器接口。而第2行和第6行配合完成了一个限流设置，下面解释一下这两行做的事情：

limit_req_zone命令在Nginx的配置文件中专门用于定义限流，它必须被放在http块中，否则无法生效，因为该命令只在http中被定义。

**该字段包含三个参数**：

第一个参数，就是键（key），即值$binary_remote_addr所在的位置，它代表的是我们的限流系统限制请求所用的键。

此处，我们使用了$binary_remote_addr，它是Nginx内置的一个值，代表的是客户端的IP地址的二进制表示。因此换言之，我们的示例配置，是希望限流系统以客户端的IP地址为键进行限流。

对Nginx有经验的读者可能还知道有一个Nginx内置值为binary_remote_addr是Nginx的社区推荐用值，因为它是二进制表达，占用的空间一般比字符串表达的$remote_addr要短一些，在寸土寸金的限流系统中尤为重要。

第二个参数是限流配置的共享内存占用（zone）。为了性能优势，Nginx将限流配置放在共享内存中，供所有Nginx的进程使用，因为它占用的是内存，所以我们希望开发者能够指定一个合理的、既不浪费又能存储足够信息的空间大小。根据实践经验，1MB的空间可以储存16000个IP地址。

该参数的语法是用冒号隔开的两个部分，第一部分是给该部分申请的内存一个名字，第二部分是我们希望申请的内存大小。

因此，在该声明中，我们声明了一个名叫mylimit（我的限制）的内存空间，然后它的大小是10M，即可以存储160000个IP地址，对于实验来说足够了。

第三个配置就是访问速率（rate）了，格式是用左斜杠隔开的请求数和时间单位。这里的访问速率就是最大速率，因此10r/s就是每秒10个请求。通过这台Nginx服务器访问后端服务器的请求速率无法超过每秒10个请求。

注意到第5行声明了一个资源位置/test/，因此我们第6行的配置就是针对这个资源的，通俗地说，我们在第6行的配置是针对特定API的，这个API就是路径为/test/的API，而其真正路径就是第8行声明的http://backend。注意，这个URL是不存在的，实际操作中，读者需要将它换成你已经开发好的业务逻辑所在的位置，Nginx在这里的作用只是一个反向代理，它自己本身没有资源。

第6行中，我们使用limit_req命令，声明该API需要一个限流配置，而该限流配置所在位置（zone）就是mylimit。

这样一来，所有发往该API的请求会先读到第6行的限流配置，然后根据该限流配置mylimit的名称找到声明在第2行的参数，然后决定该请求是否应该被拒绝。

但是这样还不够。不要忘了，Nginx使用的漏桶算法，不是时间窗口算法，我们前文介绍中说过，漏桶算法是有两个参数可以配置的！

（4）配置峰值。Nginx漏桶算法的峰值属性在API中设置。参数名为burst。如下：

```yaml
 http {
     limit_req_zone$binary_remote_addr zone=mylimit:10m rate=10r/s;

     server {
         location /test/ {
             limit_reqzone=mylimit burst=20;

             proxy_passhttp://backend;
         }
     }
 }
```

在第6行中，我们只需要在声明limit_req的同时，指定burst就可以了，此处我们指定burst为20，即漏桶算法中我们的“桶”最多可以接受20个请求。

这样一个Nginx的限流系统就配置完毕了，但实际操作中，我们还可能需要很多别的功能，下面笔者就介绍几个很有用的配置技巧。

# 1. 加快Nginx转发速度

相对于传统的漏桶算法慢吞吞地转发请求的缺陷，Nginx实现了一种漏桶算法的优化版，允许开发者指定快速转发，而且还不影响正常的限流功能。开发者只需要在指定limit_req的一行中指定burst之后指定另一个参数nodelay，就可以在请求总数没有超过burst指定值的情况下，迅速转发所有请求了。如下所示：

```yaml
 http {
     limit_req_zone$binary_remote_addr zone=mylimit:10m rate=10r/s;

     server {
         location /test/ {
             limit_reqzone=mylimit burst=20 nodelay;

             proxy_passhttp://backend;
         }
     }
 }
```

读者可能会担忧：这种情况下，会不会出现所有请求都被快速转发，然后接下来又有没有超过burst数量的请求出现，再次被快速转发，就好像固定窗口算法的漏洞一样，从而超过我们本来希望它能限制到的上限数量呢？答案是不会。Nginx的快速转发是这样实现的：

**·   当有没有超过burst上限的请求数量进入系统时，快速转发，然后将当前桶中可以填充的数量标为0；**

**·   按照我们设置的rate在1秒中内缓慢增加桶中余额，以我们的配置为例，每隔100毫秒，增加1个空位；**

**·   在增加空位的过程中，进来超过空位数量的请求，直接拒绝。**

举例而言，配置如上所示，假如在某个瞬时有25个请求进入系统，Nginx会先转发20个（或21个，取决于瞬时情况），然后拒绝剩下的4个请求，并将当前桶中数量标为0，然后接下来的每100毫秒，缓慢恢复1个空位。

这样我们可以看到，Nginx既做到了快速转发消息，又不会让后端服务器承担过多的流量。

# 2. 为限流系统配置日志级别

限流系统会提前拒绝请求，因此，我们在业务服务器上是肯定看不到这些请求的。假如我们收到一个报告说某用户在使用网站的时候出现错误，但是我们在业务服务器上又找不到相关的日志，我们如何确定是不是限流造成的呢？

只有限流系统的日志才能说明问题。因此，我们需要Nginx打印出它拒绝掉的请求的信息。但同时，Nginx打印的限流日志默认是错误（error），如果我们设置了一个基于日志错误扫描的警报，它扫到的限流错误，真的是我们希望给自己发警报的情况吗？

配置请求的位置就在资源中，使用的命令是limit_req_log_level，如下：

```
 http {
     limit_req_zone$binary_remote_addr zone=mylimit:10m rate=10r/s;

     server {
         location /test/ {
             limit_reqzone=mylimit burst=20;
            limit_req_log_level warn;

             proxy_passhttp://backend;
         }
     }
 }
```

在第7行中，我们将Nginx的日志改为了警告（warn）。

# 3. 更换Nginx的限流响应状态码

前文我们就说过，从语义上来说，限流的HTTP标准响应状态码是429，但是如果读者拿上述的配置文件直接去测试，会发现Nginx返回的是503（服务不可用）。到底**应该**返回什么状态码，是一个偏程序哲学的问题，此处我们不讨论，我们只讨论：如何让Nginx返回我们指定的状态码？

答案也是在同一个资源中，它的配置命令是limit_req_status，然后我们指定它为429即可：

```
 http {
     limit_req_zone$binary_remote_addr zone=mylimit:10m rate=10r/s;

     server {
         location /test/ {
             limit_reqzone=mylimit burst=20;
            limit_req_log_level warn;
            limit_req_status 429;

             proxy_passhttp://backend;
         }
     }
 }
```

