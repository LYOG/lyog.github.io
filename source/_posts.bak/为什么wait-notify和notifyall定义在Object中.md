---
title: 为什么wait，notify和notifyall定义在Object中
categories: Java基础
abbrlink: 92da4c91
date: 2018-06-04 15:56:44
tags:
---

为什么wait，notify和notifyall定义在Object中，而不是定义在Thread类中？

1. wait和nofity在Java中主要是实现线程之间的通信，把它们定义在Object类中，可以使任何Java对象都可以拥有实现线程通讯机制的能力。
2. 每个对象都可以作为锁。
3. 在Java中，为了进入临界区代码段，线程需要获得锁并且等待锁可用。它们不知道哪些线程持有锁，但是它们知道锁由哪些线程持有。它们应该等待锁而不是去查找哪个线程在代码块中去要求它们释放锁。

<!-- more --> 