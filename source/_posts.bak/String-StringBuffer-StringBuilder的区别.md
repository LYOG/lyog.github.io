---
title: String、StringBuffer、StringBuilder的区别
categories: Java基础
abbrlink: cccfa5c8
date: 2018-03-07 07:08:51
tags:
---

### 一、 字符串的不变性

String 字符串的不变性：**String 对象创建后则不能被修改，是不可变的**。  

下面看看 String 不变性的原因

```java
/** The value is used for character storage. */
private final char value[];

/** The offset is the first index of the storage that is used. *
private final int offset;

/** The count is the number of characters in the String. */
private final int count;
```

用于存放字符的数组被声明为final的，因此只能赋值一次，不可再更改。

重新赋值修改字符串**其实是创建了新的对象，所指向的内存空间不同**。  

### 二、 StringBuffer和StringBuilder介绍

StringBuilder：字符串变量（非线程安全）。在内部，StringBuilder对象被当作是一个包含字符序列的变长数组。

StringBuffer：字符串变量（Synchronized，即线程安全）。StringBuffer的线程安全，仅仅是保证jvm不抛出异常顺利的往下执行而已，它不保证逻辑正确和调用顺序正确。大多数时候，我们需要的不仅仅是线程安全，而是锁。

<!-- more --> 

### 三、String，StringBuffer和StringBuilder区别

#### 1、可变性

- String类中用于存放字符的数组被声明为final的，因此只能赋值一次，不可再更改。
- StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中用于存放字符的数组没有被声明为final的。所以可以知道StringBuilder与StringBuffer对象是可变的。

#### 2、线程安全性

- String中的对象是不可变的，也就可以理解为常量，显然线程安全。
- StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。看下源码：

```java
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```

- StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。看下源码：

```java
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```

#### 3、性能

- 普通情况下：StringBuilder >  StringBuffer  >  String 

- 由于StringBuilder 没有同步锁，所以速度上：StringBuilder >StringBuffer
- jdk1.5后，如果没有循环的情况下，把所有用加号连接的string运算都隐式的改写成StringBuilder ，单行用加号拼接字符串是没有性能损失的
- 但在有循环的情况下，编译器无法智能的替换成StringBuilder ，仍然会有不必要的性能损耗 