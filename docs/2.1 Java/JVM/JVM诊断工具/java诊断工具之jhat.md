[TOC]

# 1、简介
jhat统计工具可以在堆快照中快速预览对象拓扑，该工具替换了之前的HAT统计工具。
jhat以二进制格式来解析堆dump，例如由jmap -dump生成的堆dump文件。
jhat可以帮助debug无意识的对象保持，这个术语用于描述不再需要的对象，但由于通过来自rootset的某个路径的引用而保持其活动状态。这有可能发生，例如以下情况下：如果在不再需要对象之后，无意中仍然保留对对象的静态引用；如果Observer或者Listener在不需要的时候没有注销；如果引用对象的线程在应该终止时没有终止。无意的对象保留在Java语言中相当于内存泄漏。

# 2、基本语法
`jhat [ options ] heap-dump-file`
- options:命令行选项。
- heap-dump-file:要浏览的java二进制dump文件。对于一个dump文件包含多个堆dump，那么可以使用#<number>的方式来指定，如myfile.hprof#3。

## 2.1 描述
jhat命令解析Java堆dump文件并启动web服务器。jhat可以使用浏览器来浏览堆dump信息，jhat命令支持预先设计的查询，比如显示已知类MyClass的所有实例和对象查询语言(OQL)。OQL类似SQL语言，可以通过http://localhost:7000/oqlhelp/获得OQL帮助。
可以通过以下几种方式来生成堆dump文件：
1. 使用jmap -dump命令
2. 使用jconsole选项
3. 如果配置了-XX:+HeapDumpOnOutOfMemoryError，那么在出现OutOfMemoryError错误的时候会生成dump文件
4. 使用hprof命令

## 2.2选项
- -stack false|true:关闭跟踪对象分配调用堆栈。如果堆dump中没有分配站点信息，则必须将此标志设置为false。默认值为true。
- -refs false|true:关闭对对象的引用跟踪。默认是true，默认情况下，将为堆中的所有对象计算回指针，回指针是指向指定对象(如引用者或传入引用)的对象。
- -port port-number:设置jhat HTTP服务器的端口。默认是7000。
- - -exclude exclude-file:指定一个文件，该文件列出应该从可达对象查询中排除的数据成员。例如：如果指定了java.lang.String.value，那么会排除该属性。
- -baseline exclude-file:指定baseline堆dump文件，具有相同对象ID的两个堆转储中的对象被标记为非新对象。其他对象被标记为new。这对于比较两个不同的堆转储非常有用。
- -debug int:设置工具的调试级别，级别0表示没有调试输出，值越高越详细。
- -version:显示版本号
- -h:显示帮助信息
- -help:显示帮助信息
- -Jflag:将标志传递给运行jhat命令的Java虚拟机。例如，-J-Xmx512m表示使用最大堆大小512 MB。

# 3、故障排除
该工具提供一些标准查询，例如：root查询显示从rootset到指定对象的所有引用路径，对于查找不必要的对象保留特别有用。除了标准的查询之外，我们还可以使用对象查询语言（OQL）接口来编写自己的查询。
jhat是在指定的TCP端口上启动了HTTP服务，然后可以使用任何浏览器连接到服务器，并在指定的堆转储上执行查询。
如下示例：显示如何分析snapshot.hprof文件
```java
$ jhat snapshot.hprof
Started HTTP server on port 7000
Reading from java_pid2278.hprof...
Dump file created Fri May 19 17:18:38 BST 2006
Snapshot read, resolving...
Resolving 6162194 objects...
Chasing references, expect 12324 dots................................
Eliminating duplicate references.....................................
Snapshot resolved.
Server is ready.
```
此时，jhat已经在7000端口上启动了HTTP服务器，将浏览器指向http://localhost:7000来连接jhat服务，连接上之后，就可以使用标准查询或者自定义查询来进行查询，默认情况下显示所有的类。

