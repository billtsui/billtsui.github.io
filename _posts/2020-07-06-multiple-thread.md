---
layout: post
title: 聊聊Java的多线程(基本概念和基本用法)
categories: 技术 
date:   2020-07-06 14:19:38 +0800
tags: Java
---


## 并行和并发  
聊到多线程，必然要牵扯到多核处理器。多核处理器是指有2+个核心以上的处理器。那么有这么多核心，会带来哪些直观的好处呢？举个栗子，一个普通的哪吒，两只手拿一条火尖枪。一个变身的哪吒(三头六臂)，它还有四只手可以拿其他武器。CPU也一样，多核心以后，计算能力大幅度提升。    

假设你有A、B、C、D四件事要做，处理每件事情的时间是1个小时。对于单核心处理器来说，第1个小时做A，第2个小时做B，第3个小时做C，第4个小时做D。4件事情都做完需要花费4个小时。但对于一个有4核心的处理器来说。它可以用第1个core做A，第2个core做B，以此类推。四件事都做完只需要1个小时。这种处理方式，我们称之为: **Parallel**。中文技术圈可能会叫做**并行**，但我更喜欢叫他**平行**。更加细致的描述，就是即使core 1在做A的时候坏了，剩下的core依然可以不受干扰的继续做事情。    


我们再回到单核心处理器，假设A这件事，需要用3个20分钟来做，但这3个20分钟并不是连续的。先做第一个20分钟，然后等10分钟再做第2个20分钟酱紫。那这个时候，CPU心想反正你这还得等10分钟，我闲着也没事，先做B 10分钟，然后再接着做A。这种处理事情的方式，我们称之为:**Concurrent**。中文技术圈可能会叫做**并发**，我觉得这个词用的还是挺形象的。同时发生，但不一定同时运行。这种处理方式会在A、B、C、D这几件事情中切换，所以需要保存一些状态数据，这叫做:**上下文切换**。     


## 进程和线程    
**进程**是操作系统相关的概念，一个进程，就是一个正在执行中的程序。它读取文件，接收键盘和鼠标的输入，处理数学运算等等等等。    

**线程**是进程划分成更小的运行单位。一个运行的进程，可能有若干个执行不同任务的线程。进程与进程之间互不干扰，但线程与线程之间会有相关联的操作。比如A线程做一个操作要等到B线程结束再开始。    


## 多线程    
**多线程**顾名思义就是有多个正在执行的线程，他们分别处理不同的事情，以达到时间最优的效果。在Java中有一般用前面两种方式创建一个新的线程:    

1.  扩展Thread类覆写run方法(Thread本身就继承了Runnable接口)
2.  继承Runnable接口覆写run方法     
3.  继承Callable接口覆写Run方法

```java
/**
 * @author billtsui
 * 继承Thread接口创建线程
 */
public class CreateThreadByExtendThread extends Thread {
    public static void main(String[] args) {
        System.out.println("这是主线程打印的。ThreadName:" + Thread.currentThread().getName());
        CreateThreadByExtendThread thread = new CreateThreadByExtendThread();
        thread.start();
    }

    @Override
    public void run() {
        System.out.println("这是扩展Thread类创建的线程。ThreadName:" + Thread.currentThread().getName());
    }
}
```    

```java
/**
 * @author billtsui
 * 继承Runnable接口创建线程
 */
public class CreateThreadByImplRunnable implements Runnable {
    public static void main(String[] args) {
        System.out.println("这是主线程打印的。ThreadName:" + Thread.currentThread().getName());
        CreateThreadByImplRunnable thread = new CreateThreadByImplRunnable();
        Thread myThread = new Thread(thread);
        myThread.start();
    }

    @Override
    public void run() {
        System.out.println("这是继承Runnable接口创建的线程。ThreadName:" + Thread.currentThread().getName());
    }
}
```       

```java
package person.billtsui.multiplethread;

import java.util.LinkedList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.*;

/**
 * @author billtsui
 * 继承Callable接口创建线程
 */
public class CreateThreadByImplCallable implements Callable<Integer> {
    public static void main(String[] args) {
        CreateThreadByImplCallable threadByImplCallable = new CreateThreadByImplCallable();
        try {
            List<Future<Integer>> futureList = new LinkedList<>();
            ExecutorService exec = Executors.newFixedThreadPool(10);
            for (int i = 0; i < 100; i++) {
                futureList.add(exec.submit(new CreateThreadByImplCallable()));
            }

            int sum = 0;
            for (Future<Integer> future : futureList) {
                try {
                    sum += future.get();
                } catch (InterruptedException | ExecutionException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(sum);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public Integer call() {
        int number = new Random().nextInt(100);
        System.out.println("Value is " + number);
        System.out.println("这是继承Callable接口创建的线程。ThreadName:" + Thread.currentThread().getName());
        return number;
    }
}
```



从实现方式上面来看，扩展Thread类没有继承Runnable接口灵活，并Java是单类继承的。另外需要特别注意的是，前两种实现方式都需要通过**start()**来申请线程进入等待状态，run方法只是一个普通的方法。针对前面两种实现方式，我们会发现run方法的返回值都是void，这就有一点奇怪。如果我需要多个线程同时计算，并且将计算结果返回相加。这个时候第三种实现方式就很便捷了，并且Callable接口支持泛型。针对第三个例子，后面会有一篇专业的介绍。    

一个线程从创建到结束，会在不同的状态之间切换，Thread.State枚举给出了线程的六种状态。    


- NEW(线程创建但还没有执行start方法)
- RUNNABLE(正在执行)
- BLOCKED(等待获取锁，来继续执行)
- WAITING(等待另一个线程或是某种状态)
- TIMED_WAITING(有时间限制的WAITING，一定时间之后会重新唤醒线程)
- TERMINATED(线程执行结束之后的状态)

## 线程同步    
线程同步其实是一个很简单的问题。同样举个栗子，有一张试卷，你和小明两个人分工做题，小明做了第1题后告诉你，你就不会去做第1题了。线程之间也是一样的，用代码来解释:    

```java
i = 10;
i += 1;
```    
线程A和B同时执行了 **i = 10** 这条语句，A向后执行，B也向后执行,最后得到 **i = 11** 这个结果。这显然和我们预期的 **i = 12** 这个结果不符。我们需要用**synchronized**关键字，解决这个问题。     

**synchronized**是一种同步锁，被synchronized修饰的代码或者类就是线程同步访问的了，保证一个时间段最多只有一个线程访问那一段代码。    

**事实上，synchronized修饰非静态方法、同步代码块的synchronized (this)用法和synchronized (非this对象)的用法锁的是对象，线程想要执行对应同步代码，需要获得对象锁。**    

**synchronized修饰静态方法以及同步代码块的synchronized (类.class)用法锁的是类，线程想要执行对应同步代码，需要获得类锁。** 下面介绍了同步代码执行的过程:

1. 一段synchronized的代码被一个线程执行之前，他要先拿到执行这段代码的权限

2. 在Java里边就是拿到某个同步对象的锁（一个对象只有一把锁）

3.  如果这个时候同步对象的锁被其他线程拿走了，他（这个线程）就只能等了（线程阻塞在锁池等待队列中）。 

4. 取到锁后，他就开始执行同步代码(被synchronized修饰的代码）

5. 线程执行完同步代码后马上就把锁还给同步对象，其他在锁池中等待的某个线程就可以拿到锁执行同步代码了。    

这样就保证了同步代码在统一时刻只有一个线程在执行。

