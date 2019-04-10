---
title: Java多线程学习（二）
date: 2019-04-09 14:31:26
tags: java多线程
---

### 一、停止线程

在Java中有3种方法可以终止正在运行的线程：

* 使用退出标志，使线程正常退出，也就是当run方法完成后线程停止。
* 使用stop方法强行终止线程，但是不推荐使用此方法，这个方法是不安全的，而且已被废弃。
* 使用interrupt方法中断线程。

**interrupt()方法**

interrupt()方法的使用效果并不像for+break那样，调用就马上停止线程。调用此方法只是在当前线程中打了一个停止的标记，并不是真的停止线程。

**判断线程是否是停止状态**

有两种方法判断线程的状态是否是停止的：

* this.interrupted()：测试当前线程是否已经中断。
* this.isInterrupted():测试线程是否已经中断。

先看this.interrupted()方法，此方法是声明为static的，它指测试当前线程是否已经中断，当前线程指运行this.interrupted()方法的线程。看如下代码：

```java
public class Run7 {
    public static void main(String args[]) {
        try {
            MyThread7 myThread7 = new MyThread7();
            myThread7.start();
            Thread.sleep(1000);
            myThread7.interrupt();
            System.out.println("是否停止1？ = " + myThread7.interrupted());
            System.out.println("是否停止2？ = " + myThread7.interrupted());
        } catch (Exception e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end!");
    }
}
```

输出如下：

```
...
i=50000
是否停止1？ = false
是否停止2？ = false
end!
```

这里打印出的是否停止1和2都是false，说明线程并未停止，这表明这个“当前线程”是main，它从未中断过，所以打印结果是两个false。

我们更改代码如下：

```java
public class Run7 {
    public static void main(String args[]) {
        Thread.currentThread().interrupt();
        System.out.println("是否停止1？ = " + Thread.interrupted());
        System.out.println("是否停止2？ = " + Thread.interrupted());
    }
}
```

上面代码运行结果输出如下：

```
是否停止1？ = true
是否停止2？ = false
end!
```

从结果看，interrupted()方法判断出当前线程是停止状态。但是第二个输出是false，这是因为interrupted()方法具有清除状态的功能，即如果线程已经是中断状态，则该方法清除中断状态，所以第2次调用返回false。

接下来看isInterrupted()方法，更改代码如下：

```java
public class Run7 {
    public static void main(String args[]) {
        try {
            MyThread7 myThread7 = new MyThread7();
            myThread7.start();
            Thread.sleep(500);
            myThread7.interrupt();
            System.out.println("是否停止1？ = " + myThread7.isInterrupted());
            System.out.println("是否停止2？ = " + myThread7.isInterrupted());
        } catch (Exception e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end!");
    }
}
```

程序运行结果会输出两个true，所以isInterrupted()并未清除状态标志。

总结一下，就是：

* this.interrupted():测试**当前**线程是否已经是中断状态，执行后将清除中断状态。
* this.isInterrupted():测试线程Thread对象是否已经是中断状态，但不清除状态标志。

##### 异常法停止线程

代码如下：

```java
public class MyThread8 extends Thread {
    @Override
    public void run() {
        super.run();
        try {
            for(int i = 0; i < 500000; i++) {
                if(this.interrupted()) {
                    System.out.println("已经是停止状态！退出！");
                    throw new InterruptedException();
                }
                System.out.println("i=" + (i + 1));
            }
        } catch (InterruptedException e) {
            System.out.println("进入run方法的catch");
            e.printStackTrace();
        }
    }
}
```

```java
public class Run8 {
    public static void main(String args[]) {
        try {
            MyThread8 myThread8 = new MyThread8();
            myThread8.start();
            Thread.sleep(1000);
            myThread8.interrupt();
        } catch (InterruptedException e) {
            System.out.println("main catch");
            e.printStackTrace();
        }
        System.out.println("end!");
    }
}
```

在run()方法的for循环里判断是否是中断状态，是停止状态则抛出异常来停止线程。

### 二、yield方法

yield()方法的作用是放弃当前的cpu资源，将它让给其他的任务去占用cpu执行时间。但放弃的时间不确定，有可能刚刚放弃，就马上获得了cpu时间片。看以下示例代码：

```java
public class MyThread9 extends Thread {
    @Override
    public void run() {
        long beginTime = System.currentTimeMillis();
        int count = 0;
        for(int i = 0; i < 50000000; i++) {
            Thread.yield();
            count = count + (i + 1);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("用时：" + (endTime - beginTime) + "毫秒");
    }
}
```

启动类代码如下：

```java
public class Run9 {
    public static void main(String args[]) {
        MyThread9 myThread9 = new MyThread9();
        myThread9.start();
    }
}
```

如果注释掉`Thread.yield()`代码，运行结果应该是用时很短，几十毫秒左右，而此例在我本地运行最终结果如下：

```
用时：21734毫秒
```