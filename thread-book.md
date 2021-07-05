---
title: Java并发编程之美
date: 2021-07-05 14:14:27
tags: Java并发编程
categories: Java编码书籍

---

### Java并发编程之美

#### 一、并发编程线程基础

##### 1.1 什么是线程

线程是进程中的一个实体，线程本身是不会独立存在的。进程是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，线程则是进程的一个执行路劲，一个进程至少有一个线程，进程中的多个线程共享进程的资源。

操作系统分配资源时是吧资源分配给进程，但 CPU 是吧资源分配给线程，因此占用 CPU 的是线程，所以线程是 CPU 分配的基本单位。CPU 一般使用时间片轮转的方式让线程轮询占有，所以当前线程时间片用完之后，需要让出 CPU，等待下次轮到自己时在执行

计数器如果执行的时本地方法，则记录的地址时 undefined，只有执行的时 java 代码时，才是下一条指令的地址

##### 1.2 线程的创建与运行

继承 Thread 类

```java
public class ThreadTask extends Thread{

    @Override
    public void run() {
        System.out.println("I am a child thread");
    }

    public static void main(String[] args) {
        new ThreadTask().start();
    }
}
```

实现 Runnable 接口

```java
public class RunnableTask implements Runnable{
    @Override
    public void run() {
        System.out.println("I am a child thread");
    }

    public static void main(String[] args) {
        RunnableTask task = new RunnableTask();
        new Thread(task).start();
        new Thread(task).start();
    }
}
```

实现 Callable 接口，FutureTask 方式

```java
public class CallerTask implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "hello world";
    }

    public static void main(String[] args) {
        //异步线程
        FutureTask<String> task = new FutureTask<String>(new CallerTask());
        //启动线程
        new Thread(task).start();
        try {
            String s = task.get();
            System.out.println(s);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

**Thread 方式：**在 run() 方法中直接使用 this 可获得当前线程，但 java 不支持多继承，如果继承了 Thread 类，就不能继承其他类了，其次任务和代码没有分离，当多个线程执行一样的任务就会需要多份任务代码。

**Runnable 方式：**任务与代码分离，多个线程可以共用同一份任务，并且可以继承其他类

**Callable 方式：**可以有任务的返回值

##### 1.3 线程通知与等待

线程调用共享变量的 wait() 方法时，该线程会被阻塞挂起，直到发生以下事情才返回：（1）其他线程调用的共享变量的 notify() 或则 notifyAll() 方法；（2）其他线程调用了该线程的 interrupt() 方法吗，该线程抛出 InterruptedException 异常返回

如果调用 wait() 方法时，没有事先获取该对象的监视器锁，会抛出 IllegalMonitorStateException 异常

##### 1.9 死锁

死锁产生条件：

- 互斥条件： 指线程对己经获取到的资源进行排它性使用，即该资源同时只由一个线程占用。如果此时还有其他线程请求获取该资源，则请求者只能等待，直至占有资源的线程释放该资源。
- 请求并持有条件：指一 线程己经持有了至少一个资源，但又提出了新的资源请求 而新资源己被其他线程占有，所以当前线程会被阻塞，但阻塞的同时并不释放自己经获取的资源。
- 不可剥夺条件 指线程获取到的资源在自己使用完之前不能被其他线程抢占，只有在自己使用完毕后才由自己释放该资源。
- 环路等待条件：指在发生死锁时，必然存在一个线程→资源的环形链，即线程集合 {TO , TL T2 ，…， Tn ｝中 TO 正在等待一个Tl 占用的资源，Tl 正在等待 T2 占用的资源，……Tn 在等待己被 TO 占用资源。

##### 1.10 守护线程和用户线程

##### 1.11 ThreadLocal

##### 1.12 InheritableThreadLocal

可以继承父线程的ThreadLocalMap

#### 二、并发编程的其他基础知识

##### 2.1 什么是多线程并发编程



