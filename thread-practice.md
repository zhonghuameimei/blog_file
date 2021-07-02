---
title: 多线程练习题
date: 2021-07-02 14:25:27
tags: 多线程练习题
categories: 多线程
---

### 多线程练习题

##### 1、按序打印

我们提供了一个类：

~~~java
public class Foo {
  public void first() { print("first"); }
  public void second() { print("second"); }
  public void third() { print("third"); }
}
~~~

<!-- more -->

三个不同的线程 A、B、C 将会共用一个 Foo 实例。

- 一个将会调用 first() 方法
- 一个将会调用 second() 方法
- 还有一个将会调用 third() 方法

请设计修改程序，以确保 second() 方法在 first() 方法之后被执行，third() 方法在 second() 方法之后被执行。

链接：https://leetcode-cn.com/problems/print-in-order

```java
package com.thread;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class LockTest {
    public static void main(String[] args) {
        Foo foo = new Foo();
        Thread thread1 = new Thread(() -> {
            try {
                foo.first(() -> System.out.println("first"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread1.start();

        Thread thread2 = new Thread(() -> {
            try {
                foo.second(() -> System.out.println("second"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread2.start();

        Thread thread3 = new Thread(() -> {
            try {
                foo.third(() -> System.out.println("third"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread3.start();
    }
}

class Foo1 {

    public Foo1() {

    }
    private int flag = 0;

    ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void first(Runnable printFirst) throws InterruptedException {
        lock.lock();
        try {
            while (flag != 0){
                condition.await();
            }
            flag = 1;
            // printFirst.run() outputs "first". Do not change or remove this line.
            printFirst.run();
            condition.signalAll();
        }finally {
            lock.unlock();
        }
    }

    public void second(Runnable printSecond) throws InterruptedException {
        lock.lock();
        try {
            while (flag != 1){
                condition.await();
            }
            flag = 1;
            // printSecond.run() outputs "second". Do not change or remove this line.
            printSecond.run();
            condition.signalAll();
        }finally {
            lock.unlock();
        }

    }

    public void third(Runnable printThird) throws InterruptedException {
        lock.lock();
        try {
            while (flag != 2){
                condition.await();
            }
            flag = 1;
            // printThird.run() outputs "third". Do not change or remove this line.
            printThird.run();
            condition.signalAll();
        }finally {
            lock.unlock();
        }
    }

 }
```

##### 2、交替打印FooBar

我们提供一个类：

~~~java
class FooBar {
  public void foo() {
    for (int i = 0; i < n; i++) {
      print("foo");
    }
  }

  public void bar() {
    for (int i = 0; i < n; i++) {
      print("bar");
    }
  }
}
~~~

两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。
链接：https://leetcode-cn.com/problems/print-foobar-alternately

```java
public class FooBarTest {

    public static void main(String[] args) {
        StringBuffer sb = new StringBuffer("");
        int n = 1;
        FooBar fooBar = new FooBar(n);

        Thread thread1 = new Thread(() -> {
            try {
                fooBar.foo(() -> System.out.print("foo"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread thread2 = new Thread(() -> {
            try {
                fooBar.bar(() -> System.out.print("bar"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        thread1.start();
        thread2.start();
    }
}

class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    private int flag = 0;

    ReentrantLock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            lock.lock();
            try {
                while (flag != 0){
                    condition.await();
                }
                // printFoo.run() outputs "foo". Do not change or remove this line.
                printFoo.run();
                flag = 1;
                condition.signal();
            }finally {
                lock.unlock();
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            lock.lock();
            try {
                while (flag != 1) {
                    condition.await();
                }
                // printBar.run() outputs "bar". Do not change or remove this line.
                printBar.run();
                flag = 0;
                condition.signal();
            } finally {
                lock.unlock();
            }
        }
    }
}
```

##### 3、打印零与奇偶数

假设有这么一个类：

~~~java
class ZeroEvenOdd {
  public ZeroEvenOdd(int n) { ... }      // 构造函数
  public void zero(printNumber) { ... }  // 仅打印出 0
  public void even(printNumber) { ... }  // 仅打印出 偶数
  public void odd(printNumber) { ... }   // 仅打印出 奇数
}
~~~

