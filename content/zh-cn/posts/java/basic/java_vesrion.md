---
title: "Java各个版本的特性"
categories: ["技术博客"]
tags: ["Java"]
author: "Jack Wu"
date: 2019-11-06T10:22:19+08:00
draft: false
---

## Java各个版本的特性

#### Java 5
1、泛型

2、增强for循环

3、自动装箱拆箱

4、枚举

5、可变参数

6、静态导入

7、自定义注解 关键字 @interface

#### Java 6
1、集合框架增强
为了更好的支持双向访问集合。添加了许多新的类和接口。
新的数组拷贝方法。Arrays.copyOf和Arrays.copyOfRange

2、为了更好的支持双向访问集合。添加了许多新的类和接口。
  新的数组拷贝方法。Arrays.copyOf和Arrays.copyOfRange
  
3、支持JDBC4.0规范

#### Java 7
Swing
    
    新增 JLayer 类，是一个灵活而且功能强大的Swing组件修饰器，使用方法：How to Decorate Components with JLayer.
    Nimbus Look and Feel 外观从 com.sun.java.swing 包移到 javax.swing 包中，详情：javax.swing.plaf.nimbus
    更轻松的重量级和轻量级组件的混合
    支持透明窗体以及非矩形窗体的图形界面，请看 How to Create Translucent and Shaped Windows
    JColorChooser 类新增 HSV tab.

网络
    
    新增 URLClassLoader.close 方法，请看 Closing a URLClassLoader.
    支持 Sockets Direct Protocol (SDP) 提供高性能网络连接，详情请看 Understanding the Sockets Direct Protocol.
    
集合
    
    新增 TransferQueue 接口，是 BlockingQueue 的改进版，实现类为 LinkedTransferQueue
    
RIA/发布

    拖拽的小程序使用一个默认或者定制的标题进行修饰
    
XML

    包含 Java API for XML Processing (JAXP) 1.4.5, 支持 Java Architecture for XML Binding (JAXB) 2.2.3, 和 Java API for XML Web Services (JAX-WS) 2.2.4.
    
java.lang 包

    消除了在多线程环境下的非层次话类加载时导致的潜在死锁，详情：Multithreaded Custom Class Loaders in Java SE 7.

Java 虚拟机

    支持非 Java 语言: Java SE 7 引入一个新的 JVM 指令用于简化实现动态类型编程语言
    Garbage-First Collector 是一个服务器端的垃圾收集器用于替换 Concurrent Mark-Sweep Collector (CMS).
    提升了 Java HotSpot 虚拟机的性能
    
Java I/O
    
    java.nio.file 包以及相关的包 java.nio.file.attribute 提供对文件 I/O 以及访问文件系统的全面支持，请看 File I/O (featuring NIO.2).
    目录 /sample/nio/chatserver/ 包含使用 java.nio.file 包的演示程序
    目录 /demo/nio/zipfs/ 包含 NIO.2 NFS 文件系统的演示程序
    
安全性
    
    新的内置对多个基于 ECC 算法(ECDSA/ECDH)的支持，详情请看：Sun PKCS#11 Provider's Supported Algorithms in Java PKCS#11 Reference Guide.
    禁用了一些弱加密算法，详情请看 Appendix D: Disabling Cryptographic Algorithms in Java PKI Programmer's Guide and Disabled Cryptographic Algorithms in Java Secure Socket Extension (JSSE) Reference Guide.
    Java 安全套接字扩展中对 SSL/TLS 的增强
    
并发
    
    fork/join 框架，基于 ForkJoinPool 类，是 Executor 接口的实现，设计它用来进行高效的运行大量任务；使用 work-stealing 技术用来保证大量的 worker 线程工作，特别适合多处理器环境，详情请看 Fork/Join目录/sample/forkjoin/ 包含了 fork/join 框架的演示程序
    ThreadLocalRandom 类class 消除了使用伪随机码线程的竞争，请看 Concurrent Random Numbers.
    Phaser 类是一个新的同步的屏障，与 CyclicBarrier 类似.Java 2D
    一个新的基于 XRender 的 Java 2D 渲染管道支持现在的 X11 桌面，改善了图形性能，请看 System Properties for Java 2D Technology 中的 xrender .
    JDK 可枚举并显示出已安装的 OpenType/CFF 字体，通过 GraphicsEnvironment.getAvailableFontFamilyNames 方法 See Selecting a Font.
    TextLayout 类支持西藏语脚本
    libfontconfig, 是一个字体配置 api ，see Fontconfig.
    
