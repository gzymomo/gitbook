**一、关于easyNmon说明**

工具下载地址：[easyNmon](https://github.com/mzky/easyNmon)

说明：为了方便多场景批量监控，作者用golang写了个监控程序，可以通过web页面启动和停止nmon服务， 适配Loadrunner和jmeter进行性能测试，可以做到批量执行场景并生成监控报告！

环境适配：该执行文件默认为CentOS（6.5-7.4）版本，Ubuntu和SUSE需要下载对应版本的nmon替换！

go的http框架采用gin：https://gin-gonic.github.io/gin/[
](https://gin-gonic.github.io/gin/)

图表插件：echarts：http://echarts.baidu.com/

 

**二、下载安装**

**1、文件下载**

通过github下载该执行文件，然后上传到服务器，使用 tar -zxvf easyNmon.tar.gz 命令解压，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317124613795-1748644701.png)

解压后会生成一个easyNmon文件夹，进入该文件夹，通过命令 ./monitor& 启动easyNmon服务（后缀加&为后台运行）。

**2、常用信息查看**

在easyNmon目录下，输入 ./monitor -h 查看相关信息，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317125146774-1838206234.png)

**3、web页面**

可以通过帮助信息里面的信息，访问web页面查看该工具的页面管理功能，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317133252470-169645917.png)

PS：如果是云服务器，需要在云服务器控制台开启对应的安全组规则，否则无法访问！！！（上图是我的阿里云私有IP，访问的web地址需要换成公有IP地址）

**4、修改端口**

默认端口为9999，如果需要修改访问web页面的地址端口，需要自行修改，命令为 ./monitor -p 端口号 ，修改后查看帮助信息，如下图：

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317125909912-373943310.png)

 

**三、监控服务使用**

**1、集成jmeter启动**

安装好之后，在jmeter中添加线程组，然后按照如下格式填写对应的信息，添加仅一次控制器（因为后台服务启动后，只需要启动一次监控服务即可）

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317130321540-1520740239.png)

**2、web页面启动**

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317132247085-856872238.png)

接下来，就是启动压测脚本，进行压测并查看服务器监控报告。

 

**四、HTML格式监控报告**

**PS**：压测脚本结束后，默认生成监控报告，手动停止测试脚本，也会自动生成监控报告，可以通过访问web页面的报告页面查看，如下图：

**1、grafana测试结果**

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317133605037-613816055.png)

**2、easyNmon监控报告**

![img](https://img2018.cnblogs.com/blog/983980/201903/983980-20190317133729582-178093513.png)

 