相同的一个 ZeroEvenOdd 类实例将会传递给三个不同的线程：

- 线程 A 将调用 zero()，它只输出 0 。
- 线程 B 将调用 even()，它只输出偶数。
- 线程 C 将调用 odd()，它只输出奇数。

每个线程都有一个 printNumber 方法来输出一个整数。请修改给出的代码以输出整数序列 010203040506... ，其中序列的长度必须为 2n。
链接：https://leetcode-cn.com/problems/print-zero-even-odd

```java
public class ZeroEvenOdd {

    public static void main(String[] args) {
        Test test = new Test(10);

        new Thread(() -> {
            try {
                test.zero((int i) -> {
                    System.out.print(i);
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                test.odd((int i) -> {
                    System.out.print(i);
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                test.even((int i) -> {
                    System.out.print(i);
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}


class Test{
    private int n;

    public Test(int n) {
        this.n = n;
    }

    Object lock = new Object();
    private int index = 1;
    private int flag = 0;


    // printNumber.accept(x) outputs "x", where x is an integer.
    public void zero(IntConsumer printNumber) throws InterruptedException {
        while (index <= n){
            synchronized (lock){
                if (flag != 0){
                    lock.wait();
                    continue;
                }
                printNumber.accept(0);
                flag = 1;
                lock.notifyAll();
            }
        }
    }

    public void even(IntConsumer printNumber) throws InterruptedException {
        while (index <= n){
            synchronized (lock){
                if (index % 2 != 0 || flag != 1){
                    lock.wait();
                    continue;
                }
                printNumber.accept(index++);
                flag = 0;
                lock.notifyAll();
            }
        }
    }

    public void odd(IntConsumer printNumber) throws InterruptedException {
        while (index <= n){
            synchronized (lock){
                if (index % 2 == 0 || flag != 1){
                    lock.wait();
                    continue;
                }
                printNumber.accept(index++);
                flag = 0;
                lock.notifyAll();
            }
        }
    }
}
```

##### 4、交替打印字符串

编写一个可以从 1 到 n 输出代表这个数字的字符串的程序，但是：

- 如果这个数字可以被 3 整除，输出 "fizz"。
- 如果这个数字可以被 5 整除，输出 "buzz"。
- 如果这个数字可以同时被 3 和 5 整除，输出 "fizzbuzz"。


例如，当 n = 15，输出： 1, 2, fizz, 4, buzz, fizz, 7, 8, fizz, buzz, 11, fizz, 13, 14, fizzbuzz。

假设有这么一个类：

~~~java
class FizzBuzz {
  public FizzBuzz(int n) { ... }               // constructor
  public void fizz(printFizz) { ... }          // only output "fizz"
  public void buzz(printBuzz) { ... }          // only output "buzz"
  public void fizzbuzz(printFizzBuzz) { ... }  // only output "fizzbuzz"
  public void number(printNumber) { ... }      // only output the numbers
}
~~~

请你实现一个有四个线程的多线程版  FizzBuzz， 同一个 FizzBuzz 实例会被如下四个线程使用：

- 线程A将调用 fizz() 来判断是否能被 3 整除，如果可以，则输出 fizz。
- 线程B将调用 buzz() 来判断是否能被 5 整除，如果可以，则输出 buzz。
- 线程C将调用 fizzbuzz() 来判断是否同时能被 3 和 5 整除，如果可以，则输出 fizzbuzz。
- 线程D将调用 number() 来实现输出既不能被 3 整除也不能被 5 整除的数字。

链接：https://leetcode-cn.com/problems/fizz-buzz-multithreaded

