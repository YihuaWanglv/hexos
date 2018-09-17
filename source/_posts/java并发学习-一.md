---
title: java并发学习(一)
date: 2017-09-06 23:33:27
tags: [java, 并发, 多线程]
categories: java
---



# 1. 进程和线程

进程，是运行在自己地址空间内的自包容程序。

而java并发系统，会共享内存和IO这样的资源，因此需要协调不同线程驱动的任务之间对资源的使用。

线程机制是在执行程序表示的单一进程中创建任务。这样的好处是操作系统的透明性。

java线程机制，抢占式，调度机会周期性地中断程序，将上下文切换到另一个线程。

线程在设计上的好处是，简化了设计，同时具有松散耦合。

# 2. 基本线程机制

一个线程是进程内的一个单一的顺序控制流，单进程可以同时拥有多个并发任务，其底层是切分CPU时间。

- 定义任务： 继承Runnable，实现run方法。

- yield()线程让步。

- Thread类

- 使用Executor，可以管理一部任务的执行，而无需显式管理线程生命周期，是启动线程的优选方法。

- ExecutorService：

CachedThreadPool

FixedThreadPool：定义好有限的线程集，一次性预先执行代价高昂的线程分配，需要线程时从线程池中获取，省去了创建线程的开销。

SingleThreadPool：同时只有一条线程，顺序执行，序列化任务队列，可以省去贡献资源上的同步

- 返回值，Callable接口，call()方法，Future对象，get()和isDone()方法。

- 休眠：sleep()

- 如果必须控制任务执行的顺序，最好的方式就是使用同步控制。

- 优先级：getPriority(),setPriority(),set优先级是在run的开头部分设置，由于不同系统的有限级别难以一一适配，所以，调整优先级最好是只用这3种：MAX_PRIORITY,NORM_PRIORITY,MIN_PRIORITY

- 让步：yield(),

- 后台线程：
当所有非后台线程结束时，程序终止，并且杀死所有后台线程。
setDeamon(true);
通过编写定制的ThreadFactory可以定制由Executor创建的线程的属性（后台、优先、名称）

一个后台线程创建的任何线程，会自动设置为后台线程。
非后台的Executor通常是一种更好的方式，它控制所有任务可以同时被关闭。

- 实现Runnable接口和使用Thread类的区别在于，使用接口，你可以继承另一个不同的类。

- join(),在某个线程上调用，表示加入到某个线程，等待这个线程结束后，才开始执行。
可使用interrupt()中断。

- interrupt()时会抛出异常，然后就清除了interrupt的标记，这时调用isInterrupted()检查时就会是false。

- 捕获异常：
main主体放到try-catch语句中没有作用。

Thread.UncaughtExceptionHandler接口，允许你在每个Thread对象上都附上一个异常处理器。

Thread.UncaughtExceptionHandler.uncaughtExcetion()会在线程因未捕获的异常而临近死亡时被调用。

使用方法：

1，可以创建一个新的ThreadFactory，在每个新建的Thread对象上附上一个Thread.UncaughtExceptionHandler，然后将这个ThreadFactory传递给Executors创建新的ExecutorService。

2，如果知道要在代码中处处使用相同的异常处理，那么可以简单的在Thread类中设置一个静态域，并将这个处理器设置为默认的未捕获异常处理器。它只有在不存在线程专有的未捕获异常处理器时才被调用。



# 3. 共享受限资源

- 本质：基本上所有并发模式在解决线程冲突问题的时候，都是采用序列化访问共享资源的方案。

- 互斥量。
对于某个特定对象而言，其所有synchronized方法共享同一个锁，可防止多个任务同时访问被编码为对象内存。

synchronized static方法可以在类的范围内防止对static数据的并发访问。

- Brian同步规则：如果你正在写一个变量，它可能接下来将被另一个线程读取，或者正在读取一个上一次已经被另一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须用相同的监视器锁同步。

- 重点：如果一个类中有超过一个方法在处理临界数据，那么必须同步所有相关方法。每个访问临界共享资源的方法都必须被同步，否则它们就不会正确工作。

- 使用显式的Lock对象：ReentrantLock

Lock的好处是可以在finally子句中维护抛出异常后的清理，同时具有更细粒度的控制力。

- 平时用synchronized，需要特殊情况时使用Lock。

- 原子性：原子性可以用于除long和double之外的所有基本类型之上的“简单操作”。
为什么long和double不行？因为jvm可以将64位的读取和写入当做两个分离的32位操作来执行，这就产生了读写两个操作之间发生上下文切换的可能。

- volitle，用于定义变量，提供原子性。它提供了变量在内存中的可视性，即更改会刷新到内存中，可以让别的线程看到。

- 一般还是使用synchronized，只有在类中只有一个可变的域时，volitile才是安全的。

- 原子类：
Atomic
可以一定程度的去除一些别的同步方法，但同步锁通常更安全。

- 如何把一个不是线程安全的类变成线程安全？
可以创造一个对象的Manager类，Manager类持有该对象，并控制对它的一切访问，唯一的public方法是get()对象的方法，并且是synchronized的。

- 线程本地存储：ThreadLocal
根除线程间变量共享，线程隔离。

# 4. 终结任务

- cancle(),isCancled()

- 在阻塞时终结：

进入阻塞几种方式：

1，sleep

2，wait

3，等待某个输入输出完成

4，视图获得对象锁而还没有获取到

- 中断：interrupt()

- 安全离开线程run方法的方式：

1，calcled标志，cancle()

2，interrupt()

3，Executor.shotdownNow()

4，Executor,submit(),Future,cancle()

- 中断时要注意I/O的关闭等底层资源的关闭，因为不能中断正在试图获取synchronized锁或者I/O操作的线程。

- 一个任务应该能够调用在同一个对象中的其他synchronized方法，而这个任务已经持有锁了。

- interrupt()，可以中断被互斥锁所阻塞的调用，并且，interrupt()只能在任务处于阻塞时有效。

- 清除中断状态：如果不清除，那么它可能会提醒你2次。

- 清理策略：try-finally子句


# 5. 线程间协作

- wait(),notifyAll()

- 关键：wait()会释放锁，而sleep()和yield()则不会

- 要点：

1，wait(),notifyAll()，notify()是基于类Object的，而不是属于Thread的一部分。而且，只能在同步控制方法或同步控制块里调用wait(),notifyAll()，notify()。

2，必须用一个检查感兴趣的条件的while循环包围wait()，也即是说，不满足我条件，我就继续等的意思。



