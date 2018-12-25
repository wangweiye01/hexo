---
title: 深度剖析Java拆箱与装箱
date: 2018-12-25 15:07:05
tags:
---

# 什么是拆箱和装箱

Java中为每种基本类型都提供了对应的包装器类型。拆箱就是把包装器类型转换为基本数据类型的过程；装箱就是把基本数据类型转换为包装器类型的过程。

```java
Integer i = 10; // 装箱

int b = i; // 拆箱
```

Java中拆箱和装箱的实现过程是：

以Double类型举例

拆箱：调用doubleValue方法;

装箱：调用Double.valueOf方法

# 面试相关问题

## Integer比较

```java
Integer i1 = 100;
Integer i2 = 100;
Integer i3 = 200;
Integer i4 = 200;

assertThat(i1 == i2).isEqualTo(true);
assertThat(i3 == i4).isEqualTo(false);
```

比较过程：

1. 调用Integer.valueOf()方法将各个变量装箱;

2. 装箱后比较各个变量指向的内存地址是否相同。

但是为什么以上两个结果不同呢？看一下Integer的valueOf方法的实现便知究竟

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

通过分析源码，在通过valueOf方法创建Integer对象的时候，如果数值在[-128,127]之间，便返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象。

## Double类型比较

那么Double类型是否相同呢？

```java
Double d1 = 100.0;
Double d2 = 100.0;
Double d3 = 200.0;
Double d4 = 200.0;

assertThat(d1 == d2).isEqualTo(false);
assertThat(d3 == d4).isEqualTo(false);
```

原因同样我们去看Double.valueOf的源码实现

```java
public static Double valueOf(String s) throws NumberFormatException {
    return new Double(parseDouble(s));
}
```

通过分析源码，double类型在每一次装箱时，都会新建一个对象。

## Boolean类型比较

```java
Boolean i1 = false;
Boolean i2 = false;
Boolean i3 = true;
Boolean i4 = true;
  
assertThat(i1 == i2).isEqualTo(true);
assertThat(i3 == i4).isEqualTo(true);
```

同样看Boolean.valueOf的源码实现

```java

public static final Boolean TRUE = new Boolean(true);

public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

# 关于equals和==

## ==

如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等；

如果作用于引用类型的变量，则比较的是所指向的内存地址

## equals

如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；

诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。