---
title: JavaSE基础查漏补缺1
date: 2019-04-11 13:26:36
tags: java
---

### 一、数据类型

##### 1.整型

Java提供了4种整型。

|类型|储存需求|
|----|----|
|byte|1字节|
|short|2字节|
|int|4字节|
|long|8字节|

##### 2.浮点类型

Java提供两种浮点类型。

|类型|存储需求|
|----|----|
|float|4字节|
|double|8字节|

float类型有效位数为6~7位，double位15位，很多情况下float类型的精度很难满足需求，所以一般程序都采用double类型。float类型的数值有一个后缀F（如3.14F），没有后缀F的浮点数值默认为double类型。double类型浮点数值后面也可以添加后缀D（如3.14D）。

##### 3.char类型

char类型占16位，即2字节。

##### 4.boolean类型

两个值：false和true。

### 二、构建字符串

众所周知，String类对象是不可变字符串，字符串“Hello”永远是包含H、e、l、l、o的代码单元序列，而不能改变其中任何一个字符。所以，我们一般改变String变量，让其引用另一个字符串的方式。但是这么做在每次链接字符串时，都会构建一个新的String对象，耗时耗空间。而使用StringBuilder类就可避免此问题。如下代码：

```java
StringBuilder builder = new StringBuilder();
//添加内容时，调用append方法
builder.append("Hello");
builder.append(" World!");
//调用toString()方法转换位字符串对象
String str = builder.toString();
```

### 三、大数值

如果基本的整数和浮点数精度无法满足需求，那么可以使用java.math包中的两个类：BitInteger和BigDecimal。这两个类可以处理任何长度数字序列的数值。前者是整数，后者是浮点数。

使用静态的valueOf方法可以将普通的数值转换为大数值：

```java
BigInteger bi = BigInteger.valueOf(100);
```

大数值不能使用算术运算符（如：+和*）。而需要使用大数值类中的方法，如：

```java
BigInteger c = a.add(b);  //c = a + b
BigInteger c = a.subtract(b);  //c = a - b
BigInteger c = a.multiply(b);  //c = a * b
BigInteger c = a.divide(b);  //c = a / b
```

具体查看java文档。