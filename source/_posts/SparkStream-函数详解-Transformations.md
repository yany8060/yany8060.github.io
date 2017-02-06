---
title: SparkStream 函数详解-Transformations
date: 2017-02-05 20:36:59
tags: [sparkstream,spark]
categories: [spark]
---
一个DStream对象可以调用多种操作，主要分为如下几类：
* Transformations
* Window Operations
* Join Operations
* Output Operations

### Transformations
#### map(func)
> Return a new DStream by passing each element of the source DStream through a function func.

主要作用是，对DStream对象a，将func函数作用到a中的每一个元素上并生成新的元素，得到的DStream对象b中包含这些新的元素。
```
//拼接一个”_NEW”字符串
val linesNew = lines.map(lines => lines + "_NEW" )
```
map中操作复杂可把函数抽出来在外部定义：
```
    val mapResult = stream.map(new Transformations().customeMap(_))

    def customeMap(word: String): String = {
        word + "ss"
    }
```

#### filter(func)
> Return a new DStream by selecting only the records of the source DStream on which func returns true.

对DStream a中的每一个元素，应用func方法进行计算，如果func函数返回结果为true，则保留该元素，否则丢弃该元素，返回一个新的DStream b。
```
val filterWords = words.filter(_ != "hello" )
```
在定义函数式时，返回值必须是boolean类型，

#### flatMap(func)
> Similar to map, but each input item can be mapped to 0 or more output items.

主要作用是，对DStream对象a，将func函数作用到a中的每一个元素上并生成0个或多个新的元素，得到的DStream对象b中包含这些新的元素。
```
//将lines根据空格进行分割，分割成若干个单词
val words = lines.flatMap(_.split( " " ))
```
同map若函数复杂可提出去

#### union(otherStream)
> Return a new DStream that contains the union of the elements in the source DStream and otherDStream.

这个操作将两个DStream进行合并，生成一个包含着两个DStream中所有元素的新DStream对象。
```
val wordsOne = words.map(_ + "_one" )
val wordsTwo = words.map(_ + "_two" )
val unionWords = wordsOne.union(wordsTwo)
```
输入 hello yany tian，结果是将wordsOne和wordsTwo合成一个unionWords的DStream

```
hello_one
yany_one
tian_one
hello_two
yany_two
tian_two
```

#### repartition(numPartitions)
> Changes the level of parallelism in this DStream by creating more or fewer partitions.

Return a new DStream with an increased or decreased level of parallelism. Each RDD in the returned DStream has exactly numPartitions partitions.

#### count()
> Return a new DStream of single-element RDDs by counting the number of elements in each RDD of the source DStream.

统计DStream中每个RDD包含的元素的个数，得到一个新的DStream，这个DStream中只包含一个元素，这个元素是对应语句单词统计数值。
```
val wordsCount = words.count()
```

#### reduce(func)
> Return a new DStream of single-element RDDs by aggregating the elements in each RDD of the source DStream using a function func (which takes two arguments and returns one). The function should be associative and commutative so that it can be computed in parallel.

返回一个包含一个元素的DStream，传入的func方法会作用在调用者的每一个元素上，将其中的元素顺次的两两进行计算。
```
val reduceWords = words.reduce(_ + "-" + _)
// 或

def customerReduce(line1: String, line2: String): String = {
    line1 + "|||" + line2
  }
 // val reduceResult = flatMapResult.reduce(new Transformations().customerReduce(_, _));
    val reduceResult = flatMapResult.reduce((x, y) => new Transformations().customerReduce(x, y));
```

#### countByValue()
> When called on a DStream of elements of type K, return a new DStream of (K, Long) pairs where the value of each key is its frequency in each RDD of the source DStream.

某个DStream中的元素类型为K，调用这个方法后，返回的DStream的元素为(K, Long)对，后面这个Long值是原DStream中每个RDD元素key出现的频率。
```
val countByValueWords = words.countByValue()
```

#### reduceByKey(func, [numTasks])
> When called on a DStream of (K, V) pairs, return a new DStream of (K, V) pairs where the values for each key are aggregated using the given reduce function. Note: By default, this uses Spark's default number of parallel tasks (2 for local mode, and in cluster mode the number is determined by the config property spark.default.parallelism) to do the grouping. You can pass an optional numTasks argument to set a different number of tasks.

调用这个操作的DStream是以(K, V)的形式出现，返回一个新的元素格式为(K, V)的DStream。返回结果中，K为原来的K，V是由K经过传入func计算得到的。还可以传入一个并行计算的参数，在local模式下，默认为2。在其他模式下，默认值由参数 spark.default.parallelism 确定。

注：reduceByKey使用时DStream需要以(K, V)的形式出现
```
val pairs = words.map(word => (word , 1))
val wordCounts = pairs.reduceByKey(_ + _)
//===========
/**
    * 
    * reducebykey 通过对于两两的value进行操作,可自定义
    * @param line1
    * @param line2
    * @return
    */
  def customerReduceByKey(line1: String, line2: String): String = {
    "ss"
  }
    val reduceByKeyResult = flatMapResult.map((_, "ss")).reduceByKey(new Transformations().customerReduceByKey(_, _))
```

#### join(otherStream, [numTasks])
> When called on two DStreams of (K, V) and (K, W) pairs, return a new DStream of (K, (V, W)) pairs with all pairs of elements for each key.

由一个DStream对象调用该方法，元素内容为 (k, V) ，传入另一个DStream对象，元素内容为(k, W)，返回的DStream中包含的内容是 (k, (V, W)) 。这个方法也可以传入一个并行计算的参数，该参数与reduceByKey中是相同的。
```
val wordsOne = words.map(word => (word , word + "_one" ))
val wordsTwo = words.map(word => (word , word + "_two" ))
val joinWords = wordsOne.join(wordsTwo)

```
结果
```
// 输入 hello world hello join

(hello,(hello_one,hello_two))
(hello,(hello_one,hello_two))
(hello,(hello_one,hello_two))
(hello,(hello_one,hello_two))
(join,(join_one,join_two))
(world,(world_one,world_two))
```
如果key相同，出现笛卡尔积现象

#### cogroup(otherStream, [numTasks])
> When called on a DStream of (K, V) and (K, W) pairs, return a new DStream of (K, Seq[V], Seq[W]) tuples.

由一个DStream对象调用该方法，元素内容为(k, V)，传入另一个DStream对象，元素内容为(k, W)，返回的DStream中包含的内容是 (k, (Seq[V], Seq[W])) 。这个方法也可以传入一个并行计算的参数，该参数与reduceByKey中是相同的。

```
val wordsOne = words.map(word => (word , word + "_one" ))
val wordsTwo = words.map(word => (word , word + "_two" ))
val joinWords = wordsOne.cogroup(wordsTwo)
```

结果：
```
// 输入 hello world hello cogroup

(hello,(CompactBuffer(hello_one, hello_one),CompactBuffer(hello_two, hello_two)))
(world,(CompactBuffer(world_one),CompactBuffer(world_two)))
(cogroup,(CompactBuffer(cogroup_one),CompactBuffer(cogroup_two)))
```

#### transform(func)
#### updateStateByKey(func)


参考：
http://blog.csdn.net/dabokele/article/details/52602412?utm_source=tuicool&utm_medium=referral

博客：http://yany8060.xyz/
githup: https://github.com/yany8060/SparkDemo