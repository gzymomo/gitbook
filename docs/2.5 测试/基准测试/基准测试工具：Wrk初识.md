简单介绍下HTTP基准测试工具wrk的基本使用方法。。。

 

**一、压测工具对比**

| 工具名称  | 类型 | 多协议支持         | 优缺点                                                       | 适用人群&场景                             |
| --------- | ---- | ------------------ | ------------------------------------------------------------ | ----------------------------------------- |
| Apache ab | 开源 | HTTP               | Apache自带源生测试工具，安装部署简单，不适合多协议及复杂场景 | 开发：单机&单接口性能基准验证             |
| PTS       | 商业 | 多协议(支持不太好) | 阿里云配套收费压测工具，支持多协议链路压测，功能完善         | 技术人员：基准&链路&高并发                |
| Jmeter    | 开源 | 多协议             | 使用率高&学习成本低，多协议复杂场景支持良好，受限于机制，资源损耗较高 | 技术人员：多场景&万级以下并发全场景       |
| Locust    | 开源 | 多协议(需二次开发) | python开源压测框架，支持多协议&复杂场景(需二次开发，定制化)  | 技术人员：性能测试&支持程度取决于定制开发 |
| Wrk       | 开源 | HTTP               | HTTP基准测试工具，高并发低损耗，安装部署简单，不适合多协议及复杂场景 | 开发：单机&单接口性能基准验证             |

 

**二、简介及安装**

**1、简介**

Wrk是一个支持HTTP协议的基准测试工具，结合了多线程设计和可扩展事件通知，底层封装epoll(linux)和kqueue(bsd)，能用较少线程生成大量并发请求(使用了操作系统特定的高性能io机制)。

源生支持LuaJIT脚本，可以执行HTTP发起请求、响应处理和自定义测试报告；[SCRIPTING](https://github.com/wg/wrk/blob/master/SCRIPTING)有详细说明，并且[scripts](https://github.com/wg/wrk/tree/master/scripts)中提供了几个示例。

GitHub地址：[Wrk](https://github.com/wg/wrk)

**2、安装**

**Point**：wrk托管与github，前先安装Git；依赖gcc和OpenSSL（阿里云Centos服务默认已有）库，如下载报错，安装即可！命令如下：

```
# 下载命令
git clone https://github.com/wg/wrk.git
# 进入wrk文件夹
cd wrk
# 编译
make
```

编译需要一定时间，耐心等待即可。编译成功后，示例如下：

![img](https://img2018.cnblogs.com/blog/983980/201908/983980-20190824222002337-180048656.png)

 

**三、示例demo**

**1、参数说明**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
Usage: wrk <options> <url>                            
  Options:
# 脚本开启的HTTP连接数                                          
    -c, --connections <N>  Connections to keep open
# 测试脚本执行的时长   
    -d, --duration      <T>  Duration of test   
# 测试脚本使用的线程数        
    -t, --threads        <N>  Number of threads to use 
# 加载Lua脚本文件                          
    -s, --script           <S>  Load Lua script file    
# 添加请求的信息头   
    -H, --header        <H>  Add header to request    
# 打印响应的详细信息  
        --latency          Print latency statistics   
# 请求超时时间
        --timeout        <T>  Socket/request timeout
# 版本详细信息     
    -v, --version          Print version details 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 **2、示例脚本**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[root@localhost wrk]# ./wrk -t4 -c100 -d60s --latency http://www.cnblogs.com/imyalost
Running 1m test @ http://www.cnblogs.com/imyalost
  4 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   196.87ms  318.88ms   1.90s    86.33%
    Req/Sec   316.86    220.75     3.19k    75.19%
  Latency Distribution
     50%    5.73ms
     75%  259.62ms
     90%  615.77ms
     99%    1.47s 
  73434 requests in 1.00m, 11.06MB read
  Socket errors: connect 0, read 2, write 0, timeout 267
Requests/sec:   1222.07
Transfer/sec:    188.51KB
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

结果解析：

 4 threads and 100 connections ：4个线程，发起100个http连接请求；

 Thread Stats Avg Stdev Max +/- Stdev ：测试结果统计(精简版jmeter的聚合报告)，分别是：平均值、标准偏差、最大值、偏差比（值越高表示测试结果离散程度越高，性能波动较大）；

 Latency ：响应时间分布（即百分比响应时间范围）；

 Req/Sec ：每秒完成的请求数；

 Latency Distribution ：如上面的示例结果，分别代表50/75/90/99%的响应时间在多少ms以内；

 73434 requests in 1.00m, 11.06MB read ：本次测试共计在1min内发起73434个请求，总计读取11.06MB的数据；

 Socket errors: connect 0, read 2, write 0, timeout 267 ：本次测试中，连接失败0个，读取错误2个，超时267个；

 Requests/sec ：所有线程平均每秒钟完成1222.07个请求；

 Transfer/sec ：平均每秒读取188.51KB数据（吞吐量）；

**3、更多用法**

前文提到了wrk支持LuaJIT脚本，可以执行HTTP发起请求、响应处理和自定义测试报告，wrk提供的几个lua函数作用如下：

![img](https://img2018.cnblogs.com/blog/983980/201908/983980-20190824221046473-2002199710.png)