##Java多线程知识点总结

**什么是线程？**

线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。

 

**进程和线程有什么区别？**

线程是进程的子集，一个进程可以有很多个线程，每条线程并行执行不同的任务。不同的进程使用不同的内存空间，而所有线程共享一片相同的内存空间。

 

**如何在Java中实现线程？**

一种是继承java.lang.Thread类的实例，但是需要调用java.lang.Runnable接口来执行，重写run()方法，或者直接实现Runnable接口来重写run()方法实现线程，还可以实现Callable接口，使用ExecutorService、Callable、Future实现有返回结果的线程，或者通过FutureTask包装器来创建Thread。

 

**Thread类中的start()和run()方法有什么区别？**

start()方法被用来启动一个新线程，而且start()内部也会调用run()方法，但是这和直接调用run()方法不太一样，直接调用run()会在原有的线程中调用，不会有新的线程的启动，start()方法才会。

 

**Java中的volatile变量是什么？**

Volatile是一个特殊的修饰符，只有成员变量才能使用它，volatile关键字的作用是强制从公共堆栈中取得变量的值，而不是从线程的私有数据栈取得变量的值。使用volatile关键字增加了实例变量在多个线程之间的可见性，但volatile最致命的缺点是不支持原子性，是非线程安全的，也就是说JVM虚拟机只是保证从主内存加载到线程工作内存的值是最新的。

 

**什么是线程安全？Vector是一个线程安全类吗？**

如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量值也和预期的是一样的，就是线程安全的。Vector是用同步方法来实现线程安全的。

 

**Java中堆和栈有什么不同？**

栈是一块和线程紧密相关的内存区域，每个线程都有自己的栈内存，用于存储本地变量，方法参数和栈调用，一个线程中存储的变量对其他线程是不可见的。而堆是所有线程共享的一片公用内存区域，对象都在堆里创建，为了提升效率，线程会从堆中弄一个缓存到自己的栈。

 

**什么是线程池？为什么要使用它？**

创建线程要花费昂贵的资源和时间，如果任务来了才创建线程那么相应时间会变长，而且一个进程能创建的线程数有限。为了避免这些问题，在程序启动的时候就创建若干线程来响应处理，他们被称为线程池，里面的线程叫工作线程。从JDK1.5开始，JAVA API提供了Executor框架让你可以创建不同的线程池。线程池的创建如下所示：

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), handler);
 }  
```

corePoolSize：线程池核心线程数量

maximumPoolSize:线程池最大线程数量

keepAliverTime：当活跃线程数大于核心线程数时，空闲的多余线程最大存活时间

unit：存活时间的单位

workQueue：存放任务的队列

ThreadFactory : 创建新的线程

handler：超出线程范围和队列容量的任务的处理程序

线程池的实现原理：

提交一个任务到线程池中，线程池的处理流程如下：

1、        判断线程池里的核心线程是否都在执行任务，如果不是(核心线程空闲或者还有核心线程没有被创建)，则创建一个新的工作线程来执行任务。如果核心线程都在执行任务，则进入下个流程。

2、        线程池判断工作队列是否已满，如果工作队列没有满，则将新提交的任务存储在这个工作队列里。如果工作队列满了，则进入下个流程。

3、        判断线程池里的线程是否都处于工作状态，如果没有，则创建一个新的工作线程来执行任务，如果已经满了，则交给饱和策略来处理这个任务。

```
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
　　　　　　 //如果线程数大于等于基本线程数或者线程创建失败，将任务加入队列
        if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
　　　　　　　　　　//线程池处于运行状态并且加入队列成功
            if (runState == RUNNING && workQueue.offer(command)) {
                if (runState != RUNNING || poolSize == 0)
                    ensureQueuedTaskHandled(command);
            }
　　　　　　　　　//线程池不处于运行状态或者加入队列失败，则创建线程（创建的是非核心线程）
            else if (!addIfUnderMaximumPoolSize(command))
　　　　　　　　　　　//创建线程失败，则采取阻塞处理的方式
                reject(command); // is shutdown or saturated
        }
    }
```

创建线程的方法addIfUnderCorePoolSize()和addThread()

```
 private boolean addIfUnderCorePoolSize(Runnable firstTask) {
        Thread t = null;
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            if (poolSize < corePoolSize && runState == RUNNING)
                t = addThread(firstTask);
        } finally {
            mainLock.unlock();
        }
        if (t == null)
            return false;
        t.start();
        return true;
    }
```

```
private Thread addThread(Runnable firstTask) {
        Worker w = new Worker(firstTask);
        Thread t = threadFactory.newThread(w);
        if (t != null) {
            w.thread = t;
            workers.add(w);
            int nt = ++poolSize;
            if (nt > largestPoolSize)
                largestPoolSize = nt;
        }
        return t;
    }
```

这里将线程封装成工作线程worker，并放入工作线程组里, worker类的方法run方法如下：

```
 public void run() {
            try {
                Runnable task = firstTask;
                firstTask = null;
                while (task != null || (task = getTask()) != null) {
                    runTask(task);
                    task = null;
                }
            } finally {
                workerDone(this);
            }
  ![图片 6](/Users/alisa/Desktop/图片 6.png)![图片 6](/Users/alisa/Desktop/图片 6.png)}
