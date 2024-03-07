---
title: JUC并发编程基础-线程池
categories:
- JUC
tags:
- JUC
- ThreadPool
---
<meta name="referrer" content="no-referrer"/>

## 内容

- 线程池概念 特性
- 创建线程EXecutors
- 自定义创建线程ThreadPoolExecutor
- 源码分析：创建、淘汰、异常、中断

<!--more-->

## 线程池

其实就是⼀个容纳多个线程的容器，其中的线程可以反复使⽤，省去了频繁创建线程对象的操作，⽆需反复创建线程⽽消耗过多资源。

### 优势

线程池做的⼯作主要是**控制运⾏的线程数量**，处理过程中**将任务放⼊队列**，然后在线程创建后启动这些任务，如果线程数量超过了最⼤数量，超出数量的线程排队等候，等其他线程执⾏完毕，再从队列中取出任务来执⾏。

**降低资源消耗**。通过重复利⽤已创建的线程降低线程创建和销毁造成的销耗。

**提⾼响应速度**。当任务到达时，任务可以不需要等待线程创建就能⽴即执⾏。

**提⾼线程的可管理性**。线程是稀缺资源，如果⽆限制的创建，不仅会销耗系统资源，还会降低系统的稳定性，使⽤线程池可以进⾏**统⼀的分配，调优和监控**

## 线程池-初试

~~~java
ExecutorService executorService = Executors.newSingleThreadExecutor();
for (int i = 0; i < 10000; i++) {
    executorService.execute(new Runnable() {
        @Override
        public void run() {
            list.add(random.nextInt());
        }
    });
}
executorService.shutdown();
executorService.awaitTermination(1, TimeUnit.DAYS);  
~~~

## Executor

Java⾥⾯线程池的顶级接口是 java.util.concurrent.Executor ，但是严格意义上讲 Executor并不是⼀个线程池，⽽只是⼀个执⾏线程的⼯具。真正的线程池接接口是`java.util.concurrent.ExecutorService` 。

- `newFixedThreadPool`创建⼀个固定⻓度的线程池，当到达线程最⼤数量时，线程池的规模将不再变化。
- `newCachedThreadPool`创建⼀个可缓存的线程池，如果当前线程池的规模超出了处理需求，将回收空的线程；当需求增加时，会增加线程数量；线程池规模⽆限制。
- `newSingleThreadPoolExecutor`创建⼀个单线程的Executor，确保任务对了，串⾏执⾏
- `newScheduledThreadPool`创建⼀个固定⻓度的线程池，⽽且以延迟或者定时的⽅式来执⾏，类似 Timer；

~~~java
@Slf4j
class Task implements Runnable {
    @Override
    public void run() {
        log.info("\t 办理业务");
    }
}
~~~

### newFixedThreadPool

newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值是相等的，它使⽤
的是`LinkedBlockingQueue`执⾏⻓期任务性能好，创建⼀个线程池，**⼀池有N个固定的线程**，有固定线程
数的线程

~~~java
ExecutorService threadPool = Executors.newFixedThreadPool(5);
try {
    for (int i = 0; i <= 10; i++) {
        threadPool.execute(task);
    }
} catch (Exception e) {
    throw new RuntimeException(e);
} finally {
    threadPool.shutdownNow();
}
~~~

### newSingleThreadExecutor

newSingleThreadExecutor 创建的线程池corePoolSize和maximumPoolSize值都是1，它使⽤的是`LinkedBlockingQueue`⼀个任务⼀个任务的执⾏，⼀池⼀线程

```java
ExecutorService threadPool = Executors.newSingleThreadExecutor();
```

### newCachedThreadPool

newCachedThreadPool创建的线程池将corePoolSize设置为0，将maximumPoolSize设置
为`Integer.MAX_VALUE`，它使⽤的是`SynchronousQueue`，也就是说**来了任务就创建线程运⾏，当线程**
**空闲超过60秒，就销毁线程**。

~~~java
ExecutorService threadPool = Executors.newCachedThreadPool();
~~~

### newScheduledThreadPool

**可以固定线程池线程数和延迟固定时间执行任务**，类型是`ScheduledExecutorService`

~~~java
ScheduledExecutorService threadPool = Executors.newScheduledThreadPool(5);
~~~

