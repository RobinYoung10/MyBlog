---
title: Java多线程学习（三）
date: 2019-04-13 16:18:06
tags: java多线程
---

### 一、synchronized同步方法

synchronized同步方法主要用来解决“非线程安全”产生的“脏读”问题。在多个线程对同一个对象中的实例变量进行并发访问时容易发生“脏读”。

##### 1.方法内的变量为线程安全

在对象方法内部声明的变量，是不存在“非线程安全”问题的，“非线程安全”问题存在于“实例变量”中。

##### 2.实例变量非线程安全

当多个线程共同访问1个对象的实例变量时，有可能会出现“非线程安全”问题。看如下代码：

```java
public class HasSelfPrivateNum {
    private int num = 0;
    public void methodA(String username) {
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over!");
                Thread.sleep(2000);
            } else {
                num = 200;
                System.out.println("b set over!");
            }
            System.out.println(username + " num=" + num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ThreadA10 extends Thread {
    private HasSelfPrivateNum numRef;
    public ThreadA10(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.methodA("a");
    }
}
```

```java
public class ThreadB10 extends Thread {
    private HasSelfPrivateNum numRef;
    public ThreadB10(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.methodA("b");
    }
}
```

```java
public class Run10 {
    public static void main(String args[]) {
        HasSelfPrivateNum numRef = new HasSelfPrivateNum();
        ThreadA10 threadA10 = new ThreadA10(numRef);
        threadA10.start();
        ThreadB10 threadB10 = new ThreadB10(numRef);
        threadB10.start();
    }
}
```

两个线程类共享一个对象实例，当传入的username是“a”时，`Thread.sleep(2000)`使得线程睡眠2秒，在这2秒内线程`ThreadB10`把num改为了200，所以输出如下：

```
a set over!
b set over!
b num=200
a num=200
```

此问题的解决办法就是在对象的`public void methodA(String username)`方法前加上关键字synchronized，让此方法变成同步方法。重新运行启动类，输出如下：

```
a set over!
a num=100
b set over!
b num=200
```

结论就是，多个线程访问同一个对象实例中的同步方法时是线程安全的。

##### 3.多个对象多个锁

多个线程分别访问同一个类的两个不同实例的相同同步方法时，是以异步方式运行的。同样是上述实例类和两个线程类，我们将启动类更改为如下代码：

```java
public class Run10_1 {
    public static void main(String args[]) {
        HasSelfPrivateNum numRef1 = new HasSelfPrivateNum();
        HasSelfPrivateNum numRef2 = new HasSelfPrivateNum();
        ThreadA10 threadA10 = new ThreadA10(numRef1);
        threadA10.start();
        ThreadB10 threadB10 = new ThreadB10(numRef2);
        threadB10.start();
    }
}
```

将2个实例分别给2个线程，在系统中产生了2个锁，所以运行结果是异步的。原因是synchronized取得的锁是对象锁，而不是把一段代码或方法当成锁，所以示例里`HasSelfPrivateNum`的2个实例产生了2个锁，相互之间不共享。运行结果如下：

```
a set over!
b set over!
b num=200
a num=100
```

##### 4.synchronized方法与锁对象

假设有A、B两个线程，调用object对象。

* A线程先持有object对象的锁，B线程可以以异步的方式调用object对象中非synchronized类型的方法。
* A线程先持有object对象的锁，B线程如果在这时调用object对象的synchronized类型的其他方法则需要等待A线程，也就是同步。

### 二、synchronized同步语句块

synchronized同步方法有弊端，比如A线程调用同步方法执行一个较长时间的任务，B线程就需要等待较长时间。此问题可以使用synchronized同步语句块来解决。

##### 1.synchronized同步语句块的使用

如下代码：

```java
public class ObjectService {
    public void serviceMethod() {
        try {
            synchronized (this) {
                System.out.println("begin time=" + System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println("end time=" + System.currentTimeMillis());
            }
        } catch (InterruptedException e) {

        }
    }
}
```

当两个并发线程访问同一个对象的synchronized同步代码块时，一个时间内只能有一个线程执行，另一个线程需等待当前线程执行完才能执行。

##### 2.用同步代码块解决同步方法的弊端

看如下代码：

```java
public class Task {
    private String getData1;
    private String getData2;
    public void doLongTimeTask() {
        try {
            System.out.println("begin task");
            Thread.sleep(3000);
            String privateGetData1 = "长时间处理任务后返回值1 threadName=" + Thread.currentThread().getName();
            String privateGetData2 = "长时间处理任务后返回值2 threadName=" + Thread.currentThread().getName();
            synchronized (this) {
                getData1 = privateGetData1;
                getData2 = privateGetData2;
                System.out.println(getData1);
                System.out.println(getData2);
            }
            System.out.println("end task");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class CommonUtils {
    public static long beginTime1;
    public static long endTime1;
    public static long beginTime2;
    public static long endTime2;
}
```

