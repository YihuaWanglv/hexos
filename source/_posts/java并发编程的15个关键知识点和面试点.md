---
title: java并发编程的15个关键知识点和面试点
date: 2018-09-10 20:34:50
tags: [并发, 面试, java并发编程, 多线程]
---



# <java并发编程关键知识点/面试点>


15个常见的面试问题和关键点，我列举了出来，并且就每个问题罗列出来分析解答。其中不少资料是来自英文资料，内容非常的丰富详细，其中每一点都能单独成长文，适合慢慢看。





## 1. 如何保证多线程的顺序执行（T2在T1后执行，T3在T2后执行）？

 - 可以用 Thread 类的 join 方法实现这一效果。

## 2. Lock接口，相对于同步代码块（synchronized block）有什么优势？

- 多线程和并发编程中使用 lock 接口的最大优势是它为读和写提供两个单独的锁，可以让你构建高性能数据结构，比如 ConcurrentHashMap 和条件阻塞。


### 2.1 如果让你实现一个高性能缓存，支持并发读取和单一写入，你如何保证数据完整性。

- 由于Lock具有写锁和读锁，读锁可以实现不互斥的读操作，比同步代码块具有更好的并发性能，所以可以用Lock实现高性能缓存。
- readLock(),支持可重入读，可以共享读锁，支持多线程的并发读取；
- writeLock()，互斥的写入，单一写入。
- 代码例如如下(思路：java.util.concurrent.locks包下面ReadWriteLock接口,该接口下面的实现类ReentrantReadWriteLock维护了两个锁读锁和解锁，可用该类实现这个功能)：
```
import java.util.Date;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 你需要实现一个高效的缓存，它允许多个用户读，但只允许一个用户写，以此来保持它的完整性，你会怎样去实现它？
 * @author user
 *
 */
public class Test2 {

    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                
                @Override
                public void run() {
                    MyData.read();
                }
            }).start();
        }
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                
                @Override
                public void run() {
                    MyData.write("a");
                }
            }).start();
        }
    }
}

class MyData{
    //数据
    private static String data = "0";
    //读写锁
    private static ReadWriteLock rw = new ReentrantReadWriteLock();
    //读数据
    public static void read(){
        rw.readLock().lock();
        System.out.println(Thread.currentThread()+"读取一次数据："+data+"时间："+new Date());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rw.readLock().unlock();
        }
    }
    //写数据
    public static void write(String data){
        rw.writeLock().lock();
        System.out.println(Thread.currentThread()+"对数据进行修改一次："+data+"时间："+new Date());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rw.writeLock().unlock();
        }
    }
}
```

## 3. Java 中 wait 和 sleep 方法有什么区别？

- （扩展阅读：http://www.java67.com/2015/12/producer-consumer-solution-using-blocking-queue-java.html）
- 两者主要的区别就是等待释放锁和监视器。sleep方法在等待时不会释放任何锁或监视器。wait 方法多用于线程间通信，而 sleep 只是在执行时暂停。
- wait可以被notify/notifyAll唤醒，而sleep则不能。
- wait is called from synchronized context only while sleep can be called without synchronized block.
```
 In order to call the wait (), notify () or notifyAll () methods in Java, we must have obtained the lock for the object on which we're calling the method.

 we call wait (), notify () or notifyAll method in Java from synchronized method or synchronized block in Java to avoid:

1) IllegalMonitorStateException in Java which will occur if we don't call wait (), notify () or notifyAll () method from synchronized context.

2) Any potential race condition between wait and notify method in Java.

Read more: https://javarevisited.blogspot.com/2011/05/wait-notify-and-notifyall-in-java.html#ixzz5Q61F8eI1
```
- The wait() method  is called on an Object on which the synchronized block is locked, while sleep is called on the Thread.

## 4. 如何在 Java 中实现一个阻塞队列？

### 1.wait()和notify()方式

阻塞队列与普通队列的区别在于，当队列是空的时，从队列中获取元素的操作将会被阻塞，或者当队列是满时，往队列里添加元素的操作会被阻塞。试图从空的阻塞队列中获取元素的线程将会被阻塞，直到其他的线程往空的队列插入新的元素。同样，试图往已满的阻塞队列中添加新元素的线程同样也会被阻塞，直到其他的线程使队列重新变得空闲起来，如从队列中移除一个或者多个元素，或者完全清空队列。

线程1往阻塞队列中添加元素，而线程2从阻塞队列中移除元素。

具体实现:
```
public class BlockingQueue {
 
  private List queue = new LinkedList();
  private int  limit = 10;
 
  public BlockingQueue(int limit){
    this.limit = limit;
  }
 
 
  public synchronized void enqueue(Object item)
  throws InterruptedException  {
    while(this.queue.size() == this.limit) {
      wait();
    }
    if(this.queue.size() == 0) {
      notifyAll();
    }
    this.queue.add(item);
  }
 
 
  public synchronized Object dequeue()
  throws InterruptedException{
    while(this.queue.size() == 0){
      wait();
    }
    if(this.queue.size() == this.limit){
      notifyAll();
    }
 
    return this.queue.remove(0);
  }
 
}
```
必须注意到，在enqueue和dequeue方法内部，只有队列的大小等于上限（limit）或者下限（0）时，才调用notifyAll方法。如果队列的大小既不等于上限，也不等于下限，任何线程调用enqueue或者dequeue方法时，都不会阻塞，都能够正常的往队列中添加或者移除元素。


### 2.并发类实现。eg:使用ArrayBlockingQueue实现

```
public class BlockingQueueTest {
 
    public static void main(String[] args) {
        final BlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(3); //缓冲区允许放3个数据
 
        for(int i = 0; i < 2; i ++) {
            new Thread() { //开启两个线程不停的往缓冲区存数据
 
                @Override
                public void run() {
                    while(true) {
                        try {
                            Thread.sleep((long) (Math.random()*1000));
                            System.out.println(Thread.currentThread().getName() + "准备放数据"
                                    + (queue.size() == 3?"..队列已满，正在等待":"..."));
                            queue.put(1);
                            System.out.println(Thread.currentThread().getName() + "存入数据，" 
                                    + "队列目前有" + queue.size() + "个数据");
                        } catch (InterruptedException e) {
                            // TODO Auto-generated catch block
                            e.printStackTrace();
                        } 
                    }
                }
 
            }.start();
        }
 
        new Thread() { //开启一个线程不停的从缓冲区取数据
 
            @Override
            public void run() {
                while(true) {
                    try {
                        Thread.sleep(1000);
                        System.out.println(Thread.currentThread().getName() + "准备取数据"
                                + (queue.size() == 0?"..队列已空，正在等待":"..."));
                        queue.take();
                        System.out.println(Thread.currentThread().getName() + "取出数据，" 
                                + "队列目前有" + queue.size() + "个数据");
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    } 
                }
            }
        }.start();
    }
 
}
```

## 5. 如何在 Java 中编写代码解决生产者消费者问题？

### 5.1 用 Java 中 BlockingQueue 的解决方案（由于代码简单，并且不需要处理同步代码块，这个方案是最推荐的方式）
- why BlockingQueue？ blocking queue data structure not only provides storage but also provides flow control and thread-safety, which makes the code really simple. 

- Producer-Consumer Example in Java using BlockingQueue

Here is our sample Java program to solve the classical producer consumer problem using BlockingQueue in Java:
```
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * Producer Consumer Problem solution using BlockingQueue in Java.
 * BlockingQueue not only provide a data structure to store data
 * but also gives you flow control, require for inter thread communication.
 * 
 * @author Javin Paul
 */
public class ProducerConsumerSolution {

    public static void main(String[] args) {
        BlockingQueue<Integer> sharedQ = new LinkedBlockingQueue<Integer>();
        
        Producer p = new Producer(sharedQ);
        Consumer c = new Consumer(sharedQ);
        
        p.start();
        c.start();
    }
}

class Producer extends Thread {
    private BlockingQueue<Integer> sharedQueue;

    public Producer(BlockingQueue<Integer> aQueue) {
        super("PRODUCER");
        this.sharedQueue = aQueue;
    }

    public void run() {
        // no synchronization needed
        for (int i = 0; i < 10; i++) {
            try {
                System.out.println(getName() + " produced " + i);
                sharedQueue.put(i);
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}

class Consumer extends Thread {
    private BlockingQueue<Integer> sharedQueue;

    public Consumer(BlockingQueue<Integer> aQueue) {
        super("CONSUMER");
        this.sharedQueue = aQueue;
    }

    public void run() {
        try {
            while (true) {
                Integer item = sharedQueue.take();
                System.out.println(getName() + " consumed " + item);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

Output
PRODUCER produced 0
CONSUMER consumed 0
PRODUCER produced 1
CONSUMER consumed 1
PRODUCER produced 2
CONSUMER consumed 2
PRODUCER produced 3
CONSUMER consumed 3
PRODUCER produced 4
CONSUMER consumed 4
PRODUCER produced 5
CONSUMER consumed 5
PRODUCER produced 6
CONSUMER consumed 6
PRODUCER produced 7
CONSUMER consumed 7
PRODUCER produced 8
CONSUMER consumed 8
PRODUCER produced 9
CONSUMER consumed 9

```
Read more: http://www.java67.com/2015/12/producer-consumer-solution-using-blocking-queue-java.html#ixzz5Q65io9os


### 5.2 用wait和nofity的方式

- Java program to solve Producer Consumer Problem in Java

How to solve Producer Consumer Problem in Java with ExampleHere is complete Java program to solve producer consumer problem in Java programming language. In this program we have used wait and notify method from java.lang.Object class instead of using BlockingQueue for flow control.

