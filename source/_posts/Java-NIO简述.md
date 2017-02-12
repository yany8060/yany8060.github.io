---
title: Java NIO简述
date: 2017-02-12 16:45:16
tags: [java-nio]
categories: [java]
---
### 概述
Java NIO(New IO)是一个可以替代标准Java IO API的IO API（从Java 1.4开始)，Java NIO提供了与标准IO不同的IO工作方式。

### I/O模型
* 同步阻塞IO：在此种方式下，用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行。JAVA传统的IO模型属于此种方式！
* 同步非阻塞IO：在此种方式下，用户进程发起一个IO操作以后边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费。其中目前JAVA的NIO就属于同步非阻塞IO。
* 异步阻塞IO：此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成，那么为什么说是阻塞的呢？因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！
* 异步非阻塞IO：在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。目前Java中还没有支持此种IO模型。 

### BIO、NIO、AIO
* Java BIO ： __同步并阻塞__，服务器实现模式为__一个连接一个线程__，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。
* Java NIO：__同步非阻塞__，服务器实现模式为__一个请求一个线程__，即客户端发送的连接请求都会注册到多路复用器上，多路复用器__轮询__到连接有I/O请求时才启动一个线程进行处理。
* Java AIO：__异步非阻塞__，服务器实现模式为__一个有效请求一个线程__，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。

### Java NIO由以下几个核心部分组成：
* Channel（通道）
* Buffer（缓冲区）
* Selector（选择器）

#### channel
Java NIO的通道类似流，但又有些不同：
* 既可以从通道中读取数据，又可以写数据到通道。但流的读写通常是单向的。
* 通道可以异步地读写。
* 通道中的数据总是要先读到一个Buffer，或者总是要从一个Buffer中写入
![overview-channels-buffers.png](http://upload-images.jianshu.io/upload_images/1419542-75f7966a4f4478ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Channel的实现
* `FileChannel` 从文件中读写数据
* `DatagramChannel`能通过UDP读写网络中的数据
* `SocketChannel` 能通过TCP读写网络中的数据
* `ServerSocketChannel` 可以监听新进来的TCP连接，像Web服务器那样，对每一个新进来的连接都创建一个SocketChannel。

#### Buffer
Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

#### Selector
Selector是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。
![selectors.png](http://upload-images.jianshu.io/upload_images/1419542-683c996697d33078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。


Selector的创建
```
Selector selector = Selector.open();
```
向Selector注册通道
```
// 与Selector一起使用时，Channel必须处于非阻塞模式下
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
```
通过Selector选择通道：一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道
```
int select() //阻塞到至少有一个通道在你注册的事件上就绪了
int select(long timeout) //和select()一样，除了最长会阻塞timeout毫秒(参数)
int selectNow()  //不会阻塞，不管什么通道就绪都立刻返回
```