[JVM性能优化](https://www.cnblogs.com/itwxe/p/14884361.html)

## 一、内存溢出

内存溢出的原因：程序在申请内存时，没有足够的空间。

### 1. 栈溢出

方法死循环递归调用（StackOverflowError）、不断建立线程（OutOfMemoryError）。

### 2. 堆溢出

不断创建对象，分配对象大于最大堆的大小（OutOfMemoryError）。

### 3. 直接内存

JVM 分配的本地直接内存大小大于 JVM 的限制，可以通过-XX:MaxDirectMemorySize 来设置（不设置的话默认与堆内存最大值一样,也会出现OOM 异常)。

### 4. 方法区溢出

一个类要被垃圾收集器回收掉，判定条件是比较苛刻的，在经常动态生产大量 Class 的应用中，CGLIb 字节码增强，动态语言，大量  JSP(JSP 第一次运行需要编译成 Java 类),基于 OSGi  的应用(同一个类，被不同的加载器加载也会设为不同的类)，都可能会导致OOM。

## 二、内存泄露

程序在申请内存后，无法释放已申请的内存空间，导致这一部分的原因主要是代码写的不合理，比如以下几种情况。

### 1. 长生命周期的对象持有短生命周期对象的引用

例如将 ArrayList 设置为静态变量，然后不断地向ArrayList中添加对象，则 ArrayList 容器中的对象在程序结束之前将不能被释放，从而造成内存泄漏。

### 2. 连接未关闭

如数据库连接、网络连接和 IO 连接等，只有连接被关闭后，垃圾回收器才会回收对应的对象。

### 3. 变量作用域不合理

例如:

- 一个变量的定义的作用范围大于其使用范围。
- 如果没有及时地把对象设置为 null。

### 4. 内部类持有外部类

Java 的 **非静态内部类** 的这种创建方式，会隐式地持有外部类的引用，而且默认情况下这个引用是强引用，因此，如果内部类的生命周期长于外部类的生命周期，程序很容易就产生内存泄露（可以理解为：垃圾回收器会回收掉外部类的实例，但由于内部类持有外部类的引用，导致垃圾回收器不能正常工作）。

解决办法：将非静态内部类改为 **静态内部类**，即加上 static 修饰，例如：

```java
public class Jvm5 {
    private static String string = "SuunyBear";

    public static void show() {
        System.out.println("show");
    }

    public static void main(String[] args) {
        Jvm5 m = new Jvm5();
        // 非静态内部类的构造方式
        // Child c=m.new Child();
        Child c = new Child();
        c.test();
    }

    /**
     * 内部类Child --静态的，防止内存泄漏
     */
    static class Child {
        public int i;

        public void test() {
            System.out.println("string:" + string);
            show();
        }
    }
}
```

### 5. Hash值改变

在集合中，如果修改了对象中的那些参与计算哈希值的字段，会导致无法从集合中单独删除当前对象，造成内存泄露。

使用例子来说明。

```java
public class Jvm6 {
    private int x;
    private int y;

    public Jvm6(int x, int y) {
        super();
        this.x = x;
        this.y = y;
    }
    /**
     * 重写HashCode的方法
     */
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + x;
        result = prime * result + y;
        return result;
    }
    /**
     * 改变y的值：同时改变hashcode
     */
    public void setY(int y) {
        this.y = y;
    }

    public static void main(String[] args) {
        HashSet<Jvm6> hashSet = new HashSet<Jvm6>();
        Jvm6 data1 = new Jvm6(1, 3);
        Jvm6 data2 = new Jvm6(3, 5);
        hashSet.add(data1);
        hashSet.add(data2);
        data2.setY(7); // data2的Hash值改变
        hashSet.remove(data2); // 删掉data2节点
        System.out.println(hashSet.size()); // 2
    }
}
```

## 三、内存溢出和内存泄漏辨析

