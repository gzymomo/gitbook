sysbench并不是一个压力测试工具，是一个基准测试工具。



# sysbench简介

sysbench是跨平台的基准测试工具，支持多线程，支持多种数据库；主要包括以下几种测试：

```
cpu性能
磁盘io性能
调度程序性能
内存分配及传输速度
POSIX线程性能
数据库性能(OLTP基准测试)
```

# 安装sysbench

## 安装1

安装sysbench，sysbench的源码托管在GitHub上，下载源码：

```bash
unzip sysbench-master.zip       #解压源码
yum -y install make automake libtool pkgconfig libaio-devel  #下载依赖包
cd sysbench-master
sh autogen.sh
编译：
./configure --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib   #根据安装的MySQL的位置，设置目录位置
make
make install

这样安装之后使用sysbench命令时会报错。
[root@test3 sysbench-master]# sysbench --version
sysbench: error while loading shared libraries: libmysqlclient.so.20: cannot open shared object file: No such file or directory

解决办法：
在/etc/profile文件中加入一行：
export LD_LIBRARY_PATH=/usr/local/mysql/lib

source /etc/profile
命令可以正常使用
[root@test3 sysbench-master]# sysbench --version
sysbench 1.1.0
```

查看sysbench的一些帮助信息：

```bash
[root@test3 ~]# sysbench --help
Usage:
  sysbench [options]... [testname] [command]

Commands implemented by most tests: prepare run cleanup help

General options:
  --threads=N                     number of threads to use [1]  #线程的数量，默认是1
  --events=N                      limit for total number of events [0]  #限制的最大事件数量，默认是0，不限制
  --time=N                        limit for total execution time in seconds [10]  #整个测试执行的时间
  --warmup-time=N                 #在进行基准测试多少秒之后启用统计信息--forced-shutdown=STRING        #超过--time时间限制后，强制中断，默认是【off】
  --thread-stack-size=SIZE        size of stack per thread [64K]
  --thread-init-timeout=N         wait time in seconds for worker threads to initialize [30]
  --rate=N                        average transactions rate. 0 for unlimited rate [0]
  --report-interval=N             #打印出中间的信念，N表示每隔N秒打印一次，0表示禁用--report-checkpoints=[LIST,...] #转储完全统计信息并在指定时间点复位所有计数器，参数是逗号分隔值的列表，表示从必须执行报告检查点的测试开始所经过的时间（以秒为单位）。 默认情况下，报告检查点处于关闭状态[off]。--debug[=on|off]                print more debugging info [off]
  --validate[=on|off]             #在可能情况下执行验证检查，默认是[off]
  --help[=on|off]                 print help and exit [off]
  --version[=on|off]              print version and exit [off]
  --config-file=FILENAME          File containing command line options
  --luajit-cmd=STRING             perform LuaJIT control command. This option is equivalent to 'luajit -j'. See LuaJIT documentation for more information#上面是一些通用的配置信息，在具体测试某个测试时，会再详细说明参数设置
```

## 安装2

### **sysbench的一些安装依赖**：

```
yum -y install  make automake libtool pkgconfig libaio-devel vim-common
```

　　在我的机器上已经安装上了mysql相关的所有包，如果你机器上还没有安装过这些，那你还要安装上mysql的开发包，由于系统自带mariadb

　　这个mysql分支，所以在安装mysql-devel时应该是安装mariadb-devel

 

### **安装sysbench**：

　　1　　进入到sysbench源码目录

```
/home/jianglexing/Desktop/sysbench-master
```

　　2　　执行autogen.sh用它来生成configure这个文件

```
./autogen.sh
```

　　3　　执行configure && make && make install 来完成sysbench的安装

```
./configure --prefix=/usr/local/sysbench/ --with-mysql --with-mysql-includes=/usr/local/mysql/include --with-mysql-libs=/usr/local/mysql/lib
make
make install
```

　　我这里之所以要这样写是因为我的mysql安装在/usr/local/；而不是默认的rpm的安装位置

 

### **测试是否安装成功**：

```
[root@workstudio bin]# /usr/local/sysbench/bin/sysbench --version
sysbench 1.1.0
```

