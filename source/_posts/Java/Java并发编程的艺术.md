---
title: Java并发编程艺术
date: 2022-03-10
tags: [多线程]
categories: Java
---

# 一、并发编程的艺术挑战

## 1、上下文切换

### （1）多线程一定快吗？

### （2）测试上下文切换的次数和时长

### （3）如何减少上下文切换

### （4）减少上下文切换实战

## 2、死锁

## 3、资源限制的挑战

## 4、本章小结

# 二、Java并发机制的底层实现原理

Java所实现的并发机制依赖于JVM的实现与CPU指令

## 1、volatile的应用

volatile是轻量级的sychornized，它在多处理器的开发中保证了共享变量的“可见性”

## 2、sychornized的实现原理与应用

重量级锁

为了减少获取锁和释放锁而造成的性能损失，引入了偏向锁和轻量级锁，以及锁的存储结构和升级过程

### （1）Java对象头

### （2）锁的升级与对比

## 3、原子操作的实现原理

原子操作（atomic operation）意为“不可被中断的一个或者一组操作系统”

## 4、本章小结

# 三、Java内存模型

## 1、Java内存模型的基础

### （1）并发编程模型的两个关键问题

### （2）Java内存模型的抽象结构

### （3）从源代码到指令序列的重排序

### （4）并发编程模型的分类

### （5）happens-before简介

## 2、重排序

### （1）数据依赖性

### （2）as-if-serial语意

### （3）程序顺序规则

### （4）重排序对多线程的影响

### 3、顺序一致性

### （1）数据竞争与顺序一致性

### （2）顺序一致性内存模型

### （3）同步程序的执行特性

### （4）未同步程序的执行特性

# 四、Java并发编程基础

线程作为操作系统调度的最小单位，多个线程能够同时执行，这将显著提升程序性能，多核环境下尤为明显

## 1、线程简介

### （1）什么是线程

现代操作系统在运行一个程序时会为其创建一个进程，**现代操作系统调度的最小单位是线程，也叫轻量级进程**，在一个进程里可以创建多个线程，每个线程拥有自己的计数器、堆栈、局部变量等属性，并且能够访问共享的内存变量。

### （2）为什么使用多线程

- 更多的处理器核心
- 更快的响应时间
- 更好的线程模型

### （3）线程的优先级

通过下面的代码来设置线程的优先级

```java
int priority = i < 5 ? Thread.MIN_PRIORITY : Thread.MAX_PRIORITY;
thread.setPriority(priority);
```

线程的优先级不能作为程序正确性的依赖

### （4）线程的状态

- NEW
- RUNNABLE
- BLOCKED
- WAITING
- TIME_WAITING
- TERMINATED

线程在自身的生命周期中，并不是固定的处在某个状态而是随着代码的执行在不同的状态中切换

### （5）Daemon线程

Daemon线程是一种支持类型的线程，因为它主要被用作程序中后台调度以及支持性工作，这意味着，当一个java虚拟机中不存在非Daemon线程的时候，JAVA虚拟机将会退出，可以通过调用Thread.setDaemon(true)将线程设置为Daemon线程

如下：

```java
public class Daemon {
    public static void main(String[] args) {
        Thread thread = new Thread(new DaemonRunner());
        thread.setDaemon(true);
        thread.start();
    }

    static class DaemonRunner implements Runnable {
        @Override
        public void run() {
            try {
                SleepUtils.second(100);
            } finally {
                System.out.println("DaemonThread finally run.");
            }
        }
    }
}
```

## 2、启动和终止线程

### （1）构造线程

构造线程时要考虑线程所属的线程组、线程优先级、是否为Daemon线程等等

### （2）启动线程

通过`start()`方法启动线程

### （3）理解中断

终端可以理解为线程的一个标志位属性，他用来表示一个运行中的线程是否被其他线程进行了中断操作

### （4）过期的suspend()、resume()和stop()

以上三个方法分别对应暂停、恢复和重启

不建议使用的原因：

- `suspend()`方法调用后不释放锁进入睡眠，容易死锁
- `stop()`终结一个线程时不保证资源的正常释放

### （5）安全的终止线程

