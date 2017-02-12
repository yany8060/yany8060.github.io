---
title: Java NIO ByteBuffer概述
date: 2017-02-12 17:29:27
tags: [java-nio]
categories: [java]
---
### Buffer的基本用法
使用Buffer读写数据一般遵循以下四个步骤：
1、分配空间
2、写入数据到Buffer
3、调用flip()方法
4、从Buffer中读取数据
5、调用clear()方法或者compact()方法
当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要通过`flip()`方法`将Buffer从写模式切换到读模式`。在读模式下，可以读取之前写入到buffer的所有数据。
一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用clear()或compact()方法。`clear()方法会清空整个缓冲区`。`compact()方法只会清除已经读过的数据`。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

![Buffer-process.jpeg](http://upload-images.jianshu.io/upload_images/1419542-d832d71903ddb1e1.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 四个主要属性：
* capacity：作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
* position：
  * 当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1。
  * 当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0。 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
* limit：
  * 在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。
  * 当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）
* mark：为某一读过的位置做标记，便于某些时候回退到该位置。

### Buffer基本函数
#### Buffer的分配
```
ByteBuffer buf = ByteBuffer.allocate(48);
```

#### 向Buffer中写数据
从Channel写到Buffer的例子
```
int bytesRead = inChannel.read(buf); //read into buffer.
```
通过put方法写Buffer
```
buf.put(127);
```

#### 从Buffer中读取数据
从Buffer读取数据到Channel的例子：
```
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```
使用get()方法从Buffer中读取数据的例子
```
byte aByte = buf.get();
```
#### flip()方法
flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。
换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。
#### clear()与compact()方法
一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。
如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。
如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。
compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

### 图解ByteBuffer函数过程
#### put
Writes the given byte into this buffer at the current position, and then increments the position. 
写模式下，往buffer里写一个字节，并把postion移动一位。写模式下，一般limit与capacity相等。 
![Buffer-put.png](http://upload-images.jianshu.io/upload_images/1419542-ae958aab995c9963.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### flip
Flips this buffer.  The limit is set to the current position and then the position is set to zero.  If the mark is defined then it is discarded.
After a sequence of channel-read or `put` operations, invoke this method to prepare for a sequence of channel-write or relative  `get` operations.（即读写模型切换）
写完数据，需要开始读的时候，将postion复位到0，并将limit设为当前postion
```
public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
 }
```
![Buffer-flip.png](http://upload-images.jianshu.io/upload_images/1419542-804ddf879bbc2cdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### get
Reads the byte at this buffer's current position, and then increments the position.
从buffer里读一个字节，并把postion移动一位。上限是limit，即写入数据的最后位置
![Buffer-get.png](http://upload-images.jianshu.io/upload_images/1419542-2bbde49b79b2f805.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### clear
Clears this buffer.  The position is set to zero, the limit is set to the capacity, and the mark is discarded.
Invoke this method before using a sequence of channel-read or `put` operations to fill this buffer.
将position置为0，并不清除buffer内容。
```java
    public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }
```
![Buffer-clear.png](http://upload-images.jianshu.io/upload_images/1419542-9a36d39cd8787e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考：
http://ifeve.com/buffers/
博客：
http://yany8060.xyz/