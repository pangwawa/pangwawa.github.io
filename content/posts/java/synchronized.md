---
title: "Java并发编程——synchronized关键字"
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-06T09:31:57+08:00
draft: false
---

## CAS
CAS : compare and swap 或者 compare and exchange 比较交换。
当我们需要对内存中的数据进行修改操作时，为了避免多线程并发修改的情况，我们在对他进行修改操作前，先读取他原来的值E，然后进行计算得出新的的值V，在修改前去比较当前内存中的值N是否和我之前读到的E相同，
如果相同，认为其他线程没有修改过内存中的值，如果不同，说明被其他线程修改了，这时，要继续循环去获取最新的值E，再进行计算和比较，直到我们预期的值和当前内存中的值相等时，再对数据执行修改操作。

## synchronize 
synchronize具有原子性、有序性、可见性和可重入性
    有序性： synchronized和volatile都具有有序性
    有序性： synchronized和volatile都具有可见性
    原子性： volatile不具备原子性

synchronize 锁的实现
synchronized有两种形式上锁，一个是对方法上锁，一个是构造同步代码块。他们的底层实现其实都一样，在进入同步代码之前先获取锁，获取到锁之后锁的计数器+1，
同步代码执行完锁的计数器-1，如果获取失败就阻塞式等待锁的释放


在JVM中，对象是分成三部分存在的：对象头、实例数据、对其填充

对象头是我们需要关注的重点，它是synchronized实现锁的基础，因为synchronized申请锁、上锁、释放锁都与对象头有关。对象头主要结构是由Mark Word 和 Class Metadata Address组成，
其中Mark Word存储对象的hashCode、锁信息或分代年龄或GC标志等信息，Class Metadata Address是类型指针指向对象的类元数据，JVM通过该指针确定该对象是哪个类的实例。

锁也分不同状态，JDK6之前只有两个状态：无锁、有锁（重量级锁），而在JDK6之后对synchronized进行了优化，新增了两种状态，总共就是四个状态：无锁状态、偏向锁、轻量级锁、重量级锁，
其中无锁就是一种状态了。锁的类型和状态在对象头Mark Word中都有记录，在申请锁、锁升级等过程中JVM都需要读取对象的Mark Word数据。

每一个锁都对应一个monitor对象，在HotSpot虚拟机中它是由ObjectMonitor实现的（C++实现）。每个对象都存在着一个monitor与之关联，对象与其monitor之间的关系有存在多种实现方式，
如monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个monitor被某个线程持有后，它便处于锁定状态。

在jdk6的时候，新增了两个锁状态，通过锁消除、锁粗化、自旋锁等方法使用各种场景，给synchronized性能带来了很大的提升。

## 锁状态
无锁状态、偏向锁（单线程多次申请同一锁对象）、轻量级锁（第二个线程申请同一锁对象**交替进行而不是同时进行**）、重量级锁（同一时间多个线程申请，）

锁膨胀

上面讲到锁有四种状态，并且会因实际情况进行膨胀升级，其膨胀方向是：无锁——>偏向锁——>轻量级锁——>重量级锁，并且膨胀方向不可逆。

偏向锁

一句话总结它的作用：减少统一线程获取锁的代价。在大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得，那么此时就是偏向锁。

    核心思想：
    
    如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也就变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，
    即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作。
    
    获取过程
    
    当一个线程访问同步块时，会先判断锁标志位是否为01，如果是01，则判断是否为偏向锁，如果是，会先判断当前锁对象头中是否存储了当前的线程id，如果存储了，则直接获得锁。
    如果对象头中指向不是当前线程id，则通过CAS尝试将自己的线程id存储进当前锁对象的对象头中来获取偏向锁。当cas尝试获取偏向锁成功后则继续执行同步代码块，否则等待安全点的到来撤销原来线程的偏向锁，
    撤销时需要暂停原持有偏向锁的线程，判断线程是否活动状态，如果已经退出同步代码块则唤醒新的线程开始获取偏向锁，否则开始锁竞争进行锁升级过程，升级为轻量级锁。

轻量级锁

轻量级锁是由偏向锁升级而来，当存在第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。注意这里的第二个线程只是申请锁，不存在两个线程同时竞争锁，可以是一前一后地交替执行同步块。

锁消除
JVM对运行上下文进行扫描，去除不可能存在竞争的锁

锁粗化
锁粗化是虚拟机对另一种极端情况的优化处理，通过扩大锁的范围，避免反复加锁和释放锁。比如下面method3经过锁粗化优化之后就和method4执行效率一样了

自旋锁与自适应锁

轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。

自旋锁：许多情况下，共享数据的锁定状态持续时间较短，切换线程不值得，通过让线程执行循环等待锁的释放，不让出CPU。如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式。但是它也存在缺点：如果锁被其他线程长时间占用，一直不释放CPU，会带来许多的性能开销。

自适应自旋锁：这种相当于是对上面自旋锁优化方式的进一步优化，它的自旋的次数不再固定，其自旋的次数由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定，这就解决了自旋锁带来的缺点。

旋锁或自适应自旋锁：
因为线程阻塞后进入排队队列和唤醒都需要CPU从用户态转为核心态，尤其频繁的阻塞和唤醒对CPU来说是负荷很重的工作。同时统计发现，很多对象锁的锁定状态只会持续很短的一段时间，例如一个线程切换周期，这样的话在很短的时间内阻塞线程又很快唤醒线程显然不值得，所以引入了自旋锁概念。

所谓“自旋”，就monitor并不把线程阻塞放入排队队列，而是去执行一个无意义的循环，循环结束后看看是否锁已释放并直接进行竞争上岗步骤，如果竞争不到继续自旋循环，循环过程中线程的状态一直处于running状态。明显自旋锁使得synchronized的对象锁方式在线程之间引入了不公平。但是这样可以保证大吞吐率和执行效率。

不过虽然自旋锁方式省去了阻塞线程的时间和空间（队列的维护等）开销，但是长时间自旋也是很低效的。所以自旋的次数一般控制在一个范围内，例如10,50等，在超出这个范围后，线程就进入排队队列。

自适应自旋锁，就是自旋的次数是通过JVM在运行时收集的统计信息，动态调整自旋锁的自旋次数上界。

只有一个线程进入临界区 -------偏向锁
多个线程交替进入临界区--------轻量级锁
多个线程同时进入临界区-------重量级锁

## synchronized锁升级原理:
在锁对象的对象头里面有一个threadid字段，在第一次访问的时候threadid为空，jvm让其持有偏向锁，并将threadid设置为其线程id，再次进入的时候会先判断threadid是否与其线程id一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁，通过自旋循环一定次数来获取锁，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了synchronized锁的升级。

synchronized用的锁是存在java对象头里的。
其中最后两位代表是否加锁的标志位。锁标志位如果是01的话需要根据前一位的是否为偏向锁来判断当前的锁状态，如果前一位为0则代表无锁状态，如果为1则代表有偏向锁。
后两位：00代表轻量级锁，10代表重量级锁，11代表GC垃圾回收的标记信息。
## 锁的升级的目的:
锁升级是为了减低了锁带来的性能消耗。在Java 6之后优化synchronized的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。

## ABA 问题
其他线程修改后的值和原来的值相同，（设置一个状态标志就行）