```java
public class ThreadA11 extends Thread {
    private Task task;
    public ThreadA11(Task task) {
        super();
        this.task = task;
    }
    @Override
    public void run() {
        super.run();
        CommonUtils.beginTime1 = System.currentTimeMillis();
        task.doLongTimeTask();
        CommonUtils.endTime1 = System.currentTimeMillis();
    }
}
```

```java
public class ThreadB11 extends Thread {
    private Task task;
    public ThreadB11(Task task) {
        super();
        this.task = task;
    }
    @Override
    public void run() {
        super.run();
        CommonUtils.beginTime2 = System.currentTimeMillis();
        task.doLongTimeTask();
        CommonUtils.endTime2 = System.currentTimeMillis();
    }
}
```

```java
public class Run11 {
    public static void main(String args[]) {
        Task task = new Task();
        ThreadA11 threadA11 = new ThreadA11(task);
        threadA11.start();
        ThreadB11 threadB11 = new ThreadB11(task);
        threadB11.start();
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        long beginTime = CommonUtils.beginTime1;
        if(CommonUtils.beginTime1 > CommonUtils.beginTime2) {
            beginTime = CommonUtils.beginTime2;
        }
        long endTime = CommonUtils.endTime1;
        if(CommonUtils.endTime1 < CommonUtils.endTime2) {
            endTime = CommonUtils.endTime2;
        }
        System.out.println("耗时：" + ((endTime - beginTime) / 1000));
    }
}
```

其中Task类里使用到了synchronized同步代码块，如果`doLongTimeTask()`方法使用synchronize同步方法的方式，则线程调用时同步执行方法，每次都sleep睡眠3秒。而使用此同步代码块方式的话，当一个线程访问object的synchronized同步代码块时，另一个线程仍然可以访问该object对象中的非synchronized(this)同步代码块。运行结果如下：

```
begin task
begin task
长时间处理任务后返回值1 threadName=Thread-0
长时间处理任务后返回值2 threadName=Thread-0
end task
长时间处理任务后返回值1 threadName=Thread-1
长时间处理任务后返回值2 threadName=Thread-1
end task
耗时：3
```

需要说明的是，本例中不在synchronized同步代码块中的是异步执行，在synchronized同步代码块中的就是同步执行。

##### 3.synchronized同步代码块间的同步性

当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对同一个object中的所有其他synchronized(this)同步代码块的访问将被阻塞，这跟synchronized同步方法是一样的。

##### 4.synchronized(this)同步代码块是锁定当前对象的

跟synchronized同步方法一样，synchronized(this)同步代码块也是锁定当前对象的。

##### 5.同步方法与同步代码块

（1）synchronized同步方法

* 对其他synchronized同步方法或synchronized(this)同步代码块调用呈阻塞状态。
* 同一时间只有一个线程可以执行synchronized同步方法中的代码。

（2）synchronized(this)同步代码块

* 对其他synchronized同步方法或synchronized(this)同步代码块调用呈阻塞状态。
* 同一时间只有一个线程可以执行synchronized(this)同步代码块中的代码。

##### 6.将任意对象作为对象监视器

使用格式为：synchronized(非this对象x)。

看如下代码：

```java
public class Service {
    private String usernameParam;
    private String passwordParam;
    private String anyString = new String();
    public void setUsernamePassword(String username, String password) {
        try {
            synchronized (anyString) {
                System.out.println("线程名称：" + Thread.currentThread().getName() + "在" + System.currentTimeMillis() + "进入同步块");
                usernameParam = username;
                Thread.sleep(3000);
                passwordParam = password;
                System.out.println("线程名称：" + Thread.currentThread().getName() + "在" + System.currentTimeMillis() + "离开同步块");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class ThreadA12 extends Thread {
    private Service service;
    public ThreadA12(Service service) {
        super();
        this.service = service;
    }
    @Override
    public void run() {
        super.run();
        service.setUsernamePassword("a", "aa");
    }
}
```

```java
public class ThreadB12 extends Thread {
    private Service service;
    public ThreadB12(Service service) {
        super();
        this.service = service;
    }
    @Override
    public void run() {
        super.run();
        service.setUsernamePassword("b", "bb");
    }
}
```

```java
public class Run12 {
    public static void main(String args[]) {
        Service service = new Service();
        ThreadA12 threadA12 = new ThreadA12(service);
        threadA12.setName("A");
        threadA12.start();
        ThreadB12 threadB12 = new ThreadB12(service);
        threadB12.setName("B");
        threadB12.start();
    }
}
```

可以看出，我们使用synchronized同步代码块传入的是一个自定义的String对象，锁非this对象具有一些优点：如果在一个类中有很多个synchronized方法，这时能实现同步，但会受到阻塞；如果同步代码块锁非this对象，则synchronized(非this对象x)代码块中的程序与synchronize同步方法是异步的，可大大提高效率。