# 4、标准查询
前提是已经连接了jhat服务。

## 4.1 All Classes Query
默认页是所有类的查询，显示堆中的所有类，但是不包含平台类，此列表按完全限定类名排序，并按包分类，单击类的名称会调用类查询。
该查询还会查询出平台类，平台类包括以java，sun，javax.swing，char[开头的全限定类名，前缀列表位于名为/resources/platform_names.txt的系统资源文件中，你可以通过在JAR文件中替换它来覆盖这个列表，或者在调用jhat时对需要替换的路径进行替换。

## 4.2 Class Query
类查询显示一个类的信息，包括父类，任何子类，实例数据成员，静态数据成员。可以从这个页面跳转到任何引用的类页面，也可以跳转到实例查询页面。

## 4.3 Object Query
在堆中查询一个对象的信息，会显示对象里面的类和对象里面的成员属性，可以看到引用当前对象的类，注意：对象查询还提供了对象分配点的堆栈回溯。

## 4.4 Instances Query
查询指定类下的所有实例，使用allInstances参数会显示指定类的所有子类的实例。

## 4.5 Roots Query
显示从根集到给定对象的引用链，它为可以访问给定对象的rootset的每个成员提供一个链，在计算这些链时，该工具执行深度优先搜索，以便提供最小长度的参考链。
这里有两种类型的Roots查询：一种排除弱引用的方法 (Roots)；包含弱引用(All Roots)。如果一个对象只被一个弱引用引用，那么它通常不会被认为是被保留的，因为垃圾收集器可以在它需要空间的时候收集它。注意：这可能是jhat中调试对象是否保留最有价值的查询，当找到一个被保留的对象时，这个查询会显示它被保留的原因。
![](https://img.hacpai.com/file/2019/10/image-5139b86c.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 4.6 Reachable Objects Query
此查询可从对象查询访问，并显示从给定对象可访问的所有对象的传递闭包，此列表按大小递减顺序排列，如果大小相同那么按字母顺序排列，最后，给出了所有可达对象的总大小，这对于确定对象在内存中的总运行时占用空间很有用，至少在具有简单对象拓扑的系统中是这样。
此查询在与-exclude命令行选项结合使用时最有价值。这很有用，例如，如果被分析的对象是一个可观察的对象。默认情况下，它的所有观察者都是可访问的，这将计入总大小。-exclude选项允许你去排除数据成员java.util.Observable.obs和java.util.Observable.arr。

## 4.7 Instance Counts for All Classes Query
显示系统中每个类的实例数，其中包含平台类，是根据实例数降序排序，查看所有类的实例计数，可能会发现许多类，因为实例比你预期的要多。然后你可以分析它们为什么被保留下来了。
![](https://img.hacpai.com/file/2019/10/image-904cc497.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

## 4.8 All Roots Query
这个查询显示了rootset的所有成员，包括弱引用。

## 4.9 New Instances Query
只有在使用两个堆dump调用jhat服务器时，新实例查询才可用，这个查询类似于Instances查询，只是它只显示新实例，如果实例位于第二个堆dump中，并且在基线堆dump中没有具有相同ID的相同类型的对象，则认为该实例是新的，对象的ID是一个32位或64位的整数，它惟一地标识对象。

## 4.10 Histogram Queries
可以显示图表。
![](https://img.hacpai.com/file/2019/10/image-aaf97821.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

# 5、自定义查询
可以通过Object Query Language(OQL)接口来开发自定义查询，点击Execute OQL Query按钮来进入执行OQL页面，OQL Help工具提供了内置函数，查询的示例如：
```java
select JavaScript-expression-to-select [from [instanceof] classname identifier [where JavaScript-boolean-expression-to-filter]]
select s from java.lang.String s where s.count >= 100

select JavaScript-expression-to-select [from [instanceof] classname identifier [where JavaScript-boolean-expression-to-filter]]
select s from java.lang.String s where s.count >= 100
```

![](https://img.hacpai.com/file/2019/10/image-7b01c308.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

# 6、堆分析提示
要从jhat获得有用的信息，通常需要了解应用程序及其使用的库和api，可以使用jhat来回答两个重要的问题:

## 6.1 是什么让一个对象存活？
当我们查看一个对象实例时，可以看到哪些对象被直接引用，更重要的是，可以使用Roots查询来确定从根集到给定对象的引用链，这些引用链显示从根对象到此对象的路径，通过这些链，可以很快地看到如何从根集访问对象。
如前所述，Roots查询排除弱引用，而All Roots查询会包括弱引用。弱引用是一个引用对象，它不会阻止被引用对象成为可终结的、最终确定的、然后被回收的。如果一个对象只被一个弱引用引用，那么垃圾收集器可以在它需要空间的时候收集它。
jhat工具根据根的类型对rootset引用链进行排序，顺序如下:
1. Java类的静态数据成员。
2. Java局部变量。
3. 本地静态值。
4. 本地局部变量。

## 6.2 这个对象分配到哪里？
当显示对象实例时，标题为“从对象分配”的部分以堆栈跟踪的形式显示分配站点。通过这种方式，可以看到对象是在哪里创建的。
注意:此分配站点信息仅在使用HPROF使用heap=all选项创建堆dump时可用。这个HPROF选项包括heap=dump选项和heap=sites选项。
如果不能判断单个对象dump是否存在泄露情况，另外一种方法是收集一系列dump文件，在每个dump文件中关注该对象的情况，jhat工具使用-baseline选项来支持此功能。
-baseline选项允许两个dump文件之间进行比较，前提是他们都是通过HPROF生成的，并且是相同的VM实例。如果相同的对象出现在两个dump文件中，那么它将被排除在新对象报告外面。其中一个dump文件作为baseline，那么在第二个dump文件中可以分析对象的情况。示例显示如何指定baseline：
`$ jhat -baseline snapshot.hprof#1 snapshot.hprof#2`

在上面的示例中，两个dump文件都是snapshot.hprof，但是区别是后面的#1和#2。
当使用两个堆dump启动jhat时，所有类查询的实例计数包括一个附加列，该列是该类型的新对象数量的计数。如果实例位于第二个堆dump中，并且baseline中没有具有相同ID的相同类型的对象，则认为该实例是新的。如果单击new count，则jhat将列出该类型的新对象。然后，对于每个实例，你可以查看分配它的位置，这些新对象引用哪些对象，以及哪些其他对象引用新对象。
通常，如果需要标识的对象是在连续dump之间创建的，那么-baseline选项非常有用。

# 7、示例
## 7.1 hello world
使用jmap -dump test.hprof命令，生成dump文件，使用jhat test.hprof命令分析刚才生成的dump文件。
![](https://img.hacpai.com/file/2019/10/image-428c2552.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)
看到上面的界面，表示jhat的java服务已经起来了，可以在浏览器上查看信息了，如下图：
![](https://img.hacpai.com/file/2019/10/image-bfbd894b.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)

默认显示所有的类信息。

## 7.2 baseline
使用命令：
`jhat -baseline test.hprof test2.hprof`
输出结果：
```java
Reading from test2.hprof...
Dump file created Sat Oct 12 17:31:12 CST 2019
Snapshot read, resolving...
Resolving 1024729 objects...
Chasing references, expect 204 dots...... 
Eliminating duplicate references............ 
Snapshot resolved.
Reading baseline snapshot...
Dump file created Sat Oct 12 17:31:06 CST 2019
Resolving 1016233 objects...
Discovering new objects...
Started HTTP server on port 7000
Server is ready.
```
![](https://img.hacpai.com/file/2019/10/image-2efe7f59.png?imageView2/2/w/768/format/jpg/interlace/1/q/100)
如图会出现new的标识。