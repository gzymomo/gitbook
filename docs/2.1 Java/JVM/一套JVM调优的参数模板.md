一般合理的配置以下参数就可以获得一个比较好的性能：

| -Xms2048m                                                    | 初始堆大小                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| -Xmx2048m                                                    | 最大堆大小                                                   |
| -Xmn1024m                                                    | 年轻代大小                                                   |
| -XX:PermSize=256m                                            | 设置持久代(perm gen)初始值(JDK8以后弃用改为-XX:MetaspaceSize) |
| -XX:MaxPermSize=256m                                         | 设置持久代最大值(JDK8以后弃用改为-XX:MaxMetaspaceSize)       |
| -Xss1m                                                       | 每个线程的堆栈大小(等价于-XX:ThreadStackSize)                |
| -XX:SurvivorRatio                                            | Eden区与Survivor区的大小比值（默认为8）                      |
| -XX:+DisableExplicitGC                                       | 关闭System.gc(),如果系统中使用了nio调用堆外内存，慎用此参数  |
| -XX:MaxTenuringThreshold=15                                  | 年轻代垃圾回收最大年龄，默认15，15次后进入老年代             |
| -XX:PretenureSizeThreshold                                   | 对象超过多大直接在老年代分配                                 |
| -XX:+UseParNewGC                                             | 使用ParNewGC垃圾回收器                                       |
| -XX:+UseConcMarkSweepGC                                      | 使用CMS内存收集                                              |
| -XX:CMSInitiatingOccupancyFraction=92                        | CMS垃圾收集器，当老年代达到92%时，触发CMS垃圾回收            |
| -XX:CMSFullGCsBeforeCompaction=0                             | 多少次后进行内存压缩，默认0                                  |
| -XX:+CMSScavengeBeforeRemark                                 | CMS重新标记之前尽量进行一次Young GC                          |
| -XX:+UseCMSCompactAtFullCollection                           | 在FULL GC的时候， 对老年代的压缩，默认打开                   |
| -XX:+CMSParallelInitialMarkEnabled                           | CMS初始标记并行化，增加初始标记速度                          |
| -Xverify:no                                                  | 禁止字节码校验，提高编译速度（jdk13之后作废，生产环境不使用） |
| -XX:+UseG1GC                                                 | 使用G1垃圾回收器                                             |
| -XX:MaxGCPauseMillis=200                                     | 目标GC暂停时间，尽可能目标                                   |
| -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Xloggc:d:\idea_gc.log | 合并使用，打印GC信息到指定文件                               |
| -XX:+HeapDumpOnOutOfMemoryError                              | 表示当JVM发生OOM时，自动生成DUMP文件                         |
| -XX:HeapDumpPath=${目录}                                     | 表示生成DUMP文件的路径，也可以指定文件名称，例如：-XX:HeapDumpPath=${目录}/java_heapdump.hprof。如果不指定文件名，默认为：java_<pid>_<date>_<time>_heapDump.hprof |

 