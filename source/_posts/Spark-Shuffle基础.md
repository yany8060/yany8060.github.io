---
title: Spark Shuffle基础
date: 2017-02-06 15:45:36
tags: [spark]
categories: [spark]
---
### Shuffle 基本概念
#### 概述：
* Shuffle描述着数据从map task输出到reduce task 输入的这段过程。在分布式情况下，reduce task需要跨节点拉取其它节点上的map task结果。
* 当Map的输出结果要被Reduce使用时，输出结果需要按key哈希，并且分发到每一个Reducer上去，这个过程就是shuffle。
* 由于shuffle涉及到了磁盘的读写和网络的传输，因此shuffle性能的高低直接影响到了整个程序的运行效率。
<!--more-->
#### Spark 的Shuffle 分为 Write，Read 两阶段
* Write 对应的是ShuffleMapTask，具体的写操作ExternalSorter来负责
* Read 阶段由ShuffleRDD里的HashShuffleReader来完成。如果拉来的数据如果过大，需要落地，则也由ExternalSorter来完成的
* 所有Write 写完后，才会执行Read。 他们被分成了两个不同的Stage阶段。

Shuffle Write ,Shuffle Read 两阶段都可能需要落磁盘，并且通过Disk Merge 来完成最后的Sort归并排序。

### Spark的Shuffle机制
> Spark中的Shuffle是把一组无规则的数据尽量转换成一组具有一定规则的数据。

Shuffle就是包裹在各种需要重分区的算子之下的一个对数据进行重新组合的过程。
Shuffle将数据进行收集分配到指定Reduce分区，Reduce阶段根据函数对相应的分区做Reduce所需的函数处理。


### Shuffle的基本流程
> bucket是一个抽象概念，在实现中每个bucket可以对应一个文件，可以对应文件的一部分或是其他等

![shuffle-write-no-consolidation.png](http://upload-images.jianshu.io/upload_images/1419542-47813c3a4aeccf1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 首先每一个Mapper会根据Reducer的数量创建出相应的bucket，bucket的数量是M×R，其中M是Map的个数，R是Reduce的个数。
* 其次Mapper产生的结果会根据设置的partition算法填充到每个bucket中去。这里的partition算法是可以自定义的，当然默认的算法是根据key哈希到不同的bucket中去。
* 当Reducer启动时，它会根据自己task的id和所依赖的Mapper的id从远端或是本地的block manager中取得相应的bucket作为Reducer的输入进行处理。

### Spark中Shuffle类型
* Hash Shuffle：
第一版是每个map产生r个文件，一共产生mr个文件，但是产生的中间文件太大影响扩展性。而后进行修改，让一个core上的map共用文件，减少文件数目，这样共产生core个文件，但中间文件数目仍随任务数线性增加，仍然难以对应大作业。
* Sort Shuffle：
每个map产生一个文件，彻底解决了扩展性问题


本文只是对Shuffle作了初步的描述，了解基本概念


### 问题
今天遇到如下问题，特来了解一下。
```
17/02/06 11:50:21 ERROR Executor: Exception in task 0.0 in stage 857456.0 (TID 437542)
java.io.FileNotFoundException: /tmp/spark-be115c66-a319-4931-a2ca-81ae9e7a6198/executor-54de96d2-5256-4637-b474-4342b00e755a/blockmgr-0c1c3d9f-c5d7-4b1c-bc12-7773083fa181/18/shuffle_426055_0_0.data.5874ce88-94f5-4c34-b56a-f729d4d4e393 (No such file or directory)
     at java.io.FileOutputStream.open(Native Method)
     at java.io.FileOutputStream.<init>(FileOutputStream.java:212)
     at org.apache.spark.shuffle.sort.BypassMergeSortShuffleWriter.writePartitionedFile(BypassMergeSortShuffleWriter.java:182)
     at org.apache.spark.shuffle.sort.BypassMergeSortShuffleWriter.write(BypassMergeSortShuffleWriter.java:159)
     at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:79)
     at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:47)
     at org.apache.spark.scheduler.Task.run(Task.scala:85)
     at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:274)
     at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
     at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
     at java.lang.Thread.run(Thread.java:722)
```
一般造成此问题的是系统资源不够用
参考网上的解决方案,修改启动参数：
* 添加：--conf spark.shuffle.manager=SORT
 Spark默认的shuffle采用Hash模式，会产生相当规模的文件，与此同时带来了大量的内存开销
* 是因为一个excutor给分配的内存不够，此时，减少excutor-core的数量，加大excutor-memory的值应该就没有问题。
     


参考：
http://blog.jasonding.top/2015/07/14/Spark/【Spark】Spark的Shuffle机制/
http://www.jianshu.com/p/c83bb237caa8
https://github.com/JerryLead/SparkInternals/blob/master/markdown/4-shuffleDetails.md
