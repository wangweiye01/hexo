---
title: Java多线程编程
date: 2018-01-23 14:35:57
tags:
---

# 相关概念

线程是进程的一个实体，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源，但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

# 线程状态

## 创建

当用new操作符创建一个线程时， 例如new Thread(r)，线程还没有开始运行，此时线程处在新建状态。 当一个线程处于新建状态时，程序还没有开始运行线程中的代码

## 就绪

一个新创建的线程并不自动开始运行，要执行线程，必须调用线程的start()方法。当线程对象调用start()方法即启动了线程，start()方法创建线程运行的系统资源，并调度线程运行run()方法。当start()方法返回后，线程就处于就绪状态。此时线程中代码仍未运行

## 运行

当线程获得CPU后，它才进入运行状态，真正开始执行run()方法中的代码

## 阻塞

该线程放弃CPU的使用，暂停运行

常见线程阻塞的原因:

1. 线程执行了Thread.sleep(int millsecond)方法，当前线程放弃CPU，睡眠一段时间，然后再恢复执行

2. 线程执行一段同步代码，但是尚且无法获得相关的同步锁，只能进入阻塞状态，等到获取了同步锁，才能回复执行

3. 线程执行了一个对象的wait()方法，直接进入阻塞状态，等待其他线程执行notify()或者notifyAll()方法

4. 线程执行某些IO操作，因为等待相关的资源而进入了阻塞状态。比如说监听system.in，但是尚且没有收到键盘的输入，则进入阻塞状态

## 终止

线程执行完毕

# 线程的创建方法

## 继承Thread类

```
public class Thread1 extends Thread {
    public void run() {
        // 线程执行代码
    }

    public class Main {

    public static void main(String[] args) {
        Thread1 mTh1=new Thread1();
        Thread1 mTh2=new Thread1();
        mTh1.start();
        mTh2.start();
    }
}
```

程序启动运行main时候，java虚拟机启动一个进程，主线程main在main()调用时候被创建。随着调用main的两个对象的start方法，另外两个线程也启动了，这样，整个应用就在多线程下运行

注意：
start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的。

从程序运行的结果可以发现，多线程程序是乱序执行。

Thread.sleep()方法调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留出一定时间给其他线程执行的机会。

实际上所有的多线程代码执行顺序都是不确定的，每次执行的结果都是随机的。

## 实现Runable接口(推荐)

```
public class Thread2 implements Runnable{
    private String name;

    public Thread2(String name) {
        this.name=name;
    }

    @Override
    public void run() {
    // 线程逻辑代码
    }
}
public class Main {

    public static void main(String[] args) {
        new Thread(new Thread2("C")).start();
        new Thread(new Thread2("D")).start();
    }
}
```

## 两种方法的区别

如果一个类继承Thread，则不适合资源共享。但是如果实现了Runable接口的话，则很容易的实现资源共享。

总结：
实现Runnable接口比继承Thread类所具有的优势：
1. 适合多个相同的程序代码的线程去处理同一个资源 
2. 可以避免java中的单继承的限制
3. 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
4. 线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

提醒一下大家：main方法其实也是一个线程。在java中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。

在Java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM实际就是在操作系统中启动了一个进程。

# 线程状态

