---
title: Java源码分析之Object类
tags:
  - Java
  - 源码
categories: Java基础
abbrlink: 8946258a
date: 2017-12-31 22:31:01
---

最近在研究java常见类的源码，夯实一下基础。今天先总结一下Object类。

Object类是java中所有类的基类，位于java.lang包中。而java.lang包又是java中最基础最核心的包，无需手动导入，编译时自动导入。

### Native方法

在讲Object之前，先来补充一下知识点，什么是Native方法。

"A native method is a Java method whose implementation is provided by non-java code."

简单来说就是：**一个native方法是由非java语言来实现的java方法**。 

通常情况下在定义一个native方法时，不去提供实体类，因为实体类由非java语言在外面已经实现。所以这也意味着**native不能与abstract方法连用**。那为什么要用Native呢，当然是因为某些高效率任务用java不易实现，而像C/C++这样的语言相对来说容易些。

-----

Object类里面没有定义属性，里面只包含13种方法。

### registerNatives()

```java
private static native void registerNatives();
static {
  registerNatives();
}
```

registerNatives()被native修饰，意味着此方法并不是在Java中实现的。

简单来说registerNatives()的作用是将非java语言实现的方法与java中的native方法一一对应起来，就像英文名Zhang San和中文名张三匹配起来一样。

下面的static方法块的作用就是这个类在被类加载器加载时执行一次，且仅执行一次。

<!-- more --> 

### getClass()

```java
public final native Class<?> getClass();
```

getClass()也是一个native方法，其作用是返回当前对象的类。等一下，类不是对象的描述嘛，对象是类的实现啊，描述也能返回吗？哈哈，看见Class<?>了嘛，它是描述类的类，姑且称为**元类**吧，实际返回的是元类的对象。

### hashCode()

```java
public native int hashCode();
```
也是native方法，返回的是int型的哈希码。那它的作用是什么呢？当然是为用到哈希表功能的地方行方便了。

有了哈希码，直接计算出地址来判断是否对象存在，就不用每次都遍历然后equals()了。

### equals()

```java
public boolean equals(Object obj) {
  return (this == obj);
}
```
此方法用来判断**两个对象的内容**是否相等。常规的==判断的是内存地址是否相等。等一下，不是说判断对象不用==用equals的嘛，你不也是==实现的嘛，坑爹呢吧。哈哈，人家是可以被重写的嘛。看看String，重写成逐个char对比的。当然重写了你也**别忘了重写hashCode()方法啊**。为啥要重写hashCode()啊？来自成文的规定：

- 在java应用程序执行期间，如果在equals方法比较中所用的信息没有被修改，那么在同一个对象上多次调用hashCode方法时必须一致地返回相同的整数。如果多次执行同一个应用时，不要求该整数必须相同。
- 如果两个对象通过调用equals方法是相等的，那么这两个对象调用hashCode方法必须返回相同的整数。
- 如果两个对象通过调用equals方法是不相等的，不要求这两个对象调用hashCode方法必须返回不同的整数。但是程序员应该意识到对不同的对象产生不同的hash值可以提供哈希表的性能。

看到了吧，为啥两个中一个变了，另一个也要重写了吧。

### clone()

```java
protected native Object clone() throws CloneNotSupportedException;
```
又又又是native方法，克隆作用是在堆中新建一个和被克隆对象一摸一样的对象，然后返回这个新对象的地址。

但是有个坑要注意，对象中成员变量是基本类型好说，如果也是对象的话，成员变量的值其实可是个引用啊，被克隆回来也是个引用啊，所以那个被指向的对象内容变了，那么克隆的和被克隆对象都会被影响，这就是所谓的**浅拷贝**。

克隆就克隆，后面那throws CloneNotSupportedException是啥意思，为啥要用protected修饰。

这里我们要说一下protected，单protected就可以写一篇文章了。这里只说一下：**当两个类不在同一个包中的时候，只能在子类内部通过子类的引用访问父类的protected属性或者方法**。比如写了一个Object的子类ObjectSon，然后去另外一个包下面new ObjectSon()，这时候你是调不到ObjectSon里面的protected属性和方法的。

也就是说你想要在此对象外面用clone方法，必须将clone()改成public的。改成public后，它的子类或者子子孙孙类都具有clone功能了，这也不是我们想要的。所以设计者做了个开关：Cloneable接口，只有实现该接口，并且public clone()，才能在外面使用clone功能。不实现Cloneable接口报Object的CloneNotSupportedException错误。

要想正确使用克隆，只有通过clone()和Cloneable接口组合。

### toString()

```java
public String toString() {
  return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
看代码就知道返回的是对象的类型加上其哈希值，中间用@隔开。由上面equals()第三条规范可知，同一个哈希值并不一定代表同一个对象，所以toString()返回结果相同也未必会是同一个对象。

### notify()

```java
public final native void notify();
```
在多线程中用到的方法，其作用是唤醒在此对象监视器上等待的单个线程。

### notifyAll()

```java
public final native void notifyAll();
```
同样也是多线程方法，但是它是唤醒监视器上所有等待的线程。

### wait()

```java
public final void wait() throws InterruptedException {
  wait(0);
}

public final native void wait(long timeout) throws InterruptedException;

public final void wait(long timeout, int nanos) throws InterruptedException {
  if (timeout < 0) {
    throw new IllegalArgumentException("timeout value is negative");
  }

  if (nanos < 0 || nanos > 999999) {
    throw new IllegalArgumentException(
      "nanosecond timeout value out of range");
  }

  if (nanos > 0) {
    timeout++;
  }

  wait(timeout);
}
```
wait()的作用是将当前线程设为等待，直到被notify()/notifyAll()唤醒。

wait(long timeout)和上面相同，不过超过了参数里设定的时间后，也会自动唤醒，**timeout的单位是毫秒**。

wait(long timeout, int nanos)和wait(long timeout)的作用相同，**timeout和nanos的单位是纳秒：1000000*timeout+nanos**。
### finalize()
```java
protected void finalize() throws Throwable { }
```
其作用是告诉jvm，垃圾该回收了，然后就是默默等待回收就行了。因为有垃圾回收机制，通常我们不去主动调用它，除非一种情况是此**对象不是由java来生成的**，垃圾回收机制监管不到，只能通过主动调用告诉它该回收了。
### Object()
```java
public Object(){};
```
默认的类构造器，不需要啥特殊解释。