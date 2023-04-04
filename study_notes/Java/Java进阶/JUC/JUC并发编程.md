# JUC概述



## JUC是什么

java.util.concurrent在并发编程中使用的工具包。这是一个处理线程的工具包，JDK1.5开始出现



## 线程和进程概念

### 进程与线程

> 进程

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在当代面向线程设计的计算机结构中，进程是线程的容器。



> 线程

线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程可以并发多个线程，每条线程并行执行不同的任务。



> 创建线程的方式

- 继承Thread类

- 实现Runnable接口

- 使用Callable接口

- 使用线程池



### 线程的状态

```java
public enum State {
    /**
     * Thread state for a thread which has not yet started.
     */
    NEW,

    /**
     * Thread state for a runnable thread.  A thread in the runnable
     * state is executing in the Java virtual machine but it may
     * be waiting for other resources from the operating system
     * such as processor.
     */
    RUNNABLE,

    /**
     * Thread state for a thread blocked waiting for a monitor lock.
     * A thread in the blocked state is waiting for a monitor lock
     * to enter a synchronized block/method or
     * reenter a synchronized block/method after calling
     * {@link Object#wait() Object.wait}.
     */
    BLOCKED,

    /**
     * Thread state for a waiting thread.
     * A thread is in the waiting state due to calling one of the
     * following methods:
     * <ul>
     *   <li>{@link Object#wait() Object.wait} with no timeout</li>
     *   <li>{@link #join() Thread.join} with no timeout</li>
     *   <li>{@link LockSupport#park() LockSupport.park}</li>
     * </ul>
     *
     * <p>A thread in the waiting state is waiting for another thread to
     * perform a particular action.
     *
     * For example, a thread that has called {@code Object.wait()}
     * on an object is waiting for another thread to call
     * {@code Object.notify()} or {@code Object.notifyAll()} on
     * that object. A thread that has called {@code Thread.join()}
     * is waiting for a specified thread to terminate.
     */
    WAITING,

    /**
     * Thread state for a waiting thread with a specified waiting time.
     * A thread is in the timed waiting state due to calling one of
     * the following methods with a specified positive waiting time:
     * <ul>
     *   <li>{@link #sleep Thread.sleep}</li>
     *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
     *   <li>{@link #join(long) Thread.join} with timeout</li>
     *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
     *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
     * </ul>
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```



### wait和sleep

1. sleep是Thread的静态方法；wait是Object的方法，任何对象实例都能调用
2. sleep不会释放锁，也不需要占用锁；wait会释放锁，但调用它的前提是当前线程占有锁
3. 它们都可以被interrupt方法中断



### 管程

Java利用管程解决并发编程问题

管程（Monitor），也被称为监视器，是一种同步机制，保证同一时间，只有一个线程访问被保护的数据或者代码



### 用户线程和守护线程

用户线程：自定义线程

守护线程：守护线程一般运行在后台，比如垃圾回收





# 多线程锁



## synchronized

synchronized是Java中的关键字，是一种同步锁，它修饰的对象有几种：

- 代码块，被修饰的代码块叫做同步代码块
- 方法
- 静态方法
- 类



## Lock和synchronized的不同

1. synchronized是java内置关键字，Lock是一个java类
2. synchronized无法判断获取锁的状态，Lock可以判断是否获取到了锁
3. 发生异常时，synchronized会自动释放锁，Lock必须要手动释放锁



# 线程间通信



## 多线程编程步骤

> 编程步骤

1. 创建资源类，在资源类创建属性和方法
2. 在资源类操作方法
   1. 判断
   2. 干活
   3. 通知
3. 创建多个线程，调用资源类的操作方法
4. 防止虚假唤醒问题



> 具体案例

实现四个线程A、B、C、D，其中A和C负责将数值num+1（当且仅当num为0时），B和D负责将数值num-1（当且仅当数值为1时），从而使值num在0和1间交替变化



