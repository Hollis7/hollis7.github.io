---
title: JUC并发编程基础-线程安全
categories:
- JUC
tags:
- JUC
- thread-safety
---
<meta name="referrer" content="no-referrer"/>

## 内容

- 线程安全
- volatile
- 原子类
- 锁
- ReenTrantLock

<!--more-->

## 线程安全

多线程下并发同时对共享数据进⾏读写，会造成数据混乱

当多线程并发访问临界资源时，如果破坏其**原⼦性、可见性、有序性**，可能会造成数据不⼀致。

> 临界资源：共享资源（同⼀对象）同时读写，⼀次仅允许⼀个线程使⽤，才可保证其正确性

### 原子性

原⼦性操作指相应的操作是单⼀不可分割的操作

#### 同步锁synchronized

~~~java
static class StockRunnable implements Runnable {
    @Override
    public synchronized void run() {
        if (stock > 0) {
            // 购买.. stock=stock-1； 1000000-1
            //stock--;
            stock--;
        }
    }
}
~~~

### 可见性

可见性是指当多个线程访问同⼀个变量时，⼀个线程修改了这个变量的值，其他线程能够⽴即看得到修改的值。

1.  synchronized 和 Lock 能保证同⼀时刻只有⼀个线程获取锁然后执⾏同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中

~~~java
static Boolean always = true;
public static void main(String[] args) throws InterruptedException {
    new Thread(() -> {
        while (always) {
            synchronized (always) {
            }
        }

    }).start();
    Thread.sleep(2000);
    always = false;
}
~~~

2. 通过`System.out.println("执行...");`也可以刷新工作变量到主内存，因为这个打印方法包含了一个`synchronized`方法

~~~java
public void println(String x) {
if (getClass() == PrintStream.class) {
    writeln(String.valueOf(x));
} else {
    synchronized (this) {
        print(x);
        newLine();
    }
}
~~~

3. Java提供了volatile关键字来保证可见性。当⼀个共享变量被`volatile`修饰时，它会保证修改的值会⽴即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

~~~java
static volatile Boolean always = true;
~~~

### 有序性

有序性最终表述的现象是**CPU是否按照既定代码顺序依次执⾏指令**。编译器和CPU为了提⾼指令的执⾏效率可能会进⾏**指令重排序**，这使得代码的实际执⾏⽅式可能不是按照我们所认为的⽅式进⾏。

## 破坏临界资源

### 只读

~~~java
fianl
~~~

### 局部变量

每个线程使用自己的局部变量

## ThreadLocal

`ThreadLocal`也就是线程本地变量。如果你创建了⼀个`ThreadLocal`变量，那么访问这个变量的每个线程都会有这个变量的⼀个本地拷贝，多个线程操作这个变量的时候，实际是操作自己本地内存里面的变量，从⽽起到线程隔离的作⽤，避免了线程安全问题。

~~~java
static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
userThreadLocal.set(new User());
User user = userThreadLocal.get();
~~~

### remove

如果在线程池中使⽤ThreadLocal会造成内存泄漏，因为当ThreadLocal对象使⽤完之后，应该要把设置的key，value，也就是Entry对象进⾏回收，但线程池中的线程不会回收，⽽线程对象是通过**强引⽤**指向`ThreadLocalMap`，ThreadLocalMap也是通过强引⽤指向Entry对象，线程不被回收，Entry对象也就不会被回收，从⽽出现内存泄漏。

~~~java
userL.set(new User());
userL.remove();
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/03/08/86364_image-20240308153610276.png" alt="image-20240308153610276" style="zoom:40%;" />

###  InheritableThreadLocal

InheritableThreadLocal是ThreadLocal⼦类

:notes:**主线程和ThreadLocal线程之间的变量是隔离的**

~~~java
 public static void main(String[] args) throws InterruptedException {
    // 替换成 InheritableThreadLocal
    ThreadLocal<String> threadLocal = new ThreadLocal<>();
    threadLocal.set("test");
    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() {
            String value = threadLocal.get();
            System.out.println("value:" + value);
        }
    });
    thread.start();
    Thread.sleep(1000);
}
~~~

