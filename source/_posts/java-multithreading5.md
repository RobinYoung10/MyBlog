---
title: Java多线程学习（五）----线程间通信（1）
date: 2019-05-03 17:15:16
tags: java多线程
---

### 一、等待/通知机制

##### 什么是等待/通知机制

方法`wait()`使当前执行代码的线程进行等待，并且在`wait()`所在的代码行处停止执行，直到接收到通知或被中断为止。只能在同步方法或同步代码块中调用wait方法，执行`wait()`方法后，当前线程释放锁。

方法`notify()`用来通知那些可能等待该对象的对象锁的其他线程。此方法同样需要在同步方法或同步代码块中使用，使用时也需要获得对象锁。如果有多个线程等待，则随机挑选其中一个，发出通知。

注意，在执行`notify()`方法后，当前线程不会马上释放该对象锁，要等到执行通知方法的线程将程序执行完后，才会释放锁。

一句话总结：wait使线程停止运行进入等待状态，而notify则使停止的线程继续运行。

查看如下例子：

```java
public class MyThreadA1 extends Thread {
    private Object lock;

    public MyThreadA1(Object lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            synchronized(lock) {
                System.out.println("begin wait time=" + System.currentTimeMillis());
                lock.wait();
                System.out.println("end wait time=" + System.currentTimeMillis());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class MyThreadA2 extends Thread {
    private Object lock;

    public MyThreadA2(Object lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        try {
            synchronized (lock) {
                System.out.println("begin notify time=" + System.currentTimeMillis());
                lock.notify();
                System.out.println("end notify time=" + System.currentTimeMillis());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class RunWait1 {
    public static void main(String[] args) throws Exception {
        Object lock = new Object();
        MyThreadA1 myThreadA1 = new MyThreadA1(lock);
        myThreadA1.start();
        Thread.sleep(3000);
        MyThreadA2 myThreadA2 = new MyThreadA2(lock);
        myThreadA2.start();
    }
}
```

MyThreadA1对象在同步代码块执行`wait()`，而MyThreadA2在同步代码块执行`notify()`，在启动类里分别传入相同的一个对象实例到两个线程类，运行结果如下：

```
begin wait time=1556874836693
begin notify time=1556874839709
end notify time=1556874839709
end wait time=1556874839709
```

##### 锁释放

方法wait()被执行后，锁会自动释放。但是方法notify()被执行后，锁却不自动释放，必须执行完notify()方法所在的同步synchronized代码块后才释放锁。

##### 唤醒所有线程

方法notify()仅随机唤醒一个线程。唤醒所有线程，要使用notifyAll()方法。

##### 方法wait(long)

带一个参数的方法wait(long)的功能是等待某一个时间内，是否有线程对锁进行唤醒，如果超过这个时间则自动唤醒。

### 方法join()

主线程创建并启动子线程，如果主线程想等待子线程执行完成之后再结束，比如子线程处理一个数据，主线程要取得这个数据中的值，就要用到join()方法了。方法join()的作用是等待线程对象销毁。

##### 方法join()的实现

如下代码，joinThread随机生成0~10的秒数，输出秒数并sleep这个秒数，在运行类里start后调用join方法：

```java
public class JoinThread extends Thread {
    @Override
    public void run() {
        try {
            int secondValue = (int) (Math.random() * 10000);
            System.out.println(secondValue);
            Thread.sleep(secondValue);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class RunJoinThread {
    public static void main(String[] args) throws Exception {
        JoinThread joinThread = new JoinThread();
        joinThread.start();
        joinThread.join();
        System.out.println("I want execute after JoinThread, I did it!");
    }
}
```

调用join方法后正常执行调用的线程，而使当前线程无限期阻塞，等待调用的线程销毁后再继续执行当前线程后面的代码，结果如下：

```
9731
I want execute after JoinThread, I did it!
```