**使用synchronized方法实现**

```java
class Ticket1 {
    private int num = 0;

    public synchronized void incr() throws InterruptedException {
        while (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + "::" + num);
        this.notifyAll();
    }

    public synchronized void decr() throws InterruptedException {
        while (num != 1) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + "::" + num);
        this.notifyAll();
    }
}

public class ThreadDemo1 {
    public static void main(String[] args) {
        Ticket1 ticket = new Ticket1();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.incr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.decr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.incr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.decr();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }, "D").start();
    }
}
```



**使用Lock方法**

```java
class Ticket1 {
    private int num = 0;

    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void incr() {
        lock.lock();
        try {
            while(num != 0) {
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName() + "::" + num);
            condition.signalAll();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }

    public void decr() {
        lock.lock();
        try {
            while(num != 1) {
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName() + "::" + num);
            condition.signalAll();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
}
public class ThreadDemo {
    public static void main(String[] args) {
        Ticket1 ticket = new Ticket1();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.incr();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.decr();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.incr();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    ticket.decr();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}
```





## 线程间定制化通信



> 具体案例

启动3个线程，按照A--B--C--A的顺序轮流执行

```java
class ShareResource {
    private int flag = 1;
    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    private Condition condition = lock.newCondition();

    public void print5(int loop) {
        lock.lock();
        try {
            while (flag != 1) {
                condition.await();
            }
            for (int i = 1; i <= 5; i++) {
                System.out.println(Thread.currentThread().getName() + "::" + i + ":轮数:" + loop);
            }
            flag = 2;
            condition.signal();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
    public void print10(int loop) {
        lock.lock();
        try {
            while (flag != 2) {
                condition.await();
            }
            for (int i = 1; i <= 10; i++) {
                System.out.println(Thread.currentThread().getName() + "::" + i + ":轮数:" + loop);
            }
            flag = 3;
            condition.signal();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
    public void print15(int loop) {
        lock.lock();
        try {
            while (flag != 3) {
                condition.await();
            }
            for (int i = 1; i <= 15; i++) {
                System.out.println(Thread.currentThread().getName() + "::" + i + ":轮数:" + loop);
            }
            flag = 1;
            condition.signal();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            lock.unlock();
        }
    }
}
public class ThreadDemo1 {
    public static void main(String[] args) {
        ShareResource resource = new ShareResource();
        new Thread(() -> {
            for (int i = 1; i <= 3; i++) {
                resource.print5(i);
            }
        }, "AA").start();
        new Thread(() -> {
            for (int i = 1; i <= 3; i++) {
                resource.print10(i);
            }
        }, "BB").start();
        new Thread(() -> {
            for (int i = 1; i <= 3; i++) {
                resource.print15(i);
            }
        }, "CC").start();
    }
}
```





# 集合的线程安全



## 线程不安全集合

ArrayList线程不安全

```java
// 下面操作线程不安全，会产生java.util.ConcurrentModificationException异常
public class Demo01 {
    @Test
    public void test01() {
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(1);
                System.out.println(list);
            }).start();
        }
    }
}
```



HashSet线程不安全

```java
// 下面操作线程不安全，会产生java.util.ConcurrentModificationException异常
public class Demo01 {
    @Test
    public void test01() {
        Set<Integer> set = new HashSet<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                set.add(1);
                System.out.println(set);
            }).start();
        }
    }
}
```



HashMap线程不安全



## 线程安全集合

ArrayList不安全的替代方法

- Vector

- Collections

- CopyOnWriteArrayList：使用写时复制技术



CopyOnWriteArrayList的add方法：先将数组复制出一份newElements，当对象需要被读取时，会访问旧列表，当当前列表完成元素的添加后，再将旧列表设置为新列表

