---
title: 设计模式-单例模式
date: 2018-06-11 09:36:33
tags:
categories: 设计模式
---

# 定义

是一种常用的软件设计模式，在它的核心结构中只包含一个被称为单例的特殊类。一个类只有一个实例，即一个类只有一个对象的实例

# 形式

单例模式可以分为懒汉式和饿汉式

懒汉式：在类加载时不初始化

饿汉式：在类加载时就完成了初始化，所以类加载比较慢，但获取对象的速度快

# 懒汉式

下面试最标准也是最原始的单例模式

```
public class Singleton {
    private static Singleton singleton;

    private Singleton() {}

    public static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }

        return singleton;
    }
}
```

不考虑并发情况下，以上方法主要靠以下来限制实例的单一性

1. 静态实例，带有static关键字的属性在每一个类中都是唯一的

2. 私有化构造函数，限制了随意创造实例的可能性

3. 公有获取实例的静态方法。为什么是静态？这个方法是供消费者获取对象实例的，如果是非静态的话，需要使用对象来调用，这就形成了矛盾


## 线程安全的单例

上面的方法在高并发情况下，肯定会出现有多个实例的情况-当一个线程判断为空，但是又没创建实例时，另一线程仍判断为空，这时就会出现单例模式非单例的情况

解决以上问题首先可能想到的方法如下

```
public class BadSynchronizedSingleton {

    //一个静态的实例
    private static BadSynchronizedSingleton synchronizedSingleton;
    //私有化构造函数
    private BadSynchronizedSingleton(){}
    //给出一个公共的静态方法返回一个单一实例
    public synchronized static BadSynchronizedSingleton getInstance(){
        if (synchronizedSingleton == null) {
            synchronizedSingleton = new BadSynchronizedSingleton();
        }
        return synchronizedSingleton;
    }
}
```

上面的做法很简单，就是将整个获取实例的方法同步，这样在一个线程访问这个方法时，其它所有的线程都要处于挂起等待状态，倒是避免了刚才同步访问创造出多个实例的危险，但是我只想说，这样的设计实在是糟糕透了，这样会造成很多无谓的等待

其实我们同步的地方只是需要发生在单例的实例还未创建的时候，在实例创建以后，获取实例的方法就没必要再进行同步控制了。下面我们将上面的示例修改一下，变成标准版的单例模式，也称为双重加锁

```
public class SynchronizedSingleton {

    //一个静态的实例
    private static SynchronizedSingleton synchronizedSingleton;
    //私有化构造函数
    private SynchronizedSingleton(){}
    //给出一个公共的静态方法返回一个单一实例
    public static SynchronizedSingleton getInstance(){
        if (synchronizedSingleton == null) {
            synchronized (SynchronizedSingleton.class) {
                if (synchronizedSingleton == null) {
                    synchronizedSingleton = new SynchronizedSingleton();
                }
            }
        }
        return synchronizedSingleton;
    }
}
```

这种做法与上面那种最无脑的同步做法相比就要好很多了，因为我们只是在当前实例为null，也就是实例还未创建时才进行同步，否则就直接返回，这样就节省了很多无谓的线程等待时间，值得注意的是在同步块中，我们再次判断了synchronizedSingleton是否为null，解释下为什么要这样做。

假设我们去掉同步块中的是否为null的判断，有这样一种情况，假设A线程和B线程都在同步块外面判断了synchronizedSingleton为null，结果A线程首先获得了线程锁，进入了同步块，然后A线程会创造一个实例，此时synchronizedSingleton已经被赋予了实例，A线程退出同步块，直接返回了第一个创造的实例，此时B线程获得线程锁，也进入同步块，此时A线程其实已经创造好了实例，B线程正常情况应该直接返回的，但是因为同步块里没有判断是否为null，直接就是一条创建实例的语句，所以B线程也会创造一个实例返回，此时就造成创造了多个实例的情况。


# 饿汉式

```
public class SingletonDemo {
    private static SingletonDemo instance = new SingletonDemo();

    private SingletonDemo () {}

    public static SingletonDemo getInstance() {
        return instance;
    }
}
```

这种方式基于类加载机制避免了多线程同步问题
