---
title: "Java并发编程 juc包——线程池"
categories: ["技术博客"]
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-17T09:31:57+08:00
draft: false
---

## 线程池

### JUC Executors

通过JUC的ThreadPoolExecutor类创建各种类型线程池，包括了

#### Executors.newCachedThreadPool();
可缓存的线程池，可以进行缓存的线程池，意味着它的线程数是最大的，无限的。但是核心线程数为 0，这没关系。这里要考虑线程的摧毁，因为不能够无限的创建新的线程，所以在一定时间内要摧毁空闲的线程。
```
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }

    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>(),
                                      threadFactory);
    }
```
#### Executors.newFixedThreadPool();
传入一个固定的核心线程数，并且核心线程数等于最大线程数，而且它们的线程数存活时间都是无限的
```
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
 ```
 
 ```
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```
#### Executors.newScheduledThreadPool();
有计划性的线程池，就是在给定的延迟之后运行，或周期性地执行。
```
 public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    
    public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {
        return new ScheduledThreadPoolExecutor(corePoolSize, threadFactory);
    }
```

#### Executors.newSingleThreadExecutor();
单核心线程池，最大线程也只有一个，这里的时间为 0 意味着无限的生命，就不会被摧毁了。
```
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    

    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory));
    }
```

#### Executors.newWorkStealingPool(); (JDK 1.8 新增 用的是 ForkJoinPool 类)
是一个并行的线程池，参数中传入的是一个线程并发的数量
这个线程池不会保证任务的顺序执行，也就是 WorkStealing 的意思，抢占式的工作。

对应的源码实现
```
    public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```
```
 public static ExecutorService newWorkStealingPool() {
        return new ForkJoinPool
            (Runtime.getRuntime().availableProcessors(),
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
   }
```

## 自定义线程池

通过ThreadPoolExecutor类创建自定义线程池
```
   public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
添加ThreadFactory参数    
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             threadFactory, defaultHandler);
    }
    
添加拒绝策略
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
    }

    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

通过Spring提供的ThreadPoolTaskExecutor创建线程池

## 线程池处理线程的流程

查看核心线程是否空闲，有空闲，使用核心线程，没空闲，执行下一步；
查看线程任务队列，未满，将任务存储在任务队列，已满，执行下一步；
查看是否已满，即是否达到最大线程数，未达到，则创建线程，达到，则执行既定拒绝策略
## 线程池参数
corePoolSize、maximumPoolSize、workQueue

keepAliveTime，unit 、threadFactory、rejecthandler （线程策略）