中断状态是线程的一个标识位，中断操作是一种简便的线程间交互方式

```java
public class Shutdown {
    public static void main(String[] args) throws Exception {
        /**
         * 方式一
         */
        Runner one = new Runner();
        Thread countThread = new Thread(one, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对CountThread进行中断，使CountThread能够感知中断而结束
        TimeUnit.SECONDS.sleep(1);
        countThread.interrupt();
        /**
         * 方式二
         */
        Runner two = new Runner();
        countThread = new Thread(two, "CountThread");
        countThread.start();
        // 睡眠1秒，main线程对Runner two进行取消，使CountThread能够感知on为false而结束
        TimeUnit.SECONDS.sleep(1);
        two.cancel();
    }

    private static class Runner implements Runnable {
        private long i;
        private volatile boolean on = true;

        @Override
        public void run() {
            while (on && !Thread.currentThread().isInterrupted()) {
                i++;
            }
            System.out.println("Count i = " + i);
        }

        public void cancel() {
            on = false;
        }
    }
}
```

## 3、线程间通信

线程间需要相互配合工作

### （1）volatile和synchornized关键字

volatile保证了所有线程对变量访问的可见性

synchornized保证了多个线程在同一时刻只有一个线程处于同步块中，本质是对一个对象监视器进行获取

### （2）等待/通知机制

- notify()

- notifyAll()

- wait()

  会释放锁

直接上代码

```java
public class WaitNotify {
    static boolean flag = true;
    static Object lock = new Object();

    public static void main(String[] args) throws Exception {
        Thread waitThread = new Thread(new Wait(), "WaitThread");
        waitThread.start();
        TimeUnit.SECONDS.sleep(1);

        Thread notifyThread = new Thread(new Notify(), "NotifyThread");
        notifyThread.start();
    }

    static class Wait implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 当条件不满足时，继续wait，同时释放了lock的锁
                while (flag) {
                    try {
                        System.out.println(Thread.currentThread() + " flag is true. wait @ "
                                + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                        lock.wait();
                    } catch (InterruptedException e) {
                    }
                }
                // 条件满足时，完成工作
                System.out.println(Thread.currentThread() + " flag is false. running @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
            }
        }
    }

    static class Notify implements Runnable {
        public void run() {
            // 加锁，拥有lock的Monitor
            synchronized (lock) {
                // 获取lock的锁，然后进行通知，通知时不会释放lock的锁，
                // 直到当前线程释放了lock后，WaitThread才能从wait方法中返回
                System.out.println(Thread.currentThread() + " hold lock. notify @ " + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                lock.notifyAll();
                flag = false;
                SleepUtils.second(5);
            }
            // 再次加锁
            synchronized (lock) {
                System.out.println(Thread.currentThread() + " hold lock again. sleep @ "
                        + new SimpleDateFormat("HH:mm:ss").format(new Date()));
                SleepUtils.second(5);
            }
        }
    }
}
```

### （3）等待/通知的经典范式

生产者消费者模式

```tex
synchornized(对象) {
  while(条件不足) {
  	对象.wait();
  }
  // 处理逻辑
}
```

### （4）管道输入/输出流

管道输入输出流主要包括下面四种实现：`PipedOutputStream`、`PipedInputStream`、`PipedReader`、`PipedWriter`

```java
public class Piped {
    public static void main(String[] args) throws Exception {
        PipedWriter out = new PipedWriter();
        PipedReader in = new PipedReader();
        // 将输出流和输入流进行连接，否则在使用时会抛出IOException
        out.connect(in);
        Thread printThread = new Thread(new Print(in), "PrintThread");
        printThread.start();
        int receive;
        try {
            while ((receive = System.in.read()) != -1)
                out.write(receive);
        } finally {
            out.close();
        }
    }

    static class Print implements Runnable {
        private PipedReader in;

        public Print(PipedReader in) {
            this.in = in;
        }
        
        public void run() {
            int receive;
            try {
                while ((receive = in.read()) != -1)
                    System.out.print((char) receive);
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }
}
```

### （5）Thread.join()的使用

等待线程终止以后才会返回