国际化

    支持 Unicode 6.0.0目录 /demo/jfc/Font2DTest/ 包含 Unicode 6.0 的演示程序Java SE 7 可容纳在 ISO 4217 中新的货币，详情请看 Currency 类.
    
Java 编程语言特性
    
    二进制数字表达方式
    使用下划线对数字进行分隔表达，例如 1_322_222
    switch 语句支持字符串变量
    泛型实例创建的类型推断
    使用可变参数时，提升编译器的警告和错误信息
    try-with-resources 语句
    同时捕获多个异常处理
    
JDBC 4.1
    支持使用 try-with-resources 语句进行自动的资源释放，包括连接、语句和结果集
支持 RowSet 1.1

#### Java 8 

1、lambada表达式(Lambda Expressions)。Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中)。

2、方法引用（Method references）。方法引用提供了非常有用的语法，可以直接引用已有Java类或对象（实例）的方法或构造器。与lambda联合使用，可以使语言的构造更紧凑简洁，减少冗余代码。

3、默认方法（Default methods）。默认方法允许将新功能添加到库的接口中，并确保兼容实现老版本接口的旧有代码。

4、重复注解（Repeating Annotations）。重复注解提供了在同一声明或类型中多次应用相同注解类型的能力。

5、类型注解（Type Annotation）。在任何地方都能使用注解，而不是在声明的地方。

6、类型推断增强。

7、方法参数反射（Method Parameter Reflection）。

8、Stream API 。新添加的Stream API（java.util.stream） 把真正的函数式编程风格引入到Java中。Stream API集成到了Collections API里。

9、HashMap改进，在键值哈希冲突时能有更好表现。

10、Date Time API。加强对日期和时间的处理。LocateDateTime

11、java.util 包下的改进，提供了几个实用的工具类。

并行数组排序。
标准的Base64编解码。
支持无符号运算。

12.java.util.concurrent 包下增加了新的类和方法。

    java.util.concurrent.ConcurrentHashMap 类添加了新的方法以支持新的StreamApi和lambada表达式。
    java.util.concurrent.atomic 包下新增了类以支持可伸缩可更新的变量。
    java.util.concurrent.ForkJoinPool类新增了方法以支持 common pool。
    新增了java.util.concurrent.locks.StampedLock类，为控制读/写访问提供了一个基于性能的锁，且有三种模式可供选择。  
    
13.HotSpot

删除了 永久代（PermGen）.
方法调用的字节码指令支持默认方法。

使用Metaspace（JEP 122）代替持久代（PermGen space）。在JVM参数方面，使用-XX:MetaSpaceSize和-XX:MaxMetaspaceSize代替原来的-XX:PermSize和-XX:MaxPermSize

#### Java 9 新特性
1、模块系统  JPMS

2、G1 成为默认垃圾回收器

3、HTTP/2 Client

4、JShell

5、集合相关，更便捷的方法

6、Process API Updates

7、Stack-Walking API 

8、Variable Handles

9、Docker方面支持

#### Java 10 新特性
1、var 局部变量类型推断。

2、统一的 GC 接口

3、 应用程序类数据（AppCDS）共享

4. JEP314，使用附加的 Unicode 语言标记扩展。

#### Java 11 新特性

局部类型推断

集合、字符串新API

HTTP API 

Epsilon垃圾收集器
JDK上对这个特性的描述是：开发一个处理内存分配但不实现任何实际内存回收机制的GC，一旦可用堆内存用完，JVM就会退出。
    
    性能测试(它可以帮助过滤掉GC引起的性能假象)
    内存压力测试
    非常短的JOB任务
    VM接口测试

