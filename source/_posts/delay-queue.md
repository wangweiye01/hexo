---
title: 延时队列的使用
date: 2018-02-02 10:40:03
tags:
---
![](http://www.wailian.work/images/2018/02/02/61b7aea823aaba020bbe85f9b2cc4deeae67d58f12106-3PVHe7_fw658.jpg)

# 什么是DelayQueue

DelayQueue是一个无界的BlockingQueue，用于放置实现了Delayed接口的对象，其中的对象只能在其到期时才能从队列中取走。这种队列是有序的，即队头对象的延迟到期时间最长。注意：不能将null元素放置到这种队列中。

# 应用场景

## 订单超时关闭

订单业务中总是出现订单未支付过期关闭的情形。最简单的解决方式是定时任务轮询订单，这种方式浪费资源并不优雅。延时队列能够轻松应对这种情形。

### 创建订单类

放入DelayQueue的对象需要实现Delayed接口

```
class Order implements Delayed {
	public static AtomicInteger genId = new AtomicInteger(1);

	private final long delay; // 延迟时间
	private final long expire; // 到期时间
	private final long now; // 创建时间

	private Integer id; // 订单ID
	private Integer state; // 订单状态

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Integer getState() {
		return state;
	}

	public void setState(Integer state) {
		this.state = state;
	}

	public Order(long delay, String msg) {
		this.delay = delay;
		expire = System.currentTimeMillis() + delay; // 到期时间 = 当前时间+延迟时间
		now = System.currentTimeMillis();

		this.state = 0;
	}

	/**
	 * 需要实现的接口，获得延迟时间 用过期时间-当前时间
	 *
	 * @param unit
	 * @return
	 */
	@Override
	public long getDelay(TimeUnit unit) {
		return unit.convert(this.expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
	}

	/**
	 * 用于延迟队列内部比较排序 当前时间的延迟时间 - 比较对象的延迟时间
	 *
	 * @param o
	 * @return
	 */
	@Override
	public int compareTo(Delayed o) {
		return (int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
	}
}
```

### 生成订单

```
private static void producer(final DelayQueue<Order> delayQueue) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            while (true) {
                try {
                    TimeUnit.MILLISECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                Order element = new Order(new Random().nextInt(1000) * 10, "test");

                element.setId(Order.genId.getAndIncrement());

                delayQueue.offer(element);
            }
        }
    }).start();

    /**
     * 每秒打印延迟队列中的对象个数
     */
    new Thread(new Runnable() {
        @Override
        public void run() {
            while (true) {
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("delayQueue size:" + delayQueue.size());
            }
        }
    }).start();
}
```

### 处理超时订单

```
private static void consumer(final DelayQueue<Order> delayQueue) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            while (true) {
                Order element = null;
                try {
                    element = delayQueue.take();

                    if (element.getState().intValue() == 0) {
                        // 如果现在订单状态还未支付，关闭订单
                        element.setState(1);

                        System.out.println("订单" + element.getId() + "超时关闭");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }).start();
}
```

DelayQueue还是一个阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，此时调用 poll() 将直接返回 null，调用 take() 将会发生阻塞，直到有元素发生到期，take() 才会返回。

### 测试

```
public static void main(String[] args) {
    DelayQueue<Order> delayQueue = new DelayQueue<Order>();

    // 生产者
    producer(delayQueue);

    // 消费者
    consumer(delayQueue);

    while (true) {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

![](http://www.wailian.work/images/2018/02/02/WX20180202-110453.png)

## 多考生考试

1. 考试总时间为10秒，至少2秒后才可进行交卷。
2. 考生可在2-10秒这段时间内的任意时间交卷。
3. 考试时间一到，所有未交卷的学生必须交卷。

使用enum定义时间常量

```
enum Times {
    SUMMIT_TIME(10), //考试总时间
    SUMBMIT_LIMIT(2), // 交卷限制时间
    MAX_RAND_TIME(15); // 模拟考生所需最大时间

    private final int value;

    private Times(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

### 学生类

```
class Student implements Delayed {
    private String name;
    private long delay; // 考试花费时间，单位为毫秒
    private long expire; // 交卷时间，单位为毫秒

    // 此构造可随机生成考试花费时间
    public Student(String name) {
        this.name = name;
        this.delay = TimeUnit.MILLISECONDS.convert(getRandomSeconds(), TimeUnit.SECONDS);
        this.expire = System.currentTimeMillis() + this.delay;
    }

    //此构造可指定考试花费时间
    public Student(String name, long delay, TimeUnit unit) {
        this.name = name;
        this.delay = TimeUnit.MILLISECONDS.convert(delay, unit);
        this.expire = System.currentTimeMillis() + this.delay;
    }

    // ...

    public int getRandomSeconds() {
        // 获取随机花费时间，范围：2-10秒
        return new Random().nextInt(Times.MAX_RAND_TIME.getValue() - Times.SUMBMIT_LIMIT.getValue())
                    + Times.SUMBMIT_LIMIT.getValue();
    }
    
    @Override
    public int compareTo(Delayed o) {
        // 此方法的实现用于定义优先级
        long td = this.getDelay(TimeUnit.MILLISECONDS);
        long od = o.getDelay(TimeUnit.MILLISECONDS);
        return td > od ? 1 : td == od ? 0 : -1;
    }

    @Override
    public long getDelay(TimeUnit unit) {
        // 这里返回的是剩余延时，当延时为0时，此元素延时期满，可从take()取出
        return unit.convert(this.expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }
}
```

### 主方法实现

1. 初始化对象

```
DelayQueue<Student> queue = new DelayQueue<>();
```

2. 添加测试数据

```
queue.add(new Student("范冰冰"));
queue.add(new Student("成  龙"));
queue.add(new Student("李一桐"));
queue.add(new Student("宋小宝"));
queue.add(new Student("吴  京"));
queue.add(new Student("绿巨人"));
queue.add(new Student("洪金宝"));
queue.add(new Student("李云龙"));
queue.add(new Student("钢铁侠"));
queue.add(new Student("刘德华"));
queue.add(new Student("戴安娜"));
```

3. 添加一条用于考试结束时强制交卷的属性

```
queue.add(new Student("submit", Times.SUBMIT_TIME.getValue(),TimeUnit.SECONDS));
```

4. 开始考试

```
while (true) {
    Student s = queue.take(); // 必要时进行阻塞等待
    if (s.getName().equals("submit")) {
        System.out.println("时间已到，全部交卷！");
        // 利用Java8 Stream特性使尚未交卷学生交卷
        queue.parallelStream()
             .filter(v -> v.getExpire() >= s.getExpire())
             .map(Student::submit)
             .forEach(System.out::println);
        System.exit(0);
    }
    System.out.println(s);
}
```

### 测试结果

![](http://www.wailian.work/images/2018/02/02/WX20180202-115504.png)

### 完整代码

```
package cn.gss.juc;

import java.text.DateFormat;
import java.util.Date;
import java.util.Random;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

enum Times {
    SUBMIT_TIME(10), SUMBMIT_LIMIT(2), MAX_RAND_TIME(15);
    private final int value;

    private Times(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
/**
 * DelayQueue实现多考生考试
 * @author Gss
 */
public class TestDelayedQueue {

    public static void main(String[] args) throws InterruptedException {
        DelayQueue<Student> queue = new DelayQueue<>();
        queue.add(new Student("范冰冰"));
        queue.add(new Student("成  龙"));
        queue.add(new Student("李一桐"));
        queue.add(new Student("宋小宝"));
        queue.add(new Student("吴  京"));
        queue.add(new Student("绿巨人"));
        queue.add(new Student("洪金宝"));
        queue.add(new Student("李云龙"));
        queue.add(new Student("钢铁侠"));
        queue.add(new Student("刘德华"));
        queue.add(new Student("戴安娜"));
        queue.add(new Student("submit", Times.SUBMIT_TIME.getValue(), TimeUnit.SECONDS));
        while (true) {
            Student s = queue.take(); // 必要时进行阻塞等待
            if (s.getName().equals("submit")) {
                System.out.println("时间已到，全部交卷！");
                // 利用Java8 Stream使尚未交卷学生交卷
                queue.parallelStream()
                     .filter(v -> v.getExpire() >= s.getExpire())
                     .map(Student::submit)
                     .forEach(System.out::println);
                System.exit(0);
            }
            System.out.println(s);
        }
    }

}

class Student implements Delayed {
    private String name;
    private long delay; // 考试花费时间，单位为毫秒
    private long expire; // 交卷时间，单位为毫秒

    // 此构造可随机生成考试花费时间
    public Student(String name) {
        this.name = name;
        this.delay = TimeUnit.MILLISECONDS.convert(getRandomSeconds(), TimeUnit.SECONDS); // 随机生成考试花费时间
        this.expire = System.currentTimeMillis() + this.delay;
    }

    // 此构造可指定考试花费时间
    public Student(String name, long delay, TimeUnit unit) {
        this.name = name;
        this.delay = TimeUnit.MILLISECONDS.convert(delay, unit);
        this.expire = System.currentTimeMillis() + this.delay;
    }

    public int getRandomSeconds() { // 获取随机花费时间
        return new Random().nextInt(Times.MAX_RAND_TIME.getValue() - Times.SUMBMIT_LIMIT.getValue())
                + Times.SUMBMIT_LIMIT.getValue();
    }

    public Student submit() { // 设置花费时间和交卷时间，考试时间结束强制交卷时调用此方法
        setDelay(Times.SUBMIT_TIME.getValue(), TimeUnit.SECONDS);
        setExpire(System.currentTimeMillis());
        return this;
    }

    public String getName() {
        return name;
    }

    public long getExpire() {
        return expire;
    }

    public void setDelay(long delay, TimeUnit unit) {
        this.delay = TimeUnit.MILLISECONDS.convert(delay, TimeUnit.SECONDS);
    }

    public void setExpire(long expire) {
        this.expire = expire;
    }

    @Override
    public int compareTo(Delayed o) { // 此方法的实现用于定义优先级
        long td = this.getDelay(TimeUnit.MILLISECONDS);
        long od = o.getDelay(TimeUnit.MILLISECONDS);
        return td > od ? 1 : td == od ? 0 : -1;
    }

    @Override
    public long getDelay(TimeUnit unit) { // 这里返回的是剩余延时，当延时为0时，此元素延时期满，可从take()取出
        return unit.convert(this.expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public String toString() {
        return "学生姓名：" + this.name + ",考试用时：" + TimeUnit.SECONDS.convert(delay, TimeUnit.MILLISECONDS) + ",交卷时间："
                + DateFormat.getDateTimeInstance().format(new Date(this.expire));
    }
}
```

# 源码地址

[GitHub](https://github.com/wangweiye01/DelayQueueTest/tree/master)