```java
public class FizzBuzzTest{

    public static void main(String[] args) {
        FizzBuzz fizzBuzz = new FizzBuzz(15);
        new Thread(() -> {
            try {
                fizzBuzz.fizz(() -> {
                    System.out.print("fizz，");
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                fizzBuzz.buzz(() -> {
                    System.out.print("buzz，");
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                fizzBuzz.fizzbuzz(() -> {
                    System.out.print("fizzBuzz，");
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        new Thread(() -> {
            try {
                fizzBuzz.number((int i) -> {
                    System.out.print(i+"，");
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

    }
}

class FizzBuzz {
    private int n;

    public FizzBuzz(int n) {
        this.n = n;
    }

    Object lock = new Object();

    private int index = 1;

    // printFizz.run() outputs "fizz".
    public void fizz(Runnable printFizz) throws InterruptedException {
        while(index <= n) {
            synchronized (lock) {
                if (index % 3 == 0 && index % 5 != 0) {
                    printFizz.run();
                    index++;
                    lock.notifyAll();
                    continue;
                }
                lock.wait();
            }
        }
    }

    // printBuzz.run() outputs "buzz".
    public void buzz(Runnable printBuzz) throws InterruptedException {
        while(index <= n){
            synchronized (lock) {
                if (index % 5 == 0 && index % 3 != 0) {
                    printBuzz.run();
                    index++;
                    lock.notifyAll();
                    continue;
                }
                lock.wait();
            }
        }
    }

    // printFizzBuzz.run() outputs "fizzbuzz".
    public void fizzbuzz(Runnable printFizzBuzz) throws InterruptedException {
        while(index <= n) {
            synchronized (lock) {
                if (index % 3 == 0 && index % 5 == 0){
                    printFizzBuzz.run();
                    index++;
                    lock.notifyAll();
                    continue;
                }
                lock.wait();
            }
        }
    }

    // printNumber.accept(x) outputs "x", where x is an integer.
    public void number(IntConsumer printNumber) throws InterruptedException {
        while(index <= n){
            synchronized (lock) {
                if (index % 3 != 0 && index % 5 != 0) {
                    printNumber.accept(index);
                    index++;
                    lock.notifyAll();
                    continue;
                }
                lock.wait();
            }
        }
    }
}
```

##### 5、哲学家进餐

5 个沉默寡言的哲学家围坐在圆桌前，每人面前一盘意面。叉子放在哲学家之间的桌面上。（5 个哲学家，5 根叉子）

所有的哲学家都只会在思考和进餐两种行为间交替。哲学家只有同时拿到左边和右边的叉子才能吃到面，而同一根叉子在同一时间只能被一个哲学家使用。每个哲学家吃完面后都需要把叉子放回桌面以供其他哲学家吃面。只要条件允许，哲学家可以拿起左边或者右边的叉子，但在没有同时拿到左右叉子时不能进食。

假设面的数量没有限制，哲学家也能随便吃，不需要考虑吃不吃得下。

设计一个进餐规则（并行算法）使得每个哲学家都不会挨饿；也就是说，在没有人知道别人什么时候想吃东西或思考的情况下，每个哲学家都可以在吃饭和思考之间一直交替下去。

哲学家从 0 到 4 按 顺时针 编号。请实现函数 void wantsToEat(philosopher, pickLeftFork, pickRightFork, eat, putLeftFork, putRightFork)：

- philosopher 哲学家的编号。
- pickLeftFork 和 pickRightFork 表示拿起左边或右边的叉子。
- eat 表示吃面。
- putLeftFork 和 putRightFork 表示放下左边或右边的叉子。
- 由于哲学家不是在吃面就是在想着啥时候吃面，所以思考这个方法没有对应的回调。

给你 5 个线程，每个都代表一个哲学家，请你使用类的同一个对象来模拟这个过程。在最后一次调用结束之前，可能会为同一个哲学家多次调用该函数。

链接：https://leetcode-cn.com/problems/the-dining-philosophers

```java
public class DiningPhilosophers {

    ReentrantLock locks[] = new ReentrantLock[5];

    public DiningPhilosophers() {
        for (int i = 0; i < 5; i++) {
            locks[i] = new ReentrantLock();
        }
    }

    // call the run() method of any runnable to execute its code
    public void wantsToEat(int philosopher,
                           Runnable pickLeftFork,
                           Runnable pickRightFork,
                           Runnable eat,
                           Runnable putLeftFork,
                           Runnable putRightFork) throws InterruptedException {
        locks[philosopher].lock();
        locks[(philosopher+1) % 5].lock();

        pickLeftFork.run();
        pickRightFork.run();
        eat.run();
        putLeftFork.run();
        putRightFork.run();

        locks[philosopher].unlock();
        locks[(philosopher+1) % 5].unlock();
    }
}
```

