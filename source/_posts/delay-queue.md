---
title: 延时队列的使用
date: 2018-02-02 10:40:03
tags:
---

![](http://www.wailian.work/images/2018/02/02/61b7aea823aaba020bbe85f9b2cc4deeae67d58f12106-3PVHe7_fw658.jpg)

# 什么是DelayQueue

DelayQueue是一个无界的BlockingQueue，用于放置实现了Delayed接口的对象，其中的对象只能在其到期时才能从队列中取走。这种队列是有序的，即队头对象的延迟到期时间最长。注意：不能将null元素放置到这种队列中。

# 应用场景

订单业务中总是出现订单未支付过期关闭的情形。最简单的解决方式是定时任务轮询订单，这种方式浪费资源并不优雅。延时队列能够轻松应对这种情形。

# 创建订单类

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

# 生成订单

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

# 处理超时订单

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

# 测试

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
