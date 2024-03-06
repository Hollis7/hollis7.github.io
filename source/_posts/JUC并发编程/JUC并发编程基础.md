---
title: JUC并发编程基础
categories:
- JUC
tags:
- JUC
- basic
author: hollis7
---
<meta name="referrer" content="no-referrer"/>

## 内容

- JUC
- 进程线程
- 线程常用方法
- 线程的6种状态
- 线程状态切换
- Callable接口

<!--more-->

## 什么是J.U.C

JUC是`java.util.concurrent`包的缩写，包结构如下，说⽩了就是**并发场景进⾏多线程编程**的⼯具
类。

怎么让程序尽量通过**有限的硬件**，⾼效的处理请求，并且保证程序“**线程安全**”⽽这涉及到的知识点⾮常的庞⼤。

## 什么是进程

在操作系统中，**进程是基本的资源分配单位**，操作系统通过进程来管理计算机的资源，如CPU、内存、磁盘等。每个进程都有⼀个唯⼀的**进程标识符**（PID），⽤于区分不同的进程。
通俗说法：可看做是正在执⾏的程序如 QQ.exe

## 什么是线程

底层⻆度：**线程是操作系统中的基本执⾏单元**（能够直接执⾏的最⼩代码块），它是进程中的⼀个实
体，是**CPU调度和分派的基本单位**。⼀个进程可以包含多个线程，每个线程都可以独⽴执⾏不同的任
务，但它们共享进程的资源。

## 并⾏、并发、串⾏

**并发**是指多个任务在**同一时间段内**交替执行。这些任务可能会在同一个处理器上通过**时间片轮转**或者通过操作系统的调度器分配资源交替执行。

**并行**是指在**同一时刻**，多个任务同时执行。这些任务可以在多个处理器上同时执行。并行的主要目的是提高系统的整体性能和吞吐量。

## 创建线程

1. 通过继承Thread，重写run

~~~java
Thread t2 = new Thread() {
    @Override
    public void run() {
        System.out.println("线程Thread");
    }
};
~~~

2. 通过实现Runnable

~~~java
 // new Runnable匿名内部类
Thread t1 = new Thread(() -> {
    System.out.println("线程Runnable");
});
~~~

~~~java
Thread t1 = new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("线程Runnable");
    }
});
~~~

## 线程常⽤⽅法

### 线程原理

都会调用start方法，然后执行我们的**start0**这个原生本地方法，底层是c/c++代码，执行后回调run方法，run方法有一个Runnable类型的target判断，不为空执行Runnable.run,继承Thread方法的话直接执行run。

### start 与 run

**类型**
run⽅法是同步⽅法，⽽start⽅法是异步⽅法。
**作⽤**
run⽅法的作⽤是存放任务代码，⽽start的⽅法呢是启动线程
**线程数量⽅⾯**
执⾏run⽅法它不会产⽣新线程，⽽执⾏start⽅法会产⽣新线程，
**调⽤次数**
run⽅法可以被执⾏⽆数次，⽽star⽅法只能被执⾏⼀次，原因就在于线程不能被重复启动。

### 睡眠

~~~java
@Slf4j
public class SleepThread {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println("睡眠完毕");
            }
        });
        t1.start();
        // 线程中断
//        t1.interrupt();
    }
}
~~~

#### 应用

**在循环访问锁的过程中，可以加⼊sleep让线程阻塞时间，防⽌⼤量占⽤cpu资源。**

比如Tomcat

~~~java
try {
    awaitThread = currentThread;
    while (!stopAwait) {
        try {
            Thread.sleep(10000);
        } catch (InterruptedException ex) {
            // continue and check the flag
        }
    }
} finally {
    awaitThread = null;
}
~~~

### yeild

提示线程调度器尽⼒让出当前线程对CPU的使⽤

- Thread.yield()方法作用是：暂停当前正在执行的线程对象（及放弃当前拥有的cu资源)，并执行其他线程。
- **yield()做的是让当前运行线程回到可运行状态，以允许具有相同优先级的其他线程获得运行机会**。因此，使用yield()的目的是让相同优先级的线程之间能适当的轮转执行。但是，**实际中无法保证yield()达到让步目的，因为让步的线程还有可能被线程调度程序再次选中**。