　　到目前为止sysbench的安装就算是完成了！

### Yum安装

```
yum -y install sysbench
```

# sysbench语法

执行sysbench –help，可以看到sysbench的详细使用方法。

sysbench的基本语法如下：

**sysbench [options]... [testname] [command]**

下面说明实际使用中，常用的参数和命令。

## （1）command

command是sysbench要执行的命令，包括prepare、run和cleanup，顾名思义，prepare是为测试提前准备数据，run是执行正式的测试，cleanup是在测试完成后对数据库进行清理。

## （2）testname

testname指定了要进行的测试，在老版本的sysbench中，可以通过--test参数指定测试的脚本；而在新版本中，--test参数已经声明为废弃，可以不使用--test，而是直接指定脚本。

例如，如下两种方法效果是一样的：

```
sysbench ``--test=./tests/include/oltp_legacy/oltp.lua``sysbench ./tests/include/oltp_legacy/oltp.lua
```

测试时使用的脚本为lua脚本，可以使用sysbench自带脚本，也可以自己开发。对于大多数应用，使用sysbench自带的脚本就足够了。不同版本的sysbench中，lua脚本的位置可能不同，可以自己在sysbench路径下使用find命令搜索oltp.lua。P.S.：大多数数据服务都是oltp类型的，如果你不了解什么是oltp，那么大概率你的数据服务就是oltp类型的。

## （3）options

sysbench的参数有很多，其中比较常用的包括：

**MySQL****连接信息参数**

- --mysql-host：MySQL服务器主机名，默认localhost；如果在本机上使用localhost报错，提示无法连接MySQL服务器，改成本机的IP地址应该就可以了。
- --mysql-port：MySQL服务器端口，默认3306
- --mysql-user：用户名
- --mysql-password：密码

**MySQL****执行参数**

- --oltp-test-mode：执行模式，包括simple、nontrx和complex，默认是complex。simple模式下只测试简单的查询；nontrx不仅测试查询，还测试插入更新等，但是不使用事务；complex模式下测试最全面，会测试增删改查，而且会使用事务。可以根据自己的需要选择测试模式。
- --oltp-tables-count：测试的表数量，根据实际情况选择
- --oltp-table-size：测试的表的大小，根据实际情况选择
- --threads：客户端的并发连接数
- --time：测试执行的时间，单位是秒，该值不要太短，可以选择120
- --report-interval：生成报告的时间间隔，单位是秒，如10

# sysbench使用举例

在执行sysbench时，应该注意：

（1）尽量不要在MySQL服务器运行的机器上进行测试，一方面可能无法体现网络（哪怕是局域网）的影响，另一方面，sysbench的运行（尤其是设置的并发数较高时）会影响MySQL服务器的表现。

（2）可以逐步增加客户端的并发连接数（--thread参数），观察在连接数不同情况下，MySQL服务器的表现；如分别设置为10,20,50,100等。

（3）一般执行模式选择complex即可，如果需要特别测试服务器只读性能，或不使用事务时的性能，可以选择simple模式或nontrx模式。

（4）如果连续进行多次测试，注意确保之前测试的数据已经被清理干净。

# 建议

使用sysbench的一些建议。

1、在开始测试之前，应该首先明确：应采用针对整个系统的基准测试，还是针对MySQL的基准测试，还是二者都需要。

2、如果需要针对MySQL的基准测试，那么还需要明确精度方面的要求：是否需要使用生产环境的真实数据，还是使用工具生成也可以；前者实施起来更加繁琐。如果要使用真实数据，尽量使用全部数据，而不是部分数据。

3、基准测试要进行多次才有意义。

4、测试时需要注意主从同步的状态。

5、测试必须模拟多线程的情况，单线程情况不但无法模拟真实的效率，也无法模拟阻塞甚至死锁情况。

# sysbench除了以上的测试之外，还可以测试：

```bash
Compiled-in tests:
  fileio - File I/O test  
  cpu - CPU performance test
  memory - Memory functions speed test
  threads - Threads subsystem performance test
  mutex - Mutex performance test

See 'sysbench <testname> help' for a list of options for each test
```

