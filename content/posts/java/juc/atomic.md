---
title: "Java并发编程 juc包——Atomic包"
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-08T09:31:57+08:00
draft: false
---

## CAS

## unsafe

## Atomic包
Atomic的核心操作就是CAS（compareandset,利用CMPXCHG指令实现，它是一个原子指令）,该指令有三个操作数，变量的内存值V（value的缩写），变量的当前预期值E（exception的缩写），
变量想要更新的值U（update的缩写），当内存值和当前预期值相同时，将变量的更新值覆盖内存值

Atomic系列的类中的核心方法都会调用unsafe类中的几个本地方法。我们需要先知道一个东西就是Unsafe类，全名为：sun.misc.Unsafe，这个类包含了大量的对C代码的操作，
包括很多直接内存分配以及原子操作的调用，而它之所以标记为非安全的，是告诉你这个里面大量的方法调用都会存在安全隐患，需要小心使用，否则会导致严重的后果，例如在通过unsafe分配内存的时候，
如果自己指定某些区域可能会导致一些类似C++一样的指针越界到其他进程的问题。

Atomic包中的类按照操作的数据类型可以分成4组


### AtomicBoolean，AtomicInteger，AtomicLong
线程安全的基本类型的原子性操作

### AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
线程安全的数组类型的原子性操作，它操作的不是整个数组，而是数组中的单个元素

### AtomicLongFieldUpdater，AtomicIntegerFieldUpdater，AtomicReferenceFieldUpdater
基于反射原理对象中的基本类型（长整型、整型和引用类型）进行线程安全的操作

### AtomicReference，AtomicMarkableReference，AtomicStampedReference

## ABA问题的解决方法

使用AtomicMarkableReference，AtomicStampedReference。使用上述两个Atomic类进行操作。他们在实现compareAndSet指令的时候除了要比较当对象的前值和预期值以外，
还要比较当前（操作的）戳值和预期（操作的）戳值，当全部相同时，compareAndSet方法才能成功。每次更新成功，戳值都会发生变化，戳值的设置是由编程人员自己控制的。

## Atomic包的相关类

AtomicBoolean

Atomiclnteger

AtomiclntegerArray

AtomiclntegerFieldUpdater
原子更新整型的字段的更新器。
AtomicLong

AtomicLongArray

AtomicLongFieldUpdater

AtomicMarkableReference
原子更新带有标记位的引用类型。可以原子的更新一个布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference(V initialRef, boolean initialMark)

AtomicReference
原子更新引用类型。

AtomicReferenceArray
原子更新引用类型数组里的元素。
AtomicReferenceFieldUpdater
原子更新引用类型里的字段


AtomicStampedReference
原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于原子的更数据和数据的版本号，可以解决使用CAS进行原子更新时，可能出现的ABA问题。

DoubleAccumulator  （jdk1.8新增）
    
DoubleAdder （jdk1.8新增）

LongAccumulator （jdk1.8新增）

LongAdder （jdk1.8新增）

Striped64

## 除了atomic包提供的， Java的基本类型里还有char，float和double等 如何实现Atomic包的原子性
Atomic包里的类基本都是使用Unsafe实现的，让我们一起看下Unsafe的源码，发现Unsafe只提供了三种CAS方法，compareAndSwapObject，compareAndSwapInt和compareAndSwapLong，
再看AtomicBoolean源码，发现其是先把Boolean转换成整型，再使用compareAndSwapInt进行CAS，
所以原子更新double也可以用类似的思路来实现。