ZGC 垃圾回收器

    GC暂停时间不会超过10毫秒
    既能处理几百兆的小堆，也能处理几个T的大堆
    和G1相比，应用吞吐能力不会下降超过15%
    为未来的GC功能和利用colord指针以及Load barriers优化奠定了基础
 
ZGC是一个并发、基于region、压缩型的垃圾收集器，只有root扫描阶段会STW(strop the world，停止所有线程)，因此ZGC的停顿时间不会随着堆的增长和存活对象的增长而变长。
用法：-XX:UnlockExperimentalVMOptions -XX:+UseZGC
虽然功能如此强大，但很遗憾的是，在Windows系统的JDK中并没有提供ZGC

Flight Recorder

这是一个记录仪，用于诊断程序运行过程，那么在之前这是一个商业版的特性，是要收费的，从Java11开始，Fight Recorder免费提供使用并开源。它可以导出事件到文件中，
之后可以用Java Mission Control来分析，也可以在应用启动时配置java -XX:StartFlightRecording或者在应用启动之后使用jcmd来录制，

#### Java 12 新特性
1、switch表达式

2 默认CDS归档 

3 Shenandoah GC

Shenandoah是一种垃圾收集（GC）算法，旨在保证低延迟（10 - 500 ms的下限）。 它通过在运行Java工作线程的同时执行GC操作减少GC暂停时间。 
使用Shenandoah，暂停时间不依赖于堆的大小。 这意味着无论堆的大小如何，暂停时间都是差不多的。
这是一个实验性功能，不包含在默认（Oracle）的OpenJDK版本中。

4 JMH 基准测试

此功能为JDK源代码添加了一套微基准测试（大约100个），简化了现有微基准测试的运行和新基准测试的创建过程。 它基于Java Microbenchmark Harness（JMH）并支持JMH更新。

此功能使开发人员可以轻松运行当前的微基准测试并为JDK源代码添加新的微基准测试。 可以基于Java Microbenchmark Harness（JMH）轻松测试JDK性能。 它将支持JMH更新，并在套件中包含一组（约100个）基准测试。

5 JVM 常量 API

JEP 334引入了一个API，用于建模关键类文件和运行时artifacts，例如常量池。 此API将包括ClassDesc，MethodTypeDesc，MethodHandleDesc和DynamicConstantDesc等类。此 API 对于操作类和方法的工具很有帮助。

6 G1的可中断 mixed GC

此功能通过将Mixed GC集拆分为强制部分和可选部分，使G1垃圾收集器更有效地中止垃圾收集过程。通过允许垃圾收集过程优先处理强制集，g1可以更多满足满足暂停时间目标。

G1是一个垃圾收集器，设计用于具有大量内存的多处理器机器。由于它提高了性能效率，g1垃圾收集器最终将取代cms垃圾收集器。

G1垃圾收集器的主要目标之一是满足用户设置的暂停时间。G1采用一个分析引擎来选择在收集期间要处理的工作量。此选择过程的结果是一组称为GC集的区域。一旦GC集建立并且GC已经开始，那么G1就无法停止。

如果G1发现GC集选择选择了错误的区域，它会将GC区域的拆分为两部分（强制部分和可选部分）来切换到处理Mix GC的增量模式。如果未达到暂停时间目标，则停止对可选部分的垃圾收集。

7 G1归还不使用的内存

此功能的主要目标是改进G1垃圾收集器，以便在不活动时将Java堆内存归还给操作系统。 为实现此目标，G1将在低应用程序活动期间定期生成或持续循环检查完整的Java堆使用情况。

这将立即归还未使用的部分Java堆内存给操作系统。 用户可以选择执行FULL GC以最大化返回的内存量。

8 移除多余ARM64实现

Java 12将只有一个ARM 64位实现（aarch64）。 目标是删除所有与arm64实现相关的代码，同时保留32位ARM端口和64位aarch64实现。

这将把重点转移到单个64位ARM实现，并消除维护两个实现所需的重复工作。 当前的JDK 11实现中有两个64位ARM实现。