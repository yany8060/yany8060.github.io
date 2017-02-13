---
title: Kafka 概述
date: 2017-02-13 20:11:17
tags: [kafka]
categories: [kafka]
---
### Kafka架构
* Broker：Kafka集群包含一个或多个服务器，这种服务器被称为broker
* Topic：每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。（物理上不同Topic的消息分开存储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处）/Users/yanyong/WebStudy/Blog/YanY/source/_posts/Kafka-概述.md

* Partition：Partition（分片）是物理上的概念，每个Topic包含一个或多个Partition，创建topic时可指定parition数量。每个partition对应于一个文件夹，该文件夹下存储该partition的数据和索引文件。
* Producer：负责发布消息到Kafka broker
* Consumer：消息消费者，向kafka broker读取消息的客户端。每个consumer属于一个特定的consuer group（可为每个consumer指定group name，若不指定group name则属于默认的group）。使用consumer high level API时，同一topic的一条消息只能被同一个consumer group内的一个consumer消费，但多个consumer group可同时消费这一消息。
* Consumer Group：每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）。

Kafka拓扑结构：

![kafka-tupe-20170213.png](http://upload-images.jianshu.io/upload_images/1419542-2deb6eeb9853f960.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### Kafka的Topic & Partition
Topic在逻辑上可以被认为是一个queue，每条消费都必须指定它的Topic，可以简单理解为必须指明把这条消息放进哪个queue里。
消息发送时都被发送到一个topic，其本质就是一个目录，而__topic是由一些Partition Logs(分区日志)组成__，其组织结构如下图所示：
![kafka_log_anatomy_20170213.png](http://upload-images.jianshu.io/upload_images/1419542-0da76ff4a59ef941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证）。

### Kafka 分区机制
Kafka中可以将Topic从物理上划分成一个或多个分区（Partition），每个分区在物理上对应一个文件夹，以“topicName_partitionIndex”的命名方式命名，该文件夹下存储这个分区的所有**消息（.log）**和**索引文件（.index）**，这使得Kafka的吞吐率可以水平扩展。
生产者在生产数据的时候，可以**为每条消息指定Key**，这样消息被发送到broker时，会根据分区规则选择被存储到哪一个分区中，如果分区规则设置的合理，那么所有的消息会将被均分的分布到不同的分区中，这样就实现了负载均衡和水平扩展。另外，在消费端，同一个消费组可以多线程并发的从多个分区中同时消费数据。
默认kakfa.producer.Partitioner接口的类
```
class DefaultPartitioner(props: VerifiableProperties = null) extends Partitioner {
  def partition(key: Any, numPartitions: Int): Int = {
    Utils.abs(key.hashCode) % numPartitions
  }
}
```

### Consumer Group
Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给某一个Consumer）的手段。一个Topic可以对应多个Consumer Group。
* 单播：所有Consumer在同一个Croup里
* 广播：每个Consumer有一个独立的Group

使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，但多个Consumer Group可同时消费这一消息。

### Producer
Producer发送消息到broker时，会根据Paritition机制选择将其存储到哪一个Partition。
producer 写入消息序列图:

![kafka-producer-20170213.png.png](http://upload-images.jianshu.io/upload_images/1419542-1e171553afe93789.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

流程说明:
```
1. producer 先从 zookeeper 的 "/brokers/.../state" 节点找到该 partition 的 leader
2. producer 将消息发送给该 leader
3. leader 将消息写入本地 log
4. followers 从 leader pull 消息，写入本地 log 后 leader 发送 ACK
5. leader 收到所有 ISR 中的 replica 的 ACK 后，增加 HW（high watermark，最后 commit 的 offset） 并向 producer 发送 ACK
```

### Kafka Shell
启动：
```
 ./kafka-server-start.sh ../config/server.properties > /dev/null &
```
查看topic：
```
./kafka-topics.sh --zookeeper 10.211.55.5:2181 --list
```
创建topic：
```
./kafka-topics.sh --create --zookeeper 10.211.55.5:2181 --replication-factor 2 --partitions 1 --topic testYanY
#  replication-factor 副本参数
#  partitions 分区参数

```
删除topic:
```
./kafka-topic.sh --zookeeper 10.211.55.5:2181 --topic  testYanY --delete
```
消费者：
```
./kafka-console-consumer.sh --zookeeper 10.211.55.5:2181 --topic testYanY --from-beginning
```
生产者：
```
./kafka-console-producer.sh --broker-list 10.211.55.5:9092 --topic testYanY
```
    

参考：
http://www.infoq.com/cn/articles/kafka-analysis-part-1/
http://www.cnblogs.com/cyfonly/p/5954614.html