![](http://www.wailian.work/images/2018/01/23/20170717150938439.jpg)

# 线程调度

Java线程有优先级，优先级高的线程会优先获得运行机会(但不一定优先级高的一定先执行)
Java线程的优先级用整数表示，取值范围是1~10，Thread类有以下三个静态常量：

```
static int MAX_PRIORITY  = 10;  //线程可以具有的最高优先级
static int MIN_PRIORITY  = 1;   //线程可以具有的最低优先级
static int NORM_PRIORITY = 5;   //分配给线程的默认优先级
```
Thread类的setPriority()和getPriority()方法分别用来设置和获取线程的优先级。

# 线程睡眠

Thread.sleep(long millis)方法，使线程转到阻塞状态。millis参数设定睡眠的时间，以毫秒为单位。当睡眠结束后，就转为就绪（Runnable）状态。sleep()平台移植性好。

# 线程等待

Object类中的wait()方法，导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 唤醒方法。这个两个唤醒方法也是Object类中的方法，行为等价于调用 wait(0) 一样。

# 线程让步

Thread.yield() 方法，暂停当前正在执行的线程对象，把执行机会让给相同或者更高优先级的线程。

# 线程加入

join()方法，等待其他线程终止。在当前线程中调用另一个线程的join()方法，则当前线程转入阻塞状态，直到另一个进程运行结束，当前线程再由阻塞转为就绪状态。

# 线程唤醒

Object类中的notify()方法，唤醒在此对象监视器上等待的单个线程。如果所有线程都在此对象上等待，则会选择唤醒其中一个线程。选择是任意性的，并在对实现做出决定时发生。线程通过调用其中一个 wait 方法，在对象的监视器上等待。 直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程。被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争；例如，唤醒的线程在作为锁定此对象的下一个线程方面没有可靠的特权或劣势。类似的方法还有一个notifyAll()，唤醒在此对象监视器上等待的所有线程。

注意：Thread中suspend()和resume()两个方法在JDK1.5中已经废除，不再介绍。因为有死锁倾向。

# 常用函数说明

## sleep(long millis)

在指定的毫秒数内让当前正在执行的线程休眠（暂停执行）

# join

join是Thread类的一个方法，启动线程后直接调用，即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

## 使用场景

在很多情况下，主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

```
public class Main {

    public static void main(String[] args) {
        System.out.println("主线程运行开始!");
        Thread1 mTh1=new Thread1("A");
        Thread1 mTh2=new Thread1("B");
        mTh1.start();
        mTh2.start();
        try {
            mTh1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        try {
            mTh2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("主线程运行结束!");
    }
}
```

# yield():暂停当前正在执行的线程对象，并执行其他线程。

Thread.yield()方法作用是：暂停当前正在执行的线程对象，并执行其他线程。

yield()应该做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中。

结论：yield()从未导致线程转到等待/睡眠/阻塞状态。在大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。可看上面的图。

# sleep()和yield()的区别

sleep()使当前线程进入停滞状态，所以执行sleep()的线程在指定的时间内肯定不会被执行；yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。

sleep 方法使当前运行中的线程睡眼一段时间，进入不可运行状态，这段时间的长短是由程序设定的，yield 方法使当前线程让出 CPU 占有权，但让出的时间是不可设定的。实际上，yield()方法对应了如下操作：先检测当前是否有相同优先级的线程处于同可运行状态，如有，则把 CPU 的占有权交给此线程，否则，继续运行原来的线程。所以yield()方法称为“退让”，它把运行机会让给了同等优先级的其他线程

另外，sleep 方法允许较低优先级的线程获得运行机会，但 yield() 方法执行时，当前线程仍处在可运行状态，所以，不可能让出较低优先级的线程些时获得 CPU 占有权。在一个运行系统中，如果较高优先级的线程没有调用 sleep 方法，又没有受到 I\O 阻塞，那么，较低优先级线程只能等待所有较高优先级的线程运行结束，才有机会运行。

# wait

我们可以利用wait()来让一个线程在某些条件下暂停运行。例如，在生产者消费者模型中，生产者线程在缓冲区为满的时候，消费者在缓冲区为空的时候，都应该暂停运行。如果某些线程在等待某些条件触发，那当那些条件为真时，你可以用 notify 和 notifyAll 来通知那些等待中的线程重新开始运行。不同之处在于，notify 仅仅通知一个线程，并且我们不知道哪个线程会收到通知，然而 notifyAll 会通知所有等待中的线程。换言之，如果只有一个线程在等待一个信号灯，notify和notifyAll都会通知到这个线程。但如果多个线程在等待这个信号灯，那么notify只会通知到其中一个，而其它线程并不会收到任何通知，而notifyAll会唤醒所有等待中的线程。

我们怎么在代码里使用wait()呢？因为wait()并不是Thread类下的函数，我们并不能使用Thread.wait()。事实上很多Java程序员都喜欢这么写，因为它们习惯了使用Thread.sleep()，所以他们会试图使用wait() 来达成相同的目的，但很快他们就会发现这并不能顺利解决问题。正确的方法是对在多线程间共享的那个Object来使用wait。在生产者消费者问题中，这个共享的Object就是那个缓冲区队列。

既然我们应该在synchronized的函数或是对象里调用wait，那哪个对象应该被synchronized呢？答案是，那个你希望上锁的对象就应该被synchronized，即那个在多个线程间被共享的对象。在生产者消费者问题中，应该被synchronized的就是那个缓冲区队列。

## 永远在循环（loop）里调用 wait 和 notify，不是在 If 语句

现在你知道wait应该永远在被synchronized的背景下和那个被多线程共享的对象上调用，下一个一定要记住的问题就是，你应该永远在while循环，而不是if语句中调用wait。因为线程是在某些条件下等待的——在我们的例子里，即“如果缓冲区队列是满的话，那么生产者线程应该等待”，你可能直觉就会写一个if语句。但if语句存在一些微妙的小问题即使条件没被满足，你的线程你也有可能被错误地唤醒。所以如果你不在线程被唤醒后再次使用while循环检查唤醒条件是否被满足，你的程序就有可能会出错——例如在缓冲区为满的时候生产者继续生成数据，或者缓冲区为空的时候消费者开始消耗数据。所以记住，永远在while循环而不是if语句中使用wait！

在while循环里使用wait的目的，是在线程被唤醒的前后都持续检查条件是否被满足。如果条件并未改变，wait被调用之前notify的唤醒通知就来了，那么这个线程并不能保证被唤醒，有可能会导致死锁问题。

下面我们提供一个使用wait和notify的范例程序。在这个程序里，我们使用了上文所述的一些代码规范。我们有两个线程，分别名为PRODUCER（生产者）和CONSUMER（消费者），他们分别继承了了Producer和Consumer类，而Producer和Consumer都继承了Thread类。Producer和Consumer想要实现的代码逻辑都在run()函数内。Main线程开始了生产者和消费者线程，并声明了一个LinkedList作为缓冲区队列（在Java中，LinkedList实现了队列的接口）。生产者在无限循环中持续往LinkedList里插入随机整数直到LinkedList满。我们在while(queue.size == maxSize)循环语句中检查这个条件。请注意到我们在做这个检查条件之前已经在队列对象上使用了synchronized关键词，因而其它线程不能在我们检查条件时改变这个队列。如果队列满了，那么PRODUCER线程会在CONSUMER线程消耗掉队列里的任意一个整数，并用notify来通知PRODUCER线程之前持续等待。在我们的例子中，wait和notify都是使用在同一个共享对象上的。

```
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;
/**
* Simple Java program to demonstrate How to use wait, notify and notifyAll()
* method in Java by solving producer consumer problem.
*/
public class ProducerConsumerInJava {
    public static void main(String args[]) {
        System.out.println("How to use wait and notify method in Java");
        System.out.println("Solving Producer Consumper Problem");
        Queue<Integer> buffer = new LinkedList<>();
        int maxSize = 10;
        Thread producer = new Producer(buffer, maxSize, "PRODUCER");
        Thread consumer = new Consumer(buffer, maxSize, "CONSUMER");
        producer.start(); consumer.start(); }
    }
    /**
    * Producer Thread will keep producing values for Consumer
    * to consumer. It will use wait() method when Queue is full
    * and use notify() method to send notification to Consumer
    * Thread.
    */
    class Producer extends Thread
    {
        private Queue<Integer> queue;
        private int maxSize;
        public Producer(Queue<Integer> queue, int maxSize, String name){
            super(name); this.queue = queue; this.maxSize = maxSize;
        }
        @Override public void run()
        {
            while (true)
                {
                    synchronized (queue) {
                        while (queue.size() == maxSize) {
                            try {
                                System.out .println("Queue is full, " + "Producer thread waiting for " + "consumer to take something from queue");
                                queue.wait();
                            } catch (Exception ex) {
                                ex.printStackTrace(); }
                            }
                            Random random = new Random();
                            int i = random.nextInt();
                            System.out.println("Producing value : " + i); queue.add(i); queue.notifyAll();
                        }
                    }
                }
            }
    /**
    * Consumer Thread will consumer values form shared queue.
    * It will also use wait() method to wait if queue is
    * empty. It will also use notify method to send
    * notification to producer thread after consuming values
    * from queue.
    */
    class Consumer extends Thread {
        private Queue<Integer> queue;
        private int maxSize;
        public Consumer(Queue<Integer> queue, int maxSize, String name){
            super(name);
            this.queue = queue;
            this.maxSize = maxSize;
        }
        @Override public void run() {
            while (true) {
                synchronized (queue) {
                    while (queue.isEmpty()) {
                        System.out.println("Queue is empty," + "Consumer thread is waiting" + " for producer thread to put something in queue");
                        try {
                            queue.wait();
                        } catch (Exception ex) {
                            ex.printStackTrace();
                        }
                    }
                    System.out.println("Consuming value : " + queue.remove()); queue.notifyAll();
                }
            }
        }
    }
```

![](http://www.wailian.work/images/2018/01/23/20170805131528961.png)

为了更好地理解这个程序，我建议你在debug模式里跑这个程序。一旦你在debug模式下启动程序，它会停止在PRODUCER或者CONSUMER线程上，取决于哪个线程占据了CPU。因为两个线程都有wait()的条件，它们一定会停止，然后你就可以跑这个程序然后看发生什么了（很有可能它就会输出我们以上展示的内容）。你也可以使用Eclipse里的Step into和Step over按钮来更好地理解多线程间发生的事情。

# 阻塞队列实现生产者消费者问题

```
public class ProducerConsumerWithQueue {
    private int queueSize = 10;
    private ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(queueSize);

    public static void main(String[] args) {
        ProducerConsumerWithQueue test = new ProducerConsumerWithQueue();
        Producer producer = test.new Producer();
        Consumer consumer = test.new Consumer();

        producer.start();
        consumer.start();
    }

    class Consumer extends Thread {

        @Override
        public void run() {
            while (true) {
                try {
                    queue.take();
                    System.out.println("从队列取走一个元素，队列剩余" + queue.size() + "个元素");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    class Producer extends Thread {

        @Override
        public void run() {
            while (true) {
                try {
                    queue.put(1);
                    System.out.println("向队列取中插入一个元素，队列剩余空间：" + (queueSize - queue.size()));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 附：阻塞队列的四种处理方法

| 方法\处理方式| 抛出异常    |  返回特殊值  |一直阻塞|超时退出
| --------   | -----:   | :----: |
|插入方法 | add(e)      |   offer(e)    |put(e)|offer(e,time,unit)
| 移除方法| remove()|poll()|take()|poll(time,unit)
| 检查方法| element()|  peek()    |不可用|不可用

# 另一个例子

```
/**
* 计算输出其他线程锁计算的数据
*
*/
public class ThreadA {
    public static void main(String[] args) throws InterruptedException{
        ThreadB b = new ThreadB();
        //启动计算线程
        b.start();
        //线程A拥有b对象上的锁。线程为了调用wait()或notify()方法，该线程必须是那个对象锁的拥有者
        synchronized (b) {
            System.out.println("等待对象b完成计算。。。");
            //当前线程A等待
            b.wait();
            System.out.println("b对象计算的总和是：" + b.total);
        }
    }
}



/**
* 计算1+2+3 ... +100的和
*
*/
class ThreadB extends Thread {
    int total;

    public void run() {
        synchronized (this) {
            for (int i = 0; i < 101; i++) {
                total += i;
            }
            //（完成计算了）唤醒在此对象监视器上等待的单个线程，在本例中线程A被唤醒
            notify();
            System.out.println("计算完成");
        }
    }
}
```

执行结果：

等待对象b完成计算。。。
计算完成
b对象计算的总和是：5050

如果我们将b.wait()去掉呢？结果如下：

等待对象b完成计算。。。
b对象计算的总和是：0
计算完成

上述的结果表明，当去掉b.wait()时，新启动的线程ThreadB与主线程ThreadA是各自执行的，没有线程等待的现象。

我们想要的效果是，当线程ThreadB完成计算之后，再去取计算后的结果。所以使用了b.wait()来让主线程等待。

那为什么是使用b.wait()，而不是Thread.currentThread.wait()，或者其他的呢？

如果我们将b.wait()替换成Thread.currentThread.wait()，将会得到如下的结果：

Exception in thread “main” java.lang.IllegalMonitorStateException
at java.lang.Object.wait(Native Method)
at java.lang.Object.wait(Object.java:485)
at pa.com.thread.ThreadA.main(ThreadA.java:18)
等待对象b完成计算。。。
计算完成

替换的代码Thread.currentThread.wait()好像理所当然应该如我们预期的正确啊，让当前线程处于等待状态，让其他线程先执行。

我们忽略了一个很重要的问题：线程与锁是分不开的，线程的同步、等待、唤醒都与对象锁是密不可分的。

线程ThreadA持有对象b的锁，我们要使用这把锁去让线程释放锁，从而让其他的线程能抢到这把锁。

从我们的程序来分析就是：线程ThreadA首先持有锁对象b，然后调用b.wait()将对象锁释放，线程ThreadB争抢到对象锁b，从而执行run()方法中的计算，计算完了之后使用notify()唤醒主线程ThreadA，ThreadA得以继续执行，从而得到了我们预期的效果。

（之所以ThreadB的对象锁也是b，是因为synchronized(this)中的this指向的就是ThreadB的实例b）

Thread.currentThread.wait()调用的是当前线程对象（即主线程ThreadA）的wait()方法，当前线程对象ThreadA是没有被加锁的，它只是获取了对象锁b。我基本没有看到过这样的调用，一般使用的是锁对象的wait()，本例中为b.wait()

顺带讲一下wait()与sleep()的区别。

如果我们将b.wait()换成Thread.sleep(1000)，则会出现如下的结果：

等待对象b完成计算。。。
b对象计算的总和是：0
计算完成

从执行结果可以看出，Thread.sleep(1000)只是让主线程ThreadA睡眠了1秒钟，而并没有释放对象锁，所以在主线程ThreadA睡眠的过程中，ThreadB拿不到对象锁，从而不能执行。

所以我们也就得出了如下的结论：

wait()方法是让线程释放对象锁，让其他线程拿到锁之后去优先执行，当其他全程唤醒wait()中的线程 或者 拿到对象锁的线程都执行完释放了对象锁之后，wait()中的线程才会再次拿到对象锁从而执行。

sleep()方法是让线程睡眠，此时并没有释放对象锁，其他想要拿到睡眠线程的对象锁的线程也就拿不到相应的对象锁，从而不能抢在它前面执行。

补：

wait、notify和notifyAll方法是Object类的final native方法。所以这些方法不能被子类重写，Object类是所有类的超类，因此在程序中有以下三种形式调用wait等方法。

```
wait();//方式1：
this.wait();//方式2：
super.wait();//方式3
```

void wait()

导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒。该方法只能在同步方法中调用。如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。

# synchronized

卖火车票问题

```
public class Seller implements Runnable {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        synchronized (this) {
            if (Synchronized.ticket > 0) {
                System.out.println("正在卖票,剩余" + Synchronized.ticket);
                Synchronized.ticket--;
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

```
public class Synchronized {
    static Integer ticket = 100;

    public static void main(String[] args) {
        Seller s = new Seller();
        Thread t1 = new Thread(s);
        Thread t2 = new Thread(s);
        Thread t3 = new Thread(s);
        Thread t4 = new Thread(s);
        Thread t5 = new Thread(s);
        Thread t6 = new Thread(s);

        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();
    }
}
```

[源码地址](https://github.com/wangweiye01/mutiThreading)
