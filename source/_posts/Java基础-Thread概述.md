---
title: Java基础-Thread概述
date: 2017-03-08 19:33:39
tags: [java,thread,Java基础]
categories: [Java基础]
---
### 线程生命周期

![Threadlife_20170208.jpg](http://upload-images.jianshu.io/upload_images/1419542-7b0fe1fa20076f52.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/620)
线程状态：
* 新建状态：使用new关键字和Thread类或其子类建立一个线程对象后
* 就绪状态：当线程对象调用了start()方法之后，该线程进入就绪状态
* 运行状态：如果就绪状态的线程获取CPU资源，就可以执行run()
* 阻塞状态：
  * 等待阻塞：运行状态中的线程执行wait()方法
  * 同步阻塞：线程在获取 synchronized 同步锁失败
  * 其他阻塞：通过调用线程的sleep()或join()发出I/O请求时
* 死亡状态：一个运行状态的线程完成人文或者其他终止条件发送

### 创建线程
1、继承Thread类
```java
class ThreadDemo extends Thread
```
2、实现Runable接口
```java
public class RunTest implements Runnable
   
Thread thread1 = new Thread(new RunTest());
```
3、通过Callable、Future、FutureTask创建线程
```
// 继承实现Callable接口，声明返回类型
class CallTest implements Callable<Long> 
    
FutureTask<Long> futureTask = new FutureTask<Long>(new CallTest(searchVo));
```

### 线程内置方法
1、sleep：使当前线程（即调用该方法的线程）暂停执行一段时间，让其它线程有机会继续执行，但它不会释放对象锁。
2、yield：__暂停__当前正在执行的线程对象，并执行其他线程。yield的目的是让相同优先级的线程之间能适当的__轮转执行__
3、join：把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如线程B中调用了线程A的JOIN()方法，直到线程A执行完毕后，才会继续执行线程B。
```
    //主线程等待子线程thread执行结束才会输出结果
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(new JoinTest());
        thread.start();
        thread.join(); //加入join()
        System.out.println("主线程结束");
    }
```
4、setDaemon：
Java中线程分为两种类型：用户线程和守护线程。通过Thread.setDaemon(false)设置为用户线程；通过Thread.setDaemon(true)设置为守护线程。如果不设置次属性，默认为用户线程。
```java
  public class DaemonTest extends Thread {
    public void run() {            //永真循环线程
        for (int i = 0; ; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException ex) {
            }
            System.out.println(i);
        }
    }

    public static void main(String[] args) {
        Thread daemonTest = new DaemonTest();
        daemonTest.setDaemon(true);    //调试时可以设置为false，那么这个程序是个死循环，没有退出条件。设置为true，即可主线程结束，test线程也结束。
        daemonTest.start();
        System.out.println("isDaemon = " + daemonTest.isDaemon());
        try {
            System.in.read();   // 接受输入，使程序在此停顿，一旦接收到用户输入，main线程结束，守护线程自动结束
        } catch (IOException ex) {
        }
    }
}
```
如果线程设置为Thread.setDaemon(true)，则主线程结束该程序不会结束，必须等待子线程执行结束整个程序才能结束；如果线程设置为Thread.setDaemon(false)，则主线程结束整个程序就结束。

5、wait、notify、notifyAll
当一个线程进入wait之后，就必须等待其他线程notify/notifyAll，使用notifyAll可以唤醒所有处于wait状态的线程，使其重新进入锁的争夺队列中，而notify只能唤醒一个。
notify被唤醒的线程是随机的，所以通常是没办法指定是谁被唤醒。

### 线程关键字
* volatile
可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入
原子性：对任意单个volatile变量的读/写具有原子性，但是类似于volatile++这种复合操作不具有原子性
* synchronized
当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。然而，当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。
synchronized 关键字，它包括两种用法：synchronized 方法和 synchronized 块。  
	* synchronized 方法：通过在方法声明中加入 synchronized关键字来声明 synchronized 方法
```java
public synchronized void accessVal(int newVal);  
```
	* synchronized 块：通过 synchronized关键字来声明synchronized 块
```java
synchronized(syncObject) {  
//允许访问控制的代码  
}  
```

### ThreadLocal
ThreadLocal是一个关于创建线程局部变量的类。通常情况下，我们创建的变量是可以被任何一个线程访问修改的。而使用ThreadLocal创建的变量只能被当前线程访问，其他线程无法访问和修改。
> 在Java中，栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存。而堆内存中的对象对所有线程可见。堆内存中的对象可以被所有线程访问。

ThreadLocal并不是存放在栈上。ThreadLocal实例实际上也是被其创建的类持有（更顶端应该是被线程持有）。而ThreadLocal的值其实也是被线程实例持有。它们都是位于堆上，只是通过一些技巧将可见性修改成线程可见。
```java
//使用ThreadLocal保存Connection变量 
ThreadLocal<Connection> connThreadLocal = new ThreadLocal<Connection>(); 
//保存到线程本地变量中
connThreadLocal.set(conn);  
// 直接返回线程本地变量  
connThreadLocal.get();

```