### （6）ThreadLocal的使用

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键，任意对象为值的存储结构。这个结构被附带到线程上面，并且可以set和get，具体直接看代码

```java
public class Profiler {
    // 第一次get()方法调用时会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Integer> VALUE_THREADLOCAL = ThreadLocal.withInitial(() -> new Random().nextInt());

    public static void set() {
        VALUE_THREADLOCAL.set(new Random().nextInt());
    }

    public static String get() {
        return VALUE_THREADLOCAL.get().toString();
    }

    public static void main(String[] args) throws Exception {
        new ThreadTest().run();
        new ThreadTest().run();
        new ThreadTest().run();
    }
}

class ThreadTest extends Thread {
    @Override
    public void run() {
        Profiler.set();
        System.out.println(Profiler.get());
    }
}
```

## 4、线程应用实例

### （1）等待超时模式

场景例如：调用某个方法等待一段时间（给定一个时间段），如果方法能在给定时间内给出结果就返回，否则返回默认结果

```java
public synchronized Object get(long mills) throws InterruptedException {
  long future = System.currentTimeMillis();
  long remaining = mills;
  // 当前超时大于0 并且 返回值不满足要求
  Object result = null;
  while (result == null && remaining > 0) {
    wait(remaining);
    remaining = future - System.currentTimeMillis();
  }
  return result;
}
```

### （2）简单的数据库连接池

客户端获取连接的过程就是一个等待超时模型

```java
public class ConnectionPool {

    private LinkedList<Connection> pool = new LinkedList<>();

    public ConnectionPool(int initialSize) {
        if (initialSize > 0) {
            for (int i = 0; i < initialSize; i++) {
                pool.addLast(ConnectionDriver.createConnection());
            }
        }
    }

    public void releaseConnection(Connection connection) {
        if (connection != null) {
            synchronized (pool) {
                // 添加后需要进行通知，这样其他消费者能够感知到链接池中已经归还了一个链接
                pool.addLast(connection);
                pool.notifyAll();
            }
        }
    }

    // 在mills内无法获取到连接，将会返回null
    public Connection fetchConnection(long mills) throws InterruptedException {
        synchronized (pool) {
            // 完全超时
            if (mills <= 0) {
                while (pool.isEmpty()) {
                    pool.wait();
                }
                return pool.removeFirst();
            } else {
                long future = System.currentTimeMillis() + mills;
                long remaining = mills;
                while (pool.isEmpty() && remaining > 0) {
                    pool.wait(remaining);
                    remaining = future - System.currentTimeMillis();
                }
                Connection result = null;
                if (!pool.isEmpty()) {
                    result = pool.removeFirst();
                }
                return result;
            }
        }
    }
}
```

我们自己通过动态代理去生成Connection类

```java
public class ConnectionDriver {
    static class ConnectionHandler implements InvocationHandler {
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            if (method.getName().equals("commit")) {
                TimeUnit.MILLISECONDS.sleep(100);
            }
            return null;
        }
    }

    // 创建一个Connection的代理，在commit时休眠1秒
    public static final Connection createConnection() {
        return (Connection) Proxy.newProxyInstance(ConnectionDriver.class.getClassLoader(), new Class<?>[]{Connection.class},
                new ConnectionHandler());
    }
}
```

通过下面的函数模拟简易数据库连接池的工作

