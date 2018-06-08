---
title: 设计模式-观察者模式
date: 2018-06-08 15:27:38
tags:
top: 111
---

# 定义

观察者模式（有时又被称为发布-订阅模式、模型-视图模式、源-收听者模式或从属者模式）是软件设计模式的一种。在此种模式中，一个目标物件管理所有相依于它的观察者物件，并且在它本身的状态改变时主动发出通知。这通常透过呼叫各观察者所提供的方法来实现。此种模式通常被用来实现事件处理系统。

上面的定义当中，主要有这样几个意思，首先是有一个目标的物件，通俗点讲就是一个类，它管理了所有依赖于它的观察者物件，或者通俗点说是观察者类，并在它自己状态发生变化时，主动发出通知。

简单点概括成通俗的话来说，就是一个类管理着所有依赖于它的观察者类，并且它状态变化时会主动给这些依赖它的类发出通知。

# 应用

通常意义上如果一个对象状态的改变需要通知很多对这个对象关注的一系列对象，就可以使用观察者模式。

# 类图

![](http://www.wailian.work/images/2018/06/08/20130626145005968.jpg)

可以看到，我们的被观察者类Observable只关联了一个Observer的列表，然后在自己状态变化时，使用notifyObservers方法通知这些Observer，具体这些Observer都是什么，被观察者是不关心也不需要知道的。

上面就将观察者和被观察者二者的耦合度降到很低了，而我们具体的观察者是必须要知道自己观察的是谁，所以它依赖于被观察者。

# 实现

实现一个简单的观察者模式，使用JAVA简单诠释一下上面的类图

观察者接口:

```
 // 这个接口时为了提供一个统一的观察者做出相应行为的方法
public interface Observer {
    void update(Observable o);
}
```

具体观察者（观察者接口的实现）

```
public class ConcreteObserver1 implements Observer {
    @Override
    public void update(Observable o) {
        System.out.println("观察者1观察到" + o.getClass().getSimpleName() + "发生变化");
        System.out.println("观察者1做出相应变化");
    }
}
```

```
public class ConcreteObserver2 implements Observer {
    @Override
    public void update(Observable o) {
        System.out.println("观察者2观察到" + o.getClass().getSimpleName() + "发生变化");
        System.out.println("观察者2做出相应变化");
    }
}
```

下面是被观察者，它有一个观察者的列表，并且有一个通知所有观察者的方法，通知的方式就是调用观察者通用的接口行为update方法

```
public class Observable {
    List<Observer> observers = new ArrayList<>();

    public void addObserver(Observer o) {
        observers.add(o);
    }

    public void changed() {
        System.out.println("==我是被观察者，我发生了变化==");

        // 通知观察我的所有观察者
        notifyObservers();
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
}
```

这里面很简单，新增两个方法，一个是为了改变自己的同时通知观察者们，一个是为了给客户端一个添加观察者的公共接口

下面测试一下

```
public void testObserver() {
    Observable observable = new Observable();
    observable.addObserver(new ConcreteObserver1());
    observable.addObserver(new ConcreteObserver2());

    observable.changed();
}
```

运行结果如下

![](http://www.wailian.work/images/2018/06/08/WX20180608-154314.png)

可以看到我们在操作被观察者时，只要调用changed方法，观察者们就会做出相应的动作，而添加观察者这个行为算是准备阶段，将具体的观察者关联到被观察者上面去

下面给出一个有实际意义的例子，比如我们经常看的小说网站，都有这样的功能，就是读者可以订阅作者，这当中就有明显的观察者模式案例，就是作者和读者。他们的关系是一旦读者关注了一个作者，那么这个作者一旦有什么新书，就都要通知读者们，这明显是一个观察者模式的案例，所以我们可以使用观察者模式解决。

读者类，要实现观察者接口

```
import java.util.Observable;
import java.util.Observer;

public class Reader implements Observer {
    private String name;

    public Reader(String name) {
        super();
        this.name = name;
    }

    public String getName() {
        return name;
    }

    // 当关注的作者发布新小说时，会通知读者去看
    @Override
        public void update(Observable o, Object arg) {
            if (o instanceof Writer) {
                Writer writer = (Writer) o;
                System.out.println(name + "知道" + writer.getName() + "发布了新书《" + writer.getLastNovel() + "》非要去看");
            }
        }

    //读者可以关注某一位作者，关注则代表把自己加到作者的观察者列表里
    public void subscribe(String writerName) {
        WriterManager.getInstance().getWriter(writerName).addObserver(this);
    }

    //读者可以取消关注某一位作者，取消关注则代表把自己从作者的观察者列表里删除
    public void unsubscribe(String writerName) {
        WriterManager.getInstance().getWriter(writerName).deleteObserver(this);
    }
}
```

作者类

```
import java.util.Observable;

public class Writer extends Observable {
    private String name; // 作者名称

    private String lastNovel; //作者最新发布的小说

    public String getName() {
        return this.name;
    }

    public Writer(String name) {
        super();
        this.name = name;
        WriterManager.getInstance().add(this);
    }

    // 作者发布新小说，要通知所有关注自己的读者
    public void addNovel(String novel) {
        System.out.println(name + "发布了新书《" + novel + "》");

        lastNovel = novel;

        setChanged();

        notifyObservers();
    }

    public String getLastNovel() {
        return this.lastNovel;
    }
}
```

然后我们还需要一个管理器帮我们管理这些作者

```
import java.util.HashMap;
import java.util.Map;

public class WriterManager {
    private static WriterManager writerManager;

    private Map<String, Writer> writerMap = new HashMap<>();

    // 添加作者
    public void add(Writer writer) {
        writerMap.put(writer.getName(), writer);
    }

    // 根据作者名获得作者
    public Writer getWriter(String name) {
        return writerMap.get(name);
    }

    // 单例模式(私有构造函数)
    private WriterManager() {

    }

    public static WriterManager getInstance() {
        if (writerManager == null) {
            synchronized (WriterManager.class) {
                if (writerManager == null) {
                    writerManager = new WriterManager();
                }
            }
        }

        return writerManager;
    }
}
```

 好了，这下我们的观察者模式就做好了，这个简单的DEMO可以支持读者关注作者，当作者发布新书时，读者会观察到这个事情，会产生相应的动作。下面我们写个测试用例测试一下

 ```
 public void testJdkObserver() {
     //假设四个读者，两个作者
     Reader r1 = new Reader("谢广坤");
     Reader r2 = new Reader("赵四");
     Reader r3 = new Reader("七哥");
     Reader r4 = new Reader("刘能");
     Writer w1 = new Writer("谢大脚");
     Writer w2 = new Writer("王小蒙");
     //四人关注了谢大脚
     r1.subscribe("谢大脚");
     r2.subscribe("谢大脚");
     r3.subscribe("谢大脚");
     r4.subscribe("谢大脚");
     //七哥和刘能还关注了王小蒙
     r3.subscribe("王小蒙");
     r4.subscribe("王小蒙");

     //作者发布新书就会通知关注的读者
     //谢大脚写了设计模式
     w1.addNovel("设计模式");
     //王小蒙写了JAVA编程思想
     w2.addNovel("JAVA编程思想");
     //谢广坤取消关注谢大脚
     r1.unsubscribe("谢大脚");
     //谢大脚再写书将不会通知谢广坤
     w1.addNovel("观察者模式");
 }
 ```

 ![](http://www.wailian.work/images/2018/06/08/121313.png)

我们使用观察者模式的用意是为了作者不再需要关心他发布新书时都要去通知谁，更重要的是他不需要关心他通知的是读者还是其它什么人，他只知道这个人是实现了观察者接口的，即我们的被观察者依赖的只是一个抽象的接口观察者接口，而不关心具体的观察者都有谁都是什么，比如以后要是游客也可以关注作者了，那么只要游客类实现观察者接口，那么一样可以将游客列入到作者的观察者列表中

另外，我们让读者自己来选择自己关注的对象，这相当于被观察者将维护通知对象的职能转化给了观察者，这样做的好处是由于一个被观察者可能有N多观察者，所以让被观察者自己维护这个列表会很艰难，这就像一个老师被许多学生认识，那么是所有的学生都记住老师的名字简单，还是让老师记住N多学生的名字简单？答案显而易见，让学生们都记住一个老师的名字是最简单的

观察者模式分离了观察者和被观察者二者的责任，这样让类之间各自维护自己的功能，专注于自己的功能，会提高系统的可维护性和可重用性

***

作者：zuoxiaolong（左潇龙）

出处：博客园左潇龙的技术博客--http://www.cnblogs.com/zuoxiaolong

您的支持是对博主最大的鼓励，感谢您的认真阅读。

本文版权归作者所有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。
