---  
title: "JVM运行时数据区（JVM内存结构）"  
date: 2018-05-03
weight: 70  
draft: false  
keywords: [""]  
description: "JVM运行时数据区"  
tags: ["JVM"]
categories: ["JVM"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## JVM 内存结构
*栈是运行时的单位，而堆是存储的单位。（**栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。堆解决的是数据存储的问题，即数据怎么放、放在哪**)。*

![](/images/jvm/jvm-jrm.png "JVM运行时数据区")

### 按照线程使用情况和职责分成两大类
#### 线程独享 （程序执行区域）
* 虚拟机栈、本地方法栈、程序计数器
* 不需要垃圾回收
#### 线程共享 （数据存储区域）
* 堆和方法区
* 存储类的静态数据和对象数据
* 需要垃圾回收

## 堆
*Java堆在JVM启动时创建内存区域去实现对象、数组与运行时常量的内存分配，它是虚拟机管理最大的，也是垃圾回收的主要内存区域 。*
### 堆内存划分
**核心逻辑就是三大假说，基于程序运行情况进行不断的优化设计。**

![](/images/jvm/jvm-heap.png "堆")

### 堆内存为什么会存在新生代和老年代
分代收集理论：当前商业虚拟机的垃圾收集器，大多数都遵循了“分代收集”（GenerationalCollection）的理论进行设计，**分代收集名为理论，实质是一套符合大多数程序运行实际情况的经验法
则**，它建立在两个分代假说之上：
* 弱分代假说（Weak Generational Hypothesis）：绝大多数对象都是朝生夕灭的。
* 强分代假说（Strong Generational Hypothesis）：熬过越多次垃圾收集过程的对象就越难以消亡。

这两个分代假说共同奠定了多款常用的垃圾收集器的一致的设计原则：**收集器应该将Java堆划分出不同的区域，然后将回收对象依据其年龄（年龄即对象熬过垃圾收集过程的次数）分配到不同的区域之中存储**。 如果一个区域中大多数对象都是朝生夕灭，难以熬过垃圾收集过程的话，那么把它们集中放在一起，每次回收时只关注如何保留少量存活而不是去标记那些大量将要被回收的对象，就能以较低代价回收到大量的空间；
* 如果剩下的都是难以消亡的对象，那把它们集中放在一块，虚拟机便可以使用较低的频率来回收这个区域。

这就同时**兼顾了垃圾收集的时间开销和内存的空间有效利用。**

### 内存模型变迁
#### JDK1.7
![](/images/jvm/jvm-jdk17.png "JDK1.7")

* Young 年轻区 ：主要保存年轻对象，分为三部分，Eden区、两个Survivor区。
* Tenured 年老区 ：主要保存年长对象，当对象在Young复制转移一定的次数后，对象就会被转移到Tenured区。
* Perm 永久区 ：主要保存class、method、filed对象，这部份的空间一般不会溢出，除非一次性加载了很多的类，不过在涉及到热部署的应用服务器的时候，有时候会遇到OOM :PermGen space 的错误。
* Virtual区： 最大内存和初始内存的差值，就是Virtual区。

#### JDK1.8
![](/images/jvm/jvm-jdk18.png "JDK1.8")

* 由2部分组成，新生代（Eden + 2*Survivor ） + 年老代（OldGen ）
* JDK1.8中变化最大是，Perm永久区用Metaspace进行了替换
* 注意：Metaspace所占用的内存空间不是在虚拟机内部，而是在本地内存空间中。区别于JDK1.7

#### JDK1.9
![](/images/jvm/jvm-jdk19.png "JDK1.9")

* 取消新生代、老年代的物理划分
* 将堆划分为若干个区域（Region），这些区域中包含了有逻辑上的新生代、老年代区域

## 虚拟机栈
*栈帧(Stack Frame)是用于支持虚拟机进行方法执行的数据结构*。

