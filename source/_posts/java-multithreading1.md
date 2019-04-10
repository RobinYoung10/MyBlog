---
title: Java多线程学习（一）
date: 2019-03-29 19:24:37
tags: java多线程
---

本文参考高红岩著的Java多线程编程核心技术

### 一、进程与线程的概念

进程：进程是操作系统结构的基础，是一次程序的执行，它是系统进行资源分配和调度的一个独立单位，是受操作系统管理的基本运行单元。简单点理解，它就是“Windows任务管理器”中的列表的程序，运行再内存中的exe文件就可以理解为进程。

线程：线程可以理解为再进程中独立运行的子任务。比如一个浏览器时打开的多个标签页，每一个标签页都是一个子任务，都可以独立的运行一些操作。

---

### 二、使用多线程

实现多线程的方式主要有两种：一种时继承Thread类，另一种时实现Runnable接口。

##### 1.继承Thread类

首先来看Thread类的源码结构，如下：

```java
public class Thread implements Runnable
```

可以看出，其实Thread类实现了Runnable接口，它们之间具有多态关系。

我们来看简单的实现多线程方法，只要创建的类继承Thread，并重写run方法，再run方法中写线程执行的任务代码，如下：

```java
public class Mythread extends Thread {
    @Override
    public void run() {
        super.run();
        System.out.println("Mythread");
    }
}
```

运行线程，即新建Mythread对象，并执行start方法，代码如下：

```java
public class Run {
    public static void main(String[] args) {
        Mythread mythread = new Mythread();
        mythread.start();
        System.out.println("end");
    }
}
```

再运行多线程时，运行的结果与代码执行顺序或调用顺序无关，上面的运行类中“Mythread”和“end”输出的顺序是随机性的。

##### 2.实现Runnable接口

因为Java不支持多继承，所以如果创建的线程类已经有父类了，就需要实现Runnable接口。代码跟上面大同小异：

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("MyRunnable");
    }
}
```

使用这个线程类需要用到Thread的构造函数，传入MyRunnable对象，代码如下：

```java
public class MyRunnableTest {
    public static void main(String[] args) {
        Runnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();
        System.out.println("运行结束");
    }
}
```

需要注意的是，如上文所说，Java的Thread类也实现了Runnable接口，所以Thread的构造函数还可以传入一个Thread类的对象，这样做就可以将一个Thread对象中的run()方法交由其他的线程调用。

##### 3.实例变量与线程安全

自定义线程类中的实例变量对于其他线程有共享和不共享之分。

不共享变量数据的情况下每个线程维护着自己的变量，其他线程不能访问，如下所示：

```java
public class Mythread4 extends Thread {

    private int count = 5;

    public Mythread4(String name) {
        super();
        this.setName(name);
    }

    @Override
    public void run() {
        super.run();
        for(;count > 0; count--) {
            System.out.println("由" + this.currentThread().getName() + "计算，count=" + count);
        }
    }
}
```

运行类如下：

```java
public class Run4 {
    public static void main(String[] args) {
        Mythread4 a = new Mythread4("a");
        Mythread4 b = new Mythread4("b");
        Mythread4 c = new Mythread4("c");
        a.start();
        b.start();
        c.start();
    }
}
```

运行结果将会是每一个线程分别计算5次并输出，顺序随机。

共享变量数据需要用到上文所说的在Thread的构造函数中传入一个Thread类的对象，即在运行类中，只创建一个自定义线程类，把这个线程类交由几个不同的基本线程类操作调用，从而实现多个线程对同一个线程变量的共享，代码如下：

```java
public class Mythread5 extends Thread {
    private int count = 5;

    @Override
    public void run() {
        super.run();
        //在此处不要使用上一例的for循环，因为使用同步后一个线程就能循环完，其他线程得不到运行的机会
        count--;
        System.out.println("由" + this.currentThread().getName() + "计算，count=" + count);
    }
}
```

运行类如下：

```java
public class Run5 {
    public static void main(String[] args) {
        Mythread5 mythread = new Mythread5();
        Thread a = new Thread(mythread, "a");
        Thread b = new Thread(mythread, "b");
        Thread c = new Thread(mythread, "c");
        Thread d = new Thread(mythread, "d");
        Thread e = new Thread(mythread, "e");
        a.start();
        b.start();
        c.start();
        d.start();
        e.start();
    }
}
```

但是这样子处理会出现“非线程安全”问题，即本例中多个线程同时对count进行处理，有可能会出现多个线程输出的count值是相同的。

修改自定义线程类如下，在run方法前增加synchronized关键字，使得多个线程在执行run方法时，以排队的方式处理。当一个线程调用run之前，先判断run有没有被上锁，如果上锁了，即有其他线程在调用，必须等其他线程调用完后才可以执行：

```java
public class Mythread5 extends Thread {
    private int count = 5;

    @Override
    synchronized public void run() {
        super.run();
        //在此处不要使用for循环，因为使用同步后其他线程得不到运行的机会
        count--;
        System.out.println("由" + this.currentThread().getName() + "计算，count=" + count);
    }
}
```

非线程安全概念：多个线程对同一个对象中的同一个实例变量进行操作时会出现值被更改、值不同步的情况，进而影响程序的执行流程。

##### 4.一些线程方法

* **currentThread()方法**

此方法可返回代码段正在被哪个线程调用的信息。如下代码获取线程的名字：

```java
Thread.currentThread().getName();
```

* **isAlive()方法**

此方法判断线程当前是否还处于活动状态，返回Boolean值。

* **sleep()方法**

此方法作用是在指定的毫秒数内让当前“正在执行的线程”休眠。以下代码示例：

```java
public class Mythread6 extends Thread {
    @Override
    public void run() {
        try {
            System.out.println("run threadName=" + this.currentThread().getName() + "begin = " + System.currentTimeMillis());
            Thread.sleep(2000);
            System.out.println("run threadName=" + this.currentThread().getName() + "end = " + System.currentTimeMillis());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

上面的自定义线程类在start后输出分别输出两个毫秒数，第二个会在线程睡眠2秒后输出。启动类如下：

```java
public class Run6 {
    public static void main(String[] args) {
        Mythread6 mythread6 = new Mythread6();
        System.out.println("begin = " + System.currentTimeMillis());
        mythread6.start();
        System.out.println("end = " + System.currentTimeMillis());
    }
}
```

运行这个启动类后，由于Mythread6线程是后main线程运行的，所以会先打印main的begin和end输出，后打印Mythread6的begin，2秒后再输出end，如下：

```
begin = 1553944198222
end = 1553944198222
run threadName=Thread-0begin = 1553944198222
run threadName=Thread-0end = 1553944200223
```

* **getId()方法**

getId()方法的作用是取得线程的唯一标识。