这段代码value的值为null

采用InheritableThreadLocal可以获取到test的值，但是使用

~~~java
InheritableThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
~~~

当我们改变了`InheritableThreadLocal`中的值过后，输出的值都是⼀样的。这是因为，**使⽤线程池时，核⼼线程在⾸次使⽤被创建的时候能正确复制⽗线程的上下⽂，但之后已复制上下⽂的核⼼线程不会回收的情况下，线程池中的线程上下⽂将不会再被改变！**

~~~java
threadLocal.set("test111");
~~~

## volatile

volatile关键字具备两个特性，⼀是**可见性**，⼀是**禁⽌指令重排**。

### 可见性

可见性：当⼀个变量被声明为volatile时，它会告诉编译器和CPU将该变量存储在主内存中，⽽不是线程的本地内存中。即每个线程读取的都是主内存中最新的值，避免了多线程并发下的数据不⼀致问题。

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/03/11/93032_image-20240311204210678.png" alt="image-20240311204210678" style="zoom:50%;" />

### Volatile缓存可见性实现原理

底层实现主要是通过**汇编lock前缀指令**，它会锁定这块内存区域的缓存（缓存行锁定）并回写到主内存
IA-32和Intel64架构软件开发者手册**对lock指令的解释**：
**1)会将当前处理器缓存行的数据立即写回到系统内存。**
**2)这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效(MESI协议)**
**3)提供内存屏障功能，使lock前后指令不能重排序**



### 指令重排

指令重排序：在不影响单线程程序执行结果的前提下，计算机为了最大限度的发挥机器性能，会对机器指令重排序优化。

重排序会遵循`as-if-serial`与`happens-before`原则

**as-if-serial**
as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

**happens-before**

happens-before关系的定义如下： 1. 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。 2. **两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么JMM也允许这样的重排序。**

> as-if-serial语义保证单线程内重排序后的执行结果和程序代码本身应有的结果是一致的，happens-before关系保证正确同步的多线程程序的执行结果不被重排序改变。

### DCL双重检测锁

