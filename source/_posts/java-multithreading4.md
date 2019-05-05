---
title: Java多线程学习（四）
date: 2019-05-03 15:44:33
tags: java多线程
---

### 一、volatile关键字

##### 单个线程死循环

看如下示例，自定义类PrintString的方法里面定义了一个while循环，判断条件为类的一个属性值为true，我们试图在运行类里面调用这个方法后，改变其属性值从而让判断条件为false：

```java
public class PrintString {
    private boolean isContinuePrint = true;

    public boolean isContinuePrint() {
        return isContinuePrint;
    }

    public void setContinuePrint(boolean continuePrint) {
        isContinuePrint = continuePrint;
    }

    public void printStringMethod() {
        try {
            while(isContinuePrint == true) {
                System.out.println("run printStringMethod threadName=" + Thread.currentThread().getName());
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class RunPrintString {
    public static void main(String[] args) {
        PrintString printString = new PrintString();
        printString.printStringMethod();
        System.out.println("I want to stop it! stopThread=" + Thread.currentThread().getName());
        printString.setContinuePrint(false);
    }
}
```

启动后的结果为，程序根本停不下来，因为while循环已经占据了线程，使得后面的代码不能执行，输出如下：

```
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
run printStringMethod threadName=main
...
```

##### 解决同步死循环

解决办法就是使用多线程，更改自定义类如下：

```java
public class PrintString implements Runnable {
    private boolean isContinuePrint = true;

    public boolean isContinuePrint() {
        return isContinuePrint;
    }

    public void setContinuePrint(boolean continuePrint) {
        isContinuePrint = continuePrint;
    }

    public void printStringMethod() {
        try {
            while(isContinuePrint == true) {
                System.out.println("run printStringMethod threadName=" + Thread.currentThread().getName());
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        printStringMethod();
    }
}
```

更改启动类如下，启动线程：

```java
public class RunPrintString {
    public static void main(String[] args) {
        PrintString printString = new PrintString();
        new Thread(printString).start();
        System.out.println("I want to stop it! stopThread=" + Thread.currentThread().getName());
        printString.setContinuePrint(false);
    }
}
```

运行结果如下：

```
I want to stop it! stopThread=main
run printStringMethod threadName=Thread-0
```

但是上述代码运行在-server服务器模式64bit的JVM时，还是会死循环。原因是变量`isContinuePrint`存在于公共堆栈及线程的私有堆栈中，在-server模式时为了线程运行的效率，线程一直在私有堆栈中去值，而`printString.setContinuePrint(false)`虽然被执行，更新的却是公共堆栈的值，所以会死循环。

##### 解决异步死循环

上述的问题其实是私有堆栈中的值和公共堆栈的值不同步造成的，解决问题就是使用volatile关键字，它的作用是在线程访问类中的变量时，强制性从公共堆栈中进行取值。

```java
volatile private boolean isContinuePrint = true;
```