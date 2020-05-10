---
title: HashMap在多线程情况下还会导致死循环吗
categories: Java基础
abbrlink: e05a5fab
date: 2018-06-10 22:56:44
tags:
---

先放结论，这个问题出现在JDK1.7及以前版本中，而现在JDK1.8中已经解决这个问题了。

#### JDK1.7中的死循环

我们知道HashMap<K,V>存放的数据量大于了装载因子（默认75%），那么HashMap<K,V>就需要进行扩容操作，扩容的空间大小就是原来空间的两倍，但是扩容的时候需要rehash操作,然后赋给新的HashMap<K,V>。

JDK中需要使用resize()函数进行扩容，下面时resize()的代码：

```java
void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        boolean oldAltHashing = useAltHashing;
        useAltHashing |= sun.misc.VM.isBooted() &&
                (newCapacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
        boolean rehash = oldAltHashing ^ useAltHashing;
        transfer(newTable, rehash);  //transfer函数的调用
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```

在resize()这个过程中,在并发情况下也是不会出现死循环的问题，关键问题是transfer函数的调用过程。

<!-- more --> 

下面时transfer()的源码：

```java
void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) { //这里才是问题出现的关键..
            while(null != e) {
                Entry<K,V> next = e.next;  //寻找到下一个节点..
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);  //重新获取hashcode
                e.next = newTable[i];  
                newTable[i] = e;
                e = next;
            }
        }
    }
```

while中的操作我们先简化成四个步骤：

```java
next = e.next;
e.next = newTable[i];
newTable[i] = e;
e = next;
```

下面来抽象一下发生死循环的过程：

```java
// 线程一开始执行while内的操作
next = e.next;
e.next = newTable[i];
newTable[i] = e;

// 紧接着线程二执行while内的操作
next = e.next;
e.next = newTable[i]; // 经线程一操作，此时newTable[i]实际上为e
```

上述情况导致e.next = e，造成一个闭环，最终形成死循环。

#### JDK1.8中的解决方案

在1.8中resize()方法不再调用transfer()方法，而是直接将原来transfer()方法中的代码写在自己方法体内； 同时还有一个重大改变，那就是：**扩容后，新数组中的链表顺序依然与旧数组中的链表顺序保持一致！** 

下面是resize()方法的部分源码：

```java
//如果扩容后，元素的index依然与原来一样，那么使用这个head和tail指针
Node<K,V> loHead = null, loTail = null
//如果扩容后，元素的index=index+oldCap，那么使用这个head和tail指针
Node<K,V> hiHead = null, hiTail = null
Node<K,V> next;
do {
    next = e.next;
    //这个地方直接通过hash值与oldCap进行与操作得出元素在新数组的index
    if ((e.hash & oldCap) == 0) {
        if (loTail == null)
            loHead = e;
        else
            loTail.next = e;  
        loTail = e;
    }
    else {
        if (hiTail == null)
            hiHead = e;
        else
            hiTail.next = e;   
        hiTail = e;
    }
} while ((e = next) != null);
```

实际抽象出来只有两步：

```java
// 1.添加一个节点
if (tail == null)
    head = e;
else
    tail.next = e;
// 2.tail指针往后移动一位，维持顺序    
tail = e;
```

这样就可以解决死循环问题了。