* 栈帧存储了方法的**局部变量表、操作数栈、动态连接和方法返回地址**等信息。每一个方法从调用至执行完成的过程，都对应着一个栈帧在虚拟机栈里从入栈到出栈的过程。
* 栈内存为线程私有的空间，每个线程都会创建私有的栈内存，生命周期与线程相同，每个Java方法在执行的时候都会创建一个**栈帧（Stack Frame）**。栈内存大小决定了方法调用的深度，栈内存过小则会导致方法调用的深度较小，如递归调用的次数较少。
* JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着入栈（进栈/压栈），方法执行结束出栈。
* 栈不存在垃圾回收问题。

![](/images/jvm/jvm-stack.png)

### 当前栈帧
* 一个线程中方法的调用链可能会很长，所以会有很多栈帧。只有**位于JVM虚拟机栈栈顶的元素才是有效的**，即称为**当前栈帧**，与这个栈帧相关连的方法称为**当前方法**，定义这个方法的类叫做**当前类**。
* 执行引擎运行的所有**字节码指令**都只针对**当前栈帧**进行操作。如果当前方法调用了其他方法，或者当前方法执行结束，那这个方法的栈帧就不再是当前栈帧了。

* 是线程私有的，生命周期和线程一致。
* 主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
* 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器。

### 什么时候创建栈帧
调用新的方法时，新的栈帧也会随之创建。并且随着程序控制权转移到新方法，新的栈帧成为了当前栈帧。方法返回之际，原栈帧会返回方法的执行结果给之前的栈帧(返回给方法调用者)，随后虚拟机将会丢弃此栈帧。

### 栈异常的两种情况：
* 如果线程请求的栈深度大于虚拟机所允许的深度（Xss默认1m），会抛出StackOverflowError异常
* 如果在创建新的线程时，没有足够的内存去创建对应的虚拟机栈，会抛出OutOfMemoryError异常【不一定】

## 本地方法栈
*本地方法栈和虚拟机栈相似，区别就是虚拟机栈为虚拟机执行Java服务（字节码服务），而本地方法栈为虚拟机使用到的Native方法（比如C++方法）服务*。
* 简单地讲，一个Native Method就是一个Java调用非Java代码的接口。

## 方法区
* 方法区（Method Area）是可供各个线程共享的运行时内存区域，方法区本质上是Java语言**编译后代码存储区域**，它存储每一个类的结构信息，例如：**运行时常量池**、成员变量、方法数据、构造方法和普通方法的字节码指令等内容。很多语言都有类似区域。
* 方法区的具体实现有两种：**永久代（PermGen）、元空间（Metaspace）**

### 方法区存储什么数据
![](/images/jvm/jvm-methodArea.png)

* 第一：Class
  1. 类型信息，比如Class（com.hero.User类）
  2. 方法信息，比如Method（方法名称、方法参数列表、方法返回值信息）
  3. 字段信息，比如Field（字段类型，字段名称需要特殊设置才能保存的住）
  4. 类变量（静态变量）：JDK1.7之后，转移到堆中存储
  5. 方法表（方法调用的时候） 在A类的main方法中去调用B类的method1方法，是根据B类的方法表去查找合适的方法，进行调用的。
* 第二：运行时常量池（字符串常量池）：从class中的常量池加载而来，JDK1.7之后，转移到堆中存储
  * 字面量类型
  * 引用类型-->内存地址
* 第三：JIT编译器编译之后的代码缓存

如果需要访问方法区中类的其他信息，都必须先获得Class对象，才能取访问该Class对象关联的方法信息或者字段信息。

### 永久代和元空间的区别是什么
1. JDK1.8之前使用的方法区实现是**永久代**，JDK1.8及以后使用的方法区实现是**元空间**。
2. 存储位置不同：
  * 永久代所使用的内存区域是JVM进程所使用的区域，它的大小受整个JVM的大小所限制。
  * 元空间所使用的内存区域是物理内存区域。那么元空间的使用大小只会受物理内存大小的限制。
