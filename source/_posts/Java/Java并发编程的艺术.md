# 一、并发编程的艺术挑战

## 1、上下文切换

## 2、死锁

## 3、资源限制的挑战



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





# 四、Java并发编程基础

线程作为操作系统调度的最小单位，多个线程能够同时执行，这将显著提升程序性能，多核环境下尤为明显

## 1、线程简介

### （1）什么是线程

现代操作系统在运行一个程序时会为其创建一个进程，现代操作系统调度的最小单位是线程，也叫轻量级进程，在一个进程里可以创建多个线程，每个线程拥有自己的计数器、堆栈、局部变量等属性，并且能够访问共享的内存变量。

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

## 3、线程间通信

## 4、线程应用实例

## 5、本章小结