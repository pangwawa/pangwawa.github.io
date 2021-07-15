---
title: "Java并发编程 juc包——AQS"
categories: ["技术博客"]
tags: ["并发编程", "Java"]
author: "Jack Wu"
date: 2019-02-06T09:31:57+08:00
draft: false
---
## AQS
AbstractQueuedSynchronizer,抽象的队列式同步器。是除了java自带的synchronized关键字之外的锁机制

使用Node实现FIFO队列
利用int类型表示状态
使用方法是继承
排它锁和共享锁	

核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并将共享资源设置为锁定状态，如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，
这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。
CLH（Craig，Landin，and Hagersten）队列是一个虚拟的双向队列，虚拟的双向队列即不存在队列实例，仅存在节点之间的关联关系。
AQS是将每一条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node），来实现锁的分配。

用大白话来说，AQS就是基于CLH队列，用volatile修饰共享变量state，线程通过CAS去改变状态符，成功则获取锁成功，失败则进入等待队列，等待被唤醒。

**注意：AQS是自旋锁：
**在等待唤醒的时候，经常会使用自旋（while(!cas())）的方式，不停地尝试获取锁，直到被其他线程获取成功

**实现了AQS的锁有：自旋锁、互斥锁、读锁写锁、条件产量、信号量、栅栏都是AQS的衍生物**

另外 AQS 还维护了一个很重要的变量exclusiveOwnerThread，它表示的是获得锁的线程，也叫独占线程。AQS中还有一个用来存储获取锁失败线程的队列，以及head 和 tail 结点

## AQS构建锁和其他同步组件
### CountDownLatch
（计数器，执行countDown方法后会减一，await()方法保证countDown减为0）
 CountDownLatch latch=new CountDownLatch(100);
latch.countDown();
latch.await();

同步辅助类，非常实用的多线程控制工具类，
CountDownLatch(int count) //实例化一个倒计数器，count指定计数个数
countDown() // 计数减一
await() //等待，当计数减到0时，所有线程并行执行

### Semaphore信号量
控制同时并发访问的个数
final Semaphore semaphore=new Semaphore(3);
semaphore.acquire(3);
semaphore.release(3);

semaphore.tryAcquire(3，……);尝试获取许可，能获取则执行，设置等待时间和获取个数


### CyclicBarrier
设置计数器等待其他符合数量条件线程就绪再继续执行之后的操作，可循环使用

### ReentrantLock与锁
可重入锁，JDK实现，对指定代码枷锁
与synchronized锁的区别

public class ReentrantLockExample {
    //请求总数
    private static int clientcount=5000;
    //并发数
    public static int threadTotal=200;
    //计数器
    public static int count=0;

   private final static Lock lock=new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService=Executors.newCachedThreadPool();
        final Semaphore semaphore=new Semaphore(threadTotal);
        final CountDownLatch countDownLatch=new CountDownLatch(clientcount);
        for (int i=0;i<clientcount;i++){
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();;

    }
    public static void add(){
        lock.lock();;
        try {
            count++;
        } finally {
            lock.unlock();
        }

    }

}

### Condition
### FutureTask

## AQS 定义的两种资源方式
1.Exclusive：独占，只有一个线程能执行，如ReentrantLock

2.Share：共享，多个线程可以同时执行，如Semaphore、CountDownLatch、ReadWriteLock，CyclicBarrier

## 自定义同步器
自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。
这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。
自定义同步器在实现的时候只需要实现共享资源state的获取和释放方式即可，至于具体线程等待队列的维护，AQS已经在顶层实现好了。自定义同步器实现的时候主要实现下面几种方法：
isHeldExclusively()：该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)：独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)：独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

ReentrantLock为例，（可重入独占式锁）：state初始化为0，表示未锁定状态，A线程lock()时，会调用tryAcquire()独占锁并将state+1.之后其他线程再想tryAcquire的时候就会失败，直到A线程unlock（）到state=0为止，其他线程才有机会获取该锁。A释放锁之前，自己也是可以重复获取此锁（state累加），这就是可重入的概念。
注意：获取多少次锁就要释放多少次锁，保证state是能回到零态的。

以CountDownLatch为例，任务分N个子线程去执行，state就初始化 为N，N个线程并行执行，每个线程执行完之后countDown（）一次，state就会CAS减一。当N子线程全部执行完毕，state=0，会unpark()主调用线程，主调用线程就会从await()函数返回，继续之后的动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。
　在acquire() acquireShared()两种方式下，线程在等待队列中都是忽略中断的，acquireInterruptibly()/acquireSharedInterruptibly()是支持响应中断的。

同步类在实现时一般都将自定义同步器（sync）定义为内部类，供自己使用；而同步类自己（Mutex）则实现某个接口，对外服务。