~~~java
public class Singleton {
    public volatile static Singleton instance;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
~~~

**没有volatile会有单例半初始化**

对象的创建过程(new关键字),它简单的分为三个阶段:

**1.分配对象内存空间.**

**2.初始化对象.**

**3.设置对象指向内存空间.**

那么实际上第三步和第二步的关系是可以进行互换的,在JVM的优化中存在一种指令重排序的现象,为了加快JVM的运行速度,指令重排序会在不影响结果的情况下,对JVM的指令进行重新排序.

那么当出现指令重排序时,原本1,2,3的顺序则可能变为1,3,2.此时当代码运行到3时,另一个线程恰好在获取该单例,那么此时代码就会返回一个没有初始化完成的单例对象,这是非常危险的. 

### 内存屏障

1. 在每个volatile写操作的前面插入一个StoreStore屏障
2. 在每个volatile写操作的后面插入一个StoreLoad屏障
3. 在每个volatile读操作的后面插入一个LoadLoad屏障
4. 在每个volatile读操作的后面插入一个LoadStore屏障

~~~java
StoreStore屏障
a=1;//volatile写，a为volatile变量
StoreLoad屏障
b=a;//volatile读
LoadLoad屏障
LoadStore屏障
~~~

## JMM内存模型

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/03/11/14349_image-20240311203311356.png" alt="image-20240311203311356" style="zoom:50%;" />

## JMM数据原子操作

read(读取)：从主内存读取数据
load(载入)：将主内存读取到的数据写入工作内存
use(使用)：从工作内存读取数据来计算
assign(赋值)：将计算好的值重新赋值到工作内存中
store(存储)：将工作内存数据写入主内存
write(写入)：将store过去的变量值赋值给主内存中的变量
1ock(锁定)：将主内存变量加锁，标识为线程独占状态
unlock(解锁)：将主内存变量解锁，解锁后其他线程可以锁定该变量

### 缓存一致性协议(MES)

多个cpu从主内存读取同一个数据到各自的高速缓存，当其中某个cpu修改了缓存里的数据，该数据会马上同步回主内存，其它cpu通过总线嗅探机制可以感知到数据的变化从而将自己缓存里的数据失效

### 缓存加锁

缓存锁的核心机制是基于缓存一致性协议来实现的，一个处理器的缓存回写到内存会导致其他处理器的缓存无效，IA-32和lntel 64处理器使用MESI实现缓存一致性协议

## 原⼦类

1. 不可分割
2. ⼀个操作是不可中断的，即便是多线程的情况下也可以保证
3. java.util.concurrent.atomic
4. 原⼦类的作⽤和锁类似，是为了保证并发情况下的线程安全。不过原⼦类相对于锁有⼀点的优势

**粒度更细**：**原⼦变量可以把竞争范围缩⼩到变量级别**，这是我们可以获得的最细的粒度的情况了，通常锁的粒度都要⼤于原⼦变量的粒度
**效率更⾼**：通常，使⽤原⼦类的效率会⽐使⽤锁的效率更⾼，除了⾼度竞争的情况

### 基础类型

使用原子的方式更新基本类型

- `AtomicInteger`：整型原子类
- `AtomicLong`：长整型原子类
- `AtomicBoolean`：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

- `AtomicIntegerArray`：整型数组原子类
- `AtomicLongArray`：长整型数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

**引用类型**

- `AtomicReference`：引用类型原子类
- `AtomicMarkableReference`：原子更新带有标记的引用类型。该类将 boolean 标记与引用关联起来。
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。

~~~java
public static AtomicInteger atomicStock = new AtomicInteger(10000);
static class StockRunnable implements Runnable {
@Override
public synchronized void run() {
    if (atomicStock.get() > 0) {
        atomicStock.decrementAndGet();
    }
}
}
~~~

## AtomicInteger 类常⽤⽅法

1. public final int get() //获取当前的值
2. public final int getAndSet(int newValue) //获取当前的值，并设置新的值
3. public final int getAndIncrement() //获取当前的值，并⾃增
4. public final int getAndDecrement() //获取当前的值，并⾃减
5. public final int getAndAdd(int delta) //获取当前的值，并加上预期的值
6. boolean compareAndSet(int expect, int update) //如果输⼊的数值等于预期值，则以原⼦⽅式将该值更新为输⼊值（update）

在Java中，`final` 关键字可以用于不同的上下文中，其中一个是用于修饰方法。在方法声明中使用 `final` 关键字表示该方法不能被子类重写（override）。

如果在Java中使用 `final` 关键字修饰变量，这意味着这个变量的值只能被赋值一次。一旦赋值后，它的值将不能被修改。

## AtomicIntegerArray类常⽤⽅法

~~~java
public static void aia() {
    int[] value = new int[]{1, 2};

    AtomicIntegerArray aia = new AtomicIntegerArray(value);

    System.out.println("ai.getAndSet(0, 3)");
    aia.getAndAdd(0, 3);//将索引0处的值加3
    System.out.println("aia.get(0) = " + aia.get(0));
    System.out.println("value[0] = " + value[0]);

    aia.compareAndSet(1, 2, 5);
    System.out.println("aia.compareAndSet(1, 2, 5)");
    System.out.println("aia.get(1) = " + aia.get(1));
}
~~~

## AtomicReference类常⽤⽅法

~~~java
public  static  void ar() {

        User user = new User("xushu", 20);
        AtomicReference<User> atomicUserRef = new AtomicReference<User>(user);
        //atomicUserRef.set(user);
        System.out.println("atomicUserRef.get() = " + atomicUserRef.get().toString());

        User updateUser = new User("zhuge", 22);
        atomicUserRef.compareAndSet(user, updateUser);
        System.out.println("atomicUserRef.compareAndSet(user, updateUser);");

        System.out.println("atomicUserRef.get() = " + atomicUserRef.get().toString());
    }
~~~

## AtomicIntegerFieldUpdate类常⽤⽅法

~~~java
public static void au() {
        // 创建原子更新器,并设置需要更新的对象类和对象的属性
        AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.
                newUpdater(User.class, "age");
        // 设置xushu的年龄是10岁
        User xushu = new User("xushu", 10);
        // 徐庶长了一岁,但是仍然会输出旧的年龄
        System.out.println(a.getAndIncrement(xushu));
        // 输出xushu现在的年龄
        System.out.println(a.get(xushu));
    }
~~~

## Adder累加器

### LongAdder

LongAdder适合的场景是统计求和计数的场景，⽽且LongAdder基本只提供了add⽅法，⽽AtomicLong还具有cas⽅法

~~~java
// 无参构造函数  从0开始
LongAdder longAdder = new LongAdder();

longAdder.increment();
longAdder.increment();
System.out.println("longAdder = " + longAdder.longValue());
~~~

## LongAccumulator

~~~java
LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x + y, 1);
longAccumulator.accumulate(3);
longAccumulator.accumulate(3);
longAccumulator.accumulate(3);
System.out.println("longAccumulator = " + longAccumulator.get());
~~~