执行任务也有所不同`schedule`

~~~java
 threadPool.schedule(task, 5, TimeUnit.SECONDS);
~~~

### 关闭线程池

如果没有关闭线程池，线程池中线程一直占用系统资源，内存泄露，主线程只会不会退出

~~~java
// 不会立马停止正在执行的线程，会等待所有的任务执行完后才彻底关闭
threadPool.shutdown();
~~~

~~~java
//判断线程池是否关闭，异步
threadPool.isTerminated()
~~~

~~~java
//等待线程池关闭，等待线程池中所有的线程执行完，最多等待Long.MAX_VALUE的时间，同步
threadPool.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
~~~

~~~java
// 不会立马停止正在执行的线程，只会等待正在执行的线程执行完后才彻底关闭，异步
//举例，线程池5个任务执行完毕后就会关闭，不管10个任务没有执行完毕
threadPool.shutdownNow();
~~~

## excute和submit

**1.参数**

execute Runnable.run

submit callable

**2.返回值**

execute void

submit Future

**3.异常**

execute 会在子线程中抛出异常，在主线程捕捉不到

submit不会字码抛出异常，而是会讲一次暂时存起来，等Future.get()方法的时候才会抛出，可以在主线程捕捉，处理异常更方便

## 线程池参数

~~~java
public ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        ThreadFactory threadFactory,
                        RejectedExecutionHandler handler)
~~~

corePoolSize:核心线程池数量
maximumPoolSize:最大线程数量
keepAliveTime:非核心线程的空闲状态的存活时间(数字1)
unit:时间单位(天、小时、…)
workQueue:工作队列（阻塞队列）

- ArrayBlockingQueue，基于数组结构的有界阻塞队列
- LinkedBlockingQueue，⼀个基于链表结构的有界阻塞队列，默认为`Integer.MAX_VALUE`
- SynchronousQueue，不存储元素的阻塞队列
- DelayQueue，⽀持延时获取元素的⽆界阻塞队列
- PriorityBlockingQueue，⼀个具有优先级的⽆限阻塞队列

threadFactory：线程⼯⼚，主要⽤来创建线程，一般默认`Executors.defaultThreadFactory()`

handler：表示当拒绝处理任务时的策略，有以下四种取值

- `ThreadPoolExecutor.AbortPolicy`: :raising_hand_woman:丢弃任务并抛出RejectedExecutionException异常，**常用**
- ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
- ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前⾯的任务，然后重新尝试执⾏任务
  （重复此过程）
- ThreadPoolExecutor.CallerRunsPolicy：由调⽤线程处理该任务

## 自定义线程池

~~~java
 public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                10,
                20,
                0L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(10),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
        //10个顾客请求
        try {
            for (int i = 1; i <= 100; i++) {
                MyTask task = new MyTask(i);
                threadPool.execute(task);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }
~~~

1. **首先提交的10个任务**：这些任务会立即由线程池中的核心线程开始执行。
2. **接下来提交的10个任务**：由于核心线程已经满了，这些任务会被放入阻塞队列中等待执行。
3. **再接下来提交的10个任务**：阻塞队列已满，线程池会创建新的线程来执行这些任务，直到线程池达到最大线程数20。
4. **剩余的70个任务**：由于线程池和阻塞队列都已满，根据`AbortPolicy`拒绝策略，提交这些任务会导致`RejectedExecutionException`异常。

~~~java
1
2
3
4
5
6
7
8
9
26
28
22
23
24
25
10
27
21
30
29
java.util.concurrent.RejectedExecutionException: Task com.hdb.juclearn.threadpoll.MyTask@728938a9 rejected from java.util.concurrent.ThreadPoolExecutor@21b8d17c[Running, pool size = 20, active threads = 20, queued tasks = 10, completed tasks = 0]
	at java.base/java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(ThreadPoolExecutor.java:2065)
	at java.base/java.util.concurrent.ThreadPoolExecutor.reject(ThreadPoolExecutor.java:833)
	at java.base/java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1365)
	at com.hdb.juclearn.threadpoll.C2_CustomeThreadPool.main(C2_CustomeThreadPool.java:23)