```java
public class ConnectionPoolTest {
    static ConnectionPool pool = new ConnectionPool(10);
    // 保证所有ConnectionRunner能够同时开始
    static CountDownLatch start = new CountDownLatch(1);
    // main线程将会等待所有ConnectionRunner结束后才能继续执行
    static CountDownLatch end;

    public static void main(String[] args) throws Exception {
        // 线程数量，可以线程数量进行观察
        int threadCount = 50;
        end = new CountDownLatch(threadCount);
        int count = 20;
        AtomicInteger got = new AtomicInteger();
        AtomicInteger notGot = new AtomicInteger();
        for (int i = 0; i < threadCount; i++) {
            Thread thread = new Thread(new ConnetionRunner(count, got, notGot), "ConnectionRunnerThread");
            thread.start();
        }
        start.countDown();
        end.await();
        System.out.println("total invoke: " + (threadCount * count));
        System.out.println("got connection:  " + got);
        System.out.println("not got connection " + notGot);
    }

    static class ConnetionRunner implements Runnable {
        int count;
        AtomicInteger got;
        AtomicInteger notGot;

        public ConnetionRunner(int count, AtomicInteger got, AtomicInteger notGot) {
            this.count = count;
            this.got = got;
            this.notGot = notGot;
        }

        public void run() {
            try {
                start.await();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
            while (count > 0) {
                try {
                    // 从线程池中获取连接，如果1000ms内无法获取到，将会返回null
                    // 分别统计连接获取的数量got和未获取到的数量notGot
                    Connection connection = pool.fetchConnection(1000);
                    if (connection != null) {
                        try {
                            connection.createStatement();
                            connection.commit();
                        } finally {
                            pool.releaseConnection(connection);
                            got.incrementAndGet();
                        }
                    } else {
                        notGot.incrementAndGet();
                    }
                } catch (Exception ex) {
                    ex.printStackTrace();
                } finally {
                    count--;
                }
            }
            end.countDown();
        }
    }
}
```

使用`CountDownLatch`来进行线程间的同步

### （3）线程池技术以及示例

线程池一方面消除了频繁创建和消亡线程的系统资源开销，另一方面面对过量任务的提交能够平缓劣势

##### a、定义线程池接口

```java
public interface ThreadPool<Job extends Runnable> {
    // 执行一个Job，这个Job需要实现Runnable
    void execute(Job job);
    // 关闭线程池
    void shutdown();
    // 增加工作者线程
    void addWorkers(int num);
    // 减少工作者线程
    void removeWorker(int num);
    // 得到正在等待执行的任务数量
    int getJobSize();
}
```

##### b、线程池接口的默认实现

线程池添加任务时，会向`LinkedList<Job>`添加一个任务，之后在每个线程中同步去获取移除`Job`，并执行run方法

本质为使用一个线程安全的队列连接工作者线程和客户端线程，客户端线程将任务放入工作队列就返回，尔工作者线程不断的取出任务执行

```java
public class DefaultThreadPool<Job extends Runnable> implements ThreadPool<Job> {
    // 线程池最大限制数
    private static final int MAX_WORKER_NUMBERS = 10;
    // 线程池默认的数量
    private static final int DEFAULT_WORKER_NUMBERS = 5;
    // 线程池最小的数量
    private static final int MIN_WORKER_NUMBERS = 1;
    // 这是一个工作列表，将会向里面插入工作
    private final LinkedList<Job> jobs = new LinkedList<>();
    // 工作者列表
    private final List<Worker> workers = Collections.synchronizedList(new ArrayList<Worker>());
    // 工作者线程的数量
    private int workerNum = DEFAULT_WORKER_NUMBERS;
    // 线程编号生成
    private AtomicLong threadNum = new AtomicLong();

    public DefaultThreadPool() {
        initializeWokers(DEFAULT_WORKER_NUMBERS);
    }

    public DefaultThreadPool(int num) {
        workerNum = num > MAX_WORKER_NUMBERS ? MAX_WORKER_NUMBERS : num < MIN_WORKER_NUMBERS ? MIN_WORKER_NUMBERS : num;
        initializeWokers(workerNum);
    }

    public void execute(Job job) {
        if (job != null)
            // 添加一个工作，然后进行通知
            synchronized (jobs) {
                jobs.addLast(job);
                jobs.notify();
            }
    }

    public void shutdown() {
        for (Worker worker : workers)
            worker.shutdown();
    }

    public void addWorkers(int num) {
        synchronized (jobs) {
            // 限制新增的Worker数量不能超过最大值
            if (num + this.workerNum > MAX_WORKER_NUMBERS)
                num = MAX_WORKER_NUMBERS - this.workerNum;
            initializeWokers(num);
            this.workerNum += num;
        }
    }

    public void removeWorker(int num) {
        synchronized (jobs) {
            if (num >= this.workerNum)
                throw new IllegalArgumentException("beyond workNum");
            // 按照给定的数量停止Worker
            int count = 0;
            while (count < num) {
                workers.get(count).shutdown();
                count++;
            }
            this.workerNum -= count;
        }
    }

    public int getJobSize() {
        return jobs.size();
    }

    // 初始化线程工作者
    private void initializeWokers(int num) {
        for (int i = 0; i < num; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            Thread thread = new Thread(worker, "ThreadPool-Worker-" + threadNum.incrementAndGet());
            thread.start();
        }
    }

    // 工作者，负责消费任务
    class Worker implements Runnable {
        // 是否工作
        private volatile boolean running = true;

        public void run() {
            while (running) {
                Job job;
                synchronized (jobs) {
                    // 如果工作者列表是空的，那么就wait
                    while (jobs.isEmpty()) {
                        try {
                            jobs.wait();
                        } catch (InterruptedException ex) {
                            // 感知到外部对WorkerThread的中断操作，返回
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    // 取出一个Job
                    job = jobs.removeFirst();
                }
                if (job != null)
                    try {
                        job.run();
                    } catch (Exception ex) {
                        // 忽略Job执行中的Exception
                    }
            }
        }
        
        public void shutdown() {
            running = false;
        }
    }
}
```