```java
public void add(int index, E element) {
        synchronized (lock) {
            Object[] es = getArray();
            int len = es.length;
            if (index > len || index < 0)
                throw new IndexOutOfBoundsException(outOfBounds(index, len));
            Object[] newElements;
            int numMoved = len - index;
            if (numMoved == 0)
                newElements = Arrays.copyOf(es, len + 1);
            else {
                newElements = new Object[len + 1];
                System.arraycopy(es, 0, newElements, 0, index);
                System.arraycopy(es, index, newElements, index + 1,
                                 numMoved);
            }
            newElements[index] = element;
            setArray(newElements);
        }
    }
```



HashSet不安全的替代方法

- CopyOnWriteArraySet



HashMap不安全的替代方法

- ConcurrentHashMap



# 多线程锁

 

## synchronized锁的八种情况

1. 标准访问，先打印SMS还是email

```java
class Phone {
    public synchronized void sendSMS() {
        System.out.println("---SMS---");
    }
    public synchronized void sendEmail() {
        System.out.println("---email---");
    }
    public void print() {
        System.out.println("---hello---");
    }
}
public class Case8 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();

        new Thread(() -> {
            phone.sendSMS();
        }).start();

        Thread.sleep(100);

        new Thread(() -> {
            phone.sendEmail();
        }).start();
    }
}

```

答：先打印SMS

原因：第一个线程先创建和运行，提前占有锁



2. sendSMS函数内停4秒，先打印SMS还是email

```java
    public synchronized void sendSMS() {
        TimeUnit.SECONDS.sleep(4);
        System.out.println("---SMS---");
    }
```

答：SMS

原因：第一个线程先抢占锁，即使睡眠4秒，也不会释放锁



3. sendSMS仍休眠4秒，先打印SMS还是hello

答：先打印hello

原因：



4. 现在有两部手机，先打印SMS还是email

答：先打印email

原因：两部手机对应不同锁，互不影响



5. 将sendSMS和sendEmail设为静态同步方法（使用static修饰），先打印SMS还是email

答：先打印SMS

原因：静态方法的锁对象是Phone的Class类，两个方法对应同一个类



6. 将sendSMS和sendEmail设为静态同步方法（使用static修饰），并且使用两部手机，先打印SMS还是email

```java
class Phone {
    public synchronized void sendSMS() {
        System.out.println("---SMS---");
    }
    public synchronized void sendEmail() {
        System.out.println("---email---");
    }
    public void print() {
        System.out.println("---hello---");
    }
}
public class Case8 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        Phone phone2 = new Phone();

        new Thread(() -> {
            phone.sendSMS();
        }).start();

        Thread.sleep(100);

        new Thread(() -> {
            phone2.sendEmail();
        }).start();
    }
}
```

答：先打印SMS

原因：两部手机仍对应同一个Class类



7. 一个静态同步方法（sendSMS），一个普通同步方法（sendEmail），一部手机，先打印短信还是邮件

答：先打印email

原因：sendSMS锁对应Class类，sendEmail锁对应手机类



8. 一个静态同步方法，一个普通同步方法，两部手机，先打印短信还是邮件

答：先打印email

原因：sendSMS锁对应Class类，sendEmail锁对应手机类



synchronized实现同步的基础：Java中的每个对象都可以作为锁

具体形式：

- 对于普通同步方法，锁是当前实例对象
- 对于静态同步方法，锁是当前类的Class对象
- 对于同步方法块，锁是synchronized括号内配置的对象



## 公平锁和非公平锁

### 公平锁

> 概念

多个线程按照申请锁的顺序去获得锁，线程会直接进入队列去排队，永远都是队列的第一位才能得到锁



> 优缺点

优点：

- 所有的线程都能得到资源，不会饿死在队列中

缺点：

- 吞吐量会下降很多，队列里除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大



### 非公平锁

> 概念

多个线程获取锁的时候，会直接去尝试获取，如果获取不到，再进入等待队列



> 优缺点

优点：

- 减少cpu唤醒线程的开销，效率高

缺点：