3. 存储内容不同：
  * 永久代存储的信息基本上就是上面方法区存储内容中的数据。
  * 元空间只存储类的元信息，而静态变量和运行时常量池都挪到堆中了。

### 为什么要使用元空间来替换永久代
1. **字符串存在永久代中，容易出现性能问题和永久代内存溢出**。
2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
3. 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
4. Oracle 计划将HotSpot 与 JRockit 合二为一。

### 方法区实现变迁历史
![](/images/jvm/jvm-methodAreaHistory.png)

## 字符串常量池
![](/images/jvm/jvm-StringPool.png)

### 三种常量池的比较
#### class常量池：一个class文件只有一个class常量池
* 字面量：数值型（int、float、long、double）、双引号引起来的字符串值等
* 符号引用：Class、Method、Field等
#### 运行时常量池：一个class对象有一个运行时常量池
* 字面量：数值型（int、float、long、double）、双引号引起来的字符串值等
* 符号引用：Class、Method、Field等
#### 字符串常量池：全局只有一个字符串常量池
* 双引号引起来的字符串值

![](/images/jvm/jvm-StringPoolStorage.png)

### 字符串常量池如何存储数据
为了提高匹配速度， 即更快的查找某个字符串是否存在于常量池 Java 在设计字符串常量池的时候，还搞了一张StringTable， StringTable里面保存了**字符串的引用**。StringTable类似于HashTable（哈希
表）。在JDK1.7+，StringTable可以通过参数指定 -XX:StringTableSize=99991

#### 字符串常量池如何查找字符串
* 根据字符串的hashcode找到对应entry
* 如果没有冲突，它可能只是一个entry
* 如何有冲突，它可能是一个entry的链表，然后Java再遍历链表，匹配引用对应的字符串
* 如果找到字符串，返回引用
* 如果找不到字符串，在使用intern()方法的时候，会将intern()方法调用者的引用放入到stringtable中

### 代码示例
```java
/**
 * 总结：
 * 单独使用””引号创建的字符串都是常量，编译期就已经确定存储到String Pool中。
 * 使用new String(“”)创建的对象会存储到heap中，是运行期新创建的。
 * 使用只包含常量的字符串连接符如”aa”+”bb”创建的也是常量，编译期就能确定已经存储到StringPool中。
 * 使用包含变量的字符串连接如”aa”+s创建的对象是运行期才创建的，存储到heap中。
 * 运行期调用String的intern()方法可以向String Pool中动态添加对象。
 */
public class StringTableDemo {
    public static void main(String[] args) {
        HashMap<String, Integer> map = new HashMap<>();
        map.put("hello", 53);
        map.put("world", 35);
        map.put("java", 55);
        map.put("world", 52);
        map.put("通话", 51);
        map.put("重地", 55);
    //出现哈希冲突怎么办？
    //System.out.println("map = " + map);//
            test();
        }
    public static void test() {
        String str1 = "abc";
        String str2 = new String("abc");
        System.out.println(str1 == str2);//false
        String str3 = new String("abc");
        System.out.println(str3 == str2);//false
        String str4 = "a" + "b";
        System.out.println(str4 == "ab");//true
        String s1 = "a";
        String s2 = "b";
        String str6 = s1 + s2;
        System.out.println(str6 == "ab");//false
        String str7 = "abc".substring(0,2);
        System.out.println(str7 == "ab");//false
        String str8 = "abc".toUpperCase();
        System.out.println(str8 == "ABC");//false
        String s5 = "a";
        String s6 = "abc";
        String s7 = s5 + "bc";
        System.out.println(s6 == s7.intern());//true
    }
}
```

