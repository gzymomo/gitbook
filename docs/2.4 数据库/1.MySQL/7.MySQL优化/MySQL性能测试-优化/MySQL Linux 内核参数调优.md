#### 1. Swappiness

swappiness内核参数将在内存不够用的条件下，决定内存换页的策略。该参数取值范围0~100，取值越大，内核会越积极地使用swap分区，设置为0表示最大限度使用物理内存，对于数据库应用，建议该参数设置为0。

查看当前的swappiness值：
cat /proc/sys/vm/swappiness

设置swappiness值：
echo 0 > /proc/sys/vm/swappiness

持久化到配置文件：
/etc/sysctl.conf
vm.swappiness=0

#### 2. I/O Scheduler

数据库对I/O性能要求高，存储数据、读取数据都需要大量消耗I/O，因此磁盘I/O调度策略是数据库需要关注的重点内核参数。

目前 Linux 上有如下几种 I/O 调度算法：

- noop(No Operation)，noop调度算法是内核中最简单的IO调度算法。noop调度算法也叫作电梯调度算法，它将IO请求放入到一个FIFO队列中，然后逐个执行这些IO请求，当然对于一些在磁盘上连续的IO请求，noop算法会适当做一些合并。这个调度算法特别适合那些不希望调度器重新组织IO请求顺序的应用。
- cfq(Completely Fair Scheduler )，完全公平调度器，它试图为竞争块设备使用权的所有进程分配一个请求队列和一个时间片，在调度器分配给进程的时间片内，进程可以将其读写请求发送给底层块设备，当进程的时间片消耗完，进程的请求队列将被挂起，等待调度，进程平均使用IO带宽。
- deadline，deadline算法的核心在于保证每个IO请求在一定的时间内一定要被服务到，以此来避免某个请求饥饿。
- anticipatory，启发式调度，类似 deadline 算法，但是引入预测机制提高性能。

CentOS 7.x 默认支持的是deadline算法，CentOS 6.x 下默认支持的cfq算法，而一般我们会在SSD固态盘硬盘环境中使用noop算法。

查看I/O Scheduler值：
cat /sys/block/sdb/queue/scheduler

设置I/O Scheduler值：
sudo echo noop > /sys/block/sdb/queue/scheduler

#### 3. TCP

##### 3.1 tcp接受连接设置

- net.core.netdev_max_backlog = 65535，当网络接收速率大于内核处理速率时，允许发送到队列中的包数目。
- net.ipv4.tcp_max_syn_backlog = 65535，表示SYN队列长度，默认1024，改成65535，可以容纳更多等待连接的网络连接数。
- net.core.somaxconn = 65535，每个端口监听队列的最大长度。

##### 3.2 tcp失效连接资源回收

对于tcp失效连接占用系统资源的优化，加快资源回收效率：

- net.ipv4.tcp_keepalive_time = 30，表示当keepalive启用的时候，TCP发送keepalive消息的频度，默认为2小时，改为30秒。
- net.ipv4.tcp_keepalive_intvl = 3，tcp未获得响应时重发间隔。
- net.ipv4.tcp_keepalive_probes = 3，tcp未获得响应时重发数量。

控制tcp连接等待时间，加快tcp链接回收：

- net.ipv4.tcp_fin_timeout = 10，如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。
- net.ipv4.tcp_tw_reuse = 1，开启socket重用,允许将TIME-WAIT socket重新用于新的TCP连接，默认为0，表示关闭。
- net.ipv4.tcp_tw_recycle = 1，开启TCP连接中TIME-WAIT socket的快速回收，默认为0，表示关闭。

##### 3.3 tcp接受连接缓冲区大小

控制tcp接受缓冲区的大小，设置大一些比较好：

- net.core.wmem_default = 8388608
- net.core.wmem_max = 16777216
- net.core.rmem_default = 8388608
- net.core.rmem_max = 16777216

#### 4. 最大文件句柄

操作系统最大文件句柄数限制：
vim /etc/security/limits.conf

- soft nofile 65535
- hard nofile 65535

#### 5. CPU Governor

大多数现代处理器都能在许多不同的时钟频率和电压配置下工作。一般来说，时钟频率越高，电压越高，CPU可以执行的指令就越多。同样，时钟频率和电压越高，消耗的能量越多。因此，在CPU容量和处理器消耗的功率之间有一个权衡。

使用以下命令检查正在使用的驱动程序和调控器：

```
analyzing CPU 0:
  driver: intel_pstate
  CPUs which run at the same hardware frequency: 0
  CPUs which need to have their frequency coordinated by software: 0
  maximum transition latency:  Cannot determine or is not supported.
  hardware limits: 1.20 GHz - 3.20 GHz
  available cpufreq governors: performance powersave
  current policy: frequency should be within 1.20 GHz and 3.20 GHz.
                  The governor "powersave" may decide which speed to use
                  within this range.
  current CPU frequency: 2.40 GHz (asserted by call to hardware)
  boost state support:
    Supported: no
    Active: no
```

查看CPU加速设置：
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

设置CPU加速策略：
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

#### 6. NUMA

NUMA表示Non-uniform memory access，带有NUMA功能，SMP系统的处理器访问它自己的本地内存比非本地内存更快。这可能会导致内存换出(swap)，从而对数据库性能产生负面影响。当分配给InnoDB缓冲池的内存大于可用内存的数量，并且选择了默认内存分配策略时，可能会发生内存交换(swap)。启用NUMA的服务器将报告CPU节点之间的不同节点距离(distances)。

通过如下命令查看：
numactl --hardware

不管numactl显示的跨节点间的距离是多少，应当开启MySQL参数innodb_numa_interleave，来确保内存可以交错使用。