```

worker在执行完任务后，还会通过getTask方法循环获取工作队里里的任务来执行。

RejectedExecutionHandler:饱和策略

当队列和线程池都满了，说明线程池处于饱和状态，那么必须对新提交的任务采用一种特殊的策略来进行处理，这个策略默认配置是AbortPolicy，表示无法处理新的任务而抛出异常。Java中提供了4种策略：

1、        AbortPolicy:直接抛出异常。

2、        CallerRunsPolicy:这个策略重试添加当前的任务，他会自动重复调用execute()方法，直到成功。

3、        DiscardOldestPolicy:丢弃队列里等待最久的一个任务，并执行当前任务。

4、        DiscardPolicy:不处理，丢弃掉。

Executor框架类图![图片 6](/Users/alisa/Desktop/图片 6.png) 

Executor接口是异步任务执行框架的基础，该框架能够支持多种不同类型的任务执行策略。

```
 public interface Executor {    
 	void execute(Runnable command);
 }

```

Executor接口就提供了一个执行方法，任务是Runnable类型，不支持Callable类型。

ExecutorService接口实现了Executor接口，主要提供关闭线程池和submit方法：

```
public interface ExecutorService extends Executor {

    List<Runnable> shutdownNow();


    boolean isTerminated();


    <T> Future<T> submit(Callable<T> task);

 }
```

另外该接口有两个重要的实现类：ThreadPoolExecutor和ScheduledThreadPoolExecutor。其中ThreadPoolExecutor是线程池的核心实现类，用来执行被提交的任务，而ScheduledThreadPoolExecutor是一个实现类，可以在给定的延迟后运行任务，或者定期执行命令。

Executors可以创建三种类型的ThreadPoolExecutor: SingleThreadExecutor、FixedThreadExecutor和CachedThreadPool。

 

SingleThreadExecutor:单线程线程池

```
ExecutorService threadPool = Executors.newSingleThreadExecutor();
```

```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
 }
```

FixedThreadExecutor：固定大小线程池

```
ExecutorService threadPool = Executors.newFixedThreadPool(5);
```

```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
 }
```

CachedThreadPool：无界线程池

```
ExecutorService threadPool = Executors.newCachedThreadPool();
```

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

无界线程池意味着没有工作队列，任务进来就执行，线程数量不够就创建，与前面两个的区别是：空闲的线程会被回收掉，空闲的时间是60s，这个适用于执行很多短期异步的小程序或者负载较轻的服务器。

 

**Callable与Runnable的区别？**

1、Callable定义的方法是call，而Runnable定义的方法是run。

2、call方法有返回值，而run方法是没有返回值的。

3、call方法可以抛出异常，而run方法是不能抛出异常。

 

**ThreadLocal的作用？**

类ThreadLocal主要解决的就是每个线程绑定自己的值，可以将ThreadLocal类比喻成全局存放数据的盒子，盒子中可以存储每个线程的私有数据。通过覆盖initialValue()方法给全局变量赋初始值，而且是线程之间隔离的。InheritableThreadLocal类可以在子线程中取得父线程继承下来的值，但在使用InheritableThreadLocal类需要注意的一点是，如果子线程在取得值的同时，主线程将InheritableThreadLocal中的值进行更改，那么子线程取到的值还是旧值。

 

**可重入锁(ReentrantLock)实现原理？**

<https://blog.csdn.net/yanyan19880509/article/details/52345422>

ReenTrantLock支持两种获取锁的方式，一种是公平模式，一种是非公平模式。先说公平锁模式：初始化时，state=0，表示无人抢占资源，这时候A线程请求锁，state+1，如图所示：![图片 15](/Users/alisa/Desktop/图片 15.png) 

线程A取得了锁，把state原子性+1，这时候state被改为1，A线程继续执行其他任务，然后线程B请求锁，线程B无法获取锁，生成节点进行排队，如下图：![图片 16](/Users/alisa/Desktop/图片 16.png)

这时候如果线程A又请求锁，是有特权的，只需修改状态，如下图：![图片 17](/Users/alisa/Desktop/图片 17.png)

这就是可重入锁，就是一个线程在获取了锁之后，再次去获取了同一个锁，这时候仅仅把状态值进行累加，那如果线程A释放了一次锁，就如下图所示：![图片 18](/Users/alisa/Desktop/图片 18.png)

仅仅是把状态值减了，只有线程A把此锁全部释放了，状态值减到0了，其他线程才有机会获取锁。当A把锁完全释放后，state恢复为0，然后会通知队列唤醒B线程节点，使B可以再次竞争锁。当然，如果B线程后面还有C线程，C线程继续休眠，除非B执行完了，通知C线程。注意，当一个线程节点被唤醒然后取得了锁，对应节点会从队列中删除。

非公平锁模型：就是当线程A执行完之后，要唤醒线程B是需要时间的，而且线程B醒来后还要再次竞争锁，所以如果在切换过程当中，来了一个线程C，那么线程C是有可能获取到锁的，如果C获取到了锁，B就只能继续乖乖休眠。

关键字synchronized与wait()和notify()/notifyAll()方法相结合可以实现等待/通知模式，类ReenTrantLock也可以实现同样的功能，但需要借助Condition对象。Condition类是JDK5中出现的技术，可以实现多路通知功能，也就是在一个Lock对象里面可以创建多个Condition实例，从而可以有选择性地进行线程通知，在线程调度上更加灵活，而synchronized就相当于整个Lock对象只有一个单一的Condition对象，所有的线程都注册在它一个对象身上。Condition使用await()和signal()方法实现等待/通知模式。

锁Lock分为”公平锁”和”非公平锁”。公平锁表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。而非公平锁就是一种获取锁的抢占机制，是随机获得锁的。

 

**ReentrantReadWriteLock是什么？**

读写锁表示有两个锁，一个是读操作相关的锁，也称为共享锁；另一个是写操作相关的锁，也叫排它锁。也就是多个读锁之间不互斥，读锁与写锁互斥，写锁与写锁互斥，即多个Thread可以同时进行读取操作，但是同一时刻只允许一个Thread进入写入操作。

 