适⽤于需要⼤量计算，并且需要并⾏计算的场景，如果不需要并⾏计算，可⽤for循环解决问题，⽤了Accumulator累加器可利⽤多核同时计算，提供效率

计算的顺序不能成为瓶颈，线程1可能在线程5之后运⾏，也可能在之前运⾏，不影响最终结果

## 乐观锁和悲观锁

**悲观锁**：认为当前线程使用数据的时候⼀定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。

> synchronized关键字和Lock的实现类都是悲观锁，适合写操作多的场景，先加锁可以保证写操作时数据正确。

**乐观锁**：乐观锁认为当前线程使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。

> 乐观锁在Java中是通过使⽤⽆锁编程来实现，最常采⽤的是CAS算法，Java原⼦类中的递增操作就通过CAS⾃旋实现的，适合读操作多的场景，不加锁的特点能够使其读操作的性能⼤幅提升。

乐观锁直接去操作同步资源，是⼀种⽆锁算法，得之我幸不得我命，重试。

## CAS(Compare-and-Swap)

即比较并替换

~~~java
 static class StockRunnable implements Runnable {
        @Override
        public  void run() {
            int oldValue;
            int newValue;
            do {
                oldValue = stock.get();
                newValue = oldValue -1;
            } while (!stock.compareAndSet(oldValue, newValue));
        }
    }
~~~

### 缺点

1. 循环时间⻓开销很⼤`do{...}while`
2. 引出来ABA问题

### ABA问题

当第⼀个线程执⾏CAS（V，E，U）操作。在获取到当前变量V，准备修改为新值U前，另外两个线程已连续修改了两次变量V的值，使得该值⼜恢复为旧值，这样的话，我们就⽆法正确判断这个变量是否已被修改过

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/03/20/93282_image-20240320200400885.png" alt="image-20240320200400885" style="zoom: 33%;" />

### 解决方法AtomicStampedReference

在调用compareAndSet方法时会设置标记stamp，通过查看标记判断是否被修改。

~~~java
AtomicStampedReference<Integer> balance = new AtomicStampedReference(1000, 0);
 // 财务发3000工资
balance.compareAndSet(balance.getReference(), 4000, balance.getStamp(), balance.getStamp() + 1);
 // 老婆取3000工资
balance.compareAndSet(balance.getReference(), 1000, balance.getStamp(), balance.getStamp() + 1);
~~~

~~~java
if (balance.getReference() > 3000) {
    System.out.println("张三美滋滋" + balance.getReference());
} else {
    if (balance.getStamp() == 1) {
        System.out.println("张三找财务麻烦:");
    } else {
        System.out.println("张三找老婆麻烦:");
    }
}
~~~



## 自旋锁(spinlock)

是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

属于乐观锁，实现基础是CAS算法机制

## wait/sleep区别

1、所属类不同：sleep是线程中的方法，但是wait是Object中的方法。
2、语法不同：sleep方法不依赖于同步器synchronized,但是wait需要依赖synchronized关键字。
3、参数不同：sleep必须设置参数时间，wait可以不设置时间，不设置将一直休眠。
4、释放锁资源不同：sleep方法不会释放Iock,**但是wait会释放**，而且会加入到等待队列中。

