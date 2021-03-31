---
title: Java中的装箱和拆箱
categories: Java基础
abbrlink: 7c65aac7
date: 2018-03-15 07:08:51
tags:
---

### 一、什么是装箱？什么是拆箱？

Java为每种基本数据类型都提供了对应的包装器类型 ，基本类型到包装器类型之间的转换就是装箱和拆箱。

- 装箱：自动将基本数据类型转换为包装器类型
- 拆箱：自动将包装器类型转换为基本数据类型

```java
Integer i = 10;  //装箱
int n = i;   //拆箱
```

| 基本数据类型    | 包装器类型 |
| --------------- | ---------- |
| int（4字节）    | Integer    |
| byte（1字节）   | Byte       |
| short（2字节）  | Short      |
| long（8字节）   | Long       |
| float（4字节）  | Float      |
| double（8字节） | Double     |
| char（2字节）   | Character  |
| boolean（未定） | Boolean    |

<!--more-->

### 二、装箱和拆箱的实现

装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的。（xxx代表对应的基本数据类型）。 

如下面试题：

下面这段代码的输出结果是什么？

```java
public class Main {
    public static void main(String[] args) {
         
        Integer i1 = 100;
        Integer i2 = 100;
        Integer i3 = 200;
        Integer i4 = 200;
         
        System.out.println(i1==i2);
        System.out.println(i3==i4);
    }
}
```

输出结果是：

```java
true
false
```

输出结果表明i1和i2指向的是同一个对象，而i3和i4指向的是不同的对象。此时只需一看源码便知究竟，下面这段代码是Integer的valueOf方法的具体实现：

```java
public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
    }
```

而其中IntegerCache类的实现为：

```java
 private static class IntegerCache {
        static final int high;
        static final Integer cache[];

        static {
            final int low = -128;

            // high value may be configured by property
            int h = 127;
            if (integerCacheHighPropValue != null) {
                // Use Long.decode here to avoid invoking methods that
                // require Integer's autoboxing cache to be initialized
                int i = Long.decode(integerCacheHighPropValue).intValue();
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - -low);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }
```

从这两段代码可以看出，在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。

上面的代码中i1和i2的数值为100，因此会直接从cache中取已经存在的对象，所以i1和i2指向的是同一个对象，而i3和i4则是分别指向不同的对象。

**Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。** 

- Short类源码

```java
    public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }

    private static class ShortCache {
        private ShortCache(){}

        static final Short cache[] = new Short[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Short((short)(i - 128));
        }
    }
```

- Byte类源码

```java
 	public static Byte valueOf(byte b) {
        final int offset = 128;
        return ByteCache.cache[(int)b + offset];
    }

    private static class ByteCache {
        private ByteCache(){}

        static final Byte cache[] = new Byte[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Byte((byte)(i - 128));
        }
    }
```

- Character类源码

```java
    public static Character valueOf(char c) {
        if (c <= 127) { // must cache
            return CharacterCache.cache[(int)c];
        }
        return new Character(c);
    }
    
    private static class CharacterCache {
        private CharacterCache(){}

        static final Character cache[] = new Character[127 + 1];

        static {
            for (int i = 0; i < cache.length; i++)
                cache[i] = new Character((char)i);
        }
    }
```

- Long类源码

```java
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];
        }
        return new Long(l);
    }
    
    private static class LongCache {
        private LongCache(){}

        static final Long cache[] = new Long[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Long(i - 128);
        }
    }
```

**Double、Float的valueOf方法的实现是类似的。** 

- Double类源码

```java
    public static Double valueOf(double d) {
        return new Double(d);
    }
```

- Float类源码

```java
    public static Float valueOf(float f) {
        return new Float(f);
    }
```

### 三、题目测试

下面程序的输出结果是什么？ 

```java
public class Main {
    public static void main(String[] args) {
         
        Integer a = 1;
        Integer b = 2;
        Integer c = 3;
        Integer d = 3;
        Integer e = 321;
        Integer f = 321;
        Long g = 3L;
        Long h = 2L;
         
        System.out.println(c==d);
        System.out.println(e==f);
        System.out.println(c==(a+b));
        System.out.println(c.equals(a+b));
        System.out.println(g==(a+b));
        System.out.println(g.equals(a+b));
        System.out.println(g.equals(a+h));
    }
}
```

答案：

```java
true	//读取缓存内同一个对象
false	//在-128~127之外，新建对象
true	//a+b触发自动拆箱过程，比较它们值相等
true	//a+b先触发自动拆箱，再触发自动装箱
true	//a+b触发自动拆箱过程，比较它们值相等
false	//a+b先拆箱，然后通过Integer.valueOf装箱，所以类型不同
true	//a+h先拆箱，然后通过Long.valueOf装箱，为缓存内同一对象
```

注意事项：**当 "=="运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）。另外，对于包装器类型，equals方法并不会进行类型转换。** 