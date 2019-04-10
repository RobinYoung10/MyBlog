---
title: java面试题收集（一）
date: 2019-03-21 15:57:02
tags: java
---

### 一、String，StringBuilder，StringBuffer三者的区别

String是字符串常量，StringBuilder和StringBuffer均为字符串变量，即String对象创建之后是不可更改的，后两者可以更改。如下代码：

```java
//String对象声明后不可改变
String str = "abc";
str.concat("def");
System.out.println(str);  //输出abc

//StringBuilder可改变
StringBuilder strbd = new StringBuilder().append("abc");
strbd.append("def");
System.out.println(strbd);  //输出abcdef
```

从线程安全上说，StringBuilder是线程不安全的，而StringBuffer是线程安全的。

### 二、java中==和equals()的区别

==是运算符，用于比较两个变量是否相等，而equals()是Object类的基本方法，用于比较两个对象是否相等。默认Object类的equals方法是比较两个对象的地址，此时和==的结果一样。换句话说：基本类型比较用==，比较的是他们的值。默认下，对象用==比较时，比较的是内存地址，如果需要比较对象内容，需要重写equal方法。

### 三、&和&&的区别

&是位操作，而&&是逻辑运算符。

### 四、final, finalize和finally的不同之处

final是一个修饰符，可以修饰变量、方法和类。如果final修饰变量，则该变量的值在初始化后不能改变。finalize方法是在对象被回收前调用的方法，给对象一个复活机会，但是什么时候调用finalize没有保证。finally是一个关键字，与try...catch..一起用于异常调用，不管有没有发生异常，finally块一定会被执行。

### 五、编写一个java singleton模式的类

思路：

1.通过定义`私有的构造函数`，使得从单例类的外部无法初始化该类，从而确保该类只有一个实例。

2.提供`共有、静态方法`访问该类的唯一实例。

基本实现：

```java
public final class MySingleton {
    private static final MySingleton INSTANCE = new MySingleton();
    private MySingleton();
    public static MySingleton getInstance() {
        return INSTANCE;
    }
}
```


