```
import java.util.Vector;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * Java program to solve Producer Consumer problem using wait and notify
 * method in Java. Producer Consumer is also a popular concurrency design pattern.
 *
 * @author Javin Paul
 */
public class ProducerConsumerSolution {

    public static void main(String args[]) {
        Vector sharedQueue = new Vector();
        int size = 4;
        Thread prodThread = new Thread(new Producer(sharedQueue, size), "Producer");
        Thread consThread = new Thread(new Consumer(sharedQueue, size), "Consumer");
        prodThread.start();
        consThread.start();
    }
}

class Producer implements Runnable {

    private final Vector sharedQueue;
    private final int SIZE;

    public Producer(Vector sharedQueue, int size) {
        this.sharedQueue = sharedQueue;
        this.SIZE = size;
    }

    @Override
    public void run() {
        for (int i = 0; i < 7; i++) {
            System.out.println("Produced: " + i);
            try {
                produce(i);
            } catch (InterruptedException ex) {
                Logger.getLogger(Producer.class.getName()).log(Level.SEVERE, null, ex);
            }

        }
    }

    private void produce(int i) throws InterruptedException {

        //wait if queue is full
        while (sharedQueue.size() == SIZE) {
            synchronized (sharedQueue) {
                System.out.println("Queue is full " + Thread.currentThread().getName()
                                    + " is waiting , size: " + sharedQueue.size());

                sharedQueue.wait();
            }
        }

        //producing element and notify consumers
        synchronized (sharedQueue) {
            sharedQueue.add(i);
            sharedQueue.notifyAll();
        }
    }
}

class Consumer implements Runnable {

    private final Vector sharedQueue;
    private final int SIZE;

    public Consumer(Vector sharedQueue, int size) {
        this.sharedQueue = sharedQueue;
        this.SIZE = size;
    }

    @Override
    public void run() {
        while (true) {
            try {
                System.out.println("Consumed: " + consume());
                Thread.sleep(50);
            } catch (InterruptedException ex) {
                Logger.getLogger(Consumer.class.getName()).log(Level.SEVERE, null, ex);
            }

        }
    }

    private int consume() throws InterruptedException {
        //wait if queue is empty
        while (sharedQueue.isEmpty()) {
            synchronized (sharedQueue) {
                System.out.println("Queue is empty " + Thread.currentThread().getName()
                                    + " is waiting , size: " + sharedQueue.size());

                sharedQueue.wait();
            }
        }

        //Otherwise consume element and notify waiting producer
        synchronized (sharedQueue) {
            sharedQueue.notifyAll();
            return (Integer) sharedQueue.remove(0);
        }
    }
}

Output:
Produced: 0
Queue is empty Consumer is waiting , size: 0
Produced: 1
Consumed: 0
Produced: 2
Produced: 3
Produced: 4
Produced: 5
Queue is full Producer is waiting , size: 4
Consumed: 1
Produced: 6
Queue is full Producer is waiting , size: 4
Consumed: 2
Consumed: 3
Consumed: 4
Consumed: 5
Consumed: 6
Queue is empty Consumer is waiting , size: 0
```

Read more: http://www.java67.com/2012/12/producer-consumer-problem-with-wait-and-notify-example.html#ixzz5Q68twJon


### 5.3 Counting Semaphore Example in Java (Binary Semaphore)

Java 5 Semaphore Example codea Counting semaphore with one permit is known as binary semaphore because it has only two state permit available or permit unavailable. Binary semaphore can be used to implement mutual exclusion or critical section where only one thread is allowed to execute. Thread will wait on acquire() until Thread inside critical section release permit by calling release() on semaphore.


here is a simple example of counting semaphore in Java where we are using binary semaphore to provide mutual exclusive access on critical section of code in java:
```
import java.util.concurrent.Semaphore;

public class SemaphoreTest {

    Semaphore binary = new Semaphore(1);
  
    public static void main(String args[]) {
        final SemaphoreTest test = new SemaphoreTest();
        new Thread(){
            @Override
            public void run(){
              test.mutualExclusion(); 
            }
        }.start();
      
        new Thread(){
            @Override
            public void run(){
              test.mutualExclusion(); 
            }
        }.start();
      
    }
  
    private void mutualExclusion() {
        try {
            binary.acquire();

            //mutual exclusive region
            System.out.println(Thread.currentThread().getName() + " inside mutual exclusive region");
            Thread.sleep(1000);

        } catch (InterruptedException i.e.) {
            ie.printStackTrace();
        } finally {
            binary.release();
            System.out.println(Thread.currentThread().getName() + " outside of mutual exclusive region");
        }
    } 
  
}

Output:
Thread-0 inside mutual exclusive region
Thread-0 outside of mutual exclusive region
Thread-1 inside mutual exclusive region
Thread-1 outside of mutual exclusive region
```

Read more: https://javarevisited.blogspot.com/2012/05/counting-semaphore-example-in-java-5.html#ixzz5Q6BDechd

## 6. 写一段死锁代码。你在 Java 中如何解决死锁？

### 6.1 Write a Java program which will result in deadlock?

```
/**
 * Java program to create a deadlock by imposing circular wait.
 * 
 * @author WINDOWS 8
 *
 */
public class DeadLockDemo {

    /*
     * This method request two locks, first String and then Integer
     */
    public void method1() {
        synchronized (String.class) {
            System.out.println("Aquired lock on String.class object");

            synchronized (Integer.class) {
                System.out.println("Aquired lock on Integer.class object");
            }
        }
    }

    /*
     * This method also requests same two lock but in exactly
     * Opposite order i.e. first Integer and then String. 
     * This creates potential deadlock, if one thread holds String lock
     * and other holds Integer lock and they wait for each other, forever.
     */
    public void method2() {
        synchronized (Integer.class) {
            System.out.println("Aquired lock on Integer.class object");

            synchronized (String.class) {
                System.out.println("Aquired lock on String.class object");
            }
        }
    }
}
```

### 6.2 How to avoid deadlock in Java?

If you have looked above code carefully then you may have figured out that real reason for deadlock is not multiple threads but the way they are requesting a lock, if you provide an ordered access then the problem will be resolved.
死锁的原因不是多线程，而是请求锁的方式不对。如果为锁的请求提供了顺序的访问，那么问题就解决了。

fixed version, which avoids deadlock by avoiding circular wait with no preemption, one of the four conditions which need for deadlock.

```
public class DeadLockFixed {

    /**
     * Both method are now requesting lock in same order, first Integer and then String.
     * You could have also done reverse e.g. first String and then Integer,
     * both will solve the problem, as long as both method are requesting lock
     * in consistent order.
     */
    public void method1() {
        synchronized (Integer.class) {
            System.out.println("Aquired lock on Integer.class object");

            synchronized (String.class) {
                System.out.println("Aquired lock on String.class object");
            }
        }
    }

    public void method2() {
        synchronized (Integer.class) {
            System.out.println("Aquired lock on Integer.class object");

            synchronized (String.class) {
                System.out.println("Aquired lock on String.class object");
            }
        }
    }
}
```
Read more: https://javarevisited.blogspot.com/2018/08/how-to-avoid-deadlock-in-java-threads.html#ixzz5Q6H9gcNy


## 7. 什么是原子操作？Java 中有哪些原子操作？

- 什么是原子操作？
所谓原子操作,就是"不可中断的一个或一系列操作" , 在确认一个操作是原子的情况下，多线程环境里面，我们可以避免仅仅为保护这个操作在外围加上性能昂贵的锁，甚至借助于原子操作，我们可以实现互斥锁。 很多操作系统都为int类型提供了+-赋值的原子操作版本，比如 NT 提供了 InterlockedExchange 等API， Linux/UNIX也提供了atomic_set 等函数。

- 关于java中的原子性？
原子性可以应用于除long和double之外的所有基本类型之上的“简单操作”。对于读取和写入出long double之外的基本类型变量这样的操作，可以保证它们会被当作不可分（原子）的操作来操作。 因为JVM的版本和其它的问题，其它的很多操作就不好说了，比如说++操作在C++中是原子操作，但在Java中就不好说了。 另外，Java提供了AtomicInteger等原子类。再就是用原子性来控制并发比较麻烦，也容易出问题。

1）除long和double之外的基本类型的赋值操作
2）所有引用reference的赋值操作
3）java.concurrent.Atomic.* 包中所有类的一切操作
count++不是原子操作，是3个原子操作组合
1.读取主存中的count值，赋值给一个局部成员变量tmp
2.tmp+1
3.将tmp赋值给count
可能会出现线程1运行到第2步的时候，tmp值为1；这时CPU调度切换到线程2执行完毕，count值为1；切换到线程1，继续执行第3步，count被赋值为1------------结果就是两个线程执行完毕，count的值只加了1；
还有一点要注意，如果使用AtomicInteger.set(AtomicInteger.get() + 1)，会和上述情况一样有并发问题，要使用AtomicInteger.getAndIncrement()才可以避免并发问题


（https://javarevisited.blogspot.com/2011/04/synchronization-in-java-synchronized.html）
- What is Synchronization in Java

Synchronization in Java is possible by using Java keywords "synchronized" and "volatile”. 

Concurrent access of shared objects in Java introduces to kind of errors: thread interference and memory consistency errors and to avoid these errors you need to properly synchronize your Java object to allow mutual exclusive access of critical section to two threads. 

- Why do we need Synchronization in Java?

If your code is executing in a multi-threaded environment, you need synchronization for objects, which are shared among multiple threads, to avoid any corruption of state or any kind of unexpected behavior. Synchronization in Java will only be needed if shared object is mutable. if your shared object is either read-only or immutable object, then you don't need synchronization, despite running multiple threads. Same is true with what threads are doing with an object if all the threads are only reading value then you don't require synchronization in Java. JVM guarantees that Java synchronized code will only be executed by one thread at a time. 

In Summary, Java synchronized Keyword provides following functionality essential for concurrent programming:


1) The synchronized keyword in Java provides locking, which ensures mutually exclusive access to the shared resource and prevents data race.

2) synchronized keyword also prevent reordering of code statement by the compiler which can cause a subtle concurrent issue if we don't use synchronized or volatile keyword.

3) synchronized keyword involve locking and unlocking. before entering into synchronized method or block thread needs to acquire the lock, at this point it reads data from main memory than cache and when it release the lock, it flushes write operation into main memory which eliminates memory inconsistency errors.

- Example of Synchronized Method in Java

```
public class Counter{ private static int count = 0; public static synchronized int getCount(){ return count; } public synchoronized setCount(int count){ this.count = count; } }

```

In this example of Java, the synchronization code is not properly synchronized because both getCount() and setCount() are not getting locked on the same object and can run in parallel which may result in the incorrect count. Here getCount() will lock in Counter.class object while setCount() will lock on current object (this). To make this code properly synchronized in Java you need to either make both method static or nonstatic or use java synchronized block instead of java synchronized method. By the way, this is one of the common mistake Java developers make while synchronizing their code.

- Example of Synchronized Block in Java

This is a classic example of double checked locking in Singleton. 