### （4）基于线程池技术的Web服务器



## 5、本章小结



# 五、Java中的锁

Java并发包中的与锁相关的API和组件

## 1、Lock接口

锁是用来控制多个线程访问共享资源的方式，Lock接口出现之前，Java是靠`synchronized`关键字来实现锁功能的

与`synchrnized`相比：

- 需要自己显式的获取与释放锁，缺少了隐式获取释放锁的便捷性
- 拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等同步特性

Lock是一个接口，它定义了锁获取和释放的基本操作

| `void`      | `lock()`获得锁。                                             |
| ----------- | ------------------------------------------------------------ |
| `void`      | `lockInterruptibly()`获取锁定，除非当前线程是 [interrupted](https://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/Thread.html#interrupt--) 。 |
| `Condition` | `newCondition()`返回一个新[`Condition`](https://www.matools.com/file/manual/jdk_api_1.8_google/java/util/concurrent/locks/Condition.html)绑定到该实例`Lock`实例。 |
| `boolean`   | `tryLock()`只有在调用时才可以获得锁。                        |
| `boolean`   | `tryLock(long time, TimeUnit unit)`如果在给定的等待时间内是空闲的，并且当前的线程尚未得到 [interrupted，](https://www.matools.com/file/manual/jdk_api_1.8_google/java/lang/Thread.html#interrupt--)则获取该锁。 |
| `void`      | `unlock()`释放锁。                                           |

## 2、队列同步器

队列同步器（AbstractQueuedSynchornizer），是用来构建锁或者其他同步组件的基础框架

锁和同步器之间的关系：

- 锁是面向使用者的，它定义了使用者与锁交互的接口，隐藏了实现细节
- 同步器是面向锁的开发者的，简化了锁的实现方式，屏蔽了同步状态管理、线程排队、等待与唤醒灯底层操作

锁和同步器很好的隔离了使用者和实现者所需要关注的领域

同步器提供的模板方法基本分为以下三类：

- 独占式获取与释放同步状态
- 共享式获取与释放同步状态
- 查询同步队列中的等待线程情况

队列同步器的实现分析

## 3、重入锁

重入锁（ReentrantLock），顾名思义，就是支持重进入的锁，表示该锁支持一个线程对资源的重复加锁

### （1）实现重进入

- 线程再次获得锁
- 锁的最终释放

### （2）公平与非公平获取锁的区别

如果一个锁是公平的，那么获取锁的顺序就应该符合请求的绝对时间排序，也就是`FIFO`

非公平性锁虽然可能造成线程“饥饿”，但是极少的线程切换，保证了更大的吞吐量

## 4、读写锁

之前提到的锁基本都是“排他锁”，这些锁同一时刻只能允许一个线程进行访问，而读写锁在同一时刻可以允许多个线程同时访问，但是在写线程访问的时候，其他的读线程和写线程均被阻塞

### （1）读写锁的接口与示例

`ReadWriteLock`仅定义了获取读锁和写锁的两个方法：`readLock()`和`writeLock()`

下面通过代码来说明读写锁的使用

```java
public class Cache {
    private static final Map<String, Object>    map = new HashMap<>();
    private static final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    private static final Lock                   r   = rwl.readLock();
    private static final Lock                   w   = rwl.writeLock();
  
    // 获取key对应的value
    public static final Object get(String key) {
        r.lock();
        try {
            return map.get(key);
        } finally {
            r.unlock();
        }
    }
  
    // 设置key对应的value，并返回旧的value
    public static final Object put(String key, Object value) {
        w.lock();
        try {
            return map.put(key, value);
        } finally {
            w.unlock();
        }
    }
  
    // 清空所有内容
    public static final void clear() {
        w.lock();
        try {
            map.clear();
        } finally {
            w.unlock();
        }
    }
}
```

### （2）读写锁的实现分析

接下来分析`ReentrantReadWriteLock`的实现，包括：

- 读写状态的设计
- 写锁的获取与释放
- 读锁的获取与释放
- 锁降级

## 5、LockSupport工具

当需要阻塞或唤醒一个线程的时候，都会使用`LockSupport`工具类来完成相应工作

## 6、Condition接口

任意一个Java对象，都有一组监视器方法（定义在java.lang.Object上），主要包括：

- `wait()`
- `wait(long timeout)`
- `notify()`
- `notifyAll()`

这些方法与`synchornized`同步关键字配合，实现了等待/通知模式，Condition接口也提供了类似于Object的监视器方法

### （1）代码示例

```java
public class ConditionUseCase {
    Lock      lock      = new ReentrantLock();
    Condition condition = lock.newCondition();

    public void conditionWait() throws InterruptedException {
        lock.lock();
        try {
          	// 释放锁并在此等待
            condition.await();
        } finally {
            lock.unlock();
        }
    }

    public void conditionSignal() throws InterruptedException {
        lock.lock();
        try {
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

获取一个`Condition`必须通过`Lock`的`newCondition()`方法

### （2）Condition的实现分析

`ConditionObject`是`AbstractQueuedSynchronizer`的内部类，下面将分析`Condition`的实现，主要包括：等待队列、等待和通知

- 等待队列
- 等待
- 通知

## 7、本章小结

本章介绍了Java并发包中与锁相关的API和组件，并剖析了队列同步器、重入锁、读写锁以及Condition等API的实现细节



# 六、Java并发容器和框架

## 1、ConcurrentHashMap的实现原理与使用

### （1）为什么使用ConcurrentHashMap？

- 线程不安全的HashMap

- 效率低下的HashTable

  所有访问HashTable的线程都必须竞争同一把锁

- ConcurrentHashMap的锁分段技术可以有效提升并发访问率

### （2）ConcurrentHashMap的结构

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/ConcurrentHashMap.jpeg">

通过上图可知，ConcurrentHashMap由Segment数组和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock），扮演锁的角色，HashEntry用于存储键值对数据。

一个ConcurrentHashMap包含一个Segment数组，Segment结构与HashMap类似，是一种链表数组结构

一个Segment包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里面的元素

当对HashEntry里面的数据进行修改的时候，必须先获得与它对应的Segment锁

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/ConcurrentHashMap%E7%BB%93%E6%9E%84%E5%9B%BE.jpeg">

### （3）ConcurrentHashMap的初始化

通过`initialCapacity`、`loadFactory`和`concurrencyLevel`等几个参数来初始化Segment数组

通过`segmentShift（段偏移量）`、`segmentMask（段掩码）`、`每个Segment里面的HashEntry数`来实现的

### （4）定位Segment

两次散列计算，放置Hash冲突

### （5）ConcurrentHashMap的操作

- get

  使用volatile保证不会读到过期值

- put

  定位Segment，判断是否需要扩容，定位添加元素位置将其放入HashEntry中去

- size

## 2、ConcurrentLinkedQueue

实现一个线程安全的队列有两种方式：阻塞算法和非阻塞算法

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，采用了“wait-free”算法（CAS算法）来实现

### （1）ConcurrentLinkedQueue的结构

<img src="https://cos-1301609895.cos.ap-nanjing.myqcloud.com/Java/ConcurrentLinkedQueue%E7%B1%BB%E5%9B%BE.jpeg">



# 七、Java中的13个原子操作类

多线程同时更新一个变量的时候，可能得到意料之外的值，例如：`i++`

## 1、原子更新基本类型类

- AtomicBoolean
- AtomicInteger
- AtomicLong

## 2、原子更新数组

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

## 3、原子更新引用类型类

- AtomicReference
- AtomicReferenceFieldIdUpdater
- AtomicMarkableReference

## 4、原子更新字段类

- AtomicIntegerFieldIdUpdater
- AtomicLongFieldIdUpdater
- AtomicStampedReference

## 5、本章小结

在适当的场景下使用这些类

# 八、Java中的并发工具类

CountDownLatch、CyclicBarrier和Semaphore工具类提供了一种并发流程控制手段，Exchanger提供了一种在线程间交换数据的一种手段

## 1、等待多线程完成的CountDownLatch

它允许一个或多个线程等待其他线程完成操作

## 2、同步屏障CyclicBarrier

### （1）CyclicBarrier简介

### （2）CyclicBarrier的应用场景

### （3）CyclicBarrier和CountDownLatch的区别

## 3、控制并发线程数的Semaphore

## 4、线程间交换数据的Echanger

## 5、本章小结

# 九、Java中的线程池

几乎所有需要异步或者并发执行任务的程序都可以使用线程池，合理地使用线程池有三个好处：

- 降低资源的消耗
- 提高响应的速度
- 提高线程的可管理性

## 1、线程池的实现原理

线程池处理流程如下：

a、判断核心线程池里面的线程是否都在执行任务

b、判断工作队列是否已满

c、判断线程池里面的线程是否都处于工作状态

## 2、线程池的使用

### （1）线程池的创建

### （2）向线程池提交任务

### （3）关闭线程池

### （4）合理地配置线程池

- 任务的性质

  CPU密集型任务、IO密集型任务和混合型任务

- 任务的优先级

  高、中和低

- 任务的执行时间

  长、中和短

- 任务的依赖性

  是否依赖其他系统的资源，如数据库连接等

### （5）线程池的监控

## 3、本章小结

本章介绍了为什么使用线程池、如何使用线程池和线程池的使用原理。

# 十、Executor框架

在Java中，使用线程来异步执行任务。但是线程的创建与销毁都会消耗大量的系统资源。Java的线程既是工作单元，也是执行机制，JDK5开始，将工作单元与执行机制分离开来了。工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

## 1、Executor框架简介

### （1）Executor框架的两级调度

应用程序通过Executor框架控制上层的调度，而下层的调度由系统内核控制

### （2）Executor框架的结构与成员

Executor的结构和Executor框架包含的成员组件

## 2、ThreadPoolExecutor详解

Executor框架的核心类是ThreadPoolExecutor，他是线程池的实现类

### （1）FixedThreadPool详解

可重用固定线程数的线程池

### （2）SingleThreadExecutor详解

使用单个worker线程的线程池

### （3）CachedThreadPool详解

根据需要创建新线程的线程池

## 3、ScheduleThreadPoolExecutor详解

他继承自ThreadPoolExecutor，主要用来在给定的延迟之后执行任务，或者定期执行任务。

### （1）ScheduleThreadPoolExecutor的运行机制

### （2）ScheduleThreadPoolExecutor的实现

## 4、FutureTask详解

FutureTask除了实现Future接口以外，还实现了Runnable接口。因此FutureTask可以提交给Executor执行，也可以调用线程直接执行

- 未启动
- 已启动
- 已完成

### 1、FutureTask简介

### 2、FutureTask的使用

### 3、FutureTask的实现

## 5、本章小结

# 十一、Java并发编程实践

## 1、生产者和消费者模式

### （1）生产者消费者模式实战

### （2）多生产者多消费者场景

### （3）线程池与生产消费者模式

## 2、线上问题定位

## 3、性能测试

## 4、异步任务池

## 5、本章小结

# E、一些小概念

## 1、什么是线程的中断
