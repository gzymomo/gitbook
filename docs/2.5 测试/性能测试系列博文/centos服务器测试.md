[Linux服务器基准测试方案](https://www.cnblogs.com/llwxhn/p/12531458.html)

测试服务器的基准性能，包括CPU、内存、磁盘性能。

## **硬件配置**

| 服务器   | CPU     | 内存 | 磁盘 | 网络（基准/最大） | 数量 |
| -------- | ------- | ---- | ---- | ----------------- | ---- |
| ecs-cts3 | 2 Cores | 4GB  | 40G  | 1.2/4 Gbit/s      | 1    |

## CPU、内存测试工具sysbench

 sysbench是一款开源的多[线程](https://baike.baidu.com/item/线程)[性能测试](https://baike.baidu.com/item/性能测试/1916148)工具，可以执行[CPU](https://baike.baidu.com/item/CPU)/内存/线程/[IO](https://baike.baidu.com/item/IO)/数据库等方面的性能测试。

 \#yum -y install sysbench

sysbench命令的基本参数：

\#sysbench --help

表2-1

| 测试项[testname] | 备注               |
| ---------------- | ------------------ |
| fileio           | 文件I/O测试        |
| cpu              | cpu性能测试        |
| memory           | 内存速度测试       |
| threads          | 线程子系统性能测试 |
| mutex            | 互斥性能测试       |

表2-2

| 选项[options]                   | 备注                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| --threads=N                     | 需要使用的线程数，默认为1                                    |
| --events=N                      | 事件总数限制，默认为0，无限制                                |
| --time=N                        | 限制的总执行时间，默认为10s                                  |
| --forced-shutdown=STRING        | 执行时间结束后等待几秒然后强制关闭，默认为off，禁用          |
| --thread-stack-size=SIZE        | 每个线程的堆栈大小，默认为64K                                |
| --rate=N                        | 平均处理速率，默认为0，无限制                                |
| --report-interval=N             | 指定间隔（秒）报告统计信息，默认为0，禁用                    |
| --report-checkpoints=[LIST,...] | 转存完整统计信息并在指定时间点重置所有计数器。参数是一个逗号分隔的值列表，表示必须执行报表检查点时，从测试开始算起经过的时间（以秒为单位）。默认情况下，报表检查点处于关闭状态。 |
| --debug[=on\|off]               | 输出更多调试信息，默认关闭                                   |
| --validate[=on\|off]            | 尽可能执行验证检查，默认关闭                                 |
| --help[=on\|off]                | 输出帮助信息并退出，默认关闭                                 |
| --version[=on\|off]             | 输出版本信息并退出，默认关闭                                 |
| --config-file=FILENAME          | 包含命令行选项的文件                                         |

表2-3

| 命令[command] | 备注 |
| ------------- | ---- |
| prepare       | 准备 |
| run           | 运行 |
| cleanup       | 清理 |
| help          | 帮助 |

sysbenh测试工具命令

根据测试需要调整参数

sysbench [testname] [options].. [command]

## **磁盘测试工具fio**

FIO是测试IOPS的非常好的工具，用来对硬件进行压力测试和验证。

安装fio测试工具

\#yum -y install fio

fio命令常用参数：                                   表3-1

| **参数**         | **说明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| -direct          | 定义是否使用direct IO，可选值如下：                          |
|                  | 值为0，表示使用buffered IO                                   |
|                  | 值为1，表示使用direct IO                                     |
| -iodepth         | 定义测试时的IO队列深度，默认为1。                            |
|                  | 此处定义的队列深度是指每个线程的队列深度，如果有多个线程测试，意味着每个线程都是此处定义的队列深度。fio总的IO并发数=iodepth * numjobs。 |
| -rw              | 定义测试时的读写策略，可选值如下：                           |
|                  | 随机读：randread                                             |
|                  | 随机写：randwrite                                            |
|                  | 顺序读：read                                                 |
|                  | 顺序写：write                                                |
|                  | 混合随机读写：randrw0                                        |
| -ioengine        | 定义fio如何下发IO请求，通常有同步IO和异步IO：                |
|                  | 同步IO一次只能发出一个IO请求，等待内核完成后才返回。这样对于单个线程IO队列深度总是小于1，但是可以透过多个线程并发执行来解决。通常会用16~32个线程同时工作把IO队列深度塞满。 |
|                  | 异步IO则通常使用libaio这样的方式一次提交一批IO 请求，然后等待一批的完成，减少交互的次数，会更有效率。 |
| -bs              | 定义IO的块大小(block size)，单位是k、K、m和M等，默认IO块大小为4 KB。 |
| -size            | 定义测试IO操作的数据量，若未指定runtime这类参数，fio会将指定大小的数据量全部读/写完成，然后才停止测试。 |
|                  | 该参数的值，可以是带单位的数字，比如size=10G，表示读/写的数据量为10GB；也可是百分数，比如size=20%，表示读/写的数据量占该设备总文件的20%的空间。 |
| -numjobs         | 定义测试的并发线程数。                                       |
| -runtime         | 定义测试时间。                                               |
|                  | 如果未配置，则持续将size指定的文件大小，以每次bs值为分块大小读/写完。 |
| -group_reporting | 定义测试结果显示模式，group_reporting 表示汇总每个进程的统计信息，而非以不同job汇总展示信息。 |
| -filename        | 定义测试文件（设备）的名称。                                 |
|                  | 此处选择文件，则代表测试文件系统的性能。例如：**-**filename=/opt/fiotest/fiotest.txt |
|                  | 此处选择设备名称，则代表测试裸盘的性能。例：**-**filename=/dev/vdb1 |
|                  | **注意：**                                                   |
|                  | 如果在已经分区、并创建文件系统，且已写入数据的磁盘上进行性能测试，请注意filename选择指定文件，以避免覆盖文件系统和原有数据。 |
| -name            | 定义测试任务名称。                                           |

## **网络测试工具iperf**

iperf 是一个网络性能测试工具。Iperf可以测试最大TCP和UDP带宽性能，具有多种参数和UDP特性，可以报告带宽、延迟抖动和数据包丢失。

iperf测试需要两台服务器，两个系统，因为一个系统必须充当服务端，另外一个系统充当客户端，客户端连接到需要测试速度的服务端。

安装iperf测试工具

\#yum -y install iperf

iperf命令的基本参数：

表3-2

| 命令行选项                         | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| 客户端与服务器共用选项             |                                                              |
| -b, --bandwidth [kmgKMG \| pps]    | 发送的带宽,以bits/sec或pps为单位                             |
| -e, --enhancedreports              | 使用增强的报告提供更多的tcp/udp和流量信息                    |
| -f, --format [kmgKMG]              | 报告采用的单位：Kbits、Mbits、KBytes、Mbytes                 |
| -i, --interval                     | 带宽报告产生的周期，单位为秒                                 |
| -l, --len [kmKM]                   | 要读或写的缓冲区长度（字节）（默认值：TCP=128K，v4-UDP=1470，v6-UDP=1450） |
| -m, --print_mss                    | 打印TCP最大段大小（MTU-TCP/IP报头）                          |
| -o, --output <filename>            | 将报告或错误消息输出到此指定文件                             |
| -p, --port                         | 要侦听/连接到的服务器端口                                    |
| -u, --udp                          | 使用UDP而不是TCP                                             |
| -w, --window [KM]                  | TCP窗口大小（套接字缓冲区大小）                              |
| -z, --realtime                     | 实时调度程序                                                 |
| -B, --bind <host>[:<port>][%<dev>] | 监听的主机，IP（包括多播地址）和可选端口和设备               |
| -C, --compatibility                | 兼容旧版本，不发送额外的msg                                  |
| -M, --mss                          | 设置TCP最大段大小（MTU-40字节）                              |
| -N, --nodelay                      | 设置TCP无延迟，禁用Nagle算法                                 |
| -S, --tos                          | 设置套接字的IP（字节）字段                                   |
| 服务器端专用选项                   |                                                              |
| -s, --server                       | 以服务器模式运行                                             |
| -t, --time                         | 列出新连接和接收流量的时间（秒）（默认值未设置）             |
| -B, --bind <ip>[%<dev>]            | 绑定到多播地址和可选设备                                     |
| -H, --ssm-host <ip>                | 设置ssm源，与-B一起用于（S，G）                              |
| -U, --single_udp                   | 以单线程udp模式运行                                          |
| -D, --daemon                       | 作为后台程序运行服务器                                       |
| -V, --ipv6_domain                  | 通过将域和套接字设置为AF/INET6来启用ipv6接收（可以在IPv4和ipv6上接收） |
| 客户端专用选项                     |                                                              |
| -c, --client <host>                | 以客户端模式运行，连接到<host>                               |
| -d, --dualtest                     | 同时进行双向测试                                             |
| -n, --num                          | 要传输的字节数                                               |
| -r, --tradeoff                     | 单独进行双向测试                                             |
| -t, --time                         | 传输时间（秒）（默认为10秒）                                 |
| -B, --bind [<ip> \| <ip:port>]     | 绑定从其到源流量的ip（和可选端口）                           |
| -F, --fileinput <name>             | 输入要从文件传输的数据                                       |
| -I, --stdin                        | 输入从stdin传输的数据                                        |
| -L, --listenport                   | 要接收双向测试的端口                                         |
| -P, --parallel                     | 要运行的并行客户端线程数                                     |
| -R, --reverse                      | 反转测试（客户端接收，服务器发送                             |
| -T, --ttl                          | 生存时间，用于多播（默认值1）                                |
| -V, --ipv6_domain                  | 将域设置为ipv6（通过ipv6发送数据包）                         |
| -X, --peer-detect                  | 执行服务器版本检测和版本交换                                 |
| -Z, --linux-congestion <algo>      | 设置TCP拥塞控制算法（仅限linux）                             |
| 其它                               |                                                              |
| -x, --reportexclude [CDMSV]        | 排除C（连接）D（数据）M（多播）S（设置）V（服务器）报告      |
| -y, --reportstyle C                | 报告以逗号分隔值                                             |
| -h, --help                         | 打印此消息并退出                                             |
| -v, --version                      | 打印版本信息并退出                                           |

iperf测试工具命令

根据测试需要调整参数

iperf [-s|-c host] [options]

## **测试CPU性能**

使用命令sysbench cpu [options] run

查看可用[options]参数  sysbench cpu help

| 参数              | 备注                          |
| ----------------- | ----------------------------- |
| --cpu-max-prime=N | 产生质数的上限，默认值为10000 |

## **测试内存性能**

使用命令sysbench memory [options] run

查看可用[options]参数 sysbench memory help

表4-2

| 参数                        | 备注                                           |
| --------------------------- | ---------------------------------------------- |
| --memory-block-size=SIZE    | 测试内存块大小，默认为1k                       |
| --memory-total-size=SIZE    | 要传输的数据的总大小，默认为100G               |
| --memory-scope=STRING       | 内存访问作用域{global,local}，默认为global     |
| --memory-hugetlb[=on\|off]  | 从HugeTLB池分配内存[on\|off]，默认为off        |
| --memory-oper=STRING        | 内存操作的类型{read, write, none}，默认为write |
| --memory-access-mode=STRING | 内存访问模式{seq，rnd}，默认为seq              |

## **磁盘测试** 

测试随机写IOPS：

\#fio -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=10G -numjobs=1 -runtime=600

 -group_reporting -filename=/dev/vdb -name=Rand_Write_IOPS_Test

 

测试随机读IOPS：

\#fio -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=10G -numjobs=1 -runtime=600

-group_reporting -filename=/dev/vdb -name=Rand_Read_IOPS_Test

 

测试写吞吐量：

\#fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1M -size=10G -numjobs=1 -runtime=600

-group_reporting -filename=/dev/vdb -name=Write_BandWidth_Test

 

测试读吞吐量：

\#fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1M -size=10G -numjobs=1 -runtime=600

-group_reporting -filename=/dev/vdb -name=Read_BandWidth_Test

## **网络测试**

 tcp模式下测试带宽

服务端运行iperf -s

客户端运行iperf -c <服务端ip>

 

udp模式下测试带宽，延时，丢包率

服务端运行iperf -s –u

客户端运行iperf -c <服务端ip> -b 1.2G/4G

# **测试用例**

## **5.1 CPU性能测试**

| **测试用例1：CPU性能测试** |                                                              |
| -------------------------- | ------------------------------------------------------------ |
| **测试目的**               | CPU在单核及多核运行情况下的运算速度                          |
| **前置条件**               | 客户端运行正常                                               |
| **步骤**                   | 1、 根据需求测试项，调整测试参数2、 单线程测试执行sysbench cpu --cpu-max-prime=20000 --threads=1 --time=60 run  //20000素数单线程60秒3、 多线程测试，根据CPU内核数修改，修改--threads =[1,2,4,8…]sysbench cpu --cpu-max-prime=20000 --threads=2 --time=60 run  //20000素数2线程604、 记录测试结果 |
| **参数化变量**             | --threads：根据CPU内核数修改线程数，线程数小于等于内核数     |
| **获取指标**               | 1、CPU speed                                                 |

## 5.2 内存测试

| **测试用例2：内存测试** |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| **测试目的**            | 不同内存大小情况下，连续读写或随机读写测试                   |
| **前置条件**            | 客户端运行正常                                               |
| **步骤**                | 1、 根据需求测试项，调整测试参数2、 不同模式下执行内存的读写测试，修改--memory-access-mode参数，单线程、多线程都要测试，例如在内存中传输10G的数据，其中每个block 大小为4k，限制总执行时间为60秒① 随机写入模式单线程执行sysbench memory --time=60 --threads=1 --memory-block-size=4k --memory-total-size=30G --memory-access-mode=rnd run多线程执行（线程由CPU内核决定，本案例为两核CPU，最大取值至2即可）sysbench memory --time=60 --threads=2 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=rnd run  ② 顺序写入模式单线程sysbench memory --time=60 --threads=1 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=seq run多线程sysbench memory --time=60 --threads=2 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=seq run③ 随机读取模式单线程sysbench memory --time=60 --threads=1 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=rnd --memory-oper=read run多线程sysbench memory --time=60 --threads=2 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=rnd --memory-oper=read run④ 顺序读取模式单线程sysbench memory --time=60 --threads=1 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=seq --memory-oper=read run多线程sysbench memory --time=60 --threads=2 --memory-block-size=4k --memory-total-size=10G --memory-access-mode=seq --memory-oper=read run3、记录测试结果 |
| **参数化变量**          | --threads：根据CPU内核数修改线程数，线程数小于等于内核数     |
| **获取指标**            | 1、传输速度（MiB/sec）                                       |

##  5.3 磁盘性能测试

| **测试用例3：磁盘性能测试** |                                                              |
| --------------------------- | ------------------------------------------------------------ |
| **测试目的**                | 磁盘读写性能测试                                             |
| **前置条件**                | 客户端运行正常当前数据盘大小50G                              |
| **步骤**                    | 1、 根据需求测试项，调整测试参数测试随机写IOPS：fio -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=10G -numjobs=1 -runtime=600 -group_reporting -filename=/dev/vdb -name=rndwr_test测试随机读IOPS：fio -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=10G -numjobs=1 -runtime=600 -group_reporting -filename=/dev/vdb -name=rndrd_test测试写吞吐量：fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1M -size=10G -numjobs=1 -runtime=600 -group_reporting -filename=/dev/vdb -name=seqwr_test测试读吞吐量：fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1M -size=10G -numjobs=1 -runtime=600 -group_reporting -filename=/dev/vdb -name=seqrd_test2、 记录测试结果 |
| **参数化变量**              | -bs，调整测试文件块大小                                      |
| **获取指标**                | 1、 IOPS  2、BW                                              |

##  5.4 测试带宽

| **测试用例4：TCP下测试带宽** |                                                              |
| ---------------------------- | ------------------------------------------------------------ |
| **测试目的**                 | TCP下测试带宽                                                |
| **前置条件**                 | 1、服务器运行正常2、两个服务器互相通讯正常                   |
| **步骤**                     | 1、 根据需求测试项，调整测试参数2、 服务端执行iperf -s -i 1  //作为测试服务端，每1秒发送一次报告客户端执行iperf -c <服务端IP> -t 60 -i 1  //作为客户端，向服务端传输数据60秒，每秒发送一次报告3、 记录测试结果 |
| **获取指标**                 | 1、 Bandwidth                                                |

 

| **测试用例5：UDP下测试带宽，延迟抖动及数据丢包率** |                                                              |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **测试目的**                                       | UDP下测试带宽，延迟抖动及数据丢包率                          |
| **前置条件**                                       | 1、服务器运行正常2、两个服务器互相通讯正常                   |
| **步骤**                                           | 1、 根据需求测试项，调整测试参数2、 服务端执行iperf -s -u -i 1 //作为UDP测试服务端，每1秒发送一次报告客户端执行iperf -c <服务端IP> -u -t 60 -b 1.2G -i 1 //作为UDP测试客户端，设定带宽为当前购买服务器的基准带宽1.2G，向服务端传输数据60秒，每秒发送一次报告客户端执行iperf -c <服务端IP> -u -t 60 -b 4G -i 1 //作为UDP测试客户端，设定带宽为当前购买服务器的最大带宽4G，向服务端传输数据60秒，每秒发送一次报告3、 记录测试结果 |
| **获取指标**                                       | 1、 Bandwidth   2、 Jitter  3、 Datagrams                    |

#  6.测试结果

##  6.1 **CPU性能测试**

| 线程数 | 计算最大素数 | 最大执行时间 | 速度（每秒执行事件数） |
| ------ | ------------ | ------------ | ---------------------- |
| 1      | 20000        | 60s          |                        |
| 2      | 20000        | 60s          |                        |

## **6.2 内存性能测试**

| 线程     | 内存访问模式 | 最大执行时间 | 块大小 | 数据大小 | 速度（MiB/sec） |
| -------- | ------------ | ------------ | ------ | -------- | --------------- |
| 1        | 随机写入     | 60s          | 4k     | 10G      |                 |
| 顺序写入 | 60s          | 4k           | 10G    |          |                 |
| 随机读取 | 60s          | 4k           | 10G    |          |                 |
| 顺序读取 | 60s          | 4k           | 10G    |          |                 |
| 2        | 随机写入     | 60s          | 4k     | 10G      |                 |
| 顺序写入 | 60s          | 4k           | 10G    |          |                 |
| 随机读取 | 60s          | 4k           | 10G    |          |                 |
| 顺序读取 | 60s          | 4k           | 10G    |          |                 |

## **6.3 磁盘性能测试**

| 测试项目 | 块大小 | 测试文件大小 | 执行时间 | IOPS（读写操作速率） | 吞吐（读写速率） |
| -------- | ------ | ------------ | -------- | -------------------- | ---------------- |
| 随机写入 | 4k     | 10G          |          |                      |                  |
|          | 8k     | 10G          |          |                      |                  |
|          | 16k    | 10G          |          |                      |                  |
|          | 32k    | 10G          |          |                      |                  |
|          | 64k    | 10G          |          |                      |                  |
|          | 1M     | 10G          |          |                      |                  |
| 随机读取 | 4k     | 10G          |          |                      |                  |
|          | 8k     | 10G          |          |                      |                  |
|          | 16k    | 10G          |          |                      |                  |
|          | 32k    | 10G          |          |                      |                  |
|          | 64k    | 10G          |          |                      |                  |
|          | 1M     | 10G          |          |                      |                  |
| 顺序写入 | 4k     | 10G          |          |                      |                  |
|          | 8k     | 10G          |          |                      |                  |
|          | 16k    | 10G          |          |                      |                  |
|          | 32k    | 10G          |          |                      |                  |
|          | 64k    | 10G          |          |                      |                  |
|          | 1M     | 10G          |          |                      |                  |
| 顺序读取 | 4k     | 10G          |          |                      |                  |
|          | 8k     | 10G          |          |                      |                  |
|          | 16k    | 10G          |          |                      |                  |
|          | 32k    | 10G          |          |                      |                  |
|          | 64k    | 10G          |          |                      |                  |
|          | 1M     | 10G          |          |                      |                  |

##  6.4 网络测试

1)TCP参数记录

| 测试时长(s) | 传输数据(GB) | 带宽（Gbits/sec） |
| ----------- | ------------ | ----------------- |
|             |              |                   |

 

2) UDP参数记录

| 测试条件 | 测试时长(s) | 传输数据(GB) | 带宽（Gbits/sec） | 延迟抖动(ms) | 丢包率 |
| -------- | ----------- | ------------ | ----------------- | ------------ | ------ |
| 基准带宽 |             |              |                   |              |        |
| 最大带宽 |             |              |                   |              |        |

说明：截图部分省略。