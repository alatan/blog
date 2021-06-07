# LockSupport详解


##  LockSupport简介
LockSupport用来创建锁和其他同步类的基本线程阻塞原语。简而言之，当调用LockSupport.park时，表示当前线程将会等待，直至获得许可，当调用LockSupport.unpark时，必须把等待获得许可的线程作为参数进行传递，好让此线程继续运行。 

##  核心函数分析
在分析LockSupport函数之前，先引入sun.misc.Unsafe类中的park和unpark函数，因为LockSupport的核心函数都是基于Unsafe类中定义的park和unpark函数，下面给出两个函数的定义:

```java
public native void park(boolean isAbsolute, long time);
public native void unpark(Thread thread);
```

说明: 对两个函数的说明如下: 
* park函数，阻塞线程，并且该线程在下列情况发生之前都会被阻塞: ① 调用unpark函数，释放该线程的许可。② 该线程被中断。③ 设置的时间到了。并且，当time为绝对时间时，isAbsolute为true，否则，isAbsolute为false。当time为0时，表示无限等待，直到unpark发生。 
* unpark函数，释放线程的许可，即激活调用park后阻塞的线程。这个函数不是安全的，调用这个函数时要确保线程依旧存活。

###  park函数
park函数有两个重载版本，方法摘要如下

```java
public static void park()；
public static void park(Object blocker)；
```

说明: 两个函数的区别在于park()函数没有没有blocker，即没有设置线程的parkBlocker字段。park(Object)型函数如下。

```java
public static void park(Object blocker) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    // 获取许可
    UNSAFE.park(false, 0L);
    // 重点方法：重新可运行后再此设置Blocker，其他线程执行unpark()后继续
    setBlocker(t, null);
}
```
说明: 调用park函数时，首先获取当前线程，然后设置当前线程的parkBlocker字段，即调用setBlocker函数，之后调用Unsafe类的park函数，之后再调用setBlocker函数。

那么问题来了，为什么要在此park函数中要调用两次setBlocker函数呢? 

原因其实很简单，**调用park函数时，当前线程首先设置好parkBlocker字段，然后再调用Unsafe的park函数，此后，当前线程就已经阻塞了，等待该线程的unpark函数被调用，所以后面的一个setBlocker函数无法运行，unpark函数被调用，该线程获得许可后，就可以继续运行了，也就运行第二个setBlocker，把该线程的parkBlocker字段设置为null，这样就完成了整个park函数的逻辑。如果没有第二个setBlocker，那么之后没有调用park(Object blocker)，而直接调用getBlocker函数，得到的还是前一个park(Object blocker)设置的blocker，显然是不符合逻辑的。**

总之，必须要保证在park(Object blocker)整个函数执行完后，该线程的parkBlocker字段又恢复为null。所以，park(Object)型函数里必须要调用setBlocker函数两次。

setBlocker方法如下:
```java
private static void setBlocker(Thread t, Object arg) {
    // 设置线程t的parkBlocker字段的值为arg
    UNSAFE.putObject(t, parkBlockerOffset, arg);
}
```

### unpark函数
此函数表示如果给定线程的许可尚不可用，则使其可用。如果线程在 park 上受阻塞，则它将解除其阻塞状态。否则，保证下一次调用 park 不会受阻塞。如果给定线程尚未启动，则无法保证此操作有任何效果。具体函数如下:

```java
public static void unpark(Thread thread) {
    if (thread != null) // 线程为不空
        UNSAFE.unpark(thread); // 释放该线程许可
}
```
## 更深入的理解
### Thread.sleep()和Object.wait()的区别
* Thread.sleep()不会释放占有的锁，Object.wait()会释放占有的锁； 
* Thread.sleep()必须传入时间，Object.wait()可传可不传，不传表示一直阻塞下去； 
* Thread.sleep()到时间了会自动唤醒，然后继续执行； 
* Object.wait()不带时间的，需要另一个线程使用Object.notify()唤醒； 
* Object.wait()带时间的，假如没有被notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁； 
**其实，他们俩最大的区别就是Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。**

### Thread.sleep()和Condition.await()的区别
Condition.await()和Object.wait()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，**一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程**。

### Thread.sleep()和LockSupport.park()的区别 
LockSupport.park()还有几个兄弟方法——parkNanos()、parkUtil()等，我们这里说的park()方法统称这一类方法。 

* 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，**且都不会释放当前线程占有的锁资源**。 
* Thread.sleep()没法从外部唤醒，只能自己醒过来； 
* LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒； 
* Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出； 
* LockSupport.park()方法不需要捕获中断异常；
* Thread.sleep()本身就是一个native方法； 
* LockSupport.park()底层是调用的Unsafe的native方法

### Object.wait()和LockSupport.park()的区别 
二者都会阻塞当前线程的运行，他们有什么区别呢? 经过上面的分析相信你一定很清楚了，真的吗? 往下看！
* Object.wait()方法需要在synchronized块中执行； 
* LockSupport.park()可以在任意地方执行； 
* Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
* LockSupport.park()不需要捕获中断异常； 
* Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容； 
* LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，**一定会继续执行后续内容**； 
* 如果在wait()之前执行了notify()会怎样? 抛出IllegalMonitorStateException异常； 
* **如果在park()之前执行了unpark()会怎样? 线程不会被阻塞，直接跳过park()，继续执行后续内容**； 

park()/unpark()底层的原理是“二元信号量”，你可以把它相像成只有一个许可证的Semaphore，只不过这个信号量在重复执行unpark()的时候也不会再增加许可证，最多只有一个许可证。

### LockSupport.park()不会释放锁资源
LockSupport.park()不会释放锁资源，它只负责阻塞当前线程，**释放锁资源实际上是在Condition的await()方法中实现的。**
