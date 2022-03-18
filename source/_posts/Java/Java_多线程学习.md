---
title: Java多线程编程
date: 2021-05-20
tags: [多线程]
categories: Java
---

Java语言提供了非常优秀的多线程支持，程序可以通过非常简单的方式来启动多线程，接下来就让我们一起来学习Java多线程编程的相关知识吧，包括：创建、启动、控制线程，以及多线程的同步操作，还有通过Java内建支持的线程池来提高多线程的性能。

### 一、线程概述

每个运行的程序就是一个进程，而当一个程序运行时，内部可能有多个顺序执行流，每个顺序执行流就是一个线程。

#### 1、线程和进程

##### a、进程

程序进入内存运行时，就变成了一个进程（process），进程是处于运行过程中的程序，并且具有一定的独立性，进程是系统进行资源调度和分配的一个独立单位。一般而言，进程具有如下三个特征：

- 独立性：
- 动态性：
- 并发性：

> 并发行和并行性是两个不同的概念：
>
> - 并行：同一时刻，多个指令在多个处理器上同时处理
> - 并发：同一时刻只能有一条指令执行，但是多个进程指令被快速轮换执行，适得其在宏观上具有多个进程同时执行的效果

#### 2、多线程的优势



### 二、线程的创建和启动

#### 1、继承Thread类创建线程类

```java
public class FirstThread extends Thread {
    private int i;

    //重写run方法
    @Override
    public void run() {
        super.run();
        for (i = 0; i < 100; i++) {
            //当线程类启动Thread时，直接使用this即可获取当前的线程
            //Thread的getName()方法返回当前线程的名字
            System.out.println(getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                //创建并启动第一个线程
                new FirstThread().start();
                //创建并启动第二个线程
                new FirstThread().start();
            }
        }
    }
}
```



#### 2、实现Runnable接口创建线程类

```java
public class SecondThread implements Runnable {
    private int i;

    @Override
    public void run() {
        for (; i < 100; i++) {
            //当线程类实现Runnable接口时
            //如果想获取当前线程，只能用Thread.currentThread()方法得到当前进程
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                SecondThread st = new SecondThread();
                //通过new方法创建进程
                new Thread(st, "进程1").start();
                new Thread(st, "进程2").start();
            }
        }
    }
}
```



#### 3、使用Callable和Future创建线程