~~~java
class Task2 implements Runnable {
@Override
public void run() {
    for (int i = 0; i < 10; i++) {
        Thread.yield();
        System.out.println("B:" + i);
    }
}
~~~

> sleep()和yield()的区别 sleep()使当前线程进⼊停滞状态，所以执⾏sleep()的线程在指定的时间内肯定不会被执⾏； yield()只是使当前线程重新回到可执⾏状态，所以执⾏yield()的线程有可能在进⼊到可执⾏状态后⻢上⼜被执⾏。

### 优先级

~~~java
 thread1.setPriority(Thread.MAX_PRIORITY);
~~~

~~~java
public static final int MIN_PRIORITY = 1;
public static final int NORM_PRIORITY = 5;
public static final int MAX_PRIORITY = 10;
~~~

### 线程打断

~~~java
public void interrupt()
~~~

**实例⽅法**interrupt()仅仅是设置线程的中断状态为true，不会停⽌线程。

~~~java
public static void main(String[] args) {
    Thread thread = new Thread(() -> {
        while (true) {
            // 定时监控系统...
            System.out.println("定时监控...");
        }
    });
    thread.start();
    thread.interrupt();
}
~~~

~~~java
定时监控...
定时监控...
定时监控...
定时监控...
~~~

~~~java
public static boolean interrupted()
~~~

**静态⽅法**，判断线程是否被中断，并清除当前中断状态
也就是说这个⽅法做了两件事：
1、返回当前线程的中断状态
2、将当前线程的中断状态设为false

```
public static void main(String[] args) {
    Thread thread = new Thread(() -> {
        while (true) {
            // 每隔1s 将时间片清除
            try {
                TimeUnit.SECONDS.sleep(1L);
            } catch (InterruptedException e) {
                // 当出现InterruptedException  会清除中断标记      false
                e.printStackTrace();
                // 再次加上中断标记
                Thread.currentThread().interrupt();      // true
            }
            //  如果中断的标记为true
            // 获取线程中断标记，  并且会清除标记
            System.out.println(Thread.currentThread().isInterrupted());
            if (Thread.interrupted()) {
                System.out.println(Thread.currentThread().isInterrupted());
                break;
            }

            // 定时监控系统...
            System.out.println("定时监控...");
        }
    });
    thread.start();
    thread.interrupt();
}
```

### 线程的合并join

Thread中， `join()`⽅法的作⽤是调⽤线程等待该线程完成后，才能继续往下运⾏。

join是Thread类的⼀个⽅法，启动线程后直接调⽤，即join()的作⽤是：“等待该线程终⽌”，这⾥需要
理解的就是该线程是指的**主线程等待⼦线程的终⽌**。也就是在⼦线程调⽤了join()⽅法后⾯的代码，只有
等到⼦线程结束了才能执⾏。

~~~java
t1.start();// 异步
System.out.println(t1.isAlive());//true
t1.join();  // 异步变成同步
System.out.println(t1.isAlive());//false
~~~

也可以有时间限制的等待

~~~java
t1.join(1000); 
~~~

### 守护线程

默认情况下我们创建的线程都是⽤户线程（普通线程），进程需要等待所有的线程执⾏完毕后，进
程才会结束。
守护线程.setDaemon(true):设置守护线程

~~~java
public class DeamonThread {
    public static void main(String[] args) {
        //创建线程(默认前台线程)
        Thread d1 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    Thread.sleep(500);
                    System.out.println(i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        //设置线程为守护线程
        d1.setDaemon(true);//主线程结束便结束了
        d1.start();
        System.out.println("主线程结束");

    }
}
~~~

> 主线程结束

**应⽤：**
垃圾回收器线程属于守护线程
tomcat⽤来接受处理外部的请求的线程就是守护线程。

## 线程6种状态

| 状态名称     | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| NEW          | 初始状态，线程被构建，但是还没有调用start()方法              |
| RUNNABLE     | 运行状态，Java线程将操作系统中的就绪和运行两种状态统称为"运行中" |
| BLOCKED      | 阻塞状态，表示线程阻塞于锁                                   |
| WAITING      | 等待状态，表示线程进入等待状态，进入该状态表示当前线程需要其他线程通知(notify或者notifyAll) |
| TIME_WAITING | 超时等待状态，可以指定等待时间自己返回                       |
| TERMINATED   | 终止状态，表示当前线程已经执行完毕                           |

### BLOCKED

```
Table table = new Table();
Thread student1 = new Thread(() -> {
    table.use();
}, "s1");
Thread student2 = new Thread(() -> {
    table.use();
}, "s2");
student1.start();
Thread.sleep(1000);
student2.start();
Thread.sleep(500);
System.out.println(student2.getState());
```

`Thread.sleep(1000)`的作用是确保student1已经获取了锁，`Thread.sleep(500)`的作用是确保student2尝试获取锁，但是锁被student1获取没有释放，因此student2是阻塞状态。

### WAITING

会一直卡在wait()那。

~~~java
class Table1 {
    public synchronized void use() throws Exception {
        System.out.println(Thread.currentThread().getName() + "-使用桌子");
        //忘记点餐了
        System.out.println("忘记点餐了");
        wait();
        System.out.println(Thread.currentThread().getName() + "就餐结束");
    }
}
~~~

### TIMED_WAITING

~~~java
wait(3000);
~~~

##  线程状态间转换

### Blocked 进⼊ Runnable

想要从 Blocked 状态进⼊ Runnable 状态，我们上⾯说过必须要线程获得 monitor 锁，但是如果想
进⼊其他状态那么就相对⽐较特殊，因为它是没有超时机制的，也就是不会主动进⼊。

### Waiting 进⼊ Runnable

- 有当执⾏了LockSupport.unpark()，或者join的线程运⾏结束，或者被中断时才可以进⼊Runnable 状态。
- 如果通过其他线程调⽤ notify() 或 notifyAll()来唤醒它，则它会直接进⼊ Blocked 状态，这⾥⼤家
  可能会有疑问，不是应该直接进⼊ Runnable 吗？**这⾥需要注意⼀点，因为唤醒 Waiting 线程的线程如果调⽤ notify() 或 notifyAll()，要求必须⾸先持有该 monitor锁**，这也就是我们说的 wait()、notify 必须在 synchronized 代码块中。
- 所以处于Waiting状态的线程被唤醒时拿不到该锁，就会进⼊Blocked状态，直到执⾏了notify（）/notifyAll（）的唤醒它的线程执⾏完毕并释放monitor锁，才可能轮到它去抢夺这把锁，如果它能抢到，就会从Blocked状态回到Runnable状态。

## Callable接⼝

⼀般情况下，使⽤Runnable接⼝、Thread实现的线程我们都是⽆法返回结果的。但是如果对⼀些场合需要线程返回的结果。就要使⽤⽤**Callable、Future**这⼏个类。Callable只能在ExecutorService的线程池中跑，但有返回结果，也可以通过返回的Future对象查询执⾏状态。

~~~java
public interface Callable<V> {
V call() throws Exception;
}
~~~

它只有⼀个call⽅法，并且有⼀个返回V，是泛型。可以认为这⾥返回V就是线程返回的结果。

~~~java
public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
~~~



第一步，创建Callable实现类的实例，并实现call方法

第2步，创建Callable实现类实例

第3步，使用FutureTask类来包装Callable对象，可以创建匿名对象

第4步，使用FutureTask对象作为Thread对象的target创建、并启动新线程。

第5步，调用FutureTask对象的方法来获取子线程执行结束后的返回值

~~~java
@Slf4j
class Task implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        log.info("2.子线程运行中...");
        Thread.sleep(5000);
        return 5;
    }

Task task = new Task();
FutureTask<Integer> future = new FutureTask<>(task);
new Thread(future).start();
~~~



## 三种线程创建的⽅式

1. 实现Runnable接口的run⽅法
2. 继承Thread类并重写run的⽅法
3. 使⽤FutureTask⽅式(实现Callable接口的⽅式)

Java中，类仅⽀持单继承，如果⼀个类继承了Thread类，就⽆法再继承其它类，因此，**如果⼀个类既要**
**继承其它的类，⼜必须创建为⼀个线程，就可以使⽤实现Runable接⼝的⽅式。**

使⽤实现Callable接口的⽅式创建的线程，可以获取到线程执⾏的**返回值**、是否执⾏完成等信息。

### Runnable和Callable区别

1. 返回值：实现Callable接口的任务线程能返回执⾏结果；⽽实现Runnable接口的任务线程不能返回结果
2. 抛出异常：Callable接口的call()⽅法允许抛出异常；⽽Runnable接口的run()⽅法的异常只能在内部消化，不能继续上抛；