- 可能导致队列中的线程获取不到锁而导致饿死





## 可重入锁

可重入锁，也叫做递归锁，指的是同一线程外层函数获得锁之后，内层递归函数仍然能够获取该锁

synchronized和lock都是可重入锁





## 死锁

> 概念

死锁，指的是多个进程在运行过程中因争夺资源而造成相互等待的一种僵局。如果没有外力作用，它们都将无法再向前推进。



> 形成原因

- 系统资源不足
- 进程运行推进顺序不合适
- 资源分配不当



> 实例

```java
public class TestDemo {
    private final Object a = new Object();
    private final Object b = new Object();

    @Test
    public void test01() {
        new Thread(() -> {
            synchronized (a) {
                System.out.println(Thread.currentThread().getName() + " 持有锁a，试图获取锁b");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (b) {
                    System.out.println(Thread.currentThread().getName() + " 获取锁b");
                }
            }
        }, "线程一").start();

        new Thread(() -> {
            synchronized (b) {
                System.out.println(Thread.currentThread().getName() + " 持有锁b，试图获取锁a");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (a) {
                    System.out.println(Thread.currentThread().getName() + " 获取锁a");
                }
            }
        }, "线程二").start();
    }
}

```





# Callable接口

目前有两种创建线程的方法：

- 创建Thread的子类
- 使用Runnable创建线程

但Runnable缺少的一项功能是，当线程终止时，我们无法使线程返回结果，为了支持此功能，Java中提供了Callable接口





# JUC的辅助类



## CountDownLatch

假设一个线程需要在其他线程完成后才允许执行，就可以使用CountDownLatch

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "号同学离开了教室");
                countDownLatch.countDown();
            },String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println("班长锁门");
    }
}
```



## CyclicBarrier

在使用中cyclicBarrier会进行循环阻塞

```java
public class CyclicBarrierDemo {
    private static final int NUMBER = 7;

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(NUMBER, () -> {
            System.out.println("集齐7颗龙珠可以召唤神龙");
        });
        for (int i = 0; i < 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "星龙珠被收集到了");
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();


        }
    }
}
```



## Semaphore

适用于多个线程抢占多个资源

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "号车抢占了停车位");
                    TimeUnit.SECONDS.sleep(new Random().nextInt());
                    System.out.println(Thread.currentThread().getName() + "离开了停车位");
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                } finally {
                    semaphore.release();
                }

            },String.valueOf(i)).start();
        }
    }
}
```





# JUC阻塞队列



阻塞队列则是JUC提供的一个用于实现生产者-消费者模式的并发数据结构。

- 阻塞队列的特点是当队列为空时，尝试从队列中获取元素的操作会被阻塞，直到队列中有元素可供获取
- 当队列已满时，尝试往队列中添加元素的操作会被阻塞，直到队列中有空闲位置可供添加。

JUC提供了多种不同的阻塞队列实现，包括：

1. ArrayBlockingQueue：一个基于数组实现的有界阻塞队列，FIFO(先进先出)原则；
2. LinkedBlockingQueue：一个基于链表实现的可选有界阻塞队列，FIFO原则；
3. PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列，元素按照优先级顺序出队；
4. SynchronousQueue：一个不存储元素的阻塞队列，每个插入操作必须等待另一个线程的移除操作，反之亦然。

使用阻塞队列可以方便地实现线程之间的数据传递，从而更好地协调并发任务的执行。



| 方法类型 | 抛出异常 | 特殊值 | 阻塞 | 超时               |
| -------- | -------- | ------ | ---- | ------------------ |
| 插入     | add      | offer  | put  | offer(e,time,unit) |
| 删除     | remove   | poll   | take | poll(time,unit)    |
| 检查     | element  | peek   |      |                    |





# 线程池

## 线程池概述

> 简介

