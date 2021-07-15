---
title: "Java并发编程 juc包——并发容器"
categories: ["技术博客"]
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-10T09:31:57+08:00
draft: false
---


容器的线程安全
## 同步容器
同步容器可实现线程安全
ArrayList    Vector（使用synchronized实现线程安全） , Stack （使用synchronized实现线程安全）
HashMap    HashTable  （使用synchronized修饰，线程安全，key,value不能为空）
Collections.synchronizedXXX（如Set，Map）来实现线程安全

## 并发容器J.U.C 
AQS ——AbstractQueuedSynchronizedr，提供基于FIFO的队列，双向链表实现队列
Java.util.concurrent包的
ArrayList    CopyOnWriteArrayList
线程安全，1、消耗内存2、不能用于实时读场景，更适合读多写少的场景，读写分离，最终一致性，
HashSet TreeSet   CopyOnWriteArraySet ConcurrentSkipListSet

HashMap,TreeMap  ConcurrentHashMap（速度快，支持更快并发）, ConcurrentSkipListMap（key排序）


安全共享对象的策略
1、线程安全对象
2、被守护对象
3、线程限制
4、共享只读

AQS并发容器的同步器，AbstractQueuedSynchronizer
使用Node实现FIFO队列
利用int类型表示状态
使用方法是继承
排它锁和共享锁	

## HashMap的底层原理

## ConcurrentHashMap的底层原理