```
public class Singleton{

private static volatile Singleton _instance;

public static Singleton getInstance(){
   if(_instance == null){
            synchronized(Singleton.class){
              if(_instance == null)
              _instance = new Singleton();
            }
   }
   return _instance;
}

```

- Important points of synchronized keyword in Java

1. Synchronized keyword in Java is used to provide mutually exclusive access to a shared resource with multiple threads in Java. Synchronization in Java guarantees that no two threads can execute a synchronized method which requires the same lock simultaneously or concurrently.
2. You can use java synchronized keyword only on synchronized method or synchronized block.
3. Whenever a thread enters into java synchronized method or blocks it acquires a lock and whenever it leaves java synchronized method or block it releases the lock. The lock is released even if thread leaves synchronized method after completion or due to any Error or Exception.

4. Java Thread acquires an object level lock when it enters into an instance synchronized java method and acquires a class level lock when it enters into static synchronized java method.

5. Java synchronized keyword is re-entrant in nature it means if a java synchronized method calls another synchronized method which requires the same lock then the current thread which is holding lock can enter into that method without acquiring the lock.

6. Java Synchronization will throw NullPointerException if object used in java synchronized block is null e.g. synchronized (myInstance) will throw java.lang.NullPointerException if myInstance is null.

7. One Major disadvantage of Java synchronized keyword is that it doesn't allow concurrent read, which can potentially limit scalability. By using the concept of lock stripping and using different locks for reading and writing, you can overcome this limitation of synchronized in Java. You will be glad to know that java.util.concurrent.locks.ReentrantReadWriteLock provides ready-made implementation of ReadWriteLock in Java.

8. One more limitation of java synchronized keyword is that it can only be used to control access to a shared object within the same JVM. If you have more than one JVM and need to synchronize access to a shared file system or database, the Java synchronized keyword is not at all sufficient. You need to implement a kind of global lock for that.

9. Java synchronized keyword incurs a performance cost. A synchronized method in Java is very slow and can degrade performance. So use synchronization in java when it absolutely requires and consider using java synchronized block for synchronizing critical section only.

10. Java synchronized block is better than java synchronized method in Java because by using synchronized block you can only lock critical section of code and avoid locking the whole method which can possibly degrade performance. A good example of java synchronization around this concept is getting Instance() method Singleton class. See here.

11. It's possible that both static synchronized and non-static synchronized method can run simultaneously or concurrently because they lock on the different object.

12. From java 5 after a change in Java memory model reads and writes are atomic for all variables declared using the volatile keyword (including long and double variables) and simple atomic variable access is more efficient instead of accessing these variables via synchronized java code. But it requires more care and attention from the programmer to avoid memory consistency errors.

13. Java synchronized code could result in deadlock or starvation while accessing by multiple threads if synchronization is not implemented correctly. To know how to avoid deadlock in java see here.

14. According to the Java language specification you can not use Java synchronized keyword with constructor it’s illegal and result in compilation error. So you can not synchronize constructor in Java which seems logical because other threads cannot see the object being created until the thread creating it has finished it.

15. You cannot apply java synchronized keyword with variables and can not use java volatile keyword with the method.

16. Java.util.concurrent.locks extends capability provided by java synchronized keyword for writing more sophisticated programs since they offer more capabilities e.g. Reentrancy and interruptible locks.

17. Java synchronized keyword also synchronizes memory. In fact, java synchronized synchronizes the whole of thread memory with main memory.

18. Important method related to synchronization in Java are wait(), notify() and notifyAll() which is defined in Object class. Do you know, why they are defined in java.lang.object class instead of java.lang.Thread? You can find some reasons, which make sense.

19. Do not synchronize on the non-final field on synchronized block in Java. because the reference of the non-final field may change anytime and then different thread might synchronizing on different objects i.e. no synchronization at all. an example of synchronizing on the non-final field:

```
private String lock = new String("lock");
synchronized(lock){
    System.out.println("locking on :"  + lock);
}
```
any if you write synchronized code like above in java you may get a warning "Synchronization on the non-final field"  in IDE like Netbeans and InteliJ

20. It's not recommended to use String object as a lock in java synchronized block because a string is an immutable object and literal string and interned string gets stored in String pool. so by any chance if any other part of the code or any third party library used same String as there lock then they both will be locked on the same object despite being completely unrelated which could result in unexpected behavior and bad performance. instead of String object its advised to use new Object() for Synchronization in Java on synchronized block.
```
private static final String LOCK = "lock";   //not recommended
private static final Object OBJ_LOCK = new Object(); //better

public void process() {
   synchronized(LOCK) {
      ........
   }
}
```
21. From Java library, Calendar and SimpleDateFormat classes are not thread-safe and requires external synchronization in Java to be used in the multi-threaded environment.  



- some limitation of synchronized keyword in Java which is addressed by explicit locking using new concurrent package and Lock interface:

1. synchronized keyword doesn't allow separate locks for reading and writing. as we know that multiple threads can read without affecting thread-safety of class, synchronized keyword suffer performance due to contention in case of multiple readers and one or few writer.

 2. if one thread is waiting for lock then there is no way to timeout, the thread can wait indefinitely for the lock.
 
3. on a similar note if the thread is waiting for the lock to acquired there is no way to interrupt the thread.
 
All these limitations of synchronized keyword are addressed and resolved by using ReadWriteLock and ReentrantLock in Java 5.


- list of Java Synchronization facts and best practices:
 
1) synchronized keyword in internally implemented using two-byte code instructions MonitorEnter and MonitorExit, this is generated by the compiler. The compiler also ensures that there must be a MonitorExit for every MonitorEnter in different code path e.g. normal execution and abrupt execution, because of Exception.

2) java.util.concurrent package different locking mechanism than provided by synchronized keyword, they mostly used ReentrantLock, which internally use CAS operations, volatile variables and atomic variables to get better performance.
 
3) With synchronized keyword, you have to leave the lock, once you exist a synchronized method or block, there is no way you can take the lock to another method. java.util.concurrent.locks.ReentrantLock solves this problem by providing control of acquiring and releasing the lock, which means you can acquire the lock in method A and can release in method B if they both needs to be locked in same object lock. Though this could be risky as the compiler will neither check nor warn you about any accidental leak of locks. Which means, this can potentially block other threads, which are waiting for the same lock.

4) Prefer ReentrantLock over synchronized keyword, it provides more control on lock acquisition, lock release, and better performance compared to synchronized keyword.
 
5) Any thread trying to acquire a lock using synchronized method will block indefinitely until the lock is available. Instead this, tryLock() method of java.util.concurrent.locks.ReentrantLock will not block if the lock is not available.
Having said that, I must say, lots of good information.


### 你需要同步原子操作吗？

原子操作是需要同步控制的。

可以通过synchronized这个关键字来同步原子操作；
更简便的方法是，对所操作的属性用原子类来定义即可：
```
private AtomicInteger a = new AtomicInteger(1);
```


## 8. Java 中 volatile 关键字是什么？你如何使用它？它和 Java 中的同步方法有什么区别？

### 8.1 what is volatile?

volatile has semantics for memory visibility. Basically, the value of a volatile field becomes visible to all readers (other threads in particular) after a write operation completes on it. Without volatile, readers could see some non-updated value.

简言之，就是volatile定义的变量是多线程环境下直接内存可见的变量。

### 8.2 How you use volatile?

- volatile is very useful to stop threads.

Not that you should be writing your own threads, Java 1.6 has a lot of nice thread pools. But if you are sure you need a thread, you'll need to know how to stop it.

The pattern I use for threads is:
```
public class Foo extends Thread {
  private volatile boolean close = false;
  public void run() {
    while(!close) {
      // do work
    }
  }
  public void close() {
    close = true;
    // interrupt here if needed
  }
}
```
Notice how there's no need for synchronization

- I use a volatile variable to control whether some code continues a loop. 

The loop tests the volatile value and continues if it is true. The condition can be set to false by calling a "stop" method. The loop sees false and terminates when it tests the value after the stop method completes execution.


- Important point about volatile:

Synchronization in Java is possible by using Java keywords synchronized and volatile and locks.
In Java, we can not have synchronized variable. Using synchronized keyword with a variable is illegal and will result in compilation error. Instead of using the synchronized variable in Java, you can use the java volatile variable, which will instruct JVM threads to read the value of volatile variable from main memory and don’t cache it locally.
If a variable is not shared between multiple threads then there is no need to use the volatile keyword.

- Lazy initialization
A singleton implementation may use lazy initialization, where the instance is created when the static method is first invoked. If the static method might be called from multiple threads simultaneously, measures may need to be taken to prevent race conditions that could result in the creation of multiple instances of the class. The following is a thread-safe sample implementation, using lazy initialization with double-checked locking, written in Java.
```
public final class Singleton {

    private static volatile Singleton instance = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }

        return instance;
    }
}
```

简言之，
volatile定义的变量可以作为在多线程环境下停止一个县城的标记；
其次，volatile不安尖子也被用于在内存中的多线程间共享内容；一个应用例子就是懒加载的单例模式，如果没有volatile关键字标记单例的实例，就会有并发问题。


### 8.2 它和 Java 中的同步方法有什么区别？

Declaring a variable as volatile means that modifying its value immediately affects the actual memory storage for the variable. The compiler cannot optimize away any references made to the variable. This guarantees that when one thread modifies the variable, all other threads see the new value immediately. (This is not guaranteed for non-volatile variables.)

Declaring an atomic variable guarantees that operations made on the variable occur in an atomic fashion, i.e., that all of the substeps of the operation are completed within the thread they are executed and are not interrupted by other threads. For example, an increment-and-test operation requires the variable to be incremented and then compared to another value; an atomic operation guarantees that both of these steps will be completed as if they were a single indivisible/uninterruptible operation.

Synchronizing all accesses to a variable allows only a single thread at a time to access the variable, and forces all other threads to wait for that accessing thread to release its access to the variable.

Synchronized access is similar to atomic access, but the atomic operations are generally implemented at a lower level of programming. Also, it is entirely possible to synchronize only some accesses to a variable and allow other accesses to be unsynchronized (e.g., synchronize all writes to a variable but none of the reads from it).

Atomicity, synchronization, and volatility are independent attributes, but are typically used in combination to enforce proper thread cooperation for accessing variables.


volatile:

volatile is a keyword. volatile forces all threads to get latest value of the variable from main memory instead of cache. No locking is required to access volatile variables. All threads can access volatile variable value at same time.

Using volatile variables reduces the risk of memory consistency errors, because any write to a volatile variable establishes a happens-before relationship with subsequent reads of that same variable.

This means that changes to a volatile variable are always visible to other threads. What's more, it also means that when a thread reads a volatile variable, it sees not just the latest change to the volatile, but also the side effects of the code that led up the change.

When to use: One thread modifies the data and other threads have to read latest value of data. Other threads will take some action but they won't update data.

AtomicXXX:

AtomicXXX classes support lock-free thread-safe programming on single variables. These AtomicXXX classes (like AtomicInteger) resolves memory inconsistency errors / side effects of modification of volatile variables, which have been accessed in multiple threads.

When to use: Multiple threads can read and modify data.

synchronized:

synchronized is keyword used to guard a method or code block. By making method as synchronized has two effects:

First, it is not possible for two invocations of synchronized methods on the same object to interleave. When one thread is executing a synchronized method for an object, all other threads that invoke synchronized methods for the same object block (suspend execution) until the first thread is done with the object.

Second, when a synchronized method exits, it automatically establishes a happens-before relationship with any subsequent invocation of a synchronized method for the same object. This guarantees that changes to the state of the object are visible to all threads.

When to use: Multiple threads can read and modify data. Your business logic not only update the data but also executes atomic operations

AtomicXXX is equivalent of volatile + synchronized even though the implementation is different. AmtomicXXX extends volatile variables + compareAndSet methods but does not use synchronization.

简言之，
主要区别是，volatile是定义于变量的，不能修饰于方法。而synchronized则用于修饰方法或者代码块，不能用于修饰变量。


## 9. 什么是竞态条件？你如何发现并解决竞态条件？
（https://javarevisited.blogspot.com/2012/02/what-is-race-condition-in.html）

### 9.1 What is

Race condition in Java is a type of concurrency bug or issue which is introduced in your program because  parallel execution of your program by multiple threads at same time, Since Java is a multi-threaded programming language hence risk of Race condition is higher in Java which demands clear understanding of what causes a race condition and how to avoid that. Anyway Race conditions are just one of hazards or risk presented by  use of multi-threading in Java just like deadlock in Java. Race conditions occurs when two thread operate on same object without proper synchronization and there operation interleaves on each other. Classical example of Race condition is incrementing a counter since increment is not an atomic operation and can be further divided into three steps like read, update and write. if two threads tries to increment count at same time and if they read same value because of interleaving of read operation of one thread to update operation of another thread, one count will be lost when one thread overwrite increment done by other thread. atomic operations are not subject to race conditions because those operation cannot be interleaved.

### 9.2 How to find Race Conditions in Java

only sure shot way to find race condition is reviewing code manually or using code review tools which can alert you on potential race conditions based on code pattern and use of synchronization in Java. I solely rely on code review and yet to find a suitable tool for exposing race condition in java.

- Code Example of Race Condition in Java
Based on my experience in Java synchronization and where we use synchronized keyword I found that two code patterns namely "check and act" and "read modify write" can suffer race condition if not synchronized properly. both cases rely on natural assumption that a single line of code will be atomic and execute in one shot which is wrong e.g. ++ is not atomic.


- "Check and Act" race condition pattern
classical example of "check and act" race condition in Java is getInstance() method of Singleton Class, remember that was one questions which we have discussed on 10 Interview questions on Singleton pattern in Java as "How to write thread-safe Singleton in Java". getInstace() method first check for whether instance is null and than initialized the instance and return to caller. Whole purpose of Singleton is that getInstance should always return same instance of Singleton. if you call getInstance() method from two thread simultaneously its possible that while one thread is initializing singleton after null check, another thread sees value of _instance reference variable as null (quite possible in java) especially if your object takes longer time to initialize and enters into critical section which eventually results in getInstance() returning two separate instance of Singleton. This may not happen always because a fraction of delay may result in value of _instance updated in main memory. here is a code example
```
public Singleton getInstance(){
if(_instance == null){   //race condition if two threads sees _instance= null
_instance = new Singleton();
}
}
```
an easy way to fix "check and act" race conditions is to synchronized keyword and enforce locking which will make this operation atomic and guarantees that block or method will only be executed by one thread and result of operation will be visible to all threads once synchronized blocks completed or thread exited form synchronized block.

read-modify-update race conditions
This is another code pattern in Java which cause race condition, classical example is the non thread safe counter we discussed in how to write thread safe class in Java. this is also a very popular multi-threading question where they ask you to find bugs on concurrent code. read-modify-update pattern also comes due to improper synchronization of non-atomic operations or combination of two individual atomic operations which is not atomic together e.g. put if absent scenario. consider below code
```
if(!hashtable.contains(key)){
hashtable.put(key,value);
}
```
here we only insert object into hashtable if its not already there. point is both contains() and put() are atomic but still this code can result in race condition since both operation together is not atomic. consider thread T1 checks for conditions and goes inside if block now CPU is switched from T1 to thread T2 which also checks condition and goes inside if block. now we have two thread inside if block which result in either T1 overwriting T2 value or vice-versa based on which thread has CPU for execution. In order to fix this race condition in Java you need to wrap this code inside synchronized block which makes them atomic together because no thread can go inside synchronized block if one thread is already there.

These are just some of examples of race conditions in Java, there will be numerous based on your business logic and code. best approach to find Race conditions is code review but its hard because thinking concurrently is not natural and we still assume code to run sequentially. Problem can become worse if JVM reorders code in absent of proper synchronization to gain performance benefit and this usually happens on production under heavily load, which is worst. I also suggest doing load testing in production like environment which many time helps to expose race conditions in java. Please share if you have faced any race condition in java projects.


## 10. 在 Java 中你如何转储线程（thread dump）？如何分析它？

在 UNIX 中，你可以使用 kill -3 <PID>，然后线程转储日志会打印在屏幕上，可以使用 CTRL+Break 查看。

你如何分析转储日志?线程转储日志对于分析死锁情况非常有用。

### 10.2 如何分析转储日志

以下为一篇详细的介绍分析thread dump的文章：

