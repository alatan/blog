# JUC-并发编程利器


![](/images/current/juc/juc-overview.png "JUC概览")

## Lock框架和Tools类
![](/images/current/juc/juc-overview-lock.png "Lock框架和Tools类")

### 接口Condition 
> Condition为接口类型，它将 Object 监视器方法(wait、notify 和 notifyAll)分解成截然不同的对象，以便通过将这些对象与任意Lock实现组合使用，为每个对象提供多个等待set (wait-set)。其中，Lock替代了synchronized方法和语句的使用，Condition替代了Object监视器方法的使用。可以通过await(),signal()来休眠/唤醒线程。

### 接口Lock
> Lock为接口类型，Lock实现提供了比使用synchronized方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的Condition对象。

### 接口ReadWriteLock
> ReadWriteLock为接口类型， 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

### 抽象类AbstractOwnableSynchonizer
> AbstractOwnableSynchonizer为抽象类，可以由线程以独占方式拥有的同步器。此类为创建锁和相关同步器(伴随着所有权的概念)提供了基础。AbstractOwnableSynchronizer 类本身不管理或使用此信息。但是，子类和工具可以使用适当维护的值帮助控制和监视访问以及提供诊断。

### 抽象类AbstractQueuedLongSynchronizer(long)
> AbstractQueuedLongSynchronizer为抽象类，以 long 形式维护同步状态的一个 AbstractQueuedSynchronizer 版本。此类具有的结构、属性和方法与 AbstractQueuedSynchronizer 完全相同，但所有与状态相关的参数和结果都定义为 long 而不是 int。当创建需要 64 位状态的多级别锁和屏障等同步器时，此类很有用。

### 核心抽象类AbstractQueuedSynchronizer(int)
> AbstractQueuedSynchronizer为抽象类，其为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器(信号量、事件，等等)提供一个框架。此类的设计目标是成为依靠单个原子int值来表示状态的大多数同步器的一个有用基础。

### 锁常用类LockSupport
> LockSupport为常用类，主要作用就是挂起线程，唤醒线程。LockSupport的功能和"Thread中的 Thread.suspend()和Thread.resume()有点类似"，LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。

该流程在购物APP上非常常见，当你准备支付时放弃，会有一个支付失效，在支付失效期内可以随时回来支付，过期后需要重新选取支付商品。

这里基于LockSupport中park和unpark控制线程状态，实现的等待通知机制。

```java
public class LockAPI04 {
    public static void main(String[] args) throws Exception {
        OrderPay orderPay = new OrderPay("UnPaid") ;
        Thread orderThread = new Thread(orderPay) ;
        orderThread.start();
        Thread.sleep(3000);
        orderPay.changeState("Pay");
        LockSupport.unpark(orderThread);
    }
}
class OrderPay implements Runnable {
    // 支付状态
    private String orderState ;
    public OrderPay (String orderState){
        this.orderState = orderState ;
    }
    public synchronized void changeState (String orderState){
        this.orderState = orderState ;
    }
    @Override
    public void run() {
        if (orderState.equals("UnPaid")){
            System.out.println("订单待支付..."+orderState);
            LockSupport.park(orderState);
        }
        System.out.println("orderState="+orderState);
        System.out.println("订单准备发货...");
    }
}
```

### 锁常用类ReentrantLock 
> ReentrantLock为常用类，它是一个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。

### 锁常用类ReentrantReadWriteLock 
> ReentrantReadWriteLock是读写锁接口ReadWriteLock的实现类，它包括Lock子类ReadLock和WriteLock。ReadLock是共享锁，WriteLock是独占锁。

**基于读锁时，其他线程可以进行读操作，基于写锁时，其他线程读、写操作都禁止。**

```java
public class LockAPI03 {
    public static void main(String[] args) throws Exception {
        DataMap dataMap = new DataMap() ;
        Thread read = new Thread(new GetRun(dataMap)) ;
        Thread write = new Thread(new PutRun(dataMap)) ;
        write.start();
        Thread.sleep(2000);
        read.start();
    }
}
class GetRun implements Runnable {
    private DataMap dataMap ;
    public GetRun (DataMap dataMap){
        this.dataMap = dataMap ;
    }
    @Override
    public void run() {
        System.out.println("GetRun："+dataMap.get("myKey"));
    }
}
class PutRun implements Runnable {
    private DataMap dataMap ;
    public PutRun (DataMap dataMap){
        this.dataMap = dataMap ;
    }
    @Override
    public void run() {
        dataMap.put("myKey","myValue");
    }
}
class DataMap {
    Map<String,String> dataMap = new HashMap<>() ;
    ReadWriteLock rwLock = new ReentrantReadWriteLock() ;
    Lock readLock = rwLock.readLock() ;
    Lock writeLock = rwLock.writeLock() ;

    // 读取数据
    public String get (String key){
        readLock.lock();
        try{
            return dataMap.get(key) ;
        } finally {
            readLock.unlock();
        }
    }
    // 写入数据
    public void put (String key,String value){
        writeLock.lock();
        try{
            dataMap.put(key,value) ;
            System.out.println("执行写入结束...");
            Thread.sleep(10000);
        } catch (Exception e) {
            System.out.println("Exception...");
        } finally {
            writeLock.unlock();
        }
    }
}
```

