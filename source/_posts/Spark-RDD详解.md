---
title: Spark RDD详解
date: 2017-02-07 19:58:41
tags: [spark]
categories: [spark]
---
### RDD基本概念：
* RDD（ resilient distributed dataset，弹性分布式数据集）：spark的基本计算单元，可以通过一系列算子进行操作（主要有Transformation和Action操作）。
* RDD是一个不可修改的，分布的对象集合。每个RDD由多个分区组成，每个分区可以同时在集群中的不同节点上计算。RDD可以包含Python，Java和Scala中的任意对象。                
* DAG （Directed Acycle graph，有向无环图）：反应RDD之间的依赖
* 窄依赖（Narrow dependency）：子RDD依赖于父RDD中固定的data
* 宽依赖（Wide Dependency）：子RDD对父RDD中的所有data partition都有依赖

<!-- more -->

### RDD构建图
![sparkRdd.jpg](http://upload-images.jianshu.io/upload_images/1419542-ae8b332de4f1f91f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### RDD来源：
*  parallelizing an existing collection in your driver program（程序内部已经存在的数据集）
* referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase, or any data source offering a Hadoop InputFormat（外部存储系统）



### RDD特点：
1、有一个分片列表。就是能被切分，和hadoop一样的，能够切分的数据才能并行计算。
2、有一个函数计算每一个分片，这里指的是下面会提到的compute函数。
3、对其他的RDD的依赖列表，依赖还具体分为宽依赖和窄依赖，但并不是所有的RDD都有依赖。
4、可选：key-value型的RDD是根据哈希来分区的，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce。
5、可选：每一个分片的优先计算位置（preferred locations），比如HDFS的block的所在位置应该是优先计算的位置。
```
// return the set of partitions in this RDD
protected def getPartitions: Array[Partition]
/* 分片列表 */

// compute a given partition
def compute(split: Partition, context: TaskContext): Iterator[T]
/* compute 对分片进行计算,得出一个可遍历的结果 */

// return how this RDD depends on parent RDDs
protected def getDependencies: Seq[Dependency[_]] = deps
/* 只计算一次，计算RDD对父RDD的依赖 */

@transient val partitioner: Option[Partitioner] = None
 /* 可选的，分区的方法，针对第4点，类似于mapreduce当中的Paritioner接口，控制key分到哪个reduce */

protected def getPreferredLocations(split: Partition): Seq[String] = Nil
/* 可选的，指定优先位置，输入参数是split分片，输出结果是一组优先的节点位置*/
```


### RDD操作
* transformations：接受RDD并返回RDD
Transformation采用惰性调用机制，每个RDD记录父RDD转换的方法，这种调用链表称之为血缘（lineage）。
* action：接受RDD但是返回非RDD
Action调用会直接计算。

### RDD优化技巧
* RDD缓存：需要使用多次的数据需要cache，否则会进行不必要的重复操作。
可以通过`rdd.persist(newLevel: StorageLevel, allowOverride: Boolean)`或`rdd.cache()`（就是调用persist）来缓存数据
* 转换并行化：RDD的转换操作是并行化计算的，多个RDD的转换同样是可以并
* 减少shuffle网络传输：网络I/O开销是很大的，减少网络开销，可以显著加快计算效率。

### RDD运行过程（具体在Spark任务调度中详细说明）
1、创建RDD对象
2、DAGScheduler模块介入运行，计算RDD之间的依赖关系。RDD之间的依赖关系就形成了DAG
3、每一个JOB被分为多个Stage，划分Stage的一个主要依据是当前计算因子的输入是否确定的，如果是则将其分在同一个Stage，避免多个Stage之间的消息传递开销



参考：
http://www.cnblogs.com/bourneli/p/4394271.html
http://www.jianshu.com/p/4ff6afbbafe4
http://spark.apache.org/docs/latest/programming-guide.html#resilient-distributed-datasets-rdds

### 附
Transformation具体内容
`map(func)` :返回一个新的分布式数据集，由每个原元素经过func函数转换后组成
`filter(func) `: 返回一个新的数据集，由经过func函数后返回值为true的原元素组成
`flatMap(func) `: 类似于map，但是每一个输入元素，会被映射为0到多个输出元素（因此，func函数的返回值是一个Seq，而不是单一元素）
`sample(withReplacement, frac, seed) `:根据给定的随机种子seed，随机抽样出数量为frac的数据
`union(otherDataset)` : 返回一个新的数据集，由原数据集和参数联合而成
`groupByKey([numTasks])` :在一个由（K,V）对组成的数据集上调用，返回一个（K，Seq[V])对的数据集。注意：默认情况下，使用8个并行任务进行分组，你可以传入numTask可选参数，根据数据量设置不同数目的Task
`reduceByKey(func, [numTasks]) `: 在一个（K，V)对的数据集上使用，返回一个（K，V）对的数据集，key相同的值，都被使用指定的reduce函数聚合到一起。和groupbykey类似，任务的个数是可以通过第二个可选参数来配置的。
`join(otherDataset, [numTasks])` ：在类型为（K,V)和（K,W)类型的数据集上调用，返回一个（K,(V,W))对，每个key中的所有元素都在一起的数据集
`groupWith(otherDataset, [numTasks])` ：在类型为（K,V)和(K,W)类型的数据集上调用，返回一个数据集，组成元素为（K, Seq[V], Seq[W]) Tuples。这个操作在其它框架，称为CoGroup
`cartesian(otherDataset) `: 笛卡尔积。但在数据集T和U上调用时，返回一个(T，U）对的数据集，所有元素交互进行笛卡尔积。

Actions具体内容
`reduce(func) `: 通过函数func聚集数据集中的所有元素。Func函数接受2个参数，返回一个值。这个函数必须是关联性的，确保可以被正确的并发执行
`collect()` : 在Driver的程序中，以数组的形式，返回数据集的所有元素。这通常会在使用filter或者其它操作后，返回一个足够小的数据子集再使用，直接将整个RDD集Collect返回，很可能会让Driver程序OOM
`count()` : 返回数据集的元素个数
`take(n)` : 返回一个数组，由数据集的前n个元素组成。注意，这个操作目前并非在多个节点上，并行执行，而是Driver程序所在机器，单机计算所有的元素(Gateway的内存压力会增大，需要谨慎使用）
`first()` : 返回数据集的第一个元素（类似于take(1)）
`saveAsTextFile(path) `: 将数据集的元素，以textfile的形式，保存到本地文件系统，hdfs或者任何其它hadoop支持的文件系统。Spark将会调用每个元素的toString方法，并将它转换为文件中的一行文本
`saveAsSequenceFile(path)` : 将数据集的元素，以sequencefile的格式，保存到指定的目录下，本地系统，hdfs或者任何其它hadoop支持的文件系统。RDD的元素必须由key-value对组成，并都实现了Hadoop的Writable接口，或隐式可以转换为Writable（Spark包括了基本类型的转换，例如Int，Double，String等等）
`foreach(func)` : 在数据集的每一个元素上，运行函数func。这通常用于更新一个累加器变量，或者和外部存储系统做交互
