[TOC]



- [这些不可不知的JVM知识，我都用思维导图整理好了](https://www.cnblogs.com/three-fighter/p/14402197.html)
- [JVM万字总结](https://juejin.cn/post/6941242430737874974)



# 1、JVM内存模型

![](https://img-blog.csdnimg.cn/20191104180307365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuMTA5MDIzOTc4Mg==,size_16,color_FFFFFF,t_70)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS83LzIyLzE2YzFhNDI2ZWQ5YWI0OGI_aW1hZ2VWaWV3Mi8wL3cvMTI4MC9oLzk2MC9mb3JtYXQvd2VicC9pZ25vcmUtZXJyb3IvMQ?x-oss-process=image/format,png)

![](https://segmentfault.com/img/bVbG0t4)

 - 程序计数器：当前线程所执行的字节码的行号指示器，用于记录正在执行的虚拟机字节指令地址，线程私有。无GC。
 - Java虚拟栈：存放基本数据类型、对象的引用、方法出口等，线程私有。存储当前线程运行方法所需要的数据，指令，返回地址.
 - 本地(Native)方法栈：和虚拟栈相似，只不过它服务于Native方法，线程私有。
 - Java堆：java内存最大的一块，所有对象实例、数组都存放在java堆，GC回收的地方，线程共享。要GC。
 - 方法区：存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码数据等。（即永久带），回收目标主要是常量池的回收和类型的卸载，各线程共享。无GC，非堆区。(java.lang.OutOfMemoryError:PermGen Space)

JVM运行时数据区：
![](https://www.showdoc.cc/server/api/common/visitfile/sign/6fada7a92c17efdeae9a6d294eaabf30?showdoc=.jpg)

 - JDK1.6及以前：有永久代，字符串常量池和运行时常量池在方法区。
 - JDK1.7：有永久代，但已经逐步“去永久代”，字符串常量池移到堆中，运行时常量池还在方法区中（永久代）。
 - JDK1.8及以后：无永久代，字符串常量池在堆中，运行时常量池在元空间。

方法区是一种定义，概念，而所谓永久代或元空间时其一种实现机制。


## 1.1 运行时数据区内存分布实例

![](https://www.showdoc.cc/server/api/common/visitfile/sign/6c2cb92ab158b7873870f2ea3dfcedf8?showdoc=.jpg)


## 1.2 类加载的生命周期

![](https://www.showdoc.cc/server/api/common/visitfile/sign/6795a183b88753622ce24acc90ece5e4?showdoc=.jpg)

 1. 加载类文件，从class文件或jar中，或从二进制流中，以类全名标识存入方法区，供之后的使用。
 2.1 验证类文件是否符合jvm的规范，验证一些类的基本信息。格式验证、语义分析、操作验证。
 2.2 准备为类的静态变量和常量分配空间和初始值，在堆中分配空间。
 2.3 就是把常量池中的符号引用转为直接引用，可以认为是一些静态绑定的会被解析，动态绑定则只会在运行时进行解析；静态绑定包括一些final方法（不可以重写），static方法（只会属于当前类），构造器（不会被重写）。
 3. 初始化将一个类中所有被static关键字标识的代码统一执行一遍，如果执行的是静态变量，那么就会使用用户指定的值覆盖之前在准备阶段设置的初始值；如果执行的是static代码块，那么在初始化阶段，JVM就会执行static代码块中定义的所有操作。所有类变量初始化语句和静态代码块都会在编译时被前端编译器放在收集器里头，存放到一个特殊的方法中，这个方法就是<clinit>方法，即类/接口初始化方法。该方法的作用就是初始化一个变量，使用用户指定的值覆盖之前在准备阶段里设定的初始值。任何invoke之类的字节码都无法调用<clinit>方法，因为该方法只能在类加载的过程中由JVM调用。如果父类还没有被初始化，那么优先对父类初始化，但在<clinit>方法内部不会显示调用父类<clinit>方法，由JVM负责保证一个类的<clinit>方法执行之前，它的父类<clinit>方法已经被执行。JVM必须确保一个类在初始化的过程中，如果是多线程需要同时初始化它，仅仅只能允许其中一个线程对其执行初始化操作，其余线程必须等待，只有在活动线程执行完对类的初始化操作之后，才会通知正在等待的其他线程。

# 2、物理内存与虚拟内存
 - 物理内存即RAM（随机存储器）
 - 寄存器，用于存储计算单元执行指令（如浮点、整数等运算）
 - 地址总线，连接处理器和RAM
 - 虚拟内存使得多个进程在同时运行时可以共享物理内存。

# 3、Java中需要使用内存的组件
## 3.1 Java堆
 - 用于存储Java对象的内存区域
 - -Xmx表示堆的最大大小
 - -Xms表示堆的初始大小
 - 一旦分配完成，就不能在内存不够时再向操作系统重新申请。

## 3.2 线程
 - 每个线程创建JVM时会为线程创建一个堆栈，通常在256KB~756KB之间。

## 3.3 类和类加载器
 - 在Sun SDK中被存储在堆中，这个区域叫永久代（PermGen区）
   - 只有HotSpot才有PermGen Space
   - JRockit(Oracle)，J9（IBM）并没有PermGen Space
   - JDK1.8中PermSize和MaxPermGen已经无效，JDK1.8使用元空间替代PermGen空间，元空间并不在虚拟机中，而是使用本地内存。
 - 默认的3个类加载器
   - Bootstrap ClassLoader
   - ExtClassLoader
   - AppClassLoader

## 3.4 NIO
 - JDK1.4之后，引入了一种基于通道和缓冲区来执行I/O的新方式。
 - 使用java.nio.ByteBuffer.allocateDirect()方法分配内存，分配的是本机内存而不是Java堆内存
 - 每次分配内存时会调用操作系统的os::malloc()函数

## 3.5 JNI
 - JNI使得本机代码（如C语言程序）可以调用Java方法，也就是native memory。

# 4、JVM内存结构
 - PC寄存器-（记录线程当前执行到哪条指令）
 - 栈
 - 堆
 - 方法区-（用于存储类结构信息，可以背所有线程共享，它不会频繁的GC）
 - 运行时常量池-（代表运行时每个class文件中的常量表）
   - 1：数字常量
   - 2：方法
   - 3：字段的引用
 - 本地方法栈-（为JVM运行Native方法准备的空间）

# 5、JVM内存回收策略

![](https://www.showdoc.cc/server/api/common/visitfile/sign/36661bcd6450ad52452ef6ab521b42e2?showdoc=.jpg)

 1. 垃圾对象的判断：引用计数器；可达性分析（虚拟机栈-局部变量，方法区类属性和常量所引用的对象，本地方法栈所引用的，都可作为GCroot）。
 2. 回收策略：标记清除（效率差，存在内存碎片），复制（没有碎片，但浪费空间），标记整理（没有碎片，需要移动对象），分代（）。
 3. 垃圾回收器：Serial(单线程GC，会STW)，ParNew（多线程的GC，会STW），Parallel-Scavenge（多线程通ParNew，不过它关心的是吞吐量，用户代码time/(用户time+GC time))，CMS（三阶段a初始标记，STW，b并行标记，c重新标记并清除，STW---占用CPU，空间碎片，并发异常），G1（并发和并行，分代，空间整合，可预测停顿），ZGC（jdk11)。

## 5.1 回收原则
 - 引用计数法：给对象中添加一个引用计数器，每当一个地方引用这个对象时，计数器值+1；当引用失效时，计数器-1.这种算法场景很多，但是Java中却没有使用这种算法，因为这种算法很难解决对象之间相互引用的情况。
 - 可达性分析法：通过一系列“GC Roots”的对象作为起始点，从这些节点向下搜索，搜索所走过的路径成为引用链，当一个对象到GC Roots没有任何引用链（即GC Roots到对象不可达）时，则证明此对象不是不可用的。

![](https://www.showdoc.cc/server/api/common/visitfile/sign/37b3372dee83ce74e22c2babbf1e2cb9?showdoc=.jpg)

## 5.2 引用状态
 - 强引用：代码中普遍存在的类似“Objec obj = new Object()”这类的引用，只要强引用还在，垃圾收集器永远不会回收掉被引用的对象。
 - 软引用：描述有些还有用但并非必须的对象。在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围进行二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。Java中的类SoftReference表示软引用。
 - 弱引用：描述非必需对象。被弱引用关联的对象只能生存到下一次垃圾回收之前，垃圾收集器工作之后，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。Java中的类WeakReference表示弱引用。
 - 虚引用：这个引用存在的唯一目的就是在这个对象被收集器回收时收到一个系统通知，被虚拟引用关联的对象，和其生存时间完全没关系。Java中的PhantomReference表示虚引用。

## 5.3 方法区的垃圾回收
 - 废弃常量：以字面量回收为例，如果一个字符串“abc”已经进入常量池，但是当前系统没有任何一个String对象引用了叫做“abc”的字面量，那么，如果发生垃圾回收并且有必要时，“abc”就会被系统移出常量池。
 - 无用的类：
   - 该类的所有实例都已经被回收，即java堆中不存在该类的任何实例。
   - 加载该类的ClassLoader已经被回收。
   - 该类对应的Java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

## 5.4 垃圾收集算法
 - 标记-清除（Mark-Sweep）算法：分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象。标记完成后统一回收所有被标记的对象。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/4b78a248350e99e7257c293d96dbc215?showdoc=.jpg)

 - 复制（Copying）算法：它将可用的内存分为两块，每次只用其中一块，当这一块内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已经使用过的内存空间一次性清理掉。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/3f082930f7120c09c55725806c3a5455?showdoc=.jpg)

 - 标记-整理（Mark-Compact）算法：让所有存活对象都向一端移动，然后直接清理掉边界意外的内存。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/ce4526200094ce9717af1e552e130df8?showdoc=.jpg)


 - 分代收集算法：大批对象死去、少量对象存活的（新生代），使用复制算法，复制成本低；对象存活率高、没有额外空间进行分配担保的（老年代），采用标记-清理算法或标记-整理算法。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/148976f94c17e8d750c979f77c25064b?showdoc=.jpg)


## 5.5 垃圾收集器
 - G1收集器（支持收集新生代和老年代）（jdk1.7）：
   - 并行和并发。使用多个CPU来缩短Stop The World停顿时间，与用户线程并发执行。
   - 分代收集。独立管理整个堆，但是能够采用不同的方式去处理新创建对象和已经存活了一段时间、熬过多次GC的旧对象，以获得更好的收集效果。
   - 空间整合。基于标记-整理算法，无内存碎片产生。
   - 可预测的停顿。能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/6fb62669939e303539259d9c0e3f506f?showdoc=.jpg)

 - Young Generation
   - Serial收集器（jdk1.3）：采用复制算法的单线程的收集器。
   - ParNew收集器（jdk1.4）：ParNew收集器其实就是Serial收集器的多线程版本。
   - Parallel Scavenge收集器（jdk1.4）：
     - Parallel Scavenge收集器也是一个新生代收集器，也是用复制算法的收集器，也是并行的多线程收集器。
	 - Parallel Scavenge收集器是虚拟机运行在Server模式下的默认垃圾收集器。
	 - Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/a5c9d0b5fdb863b4704fabcfe79499ea?showdoc=.jpg)


 - Tenured Generation
   - Parallel Old收集器（jdk1.6）：Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。
   - CMS收集器（jdk1.5）：CMS（Conrrurent Mark Sweep）收集器是以获取最短回收停顿时间为目标的收集器。使用标记-清除算法。
     - 1：初始标记，标记GcRoots能直接关联到的对象，时间很短，暂停所有的应用程序线程。
	 - 2：并发标记，进行GCRoots Tracing（可达性分析）过程，时间很长。
	 - 3：重新标记，修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，时间较长，暂停所有的应用程序线程。
	 - 4：并发清除，回收内存空间，时间很长。

![](https://www.showdoc.cc/server/api/common/visitfile/sign/1d8d3c5c2cf2ef38be929a377e3acc6b?showdoc=.jpg)

   - Serial Old收集器（jdk1.5）:Serial收集器的老年代版本，同样是一个单线程收集器，使用“标记-整理算法”。
![](https://www.showdoc.cc/server/api/common/visitfile/sign/bc3e03052016fde1a5339b578de3c0b4?showdoc=.jpg)

## 5.6 GC
 - Minor GC：从年轻代空间（包括Eden和Survivor区域）回收内存被称为Minor GC。
   - 当JVM无法为一个新的对象分配空间时会出发Minor GC，比如当Eden区满了。所以分配率越高，越频繁执行Minor GC。
   - 内存池被填满的时候，其中的内容全部会被复制，指针会从0开始跟踪空闲内存。
   - 执行Minor Gc操作时，不会影响到永久代。
   - 质疑常规的认知，所有的Minor GC都会触发“全世界暂停（stop-the-world）”，停止应用程序的线程。
 - Major GC：Major GC是清理老年代。
 - Full GC：Full GC是清理整个堆空间-包括年轻代和老年代。

## 5.7 最终确认
![](https://www.showdoc.cc/server/api/common/visitfile/sign/620b2b81af247b7b69c761cdd7d36f83?showdoc=.jpg)

# 6、对象的访问
![](https://www.showdoc.cc/server/api/common/visitfile/sign/4dcd802769cb77f456e4ed9f94892b3a?showdoc=.jpg)

对象引用：就是通过栈帧中局部变量表所存储的对象引用用来堆内存中的对象实例进行访问或操作的！简单点理解就是栈帧中有个对象引用的指针，通过各种方法指向了堆内存中的对象实例。

![](https://www.showdoc.cc/server/api/common/visitfile/sign/53b4a93ec1e86b14cd648fb85a1693d9?showdoc=.jpg)

## 6.1 E堆内存模型
![](https://www.showdoc.cc/server/api/common/visitfile/sign/e3cd0b9c01818a63d0027590628f5307?showdoc=.jpg)

逃逸分析：就是分析对象动态作用域，当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中。
-XX:+DoEscapeAnalysis //使用-XX:-DoEscapeAnalysis //不用
使用逃逸分析，编译器可以对代码做如下优化：
 1. 同步省略：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
 2. 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。（栈上分配--HotSpot并没有实现真正意义上的栈上分配，实际上是标量替换）
 3. 分离对象或标量替换：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。如果同步块所使用的锁对象通过这种分析被证实只能够被一个线程访问，就是优化成锁消除。


## 6,2 new
![](https://www.showdoc.cc/server/api/common/visitfile/sign/83d89f4b08650112d9dfab90355aea51?showdoc=.jpg)

## 6.3 Object
![](https://www.showdoc.cc/server/api/common/visitfile/sign/eb5b35abbdd5c4f4cffa6d5354aea824?showdoc=.jpg)