### 锁常用类StampedLock 
> 它是java8在java.util.concurrent.locks新增的一个API。StampedLock控制锁有三种模式(写，读，乐观读)，一个StampedLock状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。

### 工具常用类CountDownLatch 
> CountDownLatch为常用类，它是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

### 工具常用类CyclicBarrier 
> CyclicBarrier为常用类，其是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。 

### 工具常用类Phaser 
> Phaser是JDK 7新增的一个同步辅助类，它可以实现CyclicBarrier和CountDownLatch类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。 

### 工具常用类Semaphore 
> Semaphore为常用类，其是一个计数信号量，从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。通常用于限制可以访问某些资源(物理或逻辑的)的线程数目。 

### 工具常用类Exchanger 
> Exchanger是用于线程协作的工具类, 主要用于两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange()方法交换数据，当一个线程先执行exchange()方法后，它会一直等待第二个线程也执行exchange()方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

### 对比
| 同步工具	| 同步工具与AQS的关联 |
| :-----   | :---|
|ReentrantLock	|  使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。  |
|Semaphore	|  使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。  |
|CountDownLatch	|  使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过。  |
|ReentrantReadWriteLock	|   使用AQS同步状态中的16位保存写锁持有的次数，剩下的16位用于保存读锁的持有次数。  |
|ThreadPoolExecutor	|  Worker利用AQS同步状态实现对独占线程变量的设置（tryAcquire和tryRelease）。  |

##  Collections: 并发集合
![](/images/current/juc/juc-overview-collection.png "并发集合")

## CAS,Unsafe和原子类
**JUC中多数类是通过volatile和CAS来实现的，CAS本质上提供的是一种无锁方案，而Synchronized和Lock是互斥锁方案; Java原子类本质上使用的是CAS，而CAS底层是通过Unsafe类实现的。**

### Atomic原子类
其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。 

## Executors线程池
![](/images/current/juc/juc-executors.png "线程池")

### Executor
> Executor接口提供一种将任务提交与每个任务将如何运行的机制(包括线程使用的细节、调度等)分离开来的方法。通常使用 Executor 而不是显式地创建线程。

### ExecutorService 
> ExecutorService继承自Executor接口，ExecutorService提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以关闭 ExecutorService，这将导致其停止接受新任务。关闭后，执行程序将最后终止，这时没有任务在执行，也没有任务在等待执行，并且无法提交新任务。 

### ScheduledExecutorService
> ScheduledExecutorService继承自ExecutorService接口，可安排在给定的延迟后运行或定期执行的命令。

###  AbstractExecutorService 
> AbstractExecutorService继承自ExecutorService接口，其提供 ExecutorService 执行方法的默认实现。此类使用 newTaskFor 返回的 RunnableFuture 实现 submit、invokeAny 和 invokeAll 方法，默认情况下，RunnableFuture 是此包中提供的 FutureTask 类。 

### FutureTask 
> FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由CAS来保证。

### 核心: ThreadPoolExecutor 
> ThreadPoolExecutor实现了AbstractExecutorService接口，也是一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。 线程池可以解决两个不同问题: 由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源(包括执行任务集时使用的线程)的方法。每个 ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。

### 核心: ScheduledThreadExecutor 
> ScheduledThreadPoolExecutor实现ScheduledExecutorService接口，可安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer。

### 核心: Fork/Join框架 
> ForkJoinPool 是JDK 7加入的一个线程池类。Fork/Join 技术是分治算法(Divide-and-Conquer)的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。

### 工具类: Executors 
> Executors是一个工具类，用其可以创建ExecutorService、ScheduledExecutorService、ThreadFactory、Callable等对象。它的使用融入到了ThreadPoolExecutor, ScheduledThreadExecutor和ForkJoinPool中。 
