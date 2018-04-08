## java.util.concurrent包源码解析

### 一  源码包结构

1. Atomic数据类型

   这部分都在Java.util.concurrent.atomic包里，实现了原子化操作的数据类型，包括Boolean,Integer,Long和Reference四种类型以及四种类型的数组类型。

2. 锁

   这部分都放到了java.util.concurrent.lock包里，实现了并发操作中几种类型的锁。

3. Java集合框架中的一些数据结构的并发实现

   这部分实现的主要数据结构有List、Queue、Map。

4. 多线程任务执行

   这部分包括三个概念

   Callable 被执行任务

   Executor 执行任务

   Future 异步提交任务的返回数据

5. 线程管理类

   这部分主要是对线程集合的管理实现，有CyclicBarrier、CountDownLatch、Exchanger等。

   ​

### 二  锁详解

#### Condition接口



类似与Object的wait/notify，Condition对象应该是被多线程共享的，使用lock保护器状态的一致性。关键字synchronized与wait()和notify()方法结合可以实现等待/通知模式，类ReentranLock也可以实现同样的功能，只需要借助Condition对象。Condition可以实现多路通知，也就是在一个Lock对象里面可以创建多个Condition实例，线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。但是notify()/notifyAll()方法进行的通知是有JVM随机选择的。



#### Lock和ReadWriteLock

两个接口，后者不是前者的子接口，通过以下代码可看出

```
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```

两个接口都有可重入锁的实现



#### LockSupport

工具类，基本操作是park和unpark，park会使得当前线程失效，暂时挂起，直到出现以下几种情况的一种：

1）其他线程调用unpark方法操作该线程

2）该线程被中断

3）park方法立刻返回

关于unpark

public static void UNpark(Thread thread)，没有unpark当前线程的方法，因为一个线程park的时候已经被block，没有可能调用unpark自救。