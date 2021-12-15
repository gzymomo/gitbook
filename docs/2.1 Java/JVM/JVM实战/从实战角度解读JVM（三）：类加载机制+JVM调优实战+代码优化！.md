- [从实战角度解读JVM（三）：类加载机制+JVM调优实战+代码优化！](https://blog.51cto.com/u_15152535/2921787)



## 01 前言

前面我们了解了JVM相关的理论知识，这章节主要从实战方面，去解读JVM。

## 02 类加载机制

Java源代码经过编译器编译成字节码之后，最终都需要加载到虚拟机之后才能运行。虚拟机把描述类的数据从
Class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java 类型，这就是虚拟机的类加载机制。

### 2.1 类加载时机

  一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）七个阶段，其中验证、准备、解析三个部分统称为连接（Linking）。这七个阶段的发生顺序下图所示。

![7ea704e681a648f9b0ba6f24e0cede22~tplv-obj.jpg](https://p6.toutiaoimg.com/img/pgc-image/7ea704e681a648f9b0ba6f24e0cede22~tplv-obj.jpg)

 上图中，加载、验证、准备、初始化和卸载这五个阶段的顺序是确定的，类型的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定特性（也称为动态绑定或晚期绑定）。

 关于在什么情况下需要开始类加载过程的第一个阶段“加载”，《Java虚拟机规范》中并没有进行强制约束，这点可以交给虚拟机的具体实现来自由把握。

 但是对于初始化阶段，《Java虚拟机规范》则是严格规定了有且只有六种情况必须立即对类进行“初始化”（而加
载、验证、准备自然需要在此之前开始）：

- 遇到new、getstatic、putstatic 或invokestatic 这4 条字节码指令；
- 使用java.lang.reflect 包的方法对类进行反射调用的时候；
- 当初始化一个类的时候，发现其父类还没有进行初始化的时候，需要先触发其父类的初始化；
- 当虚拟机启动时，用户需要指定一个要执行的主类，虚拟机会先初始化这个类；
- 当使用JDK 1.7 的动态语言支持时，如果一个java.lang.invoke.MethodHandle 实例最后的解析结果
- REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄，并且这个方法句柄所对应的类没有初始化。
- 当一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现。类发生了初始化，那该接口要在其之前被初始化。

 对于这六种会触发类型进行初始化的场景，《Java虚拟机规范》中使用了一个非常强烈的限定语——“有且只有”，这六种场景中的行为称为对一个类型进行主动引用。除此之外，所有引用类型的方式都不会触发初始化，称为被动引用。

比如如下几种场景就是被动引用：

- 通过子类引用父类的静态字段，不会导致子类的初始化；
- 通过数组定义来引用类，不会触发此类的初始化；
- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化；

### 2.2 类加载过程

**加载**

在加载阶段，Java虚拟机需要完成以下三件事情：

 通过一个类的全限定名来获取定义此类的二进制字节流。

 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

 在内存中生成一个代表这个类的java.lang. Class对象，作为方法区这个类的各种数据的访问入口。

**验证**

 验证是连接阶段的第一步，这一阶段的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。

验证阶段大致上会完成下面4 个阶段的检验动作：

- **文件格式验证** 第一阶段要验证字节流是否符合Class  文件格式的规范，并且能够被当前版本的虚拟机处理。验证点主要包括：是否以魔数0xCAFEBABE  开头；主、次版本号是否在当前虚拟机处理范围之内；常量池的常量中是否有不被支持的常量类型；Class  文件中各个部分及文件本身是否有被删除的或者附加的其它信息等等。
- **元数据验证** 第二阶段是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java  语言规范的要求，这个阶段的验证点包括：这个类是否有父类；这个类的父类是否继承了不允许被继承的类；如果这个类不是抽象类，是否实现了其父类或者接口之中要求实现的所有方法；类中的字段、方法是否与父类产生矛盾等等。
- **字节码验证** 第三阶段是整个验证过程中最复杂的一个阶段，主要目的是通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
- **符号引用验证** 最后一个阶段的校验发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作将在连接的第三阶段--解析阶段中发生。符号引用验证可以看做是对类自身以外（常量池中的各种符号引用）的各类信息进行匹配性校验，通俗来说就是，该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源。

**准备**

 准备阶段是正式为类变量分配内存并设置类变量初始值的阶段。

**解析**

 解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

**初始化**

 类初始化阶段是类加载过程中的最后一步，前面的类加载过程中，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全是由虚拟机主导和控制的。

 到了初始化阶段，才真正开始执行类中定义的Java 程序代码。

### 2.3 类加载器

 类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远超类加载阶段。

 对于任意一个类，都必须由加载它的类加载器和这个类本身一起共同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。

 这句话可以表达得更通俗一些：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。

**双亲委派模型**

 从Java 虚拟机的角度来讲，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap  ClassLoader），这个类加载器使用C++ 来实现，是虚拟机自身的一部分；另一种就是所有其他的类加载器，这些类加载器都由Java  来实现，独立于虚拟机外部，并且全都继承自抽象类 java.lang.ClassLoader 。

 从Java 开发者的角度来看，类加载器可以划分为:

- **启动类加载器（Bootstrap ClassLoader）**：这个类加载器负责将存放在<java_home>\lib 目录中的类库加载到虚拟机内存中。启动类加载器无法被Java 程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，那直接使用null 代替即可；
- **扩展类加载器（Extension ClassLoader）**：这个类加载器由  sun.misc.Launcher$ExtClassLoader 实现，它负责加载<java_home>\lib\ext  目录中，或者被java.ext.dirs 系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器；
- **应用程序类加载器（Application ClassLoader）**：这个类加载器由  sun.misc.Launcher$AppClassLoader 实现。 getSystemClassLoader()  方法返回的就是这个类加载器，因此也被称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库。开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

 我们的应用程序都是由这3 种类加载器互相配合进行加载的，在必要时还可以自己定义类加载器。它们的关系如下图所示：

![022e7caa230b4ff8ad8e7e7eeda83ed7~tplv-obj.jpg](https://p3.toutiaoimg.com/img/pgc-image/022e7caa230b4ff8ad8e7e7eeda83ed7~tplv-obj.jpg)

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。

双亲委派模型的工作过程是：

- 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类
- 而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此
- 因此所有的加载请求最终都应该传送到最顶层的启动类加载器中
- 只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

 这样做的好处就是Java 类随着它的类加载器一起具备了一种带有优先级的层次关系。例如java.lang.  Object，它放在rt.jar 中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型顶端的启动类加载器来加载，因此Object  类在程序的各种类加载器环境中都是同一个类。

 相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为java.lang. Object  的类，并放在程序的ClassPath 中，那系统中将会出现多个不同的Object 类，Java 类型体系中最基本的行为也就无法保证了。

 双亲委派模型对于保证Java程序的稳定运作极为重要，但它的实现却异常简单，用以实现双亲委派的代码只有短短十余行，全部集中在java.lang. ClassLoader的loadClass()方法之中：

```markup
protected synchronized Class<?> loadClass(String name, boolean resolve)
       throws ClassNotFoundException {
   // 首先，检查请求的类是不是已经被加载过
   Class<?> c = findLoadedClass(name);
   if (c == null) {
       try {
           if (parent != null) {
               c = parent.loadClass(name, false);
           } else {
               c = findBootstrapClassOrNull(name);
           }
       } catch (ClassNotFoundException e) {
           // 如果父类抛出 ClassNotFoundException 说明父类加载器无法完成加载
       }
       if (c == null) {
           // 如果父类加载器无法加载，则调用自己的 findClass 方法来进行类加载
           c = findClass(name);
       }
   }
   if (resolve) {
       resolveClass(c);
   }
   return c;
}1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.
```

## 03 JVM调优实战

### 3.1 JVM运行参数

 在jvm中有很多的参数可以进行设置，这样可以让jvm在各种环境中都能够高效的运行。绝大部分的参数保持默认即可。

**三种参数类型**

- 标准参数
  - -help
  - -version
- -X参数(非标准参数)
  - -Xint
  - -Xcomp
- XX参数(使用率较高)
  - -XX:newSize
  - -XX:+UseSerialGC

**-X参数**

 jvm的-X参数是非标准参数，在不同版本的jvm中，参数可能会有所不同，可以通过java -X查看非标准参数。

**-XX参数**

 -XX参数也是非标准参数，主要用于jvm的调优和debug操作。

 -XX参数的使用有2种方式，一种是boolean类型，一种是非boolean类型：

- boolean类型
  - 格式：-XX:[+-] 表示启用或禁用属性
  - 如：-XX:+DisableExplicitGC 表示禁用手动调用gc操作，也就是说调用System.gc()无效
- 非boolean类型
  - 格式：-XX:= 表示属性的值为
  - 如：-XX:NewRatio=4 表示新生代和老年代的比值为1:4

**-Xms和-Xmx参数**

-Xms与-Xmx分别是设置jvm的堆内存的初始大小和最大大小。
-Xmx2048m：等价于-XX:MaxHeapSize，设置JVM最大堆内存为2048M。
-Xms512m：等价于-XX:InitialHeapSize，设置JVM初始堆内存为512M。
适当的调整jvm的内存大小，可以充分利用服务器资源，让程序跑得更快。
示例：

------

```markup
[root@node01 test]# java -Xms512m -Xmx2048m TestJVM
itcast1.2.
```

**jstat**

 jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：
​ jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

**查看class加载统计**

```

F:\t>jstat -class 12076
Loaded  Bytes  Unloaded  Bytes     Time
 5962 10814.2        0     0.0       3.751.2.3.

```

说明：
Loaded：加载class的数量
Bytes：所占用空间大小
Unloaded：未加载数量
Bytes：未加载占用空间
Time：时间

**查看编译统计**

```markup
F:\t>jstat -compiler 12076
Compiled Failed Invalid   Time   FailedType FailedMethod
   3115      0       0     3.43          01.2.3.
```

说明：
Compiled：编译数量。
Failed：失败数量
Invalid：不可用数量
Time：时间
FailedType：失败类型
FailedMethod：失败的方法

**垃圾回收统计**

```

F:\t>jstat -gc 12076
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU
  CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.062
#也可以指定打印的间隔和次数，每1秒中打印一次，共打印5次
F:\t>jstat -gc 12076 1000 5
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU
  CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.062
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.062
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.062
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.062
3584.0 6656.0 3412.1  0.0   180224.0 89915.4   61440.0     5332.1   27904.0 2626
7.3 3840.0 3420.8      6    0.036   1      0.026    0.0621.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.

```

说明：
S0C：第一个Survivor区的大小（KB）
S1C：第二个Survivor区的大小（KB）
S0U：第一个Survivor区的使用大小（KB）
S1U：第二个Survivor区的使用大小（KB）
EC：Eden区的大小（KB）
EU：Eden区的使用大小（KB）
OC：Old区大小（KB）
OU：Old使用大小（KB）
MC：方法区大小（KB）
MU：方法区使用大小（KB）
CCSC：压缩类空间大小（KB）
CCSU：压缩类空间使用大小（KB）
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间

### 3.2 Jmap的使用以及内存溢出分析

 前面通过jstat可以对jvm堆的内存进行统计分析，而jmap可以获取到更加详细的内容，如：内存使用情况的汇总、对内存溢出的定位与分析。

**查看内存使用情况**

```

[root@node01 ~]# jmap -heap 6219
Attaching to process ID 6219, please wait... 
Debugger attached successfully.
Server compiler detected.
JVM version is 25.141-b15
using thread-local object allocation.
Parallel GC with 2 thread(s)
Heap Configuration: #堆内存配置信息
MinHeapFreeRatio         = 0
MaxHeapFreeRatio         = 100
MaxHeapSize              = 488636416 (466.0MB)
NewSize                  = 10485760 (10.0MB)
MaxNewSize               = 162529280 (155.0MB)
OldSize                  = 20971520 (20.0MB)
NewRatio                 = 2
SurvivorRatio            = 8
MetaspaceSize            = 21807104 (20.796875MB)
CompressedClassSpaceSize = 1073741824 (1024.0MB)
MaxMetaspaceSize         = 17592186044415 MB
G1HeapRegionSize         = 0 (0.0MB)
Heap Usage: # 堆内存的使用情况
PS Young Generation #年轻代
Eden Space:
capacity = 123731968 (118.0MB)
used     = 1384736 (1.320587158203125MB)
free     = 122347232 (116.67941284179688MB)
1.1191416594941737% used
From Space:
capacity = 9437184 (9.0MB)
used     = 0 (0.0MB)
free     = 9437184 (9.0MB)
0.0% used
To Space:
capacity = 9437184 (9.0MB)
used     = 0 (0.0MB)
free     = 9437184 (9.0MB)
0.0% used
PS Old Generation #年老代
capacity = 28311552 (27.0MB)
used     = 13698672 (13.064071655273438MB)
free     = 14612880 (13.935928344726562MB)
48.38545057508681% used
13648 interned Strings occupying 1866368 bytes.1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.

```

**查看内存中对象数量及大小**

```

#查看所有对象，包括活跃以及非活跃的
jmap -histo <pid> | more
#查看活跃对象 
jmap -histo:live <pid> | more
[root@node01 ~]# jmap -histo:live 6219 | more
num     #instances         #bytes  class name
----------------------------------------------1:         37437        7914608  [C
2:         34916         837984  java.lang.String
3:           884         654848  [B
4:         17188         550016  java.util.HashMap$Node
5:          3674         424968  java.lang.Class
6:          6322         395512  [Ljava.lang.Object;
7:          3738         328944  java.lang.reflect.Method
8:          1028         208048  [Ljava.util.HashMap$Node;
9:          2247         144264  [I
10:          4305         137760  java.util.concurrent.ConcurrentHashMap$Node
11:          1270         109080  [Ljava.lang.String;
12:            64          84128  [Ljava.util.concurrent.ConcurrentHashMap$Node;
13:          1714          82272  java.util.HashMap
14:          3285          70072  [Ljava.lang.Class;
15:          2888          69312  java.util.ArrayList
16:          3983          63728  java.lang.Object
17:          1271          61008  org.apache.tomcat.util.digester.CallMethodRule
18:          1518          60720  java.util.LinkedHashMap$Entry
19:          1671          53472  com.sun.org.apache.xerces.internal.xni.QName
20:            88          50880  [Ljava.util.WeakHashMap$Entry;
21:           618          49440  java.lang.reflect.Constructor
22:          1545          49440  java.util.Hashtable$Entry
23:          1027          41080  java.util.TreeMap$Entry
24:           846          40608  org.apache.tomcat.util.modeler.AttributeInfo
25:           142          38032  [S
26:           946          37840  java.lang.ref.SoftReference
27:           226          36816  [[C
。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。
#对象说明
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean
[  数组，如[I表示int[]
[L+类名 其他对象1.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.

```

**将内存使用情况dump到文件中**

```

#用法：
jmap -dump:format=b,file=dumpFileName <pid>
#示例
jmap -dump:format=b,file=/tmp/dump.dat 62191.2.3.4.

```

![01a6453f67204e10a20104d51129d657~tplv-obj.jpg](https://p9.toutiaoimg.com/img/pgc-image/01a6453f67204e10a20104d51129d657~tplv-obj.jpg)

可以看到已经在/tmp下生成了dump.dat的文件。

**通过jhat对dump文件进行分析**

 在上一小节中，我们将jvm的内存dump到文件中，这个文件是一个二进制的文件，不方便查看，这时我们可以借助于jhat工具进行查看。

```

#用法：
jhat -port <port> <file>
#示例：
[root@node01 tmp]# jhat -port 9999 /tmp/dump.dat 
Reading from /tmp/dump.dat...
Dump file created Mon Sep 10 01:04:21 CST 2018
Snapshot read, resolving...
Resolving 204094 objects...
Chasing references, expect 40 dots........................................
Eliminating duplicate references........................................
Snapshot resolved.
Started HTTP server on port 9999
Server is ready.1.2.3.4.5.6.7.8.9.10.11.12.13.

```

打开浏览器进行访问：http://192.168.40.133:9999/

![b756d864c46d4be3ab8df66461cff523~tplv-obj.jpg](https://p3.toutiaoimg.com/img/pgc-image/b756d864c46d4be3ab8df66461cff523~tplv-obj.jpg)

在最后由OQL查询功能

![d47daf58efe2409590f975e51ad99e16~tplv-obj.jpg](https://p6.toutiaoimg.com/img/pgc-image/d47daf58efe2409590f975e51ad99e16~tplv-obj.jpg)

![4ae1523ce42f4a2c8cb640b934cce451~tplv-obj.jpg](https://p9.toutiaoimg.com/img/pgc-image/4ae1523ce42f4a2c8cb640b934cce451~tplv-obj.jpg)

3.3 Jmp使用以及内存溢出分析

**使用MAT对内存溢出的定位与分析**

 内存溢出在实际的生产环境中经常会遇到，比如，不断地将数据写入到一个集合中，出现了死循环，读取超大的文件等等，都可能会造成内存溢出。

  如果出现了内存溢出，首先我们需要定位到发生内存溢出的环节，并且进行分析，是正常还是非正常情况，如果是正常的需求，就应该考虑加大内存的设置，如果是非正常需求，那么就要对代码进行修改，修复这个bug。首先，我们得先学会如何定位问题，然后再进行分析。如何定位问题呢，我们需要借助于jmap与MAT工具进行定位分析。

接下来，我们模拟内存溢出的场景。

**模拟内存溢出**

 编写代码，向List集合中添加100万个字符串，每个字符串由1000个UUID组成。如果程序能够正常执行，最后打印ok。

```

public class TestJvmOutOfMemory {
public static void main(String[] args) { 
    List<Object> list = new ArrayList<>();
    for (int i = 0; i < 10000000; i++) {
        	String str = "";
            for (int j = 0; j < 1000; j++) {
            str += UUID.randomUUID().toString();
            }
       		 list.add(str);
        }
    System.out.println("ok");
	}
}1.2.3.4.5.6.7.8.9.10.11.12.13.

```

为了演示效果，我们将设置执行的参数

```

#参数如下：
-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError1.2.

```

**运行测试**

```

java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid5348.hprof ...
Heap dump file created [8137186 bytes in 0.032 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at java.util.Arrays.copyOf(Arrays.java:3332)
at
java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
at java.lang.StringBuilder.append(StringBuilder.java:136)
at cn.itcast.jvm.TestJvmOutOfMemory.main(TestJvmOutOfMemory.java:14)
Process finished with exit code 11.2.3.4.5.6.7.8.9.10.11.

```

可以看到，当发生内存溢出时，会dump文件到java_pid5348.hprof。

![2d27f264ca8649d597ef91f809678c7a~tplv-obj.jpg](https://p6.toutiaoimg.com/img/pgc-image/2d27f264ca8649d597ef91f809678c7a~tplv-obj.jpg)

**导入到MA T工具中进行分析**

![362d81831b9949d08922d9e5eaa1605e~tplv-obj.jpg](https://p26.toutiaoimg.com/img/pgc-image/362d81831b9949d08922d9e5eaa1605e~tplv-obj.jpg)

可以看到，有91.03%的内存由Object[]数组占有，所以比较可疑。
分析：这个可疑是正确的，因为已经有超过90%的内存都被它占有，这是非常有可能出现内存溢出的。

![64b92e8bdcbb43a2bfdc636e2015831b~tplv-obj.jpg](https://p5.toutiaoimg.com/img/pgc-image/64b92e8bdcbb43a2bfdc636e2015831b~tplv-obj.jpg)

可以看到集合中存储了大量的uuid字符串

### 3.4 Jsatck的使用

 有些时候我们需要查看下jvm中的线程执行情况，比如，发现服务器的CPU的负载突然增高了、出现了死锁、死循环等，我们该如何分析呢？

 由于程序是正常运行的，没有任何的输出，从日志方面也看不出什么问题，所以就需要看下jvm的内部线程的执行情况，然后再进行分析查找出原因。

 这个时候，就需要借助于jstack命令了，jstack的作用是将正在运行的jvm的线程情况进行快照，并且打印出来：

```

#用法：jstack <pid>
[root@node01 bin]# jstack 2203
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.141-b15 mixed mode):
"Attach Listener" #24 daemon prio=9 os_prio=0 tid=0x00007fabb4001000 nid=0x906 waiting on
condition [0x0000000000000000]
java.lang.Thread.State: RUNNABLE
"http-bio-8080-exec-5" #23 daemon prio=5 os_prio=0 tid=0x00007fabb057c000 nid=0x8e1
waiting on condition [0x00007fabd05b8000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
- parking to wait for  <0x00000000f8508360> (a
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175) 
at
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueue
dSynchronizer.java:2039)
at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
at
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
at
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-exec-4" #22 daemon prio=5 os_prio=0 tid=0x00007fab9c113800 nid=0x8e0
waiting on condition [0x00007fabd06b9000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
- parking to wait for  <0x00000000f8508360> (a
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
at
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueue
dSynchronizer.java:2039)
at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
at
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
at
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-exec-3" #21 daemon prio=5 os_prio=0 tid=0x0000000001aeb800 nid=0x8df
waiting on condition [0x00007fabd09ba000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
- parking to wait for  <0x00000000f8508360> (a
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
at
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueue
dSynchronizer.java:2039)
at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
at
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134) 
at
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-exec-2" #20 daemon prio=5 os_prio=0 tid=0x0000000001aea000 nid=0x8de
waiting on condition [0x00007fabd0abb000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
- parking to wait for  <0x00000000f8508360> (a
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
at
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueue
dSynchronizer.java:2039)
at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
at
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
at
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-exec-1" #19 daemon prio=5 os_prio=0 tid=0x0000000001ae8800 nid=0x8dd
waiting on condition [0x00007fabd0bbc000]
java.lang.Thread.State: WAITING (parking)
at sun.misc.Unsafe.park(Native Method)
- parking to wait for  <0x00000000f8508360> (a
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
at
java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueue
dSynchronizer.java:2039)
at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:104)
at org.apache.tomcat.util.threads.TaskQueue.take(TaskQueue.java:32)
at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
at
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
at
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
at
org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
at java.lang.Thread.run(Thread.java:748)
"ajp-bio-8009-AsyncTimeout" #17 daemon prio=5 os_prio=0 tid=0x00007fabe8128000 nid=0x8d0
waiting on condition [0x00007fabd0ece000]
java.lang.Thread.State: TIMED_WAITING (sleeping) 
at java.lang.Thread.sleep(Native Method)
at org.apache.tomcat.util.net.JIoEndpoint$AsyncTimeout.run(JIoEndpoint.java:152)
at java.lang.Thread.run(Thread.java:748)
"ajp-bio-8009-Acceptor-0" #16 daemon prio=5 os_prio=0 tid=0x00007fabe82d4000 nid=0x8cf
runnable [0x00007fabd0fcf000]
java.lang.Thread.State: RUNNABLE
at java.net.PlainSocketImpl.socketAccept(Native Method)
at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:409)
at java.net.ServerSocket.implAccept(ServerSocket.java:545)
at java.net.ServerSocket.accept(ServerSocket.java:513)
at
org.apache.tomcat.util.net.DefaultServerSocketFactory.acceptSocket(DefaultServerSocketFac
tory.java:60)
at org.apache.tomcat.util.net.JIoEndpoint$Acceptor.run(JIoEndpoint.java:220)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-AsyncTimeout" #15 daemon prio=5 os_prio=0 tid=0x00007fabe82d1800 nid=0x8ce
waiting on condition [0x00007fabd10d0000]
java.lang.Thread.State: TIMED_WAITING (sleeping)
at java.lang.Thread.sleep(Native Method)
at org.apache.tomcat.util.net.JIoEndpoint$AsyncTimeout.run(JIoEndpoint.java:152)
at java.lang.Thread.run(Thread.java:748)
"http-bio-8080-Acceptor-0" #14 daemon prio=5 os_prio=0 tid=0x00007fabe82d0000 nid=0x8cd
runnable [0x00007fabd11d1000]
java.lang.Thread.State: RUNNABLE
at java.net.PlainSocketImpl.socketAccept(Native Method)
at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:409)
at java.net.ServerSocket.implAccept(ServerSocket.java:545)
at java.net.ServerSocket.accept(ServerSocket.java:513)
at
org.apache.tomcat.util.net.DefaultServerSocketFactory.acceptSocket(DefaultServerSocketFac
tory.java:60)
at org.apache.tomcat.util.net.JIoEndpoint$Acceptor.run(JIoEndpoint.java:220)
at java.lang.Thread.run(Thread.java:748)
"ContainerBackgroundProcessor[StandardEngine[Catalina]]" #13 daemon prio=5 os_prio=0
tid=0x00007fabe82ce000 nid=0x8cc waiting on condition [0x00007fabd12d2000]
java.lang.Thread.State: TIMED_WAITING (sleeping)
at java.lang.Thread.sleep(Native Method)
at
org.apache.catalina.core.ContainerBase$ContainerBackgroundProcessor.run(ContainerBase.jav
a:1513)
at java.lang.Thread.run(Thread.java:748)
"GC Daemon" #10 daemon prio=2 os_prio=0 tid=0x00007fabe83b4000 nid=0x8b3 in Object.wait()
[0x00007fabd1c2f000]
java.lang.Thread.State: TIMED_WAITING (on object monitor)
at java.lang.Object.wait(Native Method)
- waiting on <0x00000000e315c2d0> (a sun.misc.GC$LatencyLock)
at sun.misc.GC$Daemon.run(GC.java:117)
- locked <0x00000000e315c2d0> (a sun.misc.GC$LatencyLock) 
"Service Thread" #7 daemon prio=9 os_prio=0 tid=0x00007fabe80c3800 nid=0x8a5 runnable
[0x0000000000000000]
java.lang.Thread.State: RUNNABLE
"C1 CompilerThread1" #6 daemon prio=9 os_prio=0 tid=0x00007fabe80b6800 nid=0x8a4 waiting
on condition [0x0000000000000000]
java.lang.Thread.State: RUNNABLE
"C2 CompilerThread0" #5 daemon prio=9 os_prio=0 tid=0x00007fabe80b3800 nid=0x8a3 waiting
on condition [0x0000000000000000]
java.lang.Thread.State: RUNNABLE
"Signal Dispatcher" #4 daemon prio=9 os_prio=0 tid=0x00007fabe80b2000 nid=0x8a2 runnable
[0x0000000000000000]
java.lang.Thread.State: RUNNABLE
"Finalizer" #3 daemon prio=8 os_prio=0 tid=0x00007fabe807f000 nid=0x8a1 in Object.wait()
[0x00007fabd2a67000]
java.lang.Thread.State: WAITING (on object monitor)
at java.lang.Object.wait(Native Method)
- waiting on <0x00000000e3162918> (a java.lang.ref.ReferenceQueue$Lock)
at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
- locked <0x00000000e3162918> (a java.lang.ref.ReferenceQueue$Lock)
at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)
"Reference Handler" #2 daemon prio=10 os_prio=0 tid=0x00007fabe807a800 nid=0x8a0 in
Object.wait() [0x00007fabd2b68000]
java.lang.Thread.State: WAITING (on object monitor)
at java.lang.Object.wait(Native Method)
- waiting on <0x00000000e3162958> (a java.lang.ref.Reference$Lock)
at java.lang.Object.wait(Object.java:502)
at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
- locked <0x00000000e3162958> (a java.lang.ref.Reference$Lock)
at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)
"main" #1 prio=5 os_prio=0 tid=0x00007fabe8009000 nid=0x89c runnable [0x00007fabed210000]
java.lang.Thread.State: RUNNABLE
at java.net.PlainSocketImpl.socketAccept(Native Method)
at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:409)
at java.net.ServerSocket.implAccept(ServerSocket.java:545)
at java.net.ServerSocket.accept(ServerSocket.java:513)
at org.apache.catalina.core.StandardServer.await(StandardServer.java:453)
at org.apache.catalina.startup.Catalina.await(Catalina.java:777)
at org.apache.catalina.startup.Catalina.start(Catalina.java:723)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at
sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at org.apache.catalina.startup.Bootstrap.start(Bootstrap.java:321)
at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:455) 
"VM Thread" os_prio=0 tid=0x00007fabe8073000 nid=0x89f runnable
"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x00007fabe801e000 nid=0x89d runnable
"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x00007fabe8020000 nid=0x89e runnable
"VM Periodic Task Thread" os_prio=0 tid=0x00007fabe80d6800 nid=0x8a6 waiting on condition
JNI global references: 431.2.3.4.5.6.7.8.9.10.11.12.13.14.15.16.17.18.19.20.21.22.23.24.25.26.27.28.29.30.31.32.33.34.35.36.37.38.39.40.41.42.43.44.45.46.47.48.49.50.51.52.53.54.55.56.57.58.59.60.61.62.63.64.65.66.67.68.69.70.71.72.73.74.75.76.77.78.79.80.81.82.83.84.85.86.87.88.89.90.91.92.93.94.95.96.97.98.99.100.101.102.103.104.105.106.107.108.109.110.111.112.113.114.115.116.117.118.119.120.121.122.123.124.125.126.127.128.129.130.131.132.133.134.135.136.137.138.139.140.141.142.143.144.145.146.147.148.149.150.151.152.153.154.155.156.157.158.159.160.161.162.163.164.165.166.167.168.169.170.171.172.173.174.175.176.177.178.179.180.181.182.183.184.185.186.187.188.189.190.191.192.193.194.195.196.197.198.199.200.201.202.203.204.205.206.207.208.209.210.211.212.213.

```

### 3.5 VisualVM工具的使用

 VisualVM，能够监控线程，内存情况，查看方法的CPU时间和内存中的对 象，已被GC的对象，反向查看分配的堆栈(如100个String对象分别由哪几个对象分配出来的)。

 VisualVM使用简单，几乎0配置，功能还是比较丰富的，几乎囊括了其它JDK自带命令的所有功能。

- 内存信息
- 线程信息
- Dump堆（本地进程
- Dump线程（本地进程）
- 打开堆Dump。堆Dump可以用jmap来生成
- 打开线程Dump
- 生成应用快照（包含内存信息、线程信息等等）
- 性能分析。CPU分析（各个方法调用时间，检查哪些方法耗时多），内存分析（各类对象占用的内存，检查哪些类占用内存多）
- ......

**启动**

在jdk的安装目录的bin目录下，找到jvisualvm.exe，双击打开即可。

![b2e0d611ce6d4e52843ec466d7fb470f~tplv-obj.jpg](https://p26.toutiaoimg.com/img/pgc-image/b2e0d611ce6d4e52843ec466d7fb470f~tplv-obj.jpg)

![c27dd18e46bd422698db7eb565fc3619~tplv-obj.jpg](https://p26.toutiaoimg.com/img/pgc-image/c27dd18e46bd422698db7eb565fc3619~tplv-obj.jpg)

**查看 CPU、内存、类、线程运行信息**

![222163fa92a24b55a8b1e4b494f33e24~tplv-obj.jpg](https://p9.toutiaoimg.com/img/pgc-image/222163fa92a24b55a8b1e4b494f33e24~tplv-obj.jpg)

**参看线程信息**

![0ab11e9b8de84cc3a775023bdc244960~tplv-obj.jpg](https://p3.toutiaoimg.com/img/pgc-image/0ab11e9b8de84cc3a775023bdc244960~tplv-obj.jpg)

也可以点击右上角Dump按钮，将线程的信息导出，其实就是执行的jstack命令。

![9ef3eacd74404d3ba2fcc9f1e1f6b71d~tplv-obj.jpg](https://p6.toutiaoimg.com/img/pgc-image/9ef3eacd74404d3ba2fcc9f1e1f6b71d~tplv-obj.jpg)

**监控远程JVM**

VisualJVM不仅是可以监控本地jvm进程，还可以监控远程的jvm进程，需要借助于JMX技术实现。

**什么是JMX**

 JMX（Java Management Extensions，即Java管理扩展）是一个为应用程序、设备、系统等植入管理功能的框架。JMX可以跨越一系列异构操作系统平台、系统体系结构和网络传输协议，灵活地开发无缝集成的系统、网络和服务管理应用。

**监控Tomcat**

想要监控远程的tomcat，就需要在远程的tomcat进行对JMX配置，方法如下：

```

#在tomcat的bin目录下，修改catalina.sh，添加如下的参数 
JAVA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9999 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false" 
#这几个参数的意思是： 
#-Dcom.sun.management.jmxremote ：允许使用JMX远程管理 
#-Dcom.sun.management.jmxremote.port=9999 ：JMX远程连接端口 
#-Dcom.sun.management.jmxremote.authenticate=false ：不进行身份认证，任何用户都可以连接 
#-Dcom.sun.management.jmxremote.ssl=false ：不使用ssl1.2.3.4.5.6.7.8.9.

```

**使用VisualJVM远程连接Tomcat**

添加主机

![d698d7a2ff4f434788b59639bc2d27b4~tplv-obj.jpg](https://p3.toutiaoimg.com/img/pgc-image/d698d7a2ff4f434788b59639bc2d27b4~tplv-obj.jpg)

在一个主机下可能会有很多的jvm需要监控，所以接下来要在该主机上添加需要监控的jvm：

![e778c35c964f4dd38f17db5e770da7b1~tplv-obj.jpg](https://p26.toutiaoimg.com/img/pgc-image/e778c35c964f4dd38f17db5e770da7b1~tplv-obj.jpg)

![553a0b5b894d4c5aa8c68c42f0877941~tplv-obj.jpg](https://p3.toutiaoimg.com/img/pgc-image/553a0b5b894d4c5aa8c68c42f0877941~tplv-obj.jpg)

连接成功。使用方法和前面就一样了，就可以和监控本地jvm进程一样，监控远程的tomcat进程。

3.6 可视化GC日志分析工具

**GC日志输出参数**

 前面通过-XX:+PrintGCDetails可以对GC日志进行打印，我们就可以在控制台查看，这样虽然可以查看GC的信息，但是并不直观，可以借助于第三方的GC日志分析工具进行查看。

在日志打印输出涉及到的参数如下：

```

-XX:+PrintGC 输出GC日志 

-XX:+PrintGCDetails 输出GC的详细日志 

-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式） 

-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800） 

-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息 

-Xloggc:../logs/gc.log 日志文件的输出路径  1.2.3.4.5.6.7.8.9.10.11.
 
```

测试：

```

-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -Xmx256m -XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC
-Xloggc:F://test//gc.log  1.2.3.
 
```

运行后就可以在E盘下生成gc.log文件。

**使用GC Easy**

它是一款在线的可视化工具，易用、功能强大，网站：http://gceasy.io/

3.7 调优实战

**先部署一个web项目（自行准备）**

**压测**

下面我们通过jmeter进行压力测试，先测得在初始状态下的并发量等信息，然后我们在对jvm做调优处理，再与初始状态测得的数据进行比较，看调好了还是调坏了。

```

首先需要对jmeter本身的参数调整，jmeter默认的的内存大小只有1g，如果并发数到达300以上时，将无法
正常执行，会抛出内存溢出等异常，所以需要对内存大小做出调整。
修改jmeter.bat文件：
set HEAP=-Xms1g -Xmx4g -XX:MaxMetaspaceSize=512m
在该文件中可以看到，jmeter默认使用的垃圾收集器是G1.
Defaults to '-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:G1ReservePercent=20'1.2.3.4.5.6.

```

**添加gc相关参数**

```
#内存设置较小是为了更频繁的gc，方便观察效果，实际要比此设置的更大 JAVA_OPTS="-XX:+UseParallelGC -XX:+UseParallelOldGC -Xms64m -Xmx128m - XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC - Xloggc:../logs/gc.log -Dcom.sun.management.jmxremote - Dcom.sun.management.jmxremote.port=9999 - Dcom.sun.management.jmxremote.authenticate=false - Dcom.sun.management.jmxremote.ssl=false"
```

重启tomcat

**创建测试用例进行压测**

调优方向主要从以下几个方面：

- 调整内存
- 更换垃圾收集器

**对于JVM的调优，给出大家几条建议**：

- 生产环境的JVM一定要进行参数设定，不能全部默认上生产。
- 对于参数的设定，不能拍脑袋，需要通过实际并发情况或压力测试得出结论。
- 对于内存中对象临时存在居多的情况，将年轻代调大一些。如果是G1或ZGC，不需要设定。
- 仔细分析gceasy给出的报告，从中分析原因，找出问题。
- 对于低延迟的应用建议使用G1或ZGC垃圾收集器。
- 不要将焦点全部聚焦jvm参数上，影响性能的因素有很多，比如：操作系统、tomcat本身的参数等。

**PerfMa**

PerfMa提供了JVM参数分析、线程分析、堆内存分析功能，界面美观，功能强大，我们在做jvm调优时，可以作为一个辅助工具。官网：https://www.perfma.com/

## 04 Tomcat8优化

### 4.1 禁用AJP连接

在服务状态页面中可以看到，默认状态下会启用AJP服务，并且占用8009端口。

![c1e62afb6e0c4a14b894c647fcb42efd~tplv-obj.jpg](https://p26.toutiaoimg.com/img/pgc-image/c1e62afb6e0c4a14b894c647fcb42efd~tplv-obj.jpg)

什么是AJP呢？

AJP（Apache JServer Protocol）  AJPv13协议是面向包的。WEB服务器和Servlet容器通过TCP连接来交互；为了节省SOCKET创建的昂贵代价，WEB服务器会尝试维护一个永久TCP连接到servlet容器，并且在多个请求和响应周期过程会重用连接。

我们一般是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用。

修改conf下的server.xml文件，将AJP服务禁用掉即可。

```markup
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />1.
```

### 4.2 执行器（线程池）

在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。

修改server.xml文件：

```

<!--将注释打开--> <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="500" minSpareThreads="50" prestartminSpareThreads="true" maxQueueSize="100"/> 
<!-- 参数说明： maxThreads：最大并发数，默认设置 200，一般建议在 500 ~ 1000，根据硬件设施和业务来判断 minSpareThreads：Tomcat 初始化时创建的线程数，默认设置 25 prestartminSpareThreads： 在 Tomcat 初始化的时候就初始化 minSpareThreads 的参数值，如果不等于 true，minSpareThreads 的值就没啥效果了 maxQueueSize，最大的等待队列数，超过则拒绝请求 --> 

<!--在Connector中设置executor属性指向上面的执行器--> 
<Connector executor="tomcatThreadPool" port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />1.2.3.4.5.

```

保存退出，重启tomcat，查看效果。

### 4.3 三种运行模式

tomcat的运行模式有3种：

\1. bio 默认的模式,性能非常低下,没有经过任何优化处理和支持.

\2. nio nio(new I/O)，是Java SE  1.4及后续版本提供的一种新的I/O操作方式(即java.nio包及其子包)。Java  nio是一个基于缓冲区、并能提供非阻塞I/O操作的Java API，因此nio也被看成是non-blocking  I/O的缩写。它拥有比传统I/O操作(bio)更好的并发运行性能。

\3. apr 安装起来最困难,但是从操作系统级别来解决异步的IO问题,大幅度的提高性能.

推荐使用nio，不过，在tomcat8中有最新的nio2，速度更快，建议使用nio2.

设置nio2

```markup
<Connector executor="tomcatThreadPool" port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol" connectionTimeout="20000" redirectPort="8443" />1.
```

## 05 代码优化建议

**（1）尽可能使用局部变量**

调用方法时传递的参数以及在调用中创建的临时变量都保存在栈中速度较快，其他变量，如静态变量、实例变量等，都在堆中创建，速度较慢。另外，栈中创建的变量，随着方法的运行结束，这些内容就没了，不需要额外的垃圾回收。

**（2）尽量减少对变量的重复计算**

明确一个概念，对方法的调用，即使方法中只有一句语句，也是有消耗的。所以例如下面的操作：

```

for (int i = 0; i < list.size(); i++) {...}1.

```

建议替换为：

```

int length = list.size(); for (int i = 0, i < length; i++) {...}1.

```

这样，在list.size()很大的时候，就减少了很多的消耗。

**（3）尽量采用懒加载的策略，即在需要的时候才创建**

```

String str = "aaa"; 
if (i == 1){ 
  list.add(str); 
}//建议替换成 
if (i == 1){ 
  String str = "aaa"; 
  list.add(str); 
}1.2.3.4.5.6.7.8.

```

**（4）异常不应该用来控制流程**

  异常对性能不利。抛出异常首先要创建一个新的对象，Throwable接口的构造函数调用名为fifillInStackTrace()的本地同步方  法，fifillInStackTrace()方法检查堆栈，收集调用跟踪信息。只要有异常被抛出，Java虚拟机就必须调整调用堆栈，因为在处理过程中创建 了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程。

**（5）不要将数组声明为public static final**

因为这毫无意义，这样只是定义了引用为static final，数组的内容还是可以随意改变的，将数组声明为public更是一个安全漏洞，这意味着这个数组可以被外部类所改变。

**（6）不要创建一些不使用的对象，不要导入一些不使用的类**

这毫无意义，如果代码中出现"The value of the local variable i is not used"、"The import java.util is never used"，那么请删除这些无用的内容

**（7）程序运行过程中避免使用反射**

反射是Java提供给用户一个很强大的功能，功能强大往往意味着效率不高。不建议在程序运行过程中使用尤其是频繁使用反射机制，特别是 Method的invoke方法。

如果确实有必要，一种建议性的做法是将那些需要通过反射加载的类在项目启动的时候通过反射实例化出一个对象并放入内存。

**（8）使用数据库连接池和线程池**

这两个池都是用于重用对象的，前者可以避免频繁地打开和关闭连接，后者可以避免频繁地创建和销毁线程。

**（9）容器初始化时尽可能指定长度**

容器初始化时尽可能指定长度，如：new ArrayList<>(10); new HashMap<>(32); 避免容器长度不足时，扩容带来的性能损耗。

**（10）ArrayList随机遍历快，LinkedList添加删除快**

**（11）使用Entry遍历map**

**（12）不要手动调用System().gc;**

**（13）String尽量少用正则表达式**

正则表达式虽然功能强大，但是其效率较低，除非是有需要，否则尽可能少用。

replace() 不支持正则 replaceAll() 支持正则

如果仅仅是字符的替换建议使用replace()。

**（14）日志的输出要注意级别**

**（15）对资源的close()建议分开操作**