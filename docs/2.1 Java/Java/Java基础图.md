[TOC]

# 1.Java虚拟机运行时数据区图
![](https://user-gold-cdn.xitu.io/2020/4/29/171c6a4017b17e4a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

JVM内存结构是Java程序员必须掌握的基础。
**程序计数器**

- 程序计数器，可以看作当前线程所执行的字节码的行号指示器
- 它是线程私有的。

**Java虚拟机栈**

- 线程私有的，生命周期与线程相同。
- 每个方法被执行的时候都会创建一个"栈帧",用于存储局部变量表(包括参数)、操作数栈、动态链接、方法出口等信息。
- 局部变量表存放各种基本数据类型boolean、byte、char、short等

**本地方法栈**

- 与虚拟机栈基本类似，区别在于虚拟机栈为虚拟机执行的java方法服务，而本地方法栈则是为Native方法服务。

**Java堆**

- Java堆是java虚拟机所管理的内存中最大的一块内存区域，也是被各个线程共享的内存区域，在JVM启动时创建。
- 其大小通过-Xms和-Xmx参数设置，-Xms为JVM启动时申请的最小内存，-Xmx为JVM可申请的最大内存。

**方法区**

- 它用于存储虚拟机加载的类信息、常量、静态变量、是各个线程共享的内存区域。
- -可以通过-XX:PermSize 和 -XX:MaxPermSize 参数限制方法区的大小。

# 2. 堆的默认分配图
![](https://user-gold-cdn.xitu.io/2020/4/30/171c6b5bddadd635?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Java堆 = 老年代 + 新生代
- 新生代 = Eden + S0 + S1
- 新生代与老年代默认比例的值为 1:2 ，可以通过参数 –XX:NewRatio 配置。
- 默认的，Eden : from : to = 8 : 1 : 1 ，可以通过参数–XX:SurvivorRatio 来设定

# 3.方法区结构图
![](https://user-gold-cdn.xitu.io/2020/5/1/171cf2f5b90c260e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

方法区是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

# 4.对象的内存布局图
![](https://user-gold-cdn.xitu.io/2020/5/1/171d01589343138f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

一个Java对象在堆内存中包括**对象头、实例数据和补齐填充**3个部分：

- 对象头包括Mark Word（存储哈希码，GC分代年龄等） 和 类型指针（对象指向它的类型元数据的指针），如果是数组对象，还有一个保存数组长度的空间
- 实例数据是对象真正存储的有效信息，包括了对象的所有成员变量，其大小由各个成员变量的大小共同决定。
- 对齐填充不是必然存在的，仅仅起占位符的作用。

# 5.对象头的Mark Word图
![](https://user-gold-cdn.xitu.io/2020/4/30/171c81fa700b1ecb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Mark Word 用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等。
- 在32位的HotSpot虚拟机中，如果对象处于未被锁定的状态下，那么Mark Word的32bit空间里的25位用于存储对象哈希码，4bit用于存储对象分代年龄，2bit用于存储锁标志位，1bit固定为0，表示非偏向锁。

# 6.对象与Monitor关联结构图
![](https://user-gold-cdn.xitu.io/2020/4/30/171c7d666c9684f7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对象是如何跟monitor有关联的呢？

一个Java对象在堆内存中包括对象头，对象头有Mark word，Mark word存储着锁状态，锁指针指向monitor地址。这其实是Synchronized的底层

# 7.Java Monitor的工作机理图：
Java 线程同步底层就是监视锁Monitor~，如下是Java Monitor的工作机理图：
![](https://user-gold-cdn.xitu.io/2020/4/30/171c7d59c7fb2ff6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 想要获取monitor的线程,首先会进入_EntryList队列。
- 当某个线程获取到对象的monitor后,进入_Owner区域，设置为当前线程,同时计数器_count加1。
- 如果线程调用了wait()方法，则会进入_WaitSet队列。它会释放monitor锁，即将_owner赋值为null,_count自减1,进入_WaitSet队列阻塞等待。
- 如果其他线程调用 notify() / notifyAll() ，会唤醒_WaitSet中的某个线程，该线程再次尝试获取monitor锁，成功即进入_Owner区域。
- 同步方法执行完毕了，线程退出临界区，会将monitor的owner设为null，并释放监视锁。

# 8.创建一个对象内存分配流程图
![](https://user-gold-cdn.xitu.io/2020/5/1/171d05dec9c7bbfd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 对象一般是在Eden区生成。
- 如果Eden区填满，就会触发Young GC。
- 触发Young GC的时候，Eden区实现清除，没有被引用的对象直接被清除。
- 依然存活的对象，会被送到Survivor区，Survivor =S0+S1.
- 每次Young GC时，存活的对象复制到未使用的那块Survivor 区，当前正在使用的另外一块Survivor 区完全清除，接着交换两块Survivor 区的使用状态。
- 如果Young GC要移送的对象大于Survivor区上限，对象直接进入老年代。
- 一个对象不可能一直呆在新生代，如果它经过多次GC，依然活着，次数超过-XX:MaxTenuringThreshold的阀值，它直接进入老年代。简言之就是，对象经历多次滚滚长江，红尘世事，终于成为长者（进入老年代）

# 9.可达性分析算法判定对象存活
可达性分析算法是用来判断一个对象是否存活的~
![](https://user-gold-cdn.xitu.io/2020/4/30/171c7e50b8bf0371?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

算法的核心思想：

通过一系列称为“GC Roots”的对象作为起始点，从这些节点开始根据引用关系向下搜索，搜索走过的路径称为“引用链”，当一个对象到 GC Roots 没有任何的引用链相连时(从 GC Roots 到这个对象不可达)时，证明此对象不可能再被使用。

# 10.标记-清除算法示意图
![](https://user-gold-cdn.xitu.io/2020/5/1/171cef1c308a4f45?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 标记-清除算法是最基础的垃圾收集算法。
- 算法分为两个阶段，标记和清除。
- 首先标记出需要回收的对象，标记完成后，统一回收掉被标记的对象。
- 当然可以反过来，先标记存活的对象，统一回收未被标记的对象。
- 标记-清除 两个缺点是，执行效率不稳定和内存空间的碎片化问题~

# 11.标记-复制算法示意图
![](https://user-gold-cdn.xitu.io/2020/5/1/171cee8899fdf506?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 1969年 Fenichel提出“半区复制”，将内存容量划分对等两块，每次只使用一块。当这一块内存用完，将还存活的对象复制到另外一块，然后把已使用过的内存空间一次清理掉~
- 1989年，Andrew Appel提出“Appel式回收”，把新生代划分为较大的Eden和两块较小的Survivor空间。每次分配内存只使用Eden和其中一块Survivor空间。发生垃圾收集时，将Eden和Survivor中仍然存活的对象一次性复制到另外一块Survivor空间上。Eden和Survivor比例是8:1~
- “半区复制”缺点是浪费可用空间，并且，如果对象存活率高的话，复制次数就会变多，效率也会降低。

# 12.标记-整理算法示意图
![](https://user-gold-cdn.xitu.io/2020/5/2/171d3312542e805b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 1974年，Edward 提出“标记-整理”算法，标记过程跟“标记-清除”算法一样，接着让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存~
- 标记-清除算法和标记整理算法本质差异是：前者是一种非移动式的回收算法，后者是移动式的回收算法。
- 是否移动存活对象都存在优缺点，移动虽然内存回收复杂，但是从程序吞吐量来看，更划算；不移动时内存分配更复杂，但是垃圾收集的停顿时间会更短，所以看收集器取舍问题~
- Parallel Scavenge收集器是基于标记-整理算法的，因为关注吞吐。CMS收集器是基于标记-清除算法的，因为它关注的是延迟。

# 13.垃圾收集器组合图
![](https://user-gold-cdn.xitu.io/2020/5/2/171d45b536a2c859?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 新生代收集器：Serial、ParNew、Parallel Scavenge
- 老年代收集器：CMS、Serial Old、Parallel Old
- 混合收集器：G1

# 14.类的生命周期图
![](https://user-gold-cdn.xitu.io/2020/4/30/171c843fe9b784cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

一个类从被加载到虚拟机内存开始，到卸载出内存为止，这个生命周期经历了七个阶段：加载、验证、准备、解析、初始化、使用、卸载。
**加载阶段**：

- 通过一个类的全限定名来获取定义此类的二进制字节流。
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
- 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

**验证**：

- 验证的目的是确保Class文件的字节流中包含的信息满足约束要求，保证这些代码运行时不会危害虚拟机自身安全
- 验证阶段有：文件格式校验、元数据校验、字节码校验、符号引用校验。

**准备**

- 准备阶段是正式为类中定义的变量（静态变量）分配内存并设置类变量初始值的阶段。

**解析**

- 解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

**初始化**

- 到了初始化阶段，才真正开始执行类中定义的Java字节码。

# 15.类加载器双亲委派模型图
![](https://user-gold-cdn.xitu.io/2020/4/30/171c84d6f56e220d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**双亲委派模型构成**
启动类加载器，扩展类加载器，应用程序类加载器，自定义类加载器

**双亲委派模型工作过程是**
如果一个类加载器收到类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器完成。每个类加载器都是如此，只有当父加载器在自己的搜索范围内找不到指定的类时（即ClassNotFoundException），子加载器才会尝试自己去加载。

**为什么需要双亲委派模型**？
如果没有双亲委派，那么用户是不是可以自己定义一个java.lang.Object的同名类，java.lang.String的同名类，并把它放到ClassPath中,那么类之间的比较结果及类的唯一性将无法保证，因此，双亲委派模型可以防止内存中出现多份同样的字节码。

# 16.栈帧概念结构图
![](https://user-gold-cdn.xitu.io/2020/5/1/171d0288ac431ad3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

栈帧是用于支持虚拟机进行方法调用和方法执行背后的数据结构。栈帧存储了方法的局部变量表、操作数栈、动态连接和方法返回地址信息。
**局部变量表**

- 是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。
- 局部变量表的容量以变量槽(Variable Slot)为最小单位。

**操作数栈**

- 操作数栈，也称操作栈，是一个后入先出栈。
- 当一个方法刚刚开始执行的时候, 该方法的操作数栈也是空的, 在方法的执行过程中, 会有各种字节码指令往操作数栈中写入和提取内容, 也就是出栈与入栈操作。

**动态连接**

- 每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用, 持有引用是为了支持方法调用过程中的动态连接(Dynamic Linking)。

**方法返回地址**

- 当一个方法开始执行时, 只有两种方式退出这个方法 。一种是执行引擎遇到任意一个方法返回的字节码指令。另外一种退出方式是在方法执行过程中遇到了异常。

# 17.Java内存模型图
![](https://user-gold-cdn.xitu.io/2020/5/1/171cef6637426e1e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Java内存模型规定了所有的变量都存储在主内存中
- 每条线程还有自己的工作内存
- 线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝
- 线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。
- 不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。

# 18.线程状态转换关系图
![](https://user-gold-cdn.xitu.io/2020/5/2/171d4191855e2a5a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Java语言定义了6种线程池状态：

- 新建（New）：创建后尚未启动的线程处于这种状态
- 运行（Running）：线程开启start（）方法，会进入该状态。
- 无限等待（Waiting）：处于这种状态的线程不会被分配处理器执行时间，一般LockSupport::park（）,没有设置了Timeoout的Object::wait()方法，会让线程陷入无限等待状态。
- 限期等待（Timed Waiting）：处于这种状态的线程不会被分配处理器执行时间，在一定时间之后他们会由系统自动唤醒。sleep（）方法会进入该状态~
- 阻塞（Blocked）：在程序等待进入同步区域的时候，线程将进入这种状态~
- 结束（Terminated）：已终止线程的线程状态，线程已经结束执行

# 19. Class文件格式图
![](https://user-gold-cdn.xitu.io/2020/5/2/171d4bffc662b84b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- u1、u2、u4、u8 分别代表1个字节、2个字节、4个字节和8个字节的无符号数
- 表是由多个无符号数或者其他表作为数据项构成的复合数据类型
- 每个Class文件的头四个字节被称为魔数（记得以前校招面试，面试官问过我什么叫魔数。。。）
- minor和major version表示次版本号，主版本号
- 紧接着主次版本号之后，是常量池入口，常量池可以比喻为Class文件里的资源仓库~

# 20.JVM参数思维导图
![](https://user-gold-cdn.xitu.io/2020/5/1/171cfb2cb8a29672?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)