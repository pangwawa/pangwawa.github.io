---
title: "Java并发编程 juc包——ForkJoin"
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-18T09:31:57+08:00
draft: false
---

## ForkJoin 特性

## 使用
提供两个抽象类继承ForkJoinTask ，分别是RecursiveAction和RecursiveTask.他们的区别在于：RecursiveTask有返回值，RecursiveAction无返回值。
ForkJoinPool pool = new ForkJoinPool();

ForkJoinPool的submit、invoke和execute的区别

execute 异步执行tasks，没有返回结果
submit会把任务对象本身返回，返回后我们可以通过get()获取方法执行结果，带Task返回值，可通过task.get 实现同步到主线程