[How to Analyze Java Thread Dumps](https://dzone.com/articles/how-analyze-java-thread-dumps)

When there is an obstacle, or when a Java based Web application is running much slower than expected, we need to use thread dumps. If thread dumps feel like very complicated to you, this article may help you very much. Here I will explain what threads are in Java, their types, how they are created, how to manage them, how you can dump threads from a running application, and finally how you can analyze them and determine the bottleneck or blocking threads. This article is a result of long experience in Java application debugging.

####  Java and Thread

A web server uses tens to hundreds of threads to process a large number of concurrent users. If two or more threads utilize the same resources, a contention between the threads is inevitable, and sometimes deadlock occurs.

Thread contention is a status in which one thread is waiting for a lock, held by another thread, to be lifted. Different threads frequently access shared resources on a web application. For example, to record a log, the thread trying to record the log must obtain a lock and access the shared resources.

Deadlock is a special type of thread contention, in which two or more threads are waiting for the other threads to complete their tasks in order to complete their own tasks.

Different issues can arise from thread contention. To analyze such issues, you need to use the thread dump. A thread dump will give you the information on the exact status of each thread.

#### Background Information for Java Threads

- Thread Synchronization
A thread can be processed with other threads at the same time. In order to ensure compatibility when multiple threads are trying to use shared resources, one thread at a time should be allowed to access the shared resources by using thread synchronization.

Thread synchronization on Java can be done using monitor. Every Java object has a single monitor. The monitor can be owned by only one thread. For a thread to own a monitor that is owned by a different thread, it needs to wait in the wait queue until the other thread releases its monitor.

- Thread Status

In order to analyze a thread dump, you need to know the status of threads. The statuses of threads are stated on java.lang.Thread.State.

NEW: The thread is created but has not been processed yet.
RUNNABLE: The thread is occupying the CPU and processing a task. (It may be in WAITING status due to the OS's resource distribution.)
BLOCKED: The thread is waiting for a different thread to release its lock in order to get the monitor lock.
WAITING: The thread is waiting by using a wait, join or park method.
TIMED_WAITING: The thread is waiting by using a sleep, wait, join or park method. (The difference from WAITING is that the maximum waiting time is specified by the method parameter, and WAITING can be relieved by time as well as external changes.) 

- Thread Types

Java threads can be divided into two:

daemon threads;
and non-daemon threads.
Daemon threads stop working when there are no other non-daemon threads. Even if you do not create any threads, the Java application will create several threads by default. Most of them are daemon threads, mainly for processing tasks such as garbage collection or JMX.

A thread running the 'static void main(String[] args)’ method is created as a non-daemon thread, and when this thread stops working, all other daemon threads will stop as well. (The thread running this main method is called the VM thread in HotSpot VM.)

#### Getting a Thread Dump

We will introduce the three most commonly used methods. Note that there are many other ways to get a thread dump. A thread dump can only show the thread status at the time of measurement, so in order to see the change in thread status, it is recommended to extract them from 5 to 10 times with 5-second intervals.

- Getting a Thread Dump Using jstack

In JDK 1.6 and higher, it is possible to get a thread dump on MS Windows using jstack.

Use PID via jps to check the PID of the currently running Java application process.
```
[user@linux ~]$ jps -v
25780 RemoteTestRunner -Dfile.encoding=UTF-8
25590 sub.rmi.registry.RegistryImpl 2999 -Dapplication.home=/home1/user/java/jdk.1.6.0_24 -Xms8m
26300 sun.tools.jps.Jps -mlvV -Dapplication.home=/home1/user/java/jdk.1.6.0_24 -Xms8m
```

Use the extracted PID as the parameter of jstack to obtain a thread dump.
```
[user@linux ~]$ jstack -f 5824
```

- A Thread Dump Using jVisualVM

Generate a thread dump by using a program such as jVisualVM.

The task on the left indicates the list of currently running processes. Click on the process for which you want the information, and select the thread tab to check the thread information in real time. Click the Thread Dump button on the top right corner to get the thread dump file.

- Generating in a Linux Terminal

Obtain the process pid by using ps -ef command to check the pid of the currently running Java process.
```
[user@linux ~]$ ps - ef | grep java
user      2477          1    0 Dec23 ?         00:10:45 ...
user    25780 25361   0 15:02 pts/3    00:00:02 ./jstatd -J -Djava.security.policy=jstatd.all.policy -p 2999
user    26335 25361   0 15:49 pts/3    00:00:00 grep java
```

**Use the extracted pid as the parameter of kill –SIGQUIT(3) to obtain a thread dump.**

#### Thread Information from the Thread Dump File

```
"pool-1-thread-13" prio=6 tid=0x000000000729a000 nid=0x2fb4 runnable [0x0000000007f0f000] java.lang.Thread.State: RUNNABLE
              at java.net.SocketInputStream.socketRead0(Native Method)
              at java.net.SocketInputStream.read(SocketInputStream.java:129)
              at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:264)
              at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:306)
              at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:158)
              - locked <0x0000000780b7e688> (a java.io.InputStreamReader)
              at java.io.InputStreamReader.read(InputStreamReader.java:167)
              at java.io.BufferedReader.fill(BufferedReader.java:136)
              at java.io.BufferedReader.readLine(BufferedReader.java:299)
              - locked <0x0000000780b7e688> (a java.io.InputStreamReader)
              at java.io.BufferedReader.readLine(BufferedReader.java:362)
```
- Thread name: When using Java.lang.Thread class to generate a thread, the thread will be named Thread-(Number), whereas when using java.util.concurrent.ThreadFactory class, it will be named pool-(number)-thread-(number).
- Priority: Represents the priority of the threads.
- Thread ID: Represents the unique ID for the threads. (Some useful information, including the CPU usage or memory usage of the thread, can be obtained by using thread ID.)
- Thread status: Represents the status of the threads.
- Thread callstack: Represents the call stack information of the threads. 

#### Thread Dump Patterns by Type

```
"BLOCKED_TEST pool-1-thread-1" prio=6 tid=0x0000000006904800 nid=0x28f4 runnable [0x000000000785f000]
   java.lang.Thread.State: RUNNABLE
                at java.io.FileOutputStream.writeBytes(Native Method)
                at java.io.FileOutputStream.write(FileOutputStream.java:282)
                at java.io.BufferedOutputStream.flushBuffer(BufferedOutputStream.java:65)
                at java.io.BufferedOutputStream.flush(BufferedOutputStream.java:123)
                - locked <0x0000000780a31778> (a java.io.BufferedOutputStream)
                at java.io.PrintStream.write(PrintStream.java:432)
                - locked <0x0000000780a04118> (a java.io.PrintStream)
                at sun.nio.cs.StreamEncoder.writeBytes(StreamEncoder.java:202)
                at sun.nio.cs.StreamEncoder.implFlushBuffer(StreamEncoder.java:272)
                at sun.nio.cs.StreamEncoder.flushBuffer(StreamEncoder.java:85)
                - locked <0x0000000780a040c0> (a java.io.OutputStreamWriter)
                at java.io.OutputStreamWriter.flushBuffer(OutputStreamWriter.java:168)
                at java.io.PrintStream.newLine(PrintStream.java:496)
                - locked <0x0000000780a04118> (a java.io.PrintStream)
                at java.io.PrintStream.println(PrintStream.java:687)
                - locked <0x0000000780a04118> (a java.io.PrintStream)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:44)
                - locked <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$1.run(ThreadBlockedState.java:7)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)
   Locked ownable synchronizers:
                - <0x0000000780a31758> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
"BLOCKED_TEST pool-1-thread-2" prio=6 tid=0x0000000007673800 nid=0x260c waiting for monitor entry [0x0000000008abf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:43)
                - waiting to lock <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$2.run(ThreadBlockedState.java:26)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)
   Locked ownable synchronizers:
                - <0x0000000780b0c6a0> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
"BLOCKED_TEST pool-1-thread-3" prio=6 tid=0x00000000074f5800 nid=0x1994 waiting for monitor entry [0x0000000008bbf000]
   java.lang.Thread.State: BLOCKED (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadBlockedState.monitorLock(ThreadBlockedState.java:42)
                - waiting to lock <0x0000000780a000b0> (a com.nbp.theplatform.threaddump.ThreadBlockedState)
                at com.nbp.theplatform.threaddump.ThreadBlockedState$3.run(ThreadBlockedState.java:34)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)
   Locked ownable synchronizers:
                - <0x0000000780b0e1b8> (a java.util.concurrent.locks.ReentrantLock$NonfairSync)
```


#### When in Deadlock Status

This is when thread A needs to obtain thread B's lock to continue its task, while thread B needs to obtain thread A's lock to continue its task. In the thread dump, you can see that DEADLOCK_TEST-1 thread has 0x00000007d58f5e48 lock, and is trying to obtain 0x00000007d58f5e60 lock. You can also see that DEADLOCK_TEST-2 thread has 0x00000007d58f5e60 lock, and is trying to obtain 0x00000007d58f5e78 lock. Also, DEADLOCK_TEST-3 thread has 0x00000007d58f5e78 lock, and is trying to obtain 0x00000007d58f5e48 lock. As you can see, each thread is waiting to obtain another thread's lock, and this status will not change until one thread discards its lock.

```
"DEADLOCK_TEST-1" daemon prio=6 tid=0x000000000690f800 nid=0x1820 waiting for monitor entry [0x000000000805f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock <0x00000007d58f5e60> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked <0x00000007d58f5e48> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
   Locked ownable synchronizers:
                - None
"DEADLOCK_TEST-2" daemon prio=6 tid=0x0000000006858800 nid=0x17b8 waiting for monitor entry [0x000000000815f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock <0x00000007d58f5e78> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked <0x00000007d58f5e60> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
   Locked ownable synchronizers:
                - None
"DEADLOCK_TEST-3" daemon prio=6 tid=0x0000000006859000 nid=0x25dc waiting for monitor entry [0x000000000825f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.goMonitorDeadlock(ThreadDeadLockState.java:197)
                - waiting to lock <0x00000007d58f5e48> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.monitorOurLock(ThreadDeadLockState.java:182)
                - locked <0x00000007d58f5e78> (a com.nbp.theplatform.threaddump.ThreadDeadLockState$Monitor)
                at com.nbp.theplatform.threaddump.ThreadDeadLockState$DeadlockThread.run(ThreadDeadLockState.java:135)
   Locked ownable synchronizers:
                - None
```


#### When Continuously Waiting to Receive Messages from a Remote Server

The thread appears to be normal, since its state keeps showing as RUNNABLE. However, when you align the thread dumps chronologically, you can see that socketReadThread thread is waiting infinitely to read the socket.

```
"socketReadThread" prio=6 tid=0x0000000006a0d800 nid=0x1b40 runnable [0x00000000089ef000]
   java.lang.Thread.State: RUNNABLE
                at java.net.SocketInputStream.socketRead0(Native Method)
                at java.net.SocketInputStream.read(SocketInputStream.java:129)
                at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:264)
                at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:306)
                at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:158)
                - locked <0x00000007d78a2230> (a java.io.InputStreamReader)
                at sun.nio.cs.StreamDecoder.read0(StreamDecoder.java:107)
                - locked <0x00000007d78a2230> (a java.io.InputStreamReader)
                at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:93)
                at java.io.InputStreamReader.read(InputStreamReader.java:151)
                at com.nbp.theplatform.threaddump.ThreadSocketReadState$1.run(ThreadSocketReadState.java:27)
                at java.lang.Thread.run(Thread.java:662)
```


#### When Waiting

The thread is maintaining WAIT status. In the thread dump, IoWaitThread thread keeps waiting to receive a message from LinkedBlockingQueue. If there continues to be no message for LinkedBlockingQueue, then the thread status will not change.

```
"IoWaitThread" prio=6 tid=0x0000000007334800 nid=0x2b3c waiting on condition [0x000000000893f000]
   java.lang.Thread.State: WAITING (parking)
                at sun.misc.Unsafe.park(Native Method)
                - parking to wait for  <0x00000007d5c45850> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
                at java.util.concurrent.locks.LockSupport.park(LockSupport.java:156)
                at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:1987)
                at java.util.concurrent.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:440)
                at java.util.concurrent.LinkedBlockingDeque.take(LinkedBlockingDeque.java:629)
                at com.nbp.theplatform.threaddump.ThreadIoWaitState$IoWaitHandler2.run(ThreadIoWaitState.java:89)
                at java.lang.Thread.run(Thread.java:662)
```


#### When Thread Resources Cannot be Organized Normally

Unnecessary threads will pile up when thread resources cannot be organized normally. If this occurs, it is recommended to monitor the thread organization process or check the conditions for thread termination.

#### How to Solve Problems by Using Thread Dump

- Example 1: When the CPU Usage is Abnormally High

1. Extract the thread that has the highest CPU usage.
```
[user@linux ~]$ ps -mo pid.lwp.stime.time.cpu -C java
     PID         LWP          STIME                  TIME        %CPU
10029               -         Dec07          00:02:02           99.5
         -       10039        Dec07          00:00:00              0.1
         -       10040        Dec07          00:00:00           95.5
```
From the application, find out which thread is using the CPU the most.

Acquire the Light Weight Process (LWP) that uses the CPU the most and convert its unique number (10039) into a hexadecimal number (0x2737).


2. After acquiring the thread dump, check the thread's action.

Extract the thread dump of an application with a PID of 10029, then find the thread with an nid of 0x2737.

```
"NioProcessor-2" prio=10 tid=0x0a8d2800 nid=0x2737 runnable [0x49aa5000]
java.lang.Thread.State: RUNNABLE
                at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
                at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:210)
                at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:65)
                at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:69)
                - locked <0x74c52678> (a sun.nio.ch.Util$1)
                - locked <0x74c52668> (a java.util.Collections$UnmodifiableSet)
                - locked <0x74c501b0> (a sun.nio.ch.EPollSelectorImpl)
                at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:80)
                at external.org.apache.mina.transport.socket.nio.NioProcessor.select(NioProcessor.java:65)
                at external.org.apache.mina.common.AbstractPollingIoProcessor$Worker.run(AbstractPollingIoProcessor.java:708)
                at external.org.apache.mina.util.NamePreservingRunnable.run(NamePreservingRunnable.java:51)
                at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
                at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
                at java.lang.Thread.run(Thread.java:662)
```

Extract thread dumps several times every hour, and check the status change of the threads to determine the problem.

- Example 2: When the Processing Performance is Abnormally Slow  

After acquiring thread dumps several times, find the list of threads with BLOCKED status.

```
" DB-Processor-13" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f000]
java.lang.Thread.State: BLOCKED (on object monitor)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
                - waiting to lock <0xe0375410> (a beans.ConnectionPool)
                at beans.cus.ServiceCnt.getTodayCount(ServiceCnt.java:111)
                at beans.cus.ServiceCnt.insertCount(ServiceCnt.java:43)
"DB-Processor-14" daemon prio=5 tid=0x003edf98 nid=0xca waiting for monitor entry [0x000000000825f020]
java.lang.Thread.State: BLOCKED (on object monitor)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:102)
                - waiting to lock <0xe0375410> (a beans.ConnectionPool)
                at beans.cus.ServiceCnt.getTodayCount(ServiceCnt.java:111)
                at beans.cus.ServiceCnt.insertCount(ServiceCnt.java:43)
" DB-Processor-3" daemon prio=5 tid=0x00928248 nid=0x8b waiting for monitor entry [0x000000000825d080]
java.lang.Thread.State: RUNNABLE
                at oracle.jdbc.driver.OracleConnection.isClosed(OracleConnection.java:570)
                - waiting to lock <0xe03ba2e0> (a oracle.jdbc.driver.OracleConnection)
                at beans.ConnectionPool.getConnection(ConnectionPool.java:112)
                - locked <0xe0386580> (a java.util.Vector)
                - locked <0xe0375410> (a beans.ConnectionPool)
                at beans.cus.Cue_1700c.GetNationList(Cue_1700c.java:66)
                at org.apache.jsp.cue_1700c_jsp._jspService(cue_1700c_jsp.java:120)
```

Acquire the list of threads with BLOCKED status after getting the thread dumps several times.

If the threads are BLOCKED, extract the threads related to the lock that the threads are trying to obtain.

Through the thread dump, you can confirm that the thread status stays BLOCKED because <0xe0375410> lock could not be obtained. This problem can be solved by analyzing stack trace from the thread currently holding the lock.

There are two reasons why the above pattern frequently appears in applications using DBMS. The first reason is inadequate configurations. Despite the fact that the threads are still working, they cannot show their best performance because the configurations for DBCP and the like are not adequate. If you extract thread dumps multiple times and compare them, you will often see that some of the threads that were BLOCKED previously are in a different state.

The second reason is the abnormal connection. When the connection with DBMS stays abnormal, the threads wait until the time is out. In this case, even after extracting the thread dumps several times and comparing them, you will see that the threads related to DBMS are still in a BLOCKED state. By adequately changing the values, such as the timeout value, you can shorten the time in which the problem occurs.

#### Coding for Easy Thread Dump

- Naming Threads

When a thread is created using java.lang.Thread object, the thread will be named Thread-(Number). When a thread is created using java.util.concurrent.DefaultThreadFactory object, the thread will be named pool-(Number)-thread-(Number). When analyzing tens to thousands of threads for an application, if all the threads still have their default names, analyzing them becomes very difficult, because it is difficult to distinguish the threads to be analyzed.

Therefore, you are recommended to develop the habit of naming the threads whenever a new thread is created. 

When you create a thread using java.lang.Thread, you can give the thread a custom name by using the creator parameter.

```

public Thread(Runnable target, String name);
public Thread(ThreadGroup group, String name);
public Thread(ThreadGroup group, Runnable target, String name);
public Thread(ThreadGroup group, Runnable target, String name, long stackSize);
```

When you create a thread using java.util.concurrent.ThreadFactory, you can name it by generating your own ThreadFactory. If you do not need special functionalities, then you can use MyThreadFactory as described below:

```
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.atomic.AtomicInteger;
public class MyThreadFactory implements ThreadFactory {
  private static final ConcurrentHashMap<String, AtomicInteger> POOL_NUMBER =
                                                       new ConcurrentHashMap<String, AtomicInteger>();
  private final ThreadGroup group;
  private final AtomicInteger threadNumber = new AtomicInteger(1);
  private final String namePrefix;
  public MyThreadFactory(String threadPoolName) {
      if (threadPoolName == null) {
          throw new NullPointerException("threadPoolName");
      }
            POOL_NUMBER.putIfAbsent(threadPoolName, new AtomicInteger());
      SecurityManager securityManager = System.getSecurityManager();
      group = (securityManager != null) ? securityManager.getThreadGroup() :
                                                    Thread.currentThread().getThreadGroup();
      AtomicInteger poolCount = POOL_NUMBER.get(threadPoolName);
      if (poolCount == null) {
            namePrefix = threadPoolName + " pool-00-thread-";
      } else {
            namePrefix = threadPoolName + " pool-" + poolCount.getAndIncrement() + "-thread-";
      }
  }
  public Thread newThread(Runnable runnable) {
      Thread thread = new Thread(group, runnable, namePrefix + threadNumber.getAndIncrement(), 0);
      if (thread.isDaemon()) {
            thread.setDaemon(false);
      }
      if (thread.getPriority() != Thread.NORM_PRIORITY) {
            thread.setPriority(Thread.NORM_PRIORITY);
      }
      return thread;
  }
}
```

- Obtaining More Detailed Information by Using MBean

```

ThreadMXBean mxBean = ManagementFactory.getThreadMXBean();
long[] threadIds = mxBean.getAllThreadIds();
ThreadInfo[] threadInfos =
                mxBean.getThreadInfo(threadIds);
for (ThreadInfo threadInfo : threadInfos) {
  System.out.println(
      threadInfo.getThreadName());
  System.out.println(
      threadInfo.getBlockedCount());
  System.out.println(
      threadInfo.getBlockedTime());
  System.out.println(
      threadInfo.getWaitedCount());
  System.out.println(
      threadInfo.getWaitedTime());
}
```

You can acquire the amount of time that the threads WAITed or were BLOCKED by using the method in ThreadInfo, and by using this you can also obtain the list of threads that have been inactive for an abnormally long period of time.





### 10.3 stack-overflow上一个介绍分析thread dump的回答： how-to-analyze-a-java-thread-dump

The TID is thead id and NID is: Native thread ID. This ID is highly platform dependent. It's the NID in jstack thread dumps. On Windows, it's simply the OS-level thread ID within a process. On Linux and Solaris, it's the PID of the thread (which in turn is a light-weight process). On Mac OS X, it is said to be the native pthread_t value.

Go to this link: [Java-level thread ID](https://gist.github.com/843622): for a definition and a further explanation of these two terms.

On IBM's site I found this link: [How to interpret a thread dump](http://publib.boulder.ibm.com/infocenter/javasdk/v1r4m2/index.jsp?topic=/com.ibm.java.doc.diagnostics.142j9/html/interpretjavadump.html). that covers this in greater detail:

It explains what that waiting on means: A lock prevents more than one entity from accessing a shared resource. Each object in Java™ has an associated lock (gained by using a synchronized block or method). In the case of the JVM, threads compete for various resources in the JVM and locks on Java objects.

Then it describes the monitor as a special kind of locking mechanism that is used in the JVM to allow flexible synchronization between threads. For the purpose of this section, read the terms monitor and lock interchangeably.

Then it goes further:

To avoid having a monitor on every object, the JVM usually uses a flag in a class or method block to indicate that the item is locked. Most of the time, a piece of code will transit some locked section without contention. Therefore, the guardian flag is enough to protect this piece of code. This is called a flat monitor. However, if another thread wants to access some code that is locked, a genuine contention has occurred. The JVM must now create (or inflate) the monitor object to hold the second thread and arrange for a signaling mechanism to coordinate access to the code section. This monitor is now called an inflated monitor.

Here is a more in-depth explanation of what you are seeing on the lines from the thread dump. A Java thread is implemented by a native thread of the operating system. Each thread is represented by a line in bold such as:

"Thread-1" (TID:0x9017A0, sys_thread_t:0x23EAC8, state:R, native ID:0x6E4) prio=5

*The following 6 items explains this as I've matched them from the example, values in the brackets[]:

name [Thread-1],
identifier [0x9017A0],
JVM data structure address [0x23EAC8],
current state [R],
native thread identifier [0x6E4],
and priority [5].
The "wait on" appears to be a daemon thread associated with the jvm itself and not the application thread perse. When you get an "in Object.wait()", that means the daemon thread, "finalizer" here, is waiting on a notification about a lock on an object, in this case it shows you what notification it's waiting on: "- waiting on <0x27ef0288> (a java.lang.ref.ReferenceQueue$Lock)"

Definition of the ReferenceQueue is: Reference queues, to which registered reference objects are appended by the garbage collector after the appropriate reachability changes are detected.

The finalizer thread runs so the garbage collection operates to clean up resources associated with an object. If I'm seeing it corectly, the finalizer can't get the lock to this object: java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:118) because the java object is running a method, so the finalizer thread is locked until that object is finished with it's current task.

Also, the finalizer isn't just looking to reclaim memory, it's more involved than that for cleaning up resources. I need to do more study on it, but if you have files open, sockets, etc... related to an objects methods, then the finalizer is going to work on freeing those items up as well.

What is the figure in squared parenthesis after Object.wait in the thread dump?

It is a pointer in memory to the thread. Here is a more detailed description:

C.4.1 Thread Information

The first part of the thread section shows the thread that provoked the fatal error, as follows:

Current thread (0x0805ac88):  JavaThread "main" [_thread_in_native, id=21139]
                    |             |         |            |          +-- ID
                    |             |         |            +------------- state
                    |             |         +-------------------------- name
                    |             +------------------------------------ type
                    +-------------------------------------------------- pointer
The thread pointer is the pointer to the Java VM internal thread structure. It is generally of no interest unless you are debugging a live Java VM or core file.

This last description came from: [Troubleshooting Guide for Java SE 6 with HotSpot VM](http://www.oracle.com/technetwork/java/javase/felog-138657.html)

Here are a few more links on thread dumps:

[How Threads Work](http://www.javamex.com/tutorials/threads/how_threads_work.shtml)
[How to Analyze a Thread Dump](http://www.javacodegeeks.com/2012/03/jvm-how-to-analyze-thread-dump.html)
[Java Thread Dumps](http://sites.google.com/site/threaddumps/java-thread-dumps)
[Java VM threads](https://stackoverflow.com/questions/1317094/name-2-java-vm-threads)
Stackoverflow question: [How threads are mapped](https://stackoverflow.com/questions/16264118/how-jvm-stack-heap-and-threads-are-mapped-to-physical-memory-or-operation-syste)

扩展阅读：
[JVM: How to analyze Thread Dump](https://www.javacodegeeks.com/2012/03/jvm-how-to-analyze-thread-dump.html)


### 10.4 堆转储文件分析
1）获取java进程id，命令：jps -v
2）到处堆转储文件，jmap命令：jmap-dump:format=b,file=/data/.../heap.hprof pid
3）分析工具：Eclipse Memory Analyzer, IBM HeapAnalyzer, VisualVM

- Heap堆分析（堆转储、堆分析）
链接：[Heap堆分析（堆转储、堆分析）](https://www.cnblogs.com/duanxz/p/8510623.html)

1）通过jcmd获得堆直方图：jcmd 26964 GC.class_histogram | more
2）通过jmap获得堆直方图：jmap -histo:live 26964 | more

3）使用jcmd进行堆转储：jmap -dump:live,file=/home/ciadmin/pos-gateway-cloud/heap_dump2.hprof 26964
4）自动堆转储OutOfMemoryError是不可预料的，我们很难确定应该何时获得堆转储。有几个JVM标志可以起到帮助。

-XX:+HeapDumpOnOutOfMemoryError该标志默认为false，打开该标志，JVM会在抛出OutOfMemoryError时创建堆转储。

-XX:HeapDumpPath=<path>该标志知道了堆转储将被写入的位置，默认为当前工作目录下生产java_pid<pid>.hprof文件。

-XX:+HeapDumpAfterFullGC 这会在运行一次Full GC后生成一个堆转储文件。

-XX:+HeapDumpBeforeFullGC 这会在运行一次Full GC之前生成一个堆转储文件。

有的情况下，（入帮，因为执行了多次Full GC）会生成多个堆转储文件，这时JVM会在堆转储文件的名字上附加一个序号。

这两条命令都会在指定目录下创建一个命名为*.hprof的文件。生成之后，有很多工具可以打开该文件。以下是三个最常见的工具。

5）分析工具：jhat
最原始的分析工具，它会读取堆转储文件，并运行一个小型的HTTP服务器，可以通过网页链接查看堆转储信息
```
$ jhat heap_dump.hprof 
Reading from heap_dump.hprof...
Dump file created Mon Mar 05 18:33:10 CST 2018
Snapshot read, resolving...
Resolving 751016 objects...
Chasing references, expect 150 dots......................................................................................................................................................
Eliminating duplicate references......................................................................................................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

6）dump分析工具：jvisualvm
jvisualvm的监视（Monitor） 选项卡可以从一个运行中的程序获得堆转储文件，也可以打开之前生成堆转储文件。
更多信息见[《八、jdk工具之JvisualVM、JvisualVM之一--（visualVM介绍及性能分析示例）》](http://www.cnblogs.com/duanxz/p/3958504.html)

7）分析工具：eclipse的内存分析器工具mat，（EclipseLink Memory Analyzer Tool，mat）
可以加载一个或多个堆转储文件并执行分析。它可以生成报告，向我们建议可能存在问题的地方，也可以用于流量堆，并对堆执行类SQL的查询。

特别指出的是：mat内置一功能：如果打开了两个堆转储文件，mat有一个选项用来计算两个堆中的直方图之间的差别。

更多信息见[《mat之一--eclipse安装Memory Analyzer》](http://www.cnblogs.com/duanxz/p/3958504.html)

对堆的第一遍分析通常涉及保留内存。一个对象的保留内存，是指回收该对象可以释放出的内存量。
关于保留内存相关知识见[《GC之二--GC是如何回收时的判断依据、shallow（浅） size、retained（保留） size、Deep（深）size》](http://www.cnblogs.com/duanxz/p/5230856.html)

8）常见内存溢出错误
在下面情况下，jvm会抛出内存溢出错误（OutOfMemeoryError）：

JVM没有原生内存可用；
永久代（在java7和更早的版本中）或元空间（java8）内存不足；
java堆本身内存不足--对于给定的堆空间而言，应用中活跃对象太多；
JVM执行GC耗时太多；
1、原生内存不足

其原因与堆根本无关。在32位的JVM中，一个进程的最大内存是4GB，如果指定一个非常大的堆大小，比如说3.8GB，使应用的大小很接近4GB的限制，这很危险。

2、永久代或元空间内存不足

首先与堆无关，其根源可能有两种情况：

第一种情况是应用使用的类太多，超出了永久代的默认容纳范围；解决方案：增加永久代的大小
第二种情况相对来说有点棘手：它涉及类加载器的内存泄漏。这种情况经常出现于Java EE应用服务器中。类加载导致内存溢出可以通过堆转储分析，在直方图中，找到ClassLoader类的所有实例，然后跟踪他们的GC根，看哪些对象还保留了对它们的引用。

3、堆内存不足

当确实是堆内存本身不足时，错误信息会这样：

OutOfMemoryError：Java heap space

可能的原因有：

1、活跃对象数目在为其配置的堆空间中已经装不下了。

2、也可能是应用存在内存泄漏：它持续分配新对象，却没有让其他对象退出作用域。

不管哪种情况，要找出哪些对象消耗的内存最多，堆转储分析都是必要的；

4、达到GC的开销限制

JVM抛出OutOfMemoryError的最后一种情况是JVM任务在执行GC上花费了太多时间：

OutOfMemoryError：GC overhead limit exceeded

当满足下列所有条件时就会抛出该错误：

1、花在Full GC的时间超出了-XX:GCTimeLimit=N标志指定的值。默认为98

2、一次Full GC回收内存量少于-XX:GCHeapFreeLimit=N标志指定的值。默认值为2（2%）

3、上面两个条件连续5次Full GC都成立（这个值无法调整）

4、-XX:+UseGCOverhead-Limit标志为true（默认也为true）

这四个条件必须都满足。一般来说，连续5次Full GC以上，不一定会抛异常。即使98%时间在Full GC上，但每次GC期间释放的堆空间会超过2%，这种情况下可以增加-XX:GCHeapFreeLimit的值。

还请注意，如果前两个条件连续4次Full GC周期都成立，作为释放内存的最后一搏，JVM中所有的软引用都会在第五次Full GC之前被释放。这往往会防止该错误，因为第五次Full GC很可能会释放超过2%的堆内存（假设该应用使用了软引用）。









## 11. 既然 start() 方法会调用 run() 方法，为什么我们调用 start() 方法，而不直接调用 run() 方法？

当你调用 start() 方法时，它会新建一个线程然后执行 run() 方法中的代码。如果直接调用 run() 方法，并不会创建新线程，方法中的代码会在当前调用者的线程中执行。可以看这篇文章了解更多[线程中 Start 和 Run 方法的区别](http://javarevisited.blogspot.sg/2012/03/difference-between-start-and-run-method.html)。




## 12.  Java 中你如何唤醒阻塞线程？

这是有关线程的一个很狡猾的问题。有很多原因会导致阻塞，如果是 IO 阻塞，我认为没有方式可以中断线程（如果有的话请告诉我）。另一方面，如果线程阻塞是由于调用了 wait()，sleep() 或 join() 方法，你可以中断线程，通过抛出 InterruptedException 异常来唤醒该线程。可以看这篇文章了解有关处理阻塞线程的知识[Java 中如何处理阻塞方法](http://javarevisited.blogspot.sg/2012/02/what-is-blocking-methods-in-java-and.html)。




## 13. Java 中 CyclicBarriar 和 CountdownLatch 有什么区别？

两者区别之一就是 CyclicBarrier 在屏障打开之后（所有线程到达屏障点），可以重复使用。而 CountDownLatch 不行。想了解更多可以参与课程[Java 中的多线程和并行计算](https://click.linksynergy.com/fs-bin/click?id=JVFxdTr9V80&subid=0&offerid=323058.1&type=10&tmpid=14538&RD_PARM1=https%3A%2F%2Fwww.udemy.com%2Fmultithreading-and-parallel-computing-in-java%2F)。


- Examples of blocking methods in Java:
There are lots of blocking methods in Java API and good thing is that javadoc clearly informs about it and always mention whether a method call is blocking or not. In General methods related to reading or writing file, opening network connection, reading from Socket, updating GUI synchronously uses blocking call. here are some of most common methods in Java which are blocking in nature:

1) InputStream.read() which blocks until input data is available, an exception is thrown or end of Stream is detected.
2) ServerSocket.accept() which listens for incoming socket connection in Java and blocks until a connection is made.
3) InvokeAndWait() wait until code is executed from Event Dispatcher thread.

- Best practices while calling blocking method in Java:
Blocking methods are for a purpose or may be due to limitation of API but there are guidelines available in terms of common and best practices to deal with them. here I am listing some standard ways which I employ while using blocking method in Java

1) If you are writing GUI application may be in Swing never call blocking method in Event dispatcher thread or
in the event handler. for example if you are reading a file or opening a network connection when a button is clicked
don't do that on actionPerformed() method, instead just create another worker thread to do that job and return from
actionPerformed(). this will keep your GUI responsive, but again it depends upon design if the operation is something which requires user to wait than consider using invokeAndWait() for synchronous update.

2) Always let separate worker thread handles time consuming operations e.g. reading and writing to file, database or
socket.

3) Use timeout while calling blocking method. so if your blocking call doesn't return in specified time period, consider
aborting it and returning back but again this depends upon scenario. if you are using Executor Framework for managing
your worker threads, which is by the way recommended way than you can use Future object whose get() methods support timeout, but ensure that you properly terminate a blocking call.

4) Extension of first practices, don't call blocking methods on keyPressed() or paint() method which are supposed to
return as quickly as possible.

5) Use call-back functions to process result of a blocking call.


- Important points:
1. If a Thread is blocked in a blocking method it remain in any of blocking state e.g. WAITING, BLOCKED or TIMED_WAITING.

2. Some of the blocking method throws checked InterrupptedException which indicates that they may allow cancel the task and return before completion like Thread.sleep() or BlockingQueue.put() or take() throws InterruptedException.

3. interupt() method of Thread class can be used to interuupt a thread blocked inside blocking operation, but this is mere
a request not guarantee and works most of the time.


Read more: https://javarevisited.blogspot.com/2012/02/what-is-blocking-methods-in-java-and.html#ixzz5QfVAkI3l




## 14. 什么是不可变类？它对于编写并发应用有何帮助？

### 什么是不可变类？

不可变类：当你获得这个类的一个实例引用时，你不可以改变这个实例的内容。不可变类的实例一但创建，其内在成员变量的值就不能被修改。 

举个例子：String和StringBuilder，String是immutable的，每次对于String对象的修改都将产生一个新的String对象，而原来的对象保持不变，而StringBuilder是mutable，因为每次对于它的对象的修改都作用于该对象本身，并没有产生新的对象。

不可变类的优势

- 方便构造、测试和使用
- 线程安全，没有同步问题
- 不需要拷贝构造方法
- 不需要实现Clone方法
- 可以缓存类的返回值，允许hashCode使用惰性初始化方式
- 不需要防御式复制
- 适合用作Map的key和Set的元素（因为集合里这些对象的状态不能改变）
- 类一旦构造完成就是不变式，不需要再次检查
- 总是“failure atomicity”（原子性失败）：如果一个不可变对象抛出异常，它从不会保留一个烦人的或者不确定的状态




### 如何写一个不可变类？

- 不提供setter方法，避免对象的域被修改
- 将所有的域都设置为private final
- 不允许子类覆盖父类方法。最简单的方法是将class设为final。更好点的方式是将构造方法设为private，同时通过工厂方法来创建实例
- 如果域包含其他可变类的对象，也要禁止这些对象被修改：
不提供修改可变对象的方法.

不要共享指向可变对象的引用。不要存储那些传进构造方法的外部可变对象的引用；如果需要，创建拷贝，保存指向拷贝的引用。类似的，在创建方法返回值时，避免返回原始的内部可变对象，而是返回可变对象的拷贝。

写一个不可变类的例子：
```
import java.util.Date;
 
/**
* 注意实例的变量本身可能是不可变的，也可能是可变的
* 对于所有可变的成员变量，返回时需要复制一份新的
* 不可变的成员变量不用做特殊处理
* */
public final class ImmutableClass
{
 
    /**
    * Integer类是不可变的，因为它没有提供任何setter方法来改变值
    * */
    private final Integer immutableField1;
    /**
    * String类是不可变的，它也没有提供任何setter方法来改变值
    * */
    private final String immutableField2;
    /**
    * Date类是可变的，它提供了改变日期或时间的setter方法
    * */
    private final Date mutableField;
 
    // 将构造方法声明为private，确保不会有意外情况构造这个类
    private ImmutableClass(Integer fld1, String fld2, Date date)
    {
        this.immutableField1 = fld1;
        this.immutableField2 = fld2;
        this.mutableField = new Date(date.getTime());
    }
 
    // 工厂方法将创建对象的逻辑封装在一个地方
    public static ImmutableClass createNewInstance(Integer fld1, String fld2, Date date)
    {
        return new ImmutableClass(fld1, fld2, date);
    }
 
    // 不提供setter方法
 
    /**
    * Integer类是不可变的，可以直接返回成员变量的实例
    * */
    public Integer getImmutableField1() {
        return immutableField1;
    }
 
    /**
    * String类是不可变的，可以直接返回成员变量的实例
    * */
    public String getImmutableField2() {
        return immutableField2;
    }
 
    /**
    * Date类是可变的，需要注意一下
    * 不要返回原始成员变量的引用
    * 创建一个新的Date对象，内容和成员变量一样
    * */
    public Date getMutableField() {
        return new Date(mutableField.getTime());
    }
 
    @Override
    public String toString() {
        return immutableField1 +" - "+ immutableField2 +" - "+ mutableField;
    }
}
```




### 不可变类对于编写并发应用有何帮助？

不可变对象天生就是线程安全的。
因为他们不能改变状态，它们不能被线程干扰所中断或者被其他线程观察到内部不一致的状态。

用volatile发布不可变对象：
使用volatile变量（仅可见性，而不具备同步性）不能保证线程安全性，但有时不可变对象也可以提供一种弱形式的原子性。

不可变对象可以在没有任何额外同步的情况下，安全地用于任意线程；甚至发布它时也不需要同步。


### 扩展知识：

- 简单地发布安全对象

安全发布对象的条件：

1、通过静态初始化对象的引用；
2、将引用存储到volatile变量或AutomaticReference;
3、将引用存储到final域字段中；
4、将引用存储到由锁正确保护的域中；

在多线程中，常量（变量）最好是声明为不可变类型，即使用final关键字。如下是一个不正确发布的对象：
```
package net.jcip.examples;
 
/**
 * Holder
 * <p/>
 * Class at risk of failure if not properly published
 *
 * @author Brian Goetz and Tim Peierls
 */
public class Holder {
    private int n;
 
    public Holder(int n) {
        this.n = n;
    }
 
    public void assertSanity() {
        if (n != n)
            throw new AssertionError("This statement is false.");
    }
}

```

修正方式：将变量n用final修饰，使之不可变（即线程安全）。
```
package net.jcip.examples;
 
/**
 * Holder
 * <p/>
 * Class at risk of failure if not properly published
 *
 * @author Brian Goetz and Tim Peierls
 */
public class Holder {
    private final int n;
 
    public Holder(int n) {
        this.n = n;
    }
 
    public void assertSanity() {
        if (n != n)
            throw new AssertionError("This statement is false.");
    }
}
```

- Java中支持线程安全保证的类：
1、置入HashTable、synchronizedMap、ConcurrentMap中的主键以及值，会安全地发布到可以从Map获取他们的任意线程中，无论是直接还是通过迭代器（Iterator）获得。

2、置入Vector、CopyOnWriteArrayList、CopyOnWriteArraySet、synchronizedList或者synchronizedSet中的元素，会安全地发布到可以从容器中获取它的任意线程中。

3、置入BlockingQueue或ConcurrentLinkedQueue的元素，会安全地发布到可以从容器中获取它的任意线程中。


通常，以最简单最安全的方式发布一个类就是使用静态初始化：
```
 public static Holder holder=new Holder(99);
```
任何线程都可以在没有额外的同步下安全地使用一个安全发布的高效不可变对象。
```
 public Map<String,Date> lastLogin=Collections.synchronizedMap(new HashMap<String, Date>());
```
Date是可变类型，通过Collections.synchronizedMap安全的数据结构使得使用它的结果不可变，从而不需要额外的同步。


- 安全地共享对象：

在并发程序中，使用和共享对象的一些最有效的策略如下：

1)线程限制：

一个线程限制的对象，通过限制在线程中，而被线程独占，且只能被占有它的线程修改。

2)共享只读(read-only)：

一个共享的只读对象，在没有额外同步的情况下，可以被多个线程并发地访问，但任何线程都不能修改它。共享只读对象包括可变对象与高效不可变对象。

3)共享线程安全(thread-safe)：

一个线程安全的对象在内部进行同步，所以其它线程无额外同步，就可以通过公共接口随意地访问它。

4)被守护（Guarded）：

一个被守护的对象只能通过特定的锁来访问。被守护的对象包括哪些被线程安全对象封装的对象，和已知特定的锁保护起来的已发布对象。

### [Why String is Immutable or Final in Java](https://javarevisited.blogspot.com/2010/10/why-string-is-immutable-or-final-in-java.html)

- The string is Immutable in Java because String objects are cached in String pool. Since cached String literals are shared between multiple clients there is always a risk, where one client's action would affect all another client.

- Since caching of String objects was important from performance reason this risk was avoided by making String class Immutable. At the same time, String was made final so that no one can compromise invariant of String class e.g. Immutability, Caching, hashcode calculation etc by extending and overriding behaviors. 

- Another reason of why String class is immutable could die due to HashMap.
Since Strings are very popular as HashMap key, it's important for them to be immutable so that they can retrieve the value object which was stored in HashMap. Since HashMap works in the principle of hashing, which requires same has value to function properly. Mutable String would produce two different hashcodes at the time of insertion and retrieval if contents of String was modified after insertion, potentially losing the value object in the map.


1) Imagine String pool facility without making string immutable , its not possible at all because in case of string pool one string object/literal e.g. "Test" has referenced by many reference variables, so if any one of them change the value others will be automatically gets affected i.e. lets say

String A = "Test"
String B = "Test"

Now String B called, "Test".toUpperCase() which change the same object into "TEST", so A will also be "TEST" which is not desirable. Here is a nice diagram which shows how String literals are created in heap memory and String literal pool.

Why String is Immutable or Final in Java


2) String has been widely used as parameter for many Java classes e.g. for opening network connection, you can pass hostname and port number as string, you can pass database URL as a string for opening database connection, you can open any file in Java by passing the name of the file as argument to File I/O classes.

In case, if String is not immutable, this would lead serious security threat, I mean someone can access to any file for which he has authorization, and then can change the file name either deliberately or accidentally and gain access to that file. Because of immutability, you don't need to worry about that kind of threats. This reason also gels with, Why String is final in Java, by making java.lang.String final, Java designer ensured that no one overrides any behavior of String class.



3)Since String is immutable it can safely share between many threads which is very important for multithreaded programming and to avoid any synchronization issues in Java, Immutability also makes String instance thread-safe in Java, means you don't need to synchronize String operation externally. Another important point to note about String is the memory leak caused by SubString, which is not a thread related issues but something to be aware of.


4) Another reason of Why String is immutable in Java is to allow String to cache its hashcode, being immutable String in Java caches its hashcode, and do not calculate every time we call hashcode method of String, which makes it very fast as hashmap key to be used in hashmap in Java.  This one is also suggested by  Jaroslav Sedlacek in comments below. In short because String is immutable, no one can change its contents once created which guarantees hashCode of String to be same on multiple invocations.


5) Another good reason of Why String is immutable in Java suggested by Dan Bergh Johnsson on comments is: The absolutely most important reason that String is immutable is that it is used by the class loading mechanism, and thus have profound and fundamental security aspects. Had String been mutable, a request to load "java.io.Writer" could have been changed to load "mil.vogoon.DiskErasingWriter"




### 扩展阅读：

[[译] 不可变类有什么优势，应该如何创建？](https://www.jianshu.com/p/2080b524fb3a)







## 15. 你在多线程环境中遇到的最多的问题是什么？你如何解决的？

内存干扰、竞态条件、死锁、活锁、线程饥饿是多线程和并发编程中比较有代表性的问题。这类问题无休无止，而且难于定位和调试。