5、唤醒方式不同：sleep:不需要被唤醒（休眠之后退出阻塞），但是**wait需要**(不指定时间需要被别人中断)。
6、线程进入状态不同：调用sleep方法线程会进入`TIMED_WAITING`有时限等待状态，而调用无参数的wait方法，线程会进入`WAITING`无时限等待状态。

## 生产者消费者

~~~java
package com.hdb.juclearn.javalock;

import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;

public class C4_SharedQueue {

    // 声明队列最大长度
    private int queenSize = 10;
    private Random random = new Random();
    private ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<>(queenSize);

    class Producter extends Thread {
        @Override
        public void run() {
            // 保证生产者在整个过程中是线程安全的
            synchronized (queue) {
// 1、 判断当前队列长度是否小于最大长度
                if (queue.size() < queenSize) {
                    // 2、如果小于 生产者就可以生产消息了
                    // 2.1 往队列queue添加一条信息
                    queue.add(random.nextInt());
                    System.out.println("生产者往队列中加入消息，队列当前长度：" + queue.size());
                    // 2.2 唤醒消费，有活了
                    queue.notify();
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } else {
                    // 3. 如果大于 生产者停止工作， 稍微歇一歇
                    try {
                        queue.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        queue.notify();
                    }
                }
            }
        }
    }

    class Consumer extends Thread {
        @Override
        public void run() {
            // 消费者需要重复的工作
            while (true) {
                // 保证整个消费的过程是线程安全
                synchronized (queue) {
                    // 如果队列为空， 消费者睡眠
                    if (queue.isEmpty()) {
                        System.out.println("当前队列为空...");
                        try {
                            queue.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                            // 一旦出现异常，手动唤醒
                            queue.notify();
                        }

                    } else {
                        // 队列不为空， 消费者需要进行消费

                        // 消费头部的信息
                        Integer value = queue.poll();
                        System.out.println("消费者从队列中消费了信息:" + value + "，队列当前长度：" + queue.size());
                        // 唤醒生产者， 可以继续工作了
                        queue.notify();

                        // 模拟业务处理
                        try {
                            Thread.sleep(1000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }

                    }
                }
            }
        }
    }

    public static void main(String[] args) {
        C4_SharedQueue test = new C4_SharedQueue();
        // 消费者持续运行
        Consumer consumer = test.new Consumer();
        consumer.start();
        // 生产10条消息
        for (int i = 0; i < 10; i++) {
            Producter producter = test.new Producter();
            producter.start();
        }
    }
}

~~~

### synchronized

**其实在JDK 1.6之前**，Java内置锁还是⼀个重量级锁，是⼀个效率⽐较低下的锁， 会由jvm⽤户态切换到操作系统的管程来实现互斥。

### 管程

管程是指管理共享变量以及对共享变量操作的过程，让它们⽀持并发。翻译成Java领域的语⾔，就是**管理类的状态变量，让这个类是线程安全的** 。

synchronized关键字和wait()、notify()、notifyAll()这三个⽅法是Java中实现管程技术的组成部分。

Monitor有两⼤作⽤：同步和互斥

wait/notify基于monitor做的

synchronized关联了monitor

### 锁升级

在JDK 1.6之后，JVM为了提⾼锁的获取与释放效率，对synchronized的实现进⾏了优化，引⼊了偏向锁
和轻量级锁，从此以后Java内置锁的状态就有了4种（⽆锁、偏向锁、轻量级锁和重量级锁）

#### 依赖

~~~xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
    <scope>compile</scope>
</dependency>
~~~

#### 无锁状态

~~~java
class T {
        Integer age;
    }
T o = new T();
System.out.println(ClassLayout.parseInstance(o).toPrintable());
~~~

```
OFFSET  SIZE                TYPE DESCRIPTION                               VALUE
      0     4                     (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
```

锁对象刚创建，没有任何线程竞争，对象处于⽆锁状态+不可偏向状态

`00000001`代表无锁as

#### 偏向锁

jdk中偏向锁存在延迟4秒启动，也就是说在jvm启动后4秒后创建的对象才会开启偏向锁

创建的对象状态为 对象处于⽆锁状态+可偏向状态

这个锁将自己偏向了当前线程，心里默默地藏着线程id， 在这里，我们就引入了“偏向锁”的概念。

`101`

#### 轻量级锁

当两个或以上线程交替获取锁，**但并没有在对象上并发的获取锁时**，偏向锁升级为轻量级锁。在此阶段，线程采取CAS的⾃旋⽅式尝试获取锁，避免阻塞线程造成的cpu在⽤户态和内核态间转换的消耗主线程⾸先对user对象加锁，⾸次加锁为`000`偏向锁

#### 重量级锁

两个或以上线程并发的在同⼀个对象上进⾏同步时，为了避免⽆⽤⾃旋消耗cpu，轻量级锁会升级成重量级锁。这时markword中的指针指向的是monitor对象（也被称为管程或监视器锁）的起始地址

`10`

## 可重入锁

是指在同⼀个线程在外层⽅法获取锁的时候，再进⼊该线程的内层⽅法会⾃动获取锁（前提，锁对象得是同⼀个对象），不会因为之前已经获取过还没释放⽽阻塞。

~~~java
 private static final Object objectLockA = new Object();