11
12
15
18
14
13
19
17
16
20
~~~

## 创建线程方法选用

超级大坑 在工作中单一的/固定数的/可变的三种创建线程池的方法哪个用的多？

> 答案是一个都不用，我们工作中只能使用自定义的

**FixedThreadPool和SingleThreadPool:**

允许的请求队列长度为Integer.MAX_VALUE,可能会堆积大量的请求，从而导致O0M。

**CachedThreadPool和ScheduledThreadPool:**

允许的创建线程数量为Integer.MAX_VALUE,可能会创建大量的线程，从而导致OOM。

## Tomcat和JDK线程池区别

jdk创建线程池执行前是没有线程的，Tomcat创建线程池的构造函数会额外调用`prestartAllCoreThreads`

函数，预先启动所有的核心线程。

## 线程池如何创建线程

1. 未达到核心线程数（假设10个），创建新的线程执行任务

2. 线程保活，通过阻塞队列让线程执行阻塞队列中的任务，没有获取到任务线程就会阻塞

~~~java
ArrayBlockingQueue<Runnable> arrayBlockingQueue = new ArrayBlockingQueue<>(10);
new Thread(new Runnable() {
    @Override
    public void run() {
        Runnable task;
        while ((task = arrayBlockingQueue.poll()) != null) {
            task.run();
        }
    }
}).start();
~~~

3. 阻塞队列满，创建新的线程执行新来的任务，直到达到最大线程数（包括核心线程数）

## 线程池淘汰策略

假设有200个线程，淘汰到最后只剩10个，就是核心线程，不一定只淘汰后续创建的线程

注意`compareAndDecrementWorkerCount`，成功的线程被淘汰

Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();

一个是限时阻塞，一个是长期阻塞



从队列获取任务时，允许核心线程超时或者当前线程数大于corePoolSize时，使用poll方法拉取任务，超过keepAliveTimel时返回null,否则使用take方法阻塞拉取任务，直到获取到任务或者线程被中断将`timedOut`变为false表示非正常超时

~~~java
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?
 
    // 循环调用，其中会判断线程池状态
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);
    	// 线程池即将关闭状态，如果阻塞队列中也没有任务了，返回null，runWorker方法没有拿到task则退出while循环，销毁线程。
        // 这里根据shutdown和shutdownNow设置不同的线程池状态走不同的逻辑
        // 如果线程池状态是STOP则直接线程数减1，然后返回null，runWorker方法会退出while循环，线程销毁
        // 如果线程池状态是SHUTDOWN则再看看阻塞队列是否为空，为空则线程数减1，后续线程销毁，不会空则继续获取任务
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }
    	// 获取当前线程数
        int wc = workerCountOf(c);
    	// 是否允许超时标识，allowCoreThreadTimeOut核心线程是否允许超时
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
    	// 非核心线程过多或者允许超时的情况下，如果队列为空则工作线程减1，后续销毁线程，这里就返回null
        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }
 
        try {
            // 允许核心线程超时或者线程数大于核心线程数时，采用poll取数据，非阻塞，超过keepAliveTime没有获取到数据就继续自旋获取任务，
            // 不允许核心线程超时或者线程数小于等于核心线程数时，采用take取数据，阻塞等待直到获取到任务或者被中断
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
~~~

## 线程异常

工作线程结束有两种情况，一种是执行任务过程中发生异常，会将异常抛出，当前线程结束销毁（被淘汰掉），但是会创建一个新的线程；

还有一种情况是Woker的task为null或者getTask方法从阻塞队列未获取到任务，线程正常销毁结束。

## 中断

### shutdown

将线程池状态设置为SHUTDOWN后不再接受新任务，将阻塞队列的任务执行完成后，线程池关闭。

### shutdownNow

- 调用shutdownNow方法会将线程池状态设置为STOP后不再接受新任务，然后将所有线程中断（这里的中断已经拿到任务并执行不会响应中断，是在调用getTask获取下一个任务时看线程池状态为STOP则不会再取阻塞队列任务，直接返回null，然后工作线程销毁
- 还有一种情况是正在阻塞等待拿任务，阻塞在poll或take上，都会响应中断，然后再一次循环任务返回null），并将未执行的任务返回，线程池关闭。
