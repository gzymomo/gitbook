[TOC]

# 1、Java 8 中如何进行内存管理
将JVM内存分为两个不同的部分：堆（Heap）、 非堆（Non-Heap）。

JVM 内存 由 年轻代（Young Generation） 、老年代（Old Generation）组成。所有新创建的对象都位于年轻代中。当年轻代被填满时，执行次要垃圾收集（Minor GC）。更准确的说，这些位于年轻代的一部分对象成为 Eden Space。Minor GC将所有仍然使用的对象从 Eden Space 移动到 Survivor 0。对于Survivor 0 和 Survivor 1 空间执行相同的过程。在 GC 的许多循环中幸存的所有对象都被移动到老年代内存空间。从哪里移除对象是由 Major GC 负责的。

在运行 java -jar 命令时，可以使用以下参数设置 Java Heap 的内存限制：
- -Xms – JVM启动时的初始堆大小
- -Xmx – 最大堆大小
- -Xmn - 年轻代的大小，其余的空间是老年代
![](https://segmentfault.com/img/bVbGd56)

JVM内存的第二部分，它是Non-Heap。 Non-Heap 包括以下部分：
- **Thread Stacks** ：所有运行的线程的空间。可以使用 **-Xss** 参数设置最大线程大小。
- **Metaspace** ： 它替代了 PermGem（Java 7中是JVM堆的一部分）。在 Metaspace 中，通过应用程序加载所有类和方法。看看Spring Cloud 包含的包数量，我们不会在这里节省大量的内存。可以通过设置** -XX:MetaspaceSize** 和 **- XX:MaxMetaspaceSize** 参数来管理 Metaspace 大小。
- **Code Cache** ： 这是由 JIT（即时）编译器编译为本地代码的本机代码（如JNI）或 Java 方法的空间。最大大小设置  **-XX:ReservedCodeCacheSize** 参数。
- **Compressed Class Space **： 使用 **-XX：CompressedClassSpaceSize** 设置为压缩类空间保留的最大内存。
- **Direct NIO Buffers**

Heap 是用于对象，Non-Heap 是用于类。可以想像，当我们的应用程序 Non-Heap 大于 Heap 时，我们可以结束这种情况。

如果您在 Spring Boot 上启动具有内嵌 Tomcat 的 Eureka，这些配置是最低的值：
```java
-Xms16m \
-Xmx32m \
-XX:MaxMetaspaceSize=48m \
-XX:CompressedClassSpaceSize=8m \
-Xss256k \
-Xmn8m \
-XX:InitialCodeCacheSize=4m \
-XX:ReservedCodeCacheSize=8m \
-XX:MaxDirectMemorySize=16m
```

如果使用REST API 的微服务（带有 Feign 或 Ribbon），我们需要增加一些值：
```java
-Xms16m \
-Xmx48m \
-XX:MaxMetaspaceSize=64m \
-XX:CompressedClassSpaceSize=8m \
-Xss256k \
-Xmn8m \
-XX:InitialCodeCacheSize=4m \
-XX:ReservedCodeCacheSize=8m \
-XX:MaxDirectMemorySize=16m
```