    public static void main(String[] args) {

        new Thread(() -> {
            relock();
        }, "a").start();
    }

    private static Integer i = 0;

    public static void relock() {

        synchronized (objectLockA) {
            if (i == 3) {
                return;
            } else {
                i++;
                System.out.println(i + "调用");
                relock();
            }
        }
    }
~~~

## ReentrantLock

ReentrantLock是Lock的默认实现

1. 可重⼊锁：可重⼊锁是指同⼀个线程可以多次获得同⼀把锁；ReentrantLock和关键字Synchronized都是可重⼊锁
2. 可中断锁：可中断锁时⼦线程在获取锁的过程中，是否可以相应线程中断操作。synchronized是不可中断的，ReentrantLock是可中断的
3. 公平锁和⾮公平锁：公平锁是指多个线程尝试获取同⼀把锁的时候，获取锁的顺序按照线程到达的先后顺序获取，⽽不是随机插队的⽅式获取。synchronized是⾮公平锁**，⽽ReentrantLock是两种都可以实现，不过默认是⾮公平锁**

### synchronized的局限性

- 当线程尝试获取锁的时候，如果获取不到锁会⼀直阻塞，这个阻塞的过程，⽤户⽆法控制
- 如果获取锁的线程进⼊休眠或者阻塞，**除⾮当前线程异常，否则其他线程尝试获取锁必须⼀直等待**

### ReentrantLock使用

~~~java
public class C6_ReentrantLock {
    private static Integer stock = 100000;
    public static ReentrantLock lock = new ReentrantLock();

    static class StockRunnable implements Runnable {