## 程序计数器
*此内存区域是唯一一个在Java的虚拟机规范中没有规定任何OutOfMemoryError异常情况的区域*。
程序计数器（Program Counter Register），也叫PC寄存器，是一块较小的内存空间，它可以看作是**当前线程所执行的字节码指令的行号指示器**。字节码解释器的工作就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。**分支，循环，跳转，异常处理，线程回复等都需要依赖这个计数器来完成**。

### 为什么需要程序计数器？
由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（针对多核处理器来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换（**系统上下文切换**）后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。
### 存储的什么数据？
如果一个线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是一个Native方法，这个计数器的值则为空。

## 直接内存
*本机直接内存的分配不会受到Java 堆大小的限制，受到本机总内存大小限制。*
直接内存并不是虚拟机运行时数据区的一部分，也不是Java 虚拟机规范中定义的内存区域。在JDK1.4 中新加入了NIO(New Input/Output)类，引入了一种基于通道(Channel)与缓冲区（Buffer）的I/O 方式，它可以使用native 函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

### 直接内存（堆外内存）与堆内存比较：
* 直接内存申请空间耗费更高的性能，当频繁申请到一定量时尤为明显
* 直接内存IO读写的性能要优于普通的堆内存，在多次读写操作的情况下差异明显

### 直接内存案例
```java
/** 直接内存案例 */

public class ByteBufferCompare {
    public static void main(String[] args) {
        //allocateCompare(); //分配比较
        operateCompare(); //读写比较
    }

    /**
     * 直接内存 和 堆内存的 分配空间比较
     * 结论： 在数据量提升时，直接内存相比非直接内的申请，有很严重的性能问题
     */
    public static void allocateCompare() {
        int time = 1000 * 10000; //操作次数,1千万
        long st = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            //ByteBuffer.allocate(int capacity) 分配一个新的字节缓冲区。
            ByteBuffer buffer = ByteBuffer.allocate(2); //非直接内存分配申请
        }
        long et = System.currentTimeMillis();
        System.out.println("在进行" + time + "次分配操作时，堆内存 分配耗时:" +
                (et - st) + "ms");
        long st_heap = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            //ByteBuffer.allocateDirect(int capacity) 分配新的直接字节缓冲区。
            ByteBuffer buffer = ByteBuffer.allocateDirect(2); //直接内存分配申请

        }
        long et_direct = System.currentTimeMillis();
        System.out.println("在进行" + time + "次分配操作时，直接内存 分配耗时:" +
                (et_direct - st_heap) + "ms");
    }

    /**
     * 直接内存 和 堆内存的 读写性能比较
     * 结论：直接内存在直接的IO 操作上，在频繁的读写时 会有显著的性能提升
     */
    public static void operateCompare() {
        int time = 10 * 10000 * 10000; //操作次数,10亿
        ByteBuffer buffer = ByteBuffer.allocate(2 * time);
        long st = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            // putChar(char value) 用来写入 char 值的相对 put 方法
            buffer.putChar('a');
        }
        buffer.flip();
        for (int i = 0; i < time; i++) {
            buffer.getChar();
        }
        long et = System.currentTimeMillis();
        System.out.println("在进行" + time + "次读写操作时，非直接内存读写耗时：" +
                (et - st) + "ms");
        ByteBuffer buffer_d = ByteBuffer.allocateDirect(2 * time);
        long st_direct = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            /
```

#### 输出：
* 在进行10000000次分配操作时，堆内存 分配耗时:82ms
* 在进行10000000次分配操作时，直接内存 分配耗时:6817ms
* 在进行1000000000次读写操作时，堆内存 读写耗时：1137ms
* 在进行1000000000次读写操作时，直接内存 读写耗时:512ms

### 从数据流的角度，来看
* 非直接内存作用链：本地IO–>直接内存–>非直接内存–>直接内存–>本地IO
* 直接内存作用链：本地IO–>直接内存–>本地IO

### 直接内存的使用场景：
* 有很大的数据需要存储，它的生命周期很长
* 适合频繁的IO操作，例如：网络并发场景