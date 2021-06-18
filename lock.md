---
title: Java锁
date: 2021-06-18 10:41:00
tags: Java锁
categories: 多线程
---

### java锁

文章链接：https://tech.meituan.com/2018/11/15/java-lock.html

Java提供了种类丰富的锁，每种锁因其特性的不同，在适当的场景下能够展现出非常高的效率。本文旨在对锁相关源码（本文中的源码来自JDK 8和Netty 3.10.6）、使用场景进行举例，为读者介绍主流锁的知识点，以及不同的锁的适用场景。
下面给出本文内容的总体分类目录

<!-- more -->

![7f749fc8](C:\Users\kun.li\Desktop\7f749fc8.png)

##### 1、乐观锁和悲观锁

**悲观锁：**悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁

**乐观锁：**乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）

乐观锁在Java中是通过使用无锁编程来实现，最常采用的是CAS算法，Java原子类中的递增操作就通过CAS自旋实现的

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/c8703cd9.png)

**使用场景：**

- 悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确
- 乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大大提升

**代码实例**

```java
public class PessimismLock {

    //---------------- 悲观锁 -------------------------
    private static int index;
    //synchronized
    public synchronized void addInteger(){
        index++;
    }

    ReentrantLock reentrantLock = new ReentrantLock();
    public void modifyInteger(){
        reentrantLock.lock();
        try {
            index--;
        }finally {
            reentrantLock.unlock();
        }
    }

    //---------------- 乐观锁 -------------------------
    private AtomicInteger atomicInteger = new AtomicInteger();  // 需要保证多个线程使用的是同一个AtomicInteger
    public void addAtomicInteger(){
        atomicInteger.incrementAndGet();//自增
    }

}
```

**CAS (Compare And Swap)**

CAS，比较交换，是一种无锁算法，在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁

CAS算法涉及到三个操作数：

- 需要读写的内存值 V
- 进行比较的值 A
- 要写入的新值 B

当且仅当 V 等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的值（比较+更新整体是原子性操作），否则不会执行任何操作，一般情况下，更新是不断重试的操作

之前提到java.util.concurrent包中的原子类，就是通过CAS来实现了乐观锁，那么我们进入原子类AtomicInteger的源码，看一下AtomicInteger的定义：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    //获取并操作内存数据
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    //存储value在AtomicInteger中的偏移量
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    //存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的
    private volatile int value;
    ...
    ...
}
```

AtomicIntege 的自增函数 incrementAndGet() 的源码时，发现自增函数底层调用的是 unsafe.getAndAddInt()。但是由于JDK本身只有 Unsafe.class，只通过 class 文件中的参数名，并不能很好的了解方法的作用，所以我们通过 OpenJDK 8 来查看 Unsafe 的源码：

~~~java
// ------------------------- JDK 8 -------------------------
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}

// ------------------------- OpenJDK 8 -------------------------
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
~~~

根据OpenJDK 8的源码我们可以看出，getAndAddInt()循环获取给定对象o中的偏移量处的值 v，然后判断内存值是否等于 v。如果相等则将内存值设置为 v + delta，否则返回 false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。整个“比较+更新”操作封装在 compareAndSwapInt() 中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止

**CAS虽然很高效，但是它也存在三大问题：**

**1. ABA问题**

CAS 由三个步骤组成，分别是“读取->比较->写回”。 考虑这样一种情况，线程1和线程2同时执行 CAS 逻辑，两个线程的执行顺序如下：

- 时刻1：线程1执行读取操作，获取原值 A，然后线程被切换走 
- 时刻2：线程2执行完成 CAS 操作将原值由 A 修改为 B
- 时刻3：线程2再次执行 CAS 操作，并将原值由 B 修改为 A
- 时刻4：线程1恢复运行，将比较值（compareValue）与原值（oldValue）进行比较，发现两个值相等。 然后用新值（newValue）写入内存中，完成 CAS 操作

如上流程，线程1并不知道原值已经被修改过了，在它看来并没什么变化，所以它会继续往下执行流程。对于 ABA 问题，通常的处理措施是对每一次 CAS 操作设置版本号。java.util.concurrent.atomic 包下提供了一个可处理 ABA 问题的原子类 AtomicStampedReference，具体的实现这里就不分析了，有兴趣的朋友可以自己去看看

**ABA问题的解决办法**

1. 在变量前面追加版本号：每次变量更新就把版本号加1，则A-B-A就变成1A-2B-3A。 
2. atomic包下的AtomicStampedReference类：其compareAndSet方法首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用的该标志的值设置为给定的更新值。

**2. 循环时间长开销大**

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。 如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率

l