        @Override
        public void run() {
            try {
                lock.lock();
                stock--;
            } finally {
                if (lock.isHeldByCurrentThread()) {
                    lock.unlock();
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newCachedThreadPool();
        StockRunnable task = new StockRunnable();
        try {
            for (int i = 0; i < 100000; i++) {
                threadPool.execute(task);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdownNow();
            threadPool.awaitTermination(1000, TimeUnit.SECONDS);
        }
        System.out.println("剩余库存：" + stock);
    }

}

~~~

上⾯代码需要注意lock.unlock()⼀定要放在finally中，否则，若程序出现了异常，锁没有释放，那么其他线程就再也没有机会获取这个锁了。

### trylock

获取锁限时等待的⽅法`tryLock（）`，可以选择传⼊时间参数，表示等待指定的时间，⽆参则表示⽴即返回锁申请的结果：true表示获取锁成功，false表示获取锁失败。

~~~java
 if (lock1.tryLock(2, TimeUnit.SECONDS)) {  // lock()
    System.out.println(System.currentTimeMillis() + ":" + this.getName() + "获取到了锁!");
    //获取到锁之后，休眠5秒
    TimeUnit.SECONDS.sleep(5);
}
~~~

### lockInterruptibly() 

在lock的基础上可以加上了中断机制，如果在外部调用`interrupt`，会中断等待。

~~~java
 Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock.lockInterruptibly();
                    System.out.println("线程 2 获取了锁");
                } catch (InterruptedException e) {
                    System.out.println("线程 2异常");
                    e.printStackTrace();
                } finally {
                    if (lock.isHeldByCurrentThread()) {
                        lock.unlock();
                    }
                }
            }
        });
...
thread2.interrupt(); // interrupt thread2
~~~

- 线程调⽤interrupt()之后，线程的中断标志会被置为true
- 触发InterruptedException异常之后，线程的中断标志会被清空，即置为false

## 公平锁/非公平锁

公平锁和⾮公平锁：公平锁是指多个线程尝试获取同⼀把锁的时候，获取锁的顺序按照线程到达的先后顺序获取，⽽不是随机插队的⽅式获取。synchronized是⾮公平锁**，⽽ReentrantLock是两种都可以实现，不过默认是⾮公平锁**

~~~java
// 默认非公平锁，  true  公平锁
private static ReentrantLock fairLock = new ReentrantLock(true);
~~~

### 考点

**1、 为什么会有公平锁/⾮公平锁的设计为什么默认⾮公平？**

> 恢复挂起的线程到真正锁的获取还是有时间差的，从开发⼈员来看这个时间微乎其微，但是从CPU的⻆度来看，这个时间差存在的还是很明显的。**所以⾮公平锁能更充分的利⽤CPU的时间⽚，尽量减少CPU空闲状态时间。**
>
> **使⽤多线程很重要的考量点是线程切换的开销**，当采⽤⾮公平锁时，当1个线程请求锁获取同步状态，然后释放同步状态，因为不需要考虑是否还有前驱节点，所以刚释放锁的线程在此刻再次获取同步状态的概率就变得⾮常⼤，所以就减少了线程的开销。

**2、 使⽤公平锁会有什么问题**

> 公平锁保证了排队的公平性，⾮公平锁霸⽓的忽视这个规则，所以就有可能导致排队的⻓时间在排队，也没有机会获取到锁，这就是传说中的“锁饥饿”

**3、 什么时候⽤公平？什么时候⽤⾮公平**

> 如果为了更⾼的吞吐量，很显然⾮公平锁是⽐较合适的，因为节省很多线程切换时间，吞吐量⾃然就上去了；否则那就⽤公平锁，⼤家公平使⽤。

## 共享锁和排它锁

- 排它锁⼜称独占锁，获得了以后既能读⼜能写，其他没有获得锁的线程不能读也不能写，典型的synchronized就是排它锁
- 共享锁⼜称读锁，获得了共享锁以后**可以查看但⽆法修改和删除数据**，其他线程也能获得共享锁，也可以查看但不能修改和删除数据

> 在没有读写锁之前，我们虽然保证了线程安全，但是也浪费了⼀定的资源，因为多个读操作同时进⾏并没有线程安全问题

### 读写锁的规则

1. 多个线程只申请读锁，都能申请到
2. **如果有⼀个线程已经占⽤了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会⼀直等待读锁释放该锁**
3. 如果有⼀个线程已经占⽤写锁，则其他线程申请写锁或读锁都要等待它释放

> 要么多读要么⼀写

~~~java
private static ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
private static ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();
private static ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();、
// ...
readLock.lock();
readLock.unlock();
~~~

## synchronized与Lock的区别

- ⾸先synchronized是java内置关键字，在jvm层⾯，Lock是个java类；
-  synchronized⽆法判断是否获取锁的状态，Lock可以判断是否获取到锁；
- synchronized会⾃动释放锁（a线程执⾏完同步代码会释放锁；b线程执⾏过程中发⽣异常会释放锁），Lock需在finally中⼿⼯释放锁（unlock（）⽅法释放锁），否则容易造成线程死锁；
- ⽤synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会⼀直等待下去，⽽Lock锁就不⼀定会等待下去，如果尝试获取不到锁，线程可以不⽤⼀直等待就结束了；
- synchronized的锁可重⼊、不可中断、⾮公平，⽽Lock锁可重⼊、可中断、可公平（两者皆可）