线程池：一种线程使用模式。线程过多会带来调度开销，进而影响缓存局部性和整体性能。而线程池维护着多个线程，等待监督管理者分配可并发执行的任务。这避免了在处理短时间任务时创建与销毁线程的代价。线程池不仅能够保证内核的充分利用，还能防止过分调度



> 优势

线程池做的工作：主要是控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务。如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完成，再从队列中取出任务来执行





## 线程池类型

- Executors.newFixedThreadPool(int)
- Executors.newSingleThreadExecutor()
- Executors.newCachedThreadPool()



```java
public class ThreadDemo {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(5);
        try {
            for (int i = 1; i <= 10; i++) {
                pool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "正在执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            pool.shutdown();
        }

    }
}
```





## 线程池参数



corePoolSize：常驻线程数量

maximumPoolSize：最大线程数量

keepAliveTime：线程存活时间

unit

workQueue：阻塞队列

threadFactory：线程工厂

handler：拒绝工厂





# Fork/Join框架

fork：将复杂问题进行拆分

join：将拆分任务的结果进行合并





# 异步回调

Java异步回调指的是一种编程模式，在该模式中，一个方法在执行完毕后，不是直接返回结果，而是通过异步回调方式将结果传递给调用者。异步回调通常用于处理一些耗时的操作，例如网络请求或者数据库查询等，以避免阻塞主线程的执行。

在Java中，异步回调通常通过回调接口来实现。调用者将一个实现了特定回调接口的对象传递给异步操作的方法，该方法会在执行完毕后调用回调接口中的特定方法，并将结果传递给该方法。调用者可以在回调方法中处理结果，或者将结果传递给其他方法进行处理。

异步回调的优点是能够提高程序的响应速度，因为调用者不需要等待异步操作执行完毕才能继续执行。此外，异步回调还能够充分利用多核CPU，提高程序的并发性能。



# AQS框架



## synchronized底层分析





### synchronized锁升级

1. 无状态

   对象初始状态

2. 偏向锁

3. 轻量级锁

   ​	多个线程加锁

4. 重量级锁

   ​	CAS自旋不成功/锁膨胀导致重度竞争



## 锁的内部实现AQS

> 概念

AQS是AbstractQueuedSynchronizer的缩写，是一个用于实现同步器的抽象类。它是Java并发包中的一个重要类，提供了一种实现阻塞锁和相关同步器（如Semaphore、CountDownLatch、ReentrantLock等）的基础框架。

AQS的核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的。



模拟锁内部的实现方式

```java
// 锁的实现
lock.lock();
...
lock.unlock();
```



> 模拟lock实现：自旋锁

```java
volatile int status = 0;
void lock() {
	while(!compareAndSet(0, 1)) {
	
	}
}

void unlock() {
	status = 0;
}

boolean compareAndSet(int except, int newValue) {

}
```

缺点：耗费CPU资源，没有竞争到锁的线程会调用while循环进行CAS操作

解决思路：让得不到锁的线程让出CPU



> yield+自旋

```java
volatile int status = 0;
void lock() {
	while (!compareAndSet(0, 1)) {
		yield();
	}
}
void unlock() {
	status = 0;
}
```

要解决自旋锁的性能问题必须让竞争锁失败的线程不空转，yield()方法就可以让出CPU资源。

问题：让出CPU资源后，操作系统自主调度线程执行，有可能还是调用当前线程



> sleep+自旋

```java
volatile int status = 0;
void lock() {
	while (!compareAndSet(0, 1)) {
		sleep(10);
	}
}
void unlock() {
	status = 0;
}
```

问题：休眠时间不确定



> park+自旋

```java
volatile int status = 0;
Queue parkQueue;
void lock() {
	while (!compareAndSet(0, 1)) {
		park();
	}
    ...
    unlock();
}
void unlock() {
	status = 0;
    lock_notify();
}
void park() {
	parkQueue.add(currentThread);
	releaseCpu();
}
void lock_notify() {
	Thread t = parkQueue.header();
	unpark(t);
}
```



