```java
public class ThirdThread {
    public static void main(String[] args) {
        //先使用Lambda表达式创建Callable<Integer>对象
        //使用FutureTask来包装Callable对象
        FutureTask<Integer> task = new FutureTask<>( (Callable<Integer>)() -> {
                    int i = 0;
                    for (; i < 100; i++) {
                        System.out.println(Thread.currentThread().getName() + " " + i);
                    }
                    //call()可以有返回值
                    return i;
                });
        for (int i = 0; i <100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
            if (i == 20) {
                //实质还是以Callable对象来创建并启动线程的
                new Thread(task, "有返回值的线程").start();
            }
        }
        //需等线程结束后才能获得值
        try {
            //获取线程的返回值
            System.out.println("子线程的返回值" + task.get());
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



#### 4、创建线程的三种方式对比



### 三、线程的生命周期

#### 1、新建和就绪状态



#### 2、运行和阻塞状态



#### 3、线程死亡



### 四、控制线程

#### 1、join线程

```java
public class JoinThread extends Thread {
    //提供一个带参数的构造器，用于设置该线程的名称
    public JoinThread(String name) {
        super(name);
    }
    //重写run()方法，定义线程执行体
    @SneakyThrows
    @Override
    public void run() {
        super.run();
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + " " + i);
            if (i == 30)
                Thread.sleep((long) 0.1);
        }
    }
    public static void main(String[] args) throws InterruptedException {
        //启动子线程
        new JoinThread("新线程").start();
        for (int i = 0; i < 100; i++) {
            if (i == 20) {
                Thread jt = new JoinThread("被join的线程");
                jt.start();
                //main()调用了jt线程的join方法，main(线程)，必须等到jt执行结束才会向下执行
                jt.join();
            }
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```



#### 2、后台线程

```java
public class DaemonThread extends Thread {
    //定义后台线程的线程执行体与普通线程没有多大区别
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            System.out.println(getName() + " " + i);
        }
    }
    public static void main(String[] args) {
        //启动子线程
        DaemonThread t = new DaemonThread();
        //将此线程设置成后台线程
        t.setDaemon(true);
        //启动线程
        t.start();
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        //线程执行到此处，前台线程（main）结束，后台线程随之结束
        System.out.println("结束了");
    }
}
```



#### 3、线程睡眠：sleep

```java
public class SleepTest {
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
    }
}
```



#### 4、改变线程优先级

```java
public class PriorityTest extends Thread {
    //定义一个有参构造器，用于创建线程时指定名称
    public PriorityTest(String name) {
        super(name);
    }
    //线程执行体
    @Override
    public void run() {
        super.run();
        for (int i = 0; i< 500; i++) {
            System.out.println(getName() + "的优先级为：" + getPriority() + "，循环变量值为" + i);
        }
    }
    public static void main(String[] args) {
        //设置主线程的优先级
        Thread.currentThread().setPriority(6);
        for (int i = 0; i < 30; i++) {
            if (i == 10) {
                PriorityTest low = new PriorityTest("低级");
                low.start();
                System.out.println("创建之初的线程优先级：" + low.getPriority());
                //设置该线程为最低优先级
                low.setPriority(Thread.MIN_PRIORITY);
            }
            if (i == 20) {
                PriorityTest high = new PriorityTest("高级");
                high.start();
                System.out.println("创建之初的线程优先级：" + high.getPriority());
                //设置该线程为最高优先级
                high.setPriority(Thread.MAX_PRIORITY);
            }
        }
    }
}
```



### 五、线程同步

#### 1、线程安全问题

##### a、定义账户类：

```java
public class Account {
    //封装账户编号、账户余额的两个成员变量
    private String accountNo;
    private Double balance;
    //构造器
    public Account(String accountNo, Double balance) {
        this.accountNo = accountNo;
        this.balance = balance;
    }
    //setter and getter
    public String getAccountNo() {
        return accountNo;
    }
    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }
    public Double getBalance() {
        return balance;
    }
    public void setBalance(Double balance) {
        this.balance = balance;
    }
    //根据accountNo重写 equals() 和 hashCode() 方法
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        } else if (obj != null && obj.getClass() == Account.class) {
            Account target = (Account) obj;
            return target.getAccountNo().equals(accountNo);
        } else {
            return false;
        }
    }
    //hashCode()方法用于返回字符串的哈希码
    @Override
    public int hashCode() {
        return accountNo.hashCode();
    }
}
```

##### b、定义取钱线程类：

```java
public class DrawThread extends Thread {
    //模拟用户账户
    private Account account;
    //当前取钱线程所希望取得钱数
    private double drawAmount;
    public DrawThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    //当多个线程修改同一共享数据时，将涉及到数据安全问题
    @Override
    public void run() {
        super.run();
        //账户余额大于取钱数目
        if (account.getBalance() >= drawAmount) {
            //吐出钞票
            System.out.println(getName() + " " + drawAmount);
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //修改余额
            account.setBalance(account.getBalance() - drawAmount);
            System.out.println("余额为：" + account.getBalance());
        } else {
            System.out.println("取钱失败！余额不足！");
        }
    }
}
```

##### c、取钱逻辑：

```java
public class DrawTest {
    public static void main(String[] args) {
        //创建一个账户
        Account acct = new Account("123", 1000.0);
        //模拟两个线程向同一账户取钱
        new DrawThread("甲", acct, 800).start();
        new DrawThread("乙", acct, 800).start();
    }
}
```



#### 2、同步代码块

将上面的` DrawThread`修改成如下形式：

```java
public class DrawThread extends Thread {
    //模拟用户账户
    private Account account;
    //当前取钱线程所希望取得钱数
    private double drawAmount;
    public DrawThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    //当多个线程修改同一共享数据时，将涉及到数据安全问题
    @Override
    public void run() {
        super.run();
        //使用account作为同步监视器，任何线程在进入下面的代码块之前，必须先获得对account对象的锁定，其他对象无法获得锁，也就无法修改它
        //"加锁-修改-释放锁"的步骤
        synchronized (account) {
            //账户余额大于取钱数目
            if (account.getBalance() >= drawAmount) {
                //吐出钞票
                System.out.println(getName() + " " + drawAmount);
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //修改余额
                account.setBalance(account.getBalance() - drawAmount);
                System.out.println("余额为：" + account.getBalance());
            } else {
                System.out.println("取钱失败！余额不足！");
            }
        }
      	//同步代码块结束，该线程释放同步锁
    }
}
```



#### 3、同步方法



#### 4、释放同步监视器的锁定



#### 5、同步锁（lock）



#### 6、死锁及常用处理策略

```java
class A {
    public synchronized void foo(B b) {
        System.out.println("当前线程名称：" + Thread.currentThread().getName() + " 进入了A实例的foo()方法");
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("当前线程名称：" + Thread.currentThread().getName() + " 企图调用B的last()方法");
        b.last();
    }
    public synchronized void last() {
        System.out.println("进入了A类l的ast()方法内部");
    }
}