- 内存溢出：实实在在的内存空间不足导致。
- 内存泄漏：该释放的对象没有释放，常见于使用容器保存元素的情况下。

**如何避免**：

- 内存溢出：检查代码以及设置足够的空间。
- 内存泄漏：一定是代码有问题，往往很多情况下，内存溢出往往是内存泄漏造成的。

## 四、了解MAT

mat是一个内存泄露的分析工具。

### 1. 浅堆和深堆

- 浅堆（Shallow Heap）：是指一个对象所消耗的内存。
- 深堆（Retained Heap）：这个对象被 GC 回收后，可以真实释放的内存大小，也就是只能通过对象被直接或间接访问到的所有对象的集合。通俗地说，就是一个对象包含（引用）的所有对象的大小，如图：

![深堆](https://img-blog.csdnimg.cn/img_convert/c5d54cf4b5a2304633187dc86435eab9.png)

### 2. MAT的使用

1、下载MAT工具：[下载地址](https://www.eclipse.org/mat/downloads.php)

2、内存溢出例子演示

参数说明：

- -Xms5m 堆初始大小5M
- -Xmx5m 堆最大大小5M
- -XX:+PrintGCDetails 打印gc日志详情
- -XX:+HeapDumpOnOutOfMemoryError 输出内存溢出文件
- -XX:HeapDumpPath=D:/oomDump/dump.hprof 内存溢出文件保存位置，此文件用于MAT分析

```java
/**
 * VM Args：-Xms5m -Xmx5m  -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=D:/oomDump/dump.hprof
 */
public class Jvm7 {

    public static void main(String[] args) {
        // 在方法执行的过程中，它是GCRoots
        List<Object> list = new LinkedList<>();
        int i = 0;
        while (true) {
            i++;
            if (i % 10000 == 0) {
                System.out.println("i=" + i);
            }
            list.add(new Object());
        }
    }
}
```

设置参数运行后，内存溢出，程序结束，然后我们就可以用下载好的MAT来分析了，当然MAT也只是分析猜想，并不代表一定是这个原因导致内存溢出。

打开我们保存的文件目录进行分析。

![MAT使用1](https://img-blog.csdnimg.cn/img_convert/358d6035fab2ac54fdd81b999200f703.png)

分析结果。

![MAT使用2](https://img-blog.csdnimg.cn/img_convert/b66ccf0a4af75b3ab26353f020a30e6b.png)

此时可以查看详情查看具体原因，当然这个原因也只是一种猜想。

## 五、JDK提供的一些工具

| 分类       | 属性值                   | 描述                 |
| ---------- | ------------------------ | -------------------- |
| 命令行工具 | jps                      | 虚拟机进程状况工具   |
| jstat      | 虚拟机统计信息监视工具   |                      |
| jinfo      | Java配置信息工具         |                      |
| jmap       | Java内存映像工具         |                      |
| jhat       | 虚拟机堆转储快照分析工具 |                      |
| jstack     | Java堆栈跟踪工具         |                      |
| 可视化工具 | JConsole                 | Java监视与管理控制台 |
| VisualVM   | 多合一故障处理工具       |                      |

所有的工具都在jdk的安装bin目录下，比如我的在`C:\My Program Files\Java\jdk1.8.0_201\bin`。

其中一般情况命令行在线上服务器上使用，可视化工具在本地使用，当然如果你的线上服务器允许远程的话也可以使用可视化工具。

## 六、GC调优

### 1. GC调优重要参数

**生产环境推荐开启**

- -XX:+HeapDumpOnOutOfMemoryError
  - 输出内存溢出文件
- -XX:HeapDumpPath=D:/oomDump/dump.hprof
  - 内存溢出文件保存位置，此文件用于MAT分析
  - 当然，一般Linux服务器可以设置为 `./java_pid<pid>.hprof` 默认为Java进程启动位置

**调优之前开始，调优之后关闭**

- -XX:+PrintGC
  - 调试跟踪之 打印简单的 GC 信息参数:
- -XX:+PrintGCDetails和-XX:+PrintGCTimeStamps
  - 打印详细的 GC 信息
- -Xlogger:logpath:log/gc.log
  - 设置 gc 的日志路，将 gc.log 的路径设置到当前目录的 log 目录下. 应用场景： 将 gc 的日志独立写入日志文件，将 GC 日志与系统业务日志进行了分离，方便开发人员进行追踪分析

**考虑使用**

- -XX:+PrintHeapAtGC
  - 打印推信息，获取 Heap 在每次垃圾回收前后的使用状况
- -XX:+TraceClassLoading
  - 在系统控制台信息中看到 class 加载的过程和具体的 class 信息，可用以分析类的加载顺序以及是否可进行精简操作
- -XX:+DisableExplicitGC
  - 禁止在运行期显式地调用 System.gc()

### 2. GC调优的原则（很重要）

- **大多数的 java 应用不需要 GC 调优**
- 大部分需要 GC 调优的的，**不是参数问题，是代码问题**
- 在实际使用中，分析 **GC 情况优化代码** 比 **优化 GC 参数** 要多得多
- **GC 调优是最后的手段**

**调优的目的**

- GC 的时间够小
- GC 的次数够少发生
- Full GC 的周期足够的长，时间合理，最好是不发生

**注：** 如果满足下面的指标，则一般不需要进行 GC调优

- Minor GC 执行时间不到 50ms
- Minor GC 执行不频繁，约 10 秒一次
- Full GC 执行时间不到 1s
- Full GC 执行频率不算频繁，不低于 10 分钟 1 次

### 3. GC调优步骤

1、监控 GC 的状态使用各种 JVM 工具，查看当前日志，分析当前 JVM 参数设置，并且分析当前堆内存快照和 gc 日志，根据实际的各区域内存划分和 GC 执行时间，觉得是否进行优化。

2、分析结果，判断是否需要优化如果各项参数设置合理。

- 系统没有超时日志出现，GC 频率不高，GC 耗时不高，那么没有必要进行 GC 优化。
- 如果 GC 时间超过 1 秒，或者频繁 GC，则必须优化。

3、调整 GC 类型和内存分配如果内存分配过大或过小，或者采用的 GC 收集器比较慢，则应该优先调整这些参数，并且先找 1 台或几台机器进行 测试，然后比较优化过的机器和没有优化的机器的性能对比，并有针对性的做出最后选择。

4、不断的分析和调整通过不断的试验和试错，分析并找到最合适的参数5，全面应用参数如果找到了最合适的参数，则将这些参数应用到所有服务器，并进行后续跟踪。

**分析GC日志**

主要关注 MinorGC 和 FullGC 的回收效率（回收前大小和回收比较）、回收的时间。

1、-XX:+UseSerialGC

- 以参数-Xms5m -Xmx5m -XX:+PrintGCDetails -XX:+UseSerialGC 为例详细说明。
- [DefNew: 1855K->1855K(1856K), 0.0000148 secs][Tenured: 2815K->4095K(4096K), 0.0134819 secs] 4671K。
- DefNew 指明了收集器类型，而且说明了收集发生在新生代。
- 1855K->1855K(1856K)表示，回收前 新生代占用 1855K，回收后占用 1855K，新生代大小 1856K
- 0.0000148 secs 表明新生代回收耗时。
- Tenured 表明收集发生在老年代。
- 2815K->4095K(4096K), 0.0134819 secs：含义同新生代最后的 4671K 指明堆的大小。

2、-XX:+UseParNewGC

- 收集器参数变为-XX:+UseParNewGC。
- 日志变为：[ParNew: 1856K->1856K(1856K), 0.0000107 secs][Tenured: 2890K->4095K(4096K), 0.0121148 secs]。
- 收集器参数变为-XX:+ UseParallelGC 或 UseParallelOldGC。
- 日志变为：[PSYoungGen: 1024K->1022K(1536K)] [ParOldGen: 3783K->3782K(4096K)] 4807K->4804K(5632K)。

3、-XX:+UseConcMarkSweepGC 和 -XX:+UseG1GC

使用这两个收集器的日志会和UseParNewGC一样有明显的相关字样。

### 4. 项目启动调优

开启日志分析-XX:+PrintGCDetails，启动项目时，通过分析日志，不断地调整参数，减少GC次数。

例如：

1、碰到 Metadata空间 不足发生GC，那么调整 Metadata空间 `-XX:MetaspaceSize=64m` 减少 FullGC 。
 2、碰到MinorGC，那么调整堆空间 `-Xms1000m` 大小减少FullGC 。
 3、如果还是有MinorGC，那么继续增大堆空间大小，或者增大新生代比例 `-Xmn900m GC`，此时新生代空间为900m，老年代大小100m 。

### 5. 项目运行GC调优

使用 [jmeter 工具](https://jmeter.apache.org/download_jmeter.cgi) 来进行压测，然后分析原因，进行调优，当然 **正式上线的项目请谨慎操作** 。

**jmeter工具安装使用**

1、下载好对应版本的jmeter，注意jdk版本。

![jmeter下载](https://img-blog.csdnimg.cn/img_convert/78fb3cb9e14fc98fc44bc888e0e4c47f.png)

2、jmeter需要Java运行时环境，所以如果报错请先检查你的Java环境变量设置，解压到你想要的路径，例如我解压在`C:\My Program Files\apache-jmeter-5.2.1`，在bin目录下有一个 `jmeter.bat` 文件，双击启动。

至于具体怎么使用就百度吧，基本拿到软件就知道使用了，毕竟这个说来就浪费篇幅了。

**聚合报告参数**

这里放出我本地 jmeter 测试一个项目之后的 **聚合报告参数解释**。

![聚合报告参数](https://img-blog.csdnimg.cn/img_convert/c8ea2fe860ce321d4a0836e42ab803ce.png)

### 6. 推荐策略（仅作参考）

1、新生代大小选择

- 尽可能设大,直到接近系统的最低响应时间限制(根据实际情况选择).在此种情况下,新生代收集发生的频率也是最小的.同时,减少到达老年代的对象。
- 避免设置过小，当新生代设置过小时会导致：MinorGC 次数更加频繁、可能导致 MinorGC 对象直接进入老年代,如果此时老年代满了,会触发 FullGC。

2、老年代大小选择

一般吞吐量优先的应用都有一个很大的新生代和一个较小的老年代.原因是,这样可以尽可能回收掉大部分短期对象,减少中期的对象,而老年代尽存放长期存活对象

## 七、逃逸分析

补充知识，并非所有的对象都会在堆上面分配，而没有在堆上分配的对象是因为经过逃逸分析，分析之后发现该对象的大小可以在栈上分配，不会造成栈溢出，这时，对象就可以在栈上分配。

当然，如果经过逃逸分析，发现该对象在栈上分配会照成栈溢出，那么该对象就会在堆空间分配。

参数jdk1.8默认开启

- -XX:+DoEscapeAnalysis 启用逃逸分析(默认打开)
- -XX:+EliminateAllocations 标量替换(默认打开)
- -XX:+UseTLAB 本地线程分配缓冲(默认打开)

## 八、常用的性能评价/测试指标

一个 web 应用不是一个孤立的个体，它是一个系统的部分，系统中的每一部分都会影响整个系统的性能。

1、响应时间：提交请求和返回该请求的响应之间使用的时间，一般比较关注平均响应时间。
 2、并发数：同一时刻，对服务器有实际交互的请求数，和网站在线用户数的关联：1000 个同时在线用户数，可以估计并发数在 5%到 15%之间，也就是同时并发数在 50~150 之间。
 3、吞吐量：对单位时间内完成的工作量(请求)的量度，例如1秒处理5万个请求。