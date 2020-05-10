---
title: equals与hashCode方法的联系
categories: Java基础
abbrlink: d2ef011b
date: 2018-05-08 15:56:44
tags:
---

### equals方法

equals()是用来判断其他的对象是否和该对象相等。

equals()在object类中定义如下： 

```java
public boolean equals(Object obj) {  
    return (this == obj);  
}  
```

很明显是两个对象的地址值的比较。通常封装类有用到这个equals()时，都会对它进行重写。

它具有以下性质：

- 自反性：对任意不为null的引用值x，x.equals(x)一定为true
- 对称性：对任意不为null的引用值x和y，当x.equals(y)为true时，y.equals(x)也为true
- 传递性：对任意不为null的引用值x、y、z，当x.equals(y)为true，并且y.equals(z)也为true时，x.equals(z)也为true
- 一致性：对任意不为null的引用值x和y，在对象信息没有被改的情况下，多次调用x.equals(y)结果不变

**当equals()方法被override时，hashCode()也要被override，因为相等的对象，hashCode一定相等。**

<!-- more --> 

### hashCode方法

hashCode()会给对象返回一个hashCode值。

它的性质：

- 在一个Java应用的执行期间，如果一个对象提供给equals做比较的信息没有被修改的话，该对象多次调用hashCode方法，该方法必须始终如一返回同一个Integer。
- 如果两个对象根据equals方法是相等的，那么调用二者各自的hashCode方法必须产生同一个Integer结果。
- 并不要求根据equals(java.lang.Object)方法不相等的两个对象，调用二者各自的hashCode方法必须产生不同的integer结果。 

### hashCode方法和equals方法的联系

如何判断元素是否在HashSet中重复，过程如下：

1. 判断两个对象hashCode是否相等
   - 如果不相等，则认为两个对象也不相等
   - 如果相等，进行下一步，判断equals是否相等
2. 判断两个对象equals运算是否相等
   - 如果不相等，两个对象不相等
   - 如果相等，两个对象相等

**步骤一的作用主要是为了提高存储效率。**

