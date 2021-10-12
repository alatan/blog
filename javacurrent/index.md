# Java并发编程概览


![Java并发编程概览](/images/current/concurrentOverview.png "Java并发编程概览")

## 并发三要素
### 可见性
* CPU缓存引起：CPU增加了缓存，以均衡与内存的速度差异导致。
* 一个线程对共享变量的修改，另外一个线程能够立刻看到。

### 原子性
* 分时复用引起：操作系统增加了进程、线程，以分时复用CPU，进而均衡CPU与I/O设备的速度差异导致。
* 一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

```java
    x = 10;        //语句1: 直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中
    y = x;         //语句2: 包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是合起来就不是原子性操作了。
    x++;           //语句3： x++包括3个操作：读取x的值，进行加1操作，写入新的值。
    x = x + 1;     //语句4： 同语句3
```
### 有序性
* 重排序引起：由于编译程序指令重排序优化指令执行次序，使得缓存能够得到更加合理地利用导致。
* 程序执行的顺序按照代码的先后顺序执行。
* 可参考多线程环境下初始化一个对象的过程来理解。<a href="#head">`点击查看`</a>


## 线程安全的实现方法
### 互斥同步(阻塞同步)
#### synchronized(JVM实现)
#### Lock&ReentrantLock(JDK实现)

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种**悲观的并发策略**，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁(这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

### 非阻塞同步
#### CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略: 先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施(不断地重试，直到成功为止)。

这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。 

**乐观锁需要操作和冲突检测这两个步骤具备原子性**，这里就不能再使用互斥同步来保证了，只能**靠硬件来完成**。

硬件支持的原子性操作最典型的是: 比较并交换(Compare-and-Swap，CAS)。

CAS指令需要有3个操作数，分别是内存地址V旧的预期值A和新值B。当执行操作时，只有当V的值等于A，才将V的值更新为B。

##### ABA问题
如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。 

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。

大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

#### AtomicInteger
J.U.C 包里面的整数原子类 AtomicInteger，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 Unsafe 类的 CAS 操作。

```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

### 无需同步方案
#### 栈封闭
多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为**局部变量存储在虚拟机栈中，属于线程私有的**。

#### 线程本地存储(ThreadLocal)
* 如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

* 符合这种特点的应用并不少见，大部分使用消费队列的架构模式(如“生产者-消费者”模式)都会将产品的消费过程尽量在一个线程中消费完。

* 其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”(Thread-per-Request)的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

#### 可重入代码
这种代码也叫做纯代码(Pure Code)，可以在代码执行的任何时刻中断它，转而去执行另外一段代码(包括递归调用它本身)，而在控制权返回后，原来的程序不会出现任何错误。 可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

## 解决并发
### 3个关键字
#### synchronized：原子性，可见性，有序性
#### volatile：有序性，可见性
##### 防重排序

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
<a id="head"/>
现在我们分析一下为什么要在变量singleton之间加上volatile关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤： 
1. 分配内存空间。
2. 初始化对象。 
3. 将内存空间的地址赋值给对应的引用。 

但是由于操作系统可以对指令进行重排序，所以上面的过程也可能会变成如下过程： 
1. 分配内存空间。 
2. 将内存空间的地址赋值给对应的引用。 
3. 初始化对象 
 
如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。

##### 实现可见性
volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现: 
* 内存屏障，又称内存栅栏，是一个 CPU 指令。 
* 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。
* 详细见：[volatile理论基础](https://www.pdai.tech/md/java/thread/java-thread-x-key-volatile.html  "volatile理论基础")

##### 使用 volatile 必须具备的条件
* 对变量的写操作不依赖于当前值。
* 该变量没有包含在具有其他变量的不变式中。
* 只有在状态真正独立于程序内其他内容时才能使用volatile。

#### final：有序性
* 写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域就不具有这个保障。
* 读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用。

### Happens-Before 规则 
*上面提到了可以用volatile和synchronized来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。*

1. 单一线程原则（在一个线程内，在程序前面的操作先行发生于后面的操作。）
2. 管程锁定规则（一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。）
3. volatile 变量规则（对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。）
4. 线程启动规则（Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。）
5. 线程加入规则（Thread 对象的结束先行发生于 join() 方法返回。）
6. 线程中断规则（对线程 interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过interrupted()方法检测到是否有中断发生。）
7. 对象终结规则 一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。
8. 传递性（如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。）

### 锁优化及JMM
新的JDK优化锁的实现保证并发，内存模型也会保证可见性。

## J.U.C框架
### Unsafe(CAS)和原子类
### AQS框架
**AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)。**

### 锁
* LockSupport
* ReentrantLock
* ReentrantReadWriteLock

### 并发集合
* ConcurrentHashMap
* CopyOnWriteArrayList
* ConcurrentLinkedQueue
* BlockingQueue

### 线程池
* FutureTask
* ThreadPoolExecutor
* ScheduledThreadPoolExecutor
* Fork/Join

### 工具类
* CountDownLatch
* CyclicBarrier
* Semaphore
* Phaser
* Exchanger
* ThreadLocal


## 参考文章
[Java 并发 - 理论基础](https://www.pdai.tech/md/java/thread/java-thread-x-theorty.html "Java 并发 - 理论基础")

