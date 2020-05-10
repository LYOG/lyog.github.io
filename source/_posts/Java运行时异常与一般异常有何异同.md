---
title: Java运行时异常与一般异常有何异同
categories: Java基础
abbrlink: 873428a7
date: 2018-03-09 07:08:51
tags:
---

Throwable是所有Java程序中错误处理的父类，有两种资类：Error和Exception。

- Error：表示由JVM所侦测到的无法预期的错误，由于这是属于JVM层次的严重错误，导致JVM无法继续执行，因此，这是不可捕捉到的，无法采取任何恢复的操作，顶多只能显示错误信息。
- Exception：表示可恢复的例外，这是可捕捉到的。 

Java提供了两类主要的异常:runtime exception和checked exception 

- checked 异常也就是我们经常遇到的IO异常，以及SQL异常都是这种异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch。这类异常一般是外部错误,例如试图从文件尾后读取数据等,这并不是程序本身的错误,而是在应用环境中出现的外部错误 。
- runtime exception，也称运行时异常，我们可以不处理。当出现这样的异常时，总是由虚拟机接管。出现运行时异常后，系统会把异常一直往上层抛，一直遇到处理代码。如果没有处理块，到最上层，如果是多线程就由Thread.run()抛出，如果是单线程就被main()抛出。抛出之后，如果是线程，这个线程也就退出了。如果是主程序抛出的异常，那么这整个程序也就退出了 。

<!-- more --> 

常见的运行时异常：

- ArrayStoreException：试图将错误类型的对象存储到一个对象数组时抛出的异常
- ClassCastException：试图将对象强制转换为不是实例的子类时，抛出该异常 
- IllegalArgumentException：抛出的异常表明向方法传递了一个不合法或不正确的参数
- IndexOutOfBoundsException ：指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出 
- NoSuchElementException：表明枚举中没有更多的元素 
- NullPointerException：当应用程序试图在需要对象的地方使用 null 时，抛出该异常 