class B {
    public synchronized void bar(A a) {
        System.out.println("当前线程名称：" + Thread.currentThread().getName() + " 进入了B实例的bar()方法");
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("当前线程名称：" + Thread.currentThread().getName() + " 企图调用A的last()方法");
        a.last();
    }
    public synchronized void last() {
        System.out.println("进入了B类的last()方法内部");
    }
}

public class DeadLock implements Runnable {
    A a = new A();
    B b = new B();
    public void init() {
        Thread.currentThread().setName("主线程");
        //调用A的foo()方法
        a.foo(b);
        System.out.println("进入了主线程之后");
    }
    @Override
    public void run() {
        Thread.currentThread().setName("副线程");
        //调用b对象的bar()方法
        b.bar(a);
        System.out.println("进入了副线程之后");
    }
    public static void main(String[] args) {
        DeadLock d1 = new DeadLock();
        //以d1为target启动新线程
        new Thread(d1).start();
        //调用init()方法
        d1.init();
    }
}
```



### 六、线程通信

#### 1、传统的线程通信

##### a、定义账户类：

```java
@Data
public class Account {
    //封装账户编号、账户余额
    private String accountNo;
    private double balance;
    //标识账户中是否有存款
    private boolean flag = false;
    //构造器
    public Account(String accountNo, double balance) {
        this.accountNo = accountNo;
        this.balance = balance;
    }
    //因为账户余额不允许修改，因此只提供getter方法
    public double getBalance() {
        return this.balance;
    }
    public synchronized void draw(double drawAmount) {
        try {
            //如果flag为假，表明账户中没有人存钱进去，取钱方法阻塞
            if (!flag) {
                wait();
            } else {
                //执行取钱操作
                System.out.println(Thread.currentThread().getName() + "取钱：" + drawAmount);
                balance -= drawAmount;
                System.out.println("账户余额为：" + balance);
                //将标识账户是否已有存款的旗标设为false
                flag = false;
                //唤醒其他线程
                notifyAll();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public synchronized void deposit(double depositAmount) {
        try {
            //如果flag为真，表明账户中已经有人存钱进去，存钱方法阻塞
            if (flag) {
                wait();
            } else {
                //执行存钱操作
                System.out.println(Thread.currentThread().getName() + "存钱：" + depositAmount);
                balance += depositAmount;
                System.out.println("账户余额为：" + balance);
                //将标识账户是否已有存款的旗标设为true
                flag = true;
                //唤醒其他线程
                notifyAll();
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

##### b、定义取钱线程

```java
public class DrawThread extends Thread {
    //模拟用户账户
    private Account account;
    //当前取钱线程所希望的线程
    private double drawAmount;
    public DrawThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    //重复100次存钱取钱操作
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            account.draw(drawAmount);
        }
    }
}
```

##### c、定义存钱类：

```java
public class DepositThread extends Thread {
    //模拟用户账户
    private Account account;
    //当前取钱线程所希望的线程
    private double drawAmount;
    public DepositThread(String name, Account account, double drawAmount) {
        super(name);
        this.account = account;
        this.drawAmount = drawAmount;
    }
    //重复100次存钱取钱操作
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            account.deposit(drawAmount);
        }
    }
}
```



#### 2、使用Condition控制线程通信

```java
public class Account {
  //显示定义Lock对象
  private final Lock lock = new ReentrantLock();
  //获得指定Lock对象对应的Condition
  private final Condition cond = lock.newCondition();
  //封装账户编号、账户余额两个成员变量
  private String accountNo;
  private double balance;
  //标识账户中是否已有存款的旗标
  private boolean flag = false;
  //构造器
  public Account() {
  }
  public Account(String accountNo, double balance) {
    this.accountNo = accountNo;
    this.balance = balance;
  }
  //getter 和 setter方法
  public String getAccountNo() {
    return accountNo;
  }
  public void setAccountNo(String accountNo) {
    this.accountNo = accountNo;
  }
  public double getBalance() {
    return balance;
  }
  //取钱操作
  public void draw(double drawAmount) {
    //加锁
    lock.lock();
    try {
      //如果flag为假，表明账户中还没有人存钱进去，取钱方法阻塞
      if(!flag) {
        cond.await();
      } else {
        //执行取钱操作
        System.out.println(Thread.currentThread().getName() + "取钱：" + drawAmount);
        balance -= drawAmount;
        System.out.println("账户余额为：" + balance);
        //将标识账户是否有存款的标志设为false
        flag = false;
        //唤醒其他线程
        cond.signalAll();
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
  }
  //存钱操作
  public void deposit(double depositAmount) {
    //加锁
    lock.lock();
    try {
      //如果flag为真，表明账户中已经有人存钱进去，存钱方法阻塞
      if(flag) {
        cond.await();
      } else {
        //执行存钱操作
        System.out.println(Thread.currentThread().getName() + "存钱：" + depositAmount);
        balance += depositAmount;
        System.out.println("账户余额为：" + balance);
        //将标识账户是否有存款的标志设为true
        flag = true;
        //唤醒其他线程
        cond.signalAll();
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
  }
  //hashCode()和equals()方法
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Account account = (Account) o;
    return Double.compare(account.balance, balance) == 0 && flag == account.flag && Objects.equals(lock, account.lock) && Objects.equals(cond, account.cond) && Objects.equals(accountNo, account.accountNo);
  }
  @Override
  public int hashCode() {
    return Objects.hash(lock, cond, accountNo, balance, flag);
  }
}
```



#### 3、使用阻塞队列（Blocking Queue）控制线程通信

阻塞队列简单使用：

```java
public class BlockingQueueTest {
    public static void main(String[] args) throws InterruptedException {
        //定义一个长度为2的阻塞队列
        BlockingQueue<String> bq = new ArrayBlockingQueue<>(2);
        //也可以用bq.add("Java");和bq.offer("Java");
        bq.put("Java");
        bq.put("Java");
        //此时会形成阻塞
        bq.put("Java");
    }
}
```

使用` BlockingQueue（阻塞队列）`实现线程通信

```java
//生产者，往队列里放元素
class Producer extends Thread {
    private BlockingQueue<String> bq;
    public Producer(BlockingQueue<String> bq) {
        this.bq = bq;
    }
    @Override
    public void run() {
        var strArr = new String[] {"Java","Structs","Spring"};
        for (var i = 0; i < 9999999999L; i++) {
            System.out.println(getName() + "生产者准备生产集合元素");
            try {
                Thread.sleep(200);
                //尝试放入元素，如果队列已满，则线程被阻塞
                bq.put(strArr[i % 3]);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(getName() + "生产完成：" + bq);
        }
    }
}

//消费者，往队列里取元素
class Consumer extends Thread {
    private BlockingQueue<String> bq;
    public Consumer(BlockingQueue<String> bq) {
        this.bq = bq;
    }
    @Override
    public void run() {
        while (true) {
            System.out.println(getName() + "消费者准备消费集合元素！");
            try {
                Thread.sleep(200);
                //尝试取出元素，如果队列已空，则线程被阻塞
                bq.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(getName() + "消费完成：" + bq);
        }
    }
}

//主类
public class BlockingQueueTest2 {
    public static void main(String[] args) {
        //创建一个容量为1的BlockingQueue
        BlockingQueue<String> bq = new ArrayBlockingQueue<>(1);
        //启动三个生产者线程
        new Producer(bq).start();
        new Producer(bq).start();
        new Producer(bq).start();
        //启动一个消费者线程
        new Consumer(bq).start();
    }
}
```



### 七、线程组和未处理的异常

```java
//线程类
class MyThread extends Thread {
    //提供指定线程名的构造器
    public MyThread(String name) {
        super(name);
    }
    //提供指定线程名、线程组的构造器
    public MyThread(ThreadGroup threadGroup, String name) {
        super(threadGroup, name);
    }
    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            System.out.println(getName() + " 线程i的变量" + i);
        }
    }
}
//线程测试主类
public class ThreadGroupTest {
    public static void main(String[] args) {
        //获取主线程所在的线程组，这是所有线程默认的线程组
        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        System.out.println("主线程组的名称：" + mainGroup.getName());
        System.out.println("主线程组是否是后台线程组：" + mainGroup.isDaemon());
        new MyThread("主线程组的线程").start();
      	//创建了一个新的线程组，并将该线程组设置为后台线程组
        var tg = new ThreadGroup("新线程组");
        tg.setDaemon(true);
        System.out.println("tg线程组是否是后台线程组：" + tg.isDaemon());
        var tt = new MyThread(tg, "tg组的线程甲");
        tt.start();
        new MyThread(tg, "tg组的线程乙").start();
    }
}
```



下面程序为主线程设置了异常处理器，当主线程运行抛出未处理异常时，该异常处理器将会起作用：

```java
//定义自己的异常处理器
class MyExHandler implements Thread.UncaughtExceptionHandler {
    //实现uncaughtException()方法，该方法将处理线程的未处理异常
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println(t + "线程出现了异常：" + e);
    }
}
public class ExHandler {
    public static void main(String[] args) {
        //设置主线程的异常处理器
        Thread.currentThread().setUncaughtExceptionHandler(new MyExHandler());
        var a = 5 / 0;
        System.out.println("程序正常结束！");
    }
}
```





### 八、线程池

#### 1、使用线程池管理线程

```java
public class ThreadPoolTest {
    public static void main(String[] args) {
        //创建一个具有固定线程数（6）的线程池
        ExecutorService pool = Executors.newFixedThreadPool(6);
        //使用Lambda表达式创建Runnable对象
        Runnable target = () -> {
            for (int i = 0; i < 100; i++) {
                System.out.println(Thread.currentThread().getName() + "值为" + i);
            }
        };
        //向线程池中提交两个线程
        pool.submit(target);
        pool.submit(target);
        pool.submit(target);
        //关闭线程池
        pool.shutdown();
    }
}
```



#### 2、使用ForkJoinPool利用多CPU

```java
//继承RecursiveAction来实现"可分解"的任务
class PrintTask extends RecursiveAction {
    //每个小任务最多只打印50个数
    private static final int THRESHOLD = 50;
    //开始和结束
    private int start,end;
    //打印从start到end的任务
    public PrintTask(int start, int end) {
        this.end = end;
        this.start = start;
    }
    //总的大任务
    @Override
    protected void compute() {
        //当end和start之间的差距小于THRESHOLD时开始打印
        if (end - start < THRESHOLD) {
            for (int i = start; i < end; i++) {
                System.out.println(Thread.currentThread().getName() + "值为" + i);
            }
        } else {
            //当end和start之间的差距大于THRESHOLD时，将大任务分解成小任务
            int middle = (start + end) / 2;
            PrintTask left = new PrintTask(start, middle);
            PrintTask right = new PrintTask(middle, end);
            //执行两个小任务
            left.fork();
            right.fork();
        }
    }
}

public class ForkJoinPoolTest {
    public static void main(String[] args) throws InterruptedException {
        ForkJoinPool pool = new ForkJoinPool();
        //提交可分解的PrintTask任务
        pool.submit(new PrintTask(0, 300));
        pool.awaitTermination(2, TimeUnit.SECONDS);
      	//关闭线程池
        pool.shutdown();
    }
}
```



下面程序示范使用了` RecursiveTask`对一个长度为100的数组元素值进行累加

```java
//继承RecursiveTask来实现"可分解"的任务
class CalTask extends RecursiveTask<Integer> {
    //每个小任务最多只打印20个数
    private static final int THRESHOLD = 20;
    //开始和结束
    private int start,end;
    //数组
    int arr[];
    //打印从start到end的任务
    public CalTask(int[] arr, int start, int end) {
        this.arr = arr;
        this.end = end;
        this.start = start;
    }
    //总的大任务
    @Override
    protected Integer compute() {
        int sum = 0;
        //当end和start之间的差距小于THRESHOLD时开始累加
        if (end - start < THRESHOLD) {
            for (int i = start; i < end; i++) {
                sum += arr[i];
            }
            return sum;
        } else {
            //当end和start之间的差距大于THRESHOLD时，将大任务分解成小任务
            int middle = (start + end) / 2;
            CalTask left = new CalTask(arr, start, middle);
            CalTask right = new CalTask(arr, middle, end);
            //执行两个小任务
            left.fork();
            right.fork();
            //把两个小任务的累加结果合并起来
            return left.join() + right.join();
        }
    }
}

public class Sum {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        int[] arr = new int[100];
        Random rand = new Random();
        int total = 0;
        //初始化100个数字元素
        for (int i = 0, len = arr.length; i < len; i++) {
            int tmp = rand.nextInt(20);
            //对数组元素赋值
            total += (arr[i] = tmp);
        }
        System.out.println(total);
        //创建一个通用池
        ForkJoinPool pool = ForkJoinPool.commonPool();
        //提交可分解的CalTask任务
        Future<Integer> future = pool.submit(new CalTask(arr, 0, arr.length));
        System.out.println(future.get());
        //关闭线程池
        pool.shutdown();
    }
}
```



### 九、线程相关类

#### 1、ThreadLocal类

```java
class Account {
    //定义一个ThreadLocal类型的变量，该变量将是一个线程局部变量
    //每个线程将会保留该变量的一个副本
    private ThreadLocal<String> name = new ThreadLocal<>();
    //定义一个初始化name成员变量的构造器
    public Account(String str) {
        this.name.set(str);
        //下面代码用于访问当前线程的name副本的值
        System.out.println("---" + this.name.get());
    }
    //name的setter和getter方法
    public String getName() {
        return this.name.get();
    }
    public void setName(String str) {
        this.name.set(str);
    }
}

class MyTest extends Thread {
    //定义一个Account类型的成员变量
    private Account account;
    public MyTest(Account account, String name) {
        super(name);
        this.account = account;
    }
    public void run() {
        //循环10次
        for(var i = 0; i < 10; i++) {
            //当i==6时输出账户名代替换成当前线程名
            if(i == 6) {
                account.setName(getName());
            }
            //输出同一个账户的账户名称和循环变量
            System.out.println(account.getName() + "账户的i值" + i);
        }
    }
}
public class ThreadLocalTest {
    public static void main(String[] args) {
        //启动两个线程，两个线程共享一个Account
        var at = new Account("初始名");
        /**
         * 虽然两个线程共享一个账户，但是只有一个账户名
         * 但由于账户名是ThreadLocal类型的，所以每个线程
         * 都完全拥有各自的账本账户名副本，因此在i==6之后将看到
         * 两个线程访问同一个账户时出现两个不同的账户名
         */
        new MyTest(at, "线程甲").start();
        new MyTest(at, "线程乙").start();
    }
}
```



#### 2、包装线程不安全的集合

```java
//使用 Collections 的 synchronizedMap 方法将一个HashMap包装成线程安全的类
HashMap m = (HashMap) Collections.synchronizedMap(new HashMap());
```



#### 3、线程安全的集合类



#### 4、Java9新增的发布-订阅框架

```java
//创建我自己的订阅者
class MySubscriber<T> implements Flow.Subscriber {
    //发布者与订阅者之间的纽带
    private Flow.Subscription subscription;
    //订阅时会触发该问题
    @Override
    public void onSubscribe(Flow.Subscription subscription) {
        this.subscription = subscription;
        //开始请求数据
        subscription.request(1);
    }
    //接收到数据是会触发该方法
    @Override
    public void onNext(Object item) {
        System.out.println("获取到数据" + item);
        //请求下一条数据
        subscription.request(1);
    }
    //订阅出错时
    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();
        synchronized ("java") {
            "java".notifyAll();
        }
    }
    //订阅结束时触发该方法
    @Override
    public void onComplete() {
        System.out.println("订阅结束时");
        synchronized ("java") {
            "java".notifyAll();
        }
    }
}
public class PubSubTest {
    public static void main(String[] args) {
        //创建一个SubmissionPublisher作为发布者
        SubmissionPublisher<String> publisher = new SubmissionPublisher<>();
        //创建订阅者
        MySubscriber<String> subscriber = new MySubscriber<>();
        //注册订阅者
        publisher.subscribe(subscriber);
        //发布几个数据项
        System.out.println("开发发布数据...");
        List.of("Java", "Go", "Erlang", "Swift", "Lua")
                .forEach(im -> {
                    //提交数据
                    publisher.submit(im);
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                });
        //发布结束
        publisher.close();
        //发布结束之后，为了让线程不会死亡，暂停线程
        synchronized ("java") {
            try {
                "java".wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



### 十、本章小结











