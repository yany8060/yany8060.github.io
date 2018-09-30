---
title: TensorFlow 基本概念与函数-1
year: 2018
date: 2018-09-03 20:18:38
tags: tensorflow
categories: tensorflow
---

### 编程模型
> TensorFlow的数据流图是由`节点（Node） `和`边（edge）`组成的`有向无环图（directed acycline graph，DAG）`。TensorFlow 由 Tensor 和 Flow 两部 分组成，Tensor(张量)代表了数据流图中的边，而 Flow(流动)这个动作就代表了数据流图中节点所做的操作。
> 

计算过程：首先从输入层开始，经过塑形层后，一层一层进行前向传播运算。Relu层（隐藏层）里两个参数，即Wh1和bh1，在输出前使用Rule（Rectified Linear Units）激活函数做非线性处理。然后进入Logit层（输出层），学习两个参数Wsm 和 bsm。用Softmax来计算输出结果中各个类别的概率分布。用交叉熵来度量两个概率分布（源样本的概率分布和输出结果 的概率分布）直接的相似性。然后开始计算梯度，这是需要参数Wh1、bh1、Wsm 和 bsm，以及 交叉熵后的结果。随后进入 SGD 训练，也就是反向传播的过程，从上往下计算每一层的参数， 依次进行更新。也就是说，计算和更新的顺序为 bsm、Wsm、bh1 和 Wh1。

![avatar](https://wx3.sinaimg.cn/large/007h1WTYly1fup5sc9m7ag30700cgwol.gif)

### 张量
TensorFlow用张量这种数据结构来表示所有的数据。一个张量有一个静态类型和动态类型的维数。张量可以在图中的节点之间流通。
张量的维数来被描述为`阶`
> 秩：Tensor维度的数量

#### 属性
* 数据类型（例如 float32、int32 或 string）
* 形状：即张量的维数和每个维度的大小

#### 方法
* tf.rank：返回张量的秩

```python
# shape of tensor 't' is [2, 2, 3]
t = tf.constant([[[1, 1, 1], [2, 2, 2]], [[3, 3, 3], [4, 4, 4]]])
tf.rank(t)  # 3
```

* tf.shape：返回某个张量的形状

```python
t = tf.constant([[[1, 1, 1], [2, 2, 2]], [[3, 3, 3], [4, 4, 4]]])
tf.shape(t)  # [2, 2, 3]
```
* tf.reshape：重构张量
* tf.cast：将 tf.Tensor 从一种数据类型转型为另一种

### 变量

#### 变量集合
默认情况下，每个 tf.Variable 都放置在以下两个集合中

* `tf.GraphKeys.GLOBAL_VARIABLES` - 可以在多台设备间共享的变量，
* `tf.GraphKeys.TRAINABLE_VARIABLES` - TensorFlow 将计算其梯度的变量

```
# 如果不希望变量可训练，可以将其添加到 tf.GraphKeys.LOCAL_VARIABLES 集合中
my_local = tf.get_variable("my_local", shape=(),collections=[tf.GraphKeys.LOCAL_VARIABLES])
# 或
my_non_trainable = tf.get_variable("my_non_trainable",
                                   shape=(),
                                   trainable=False)
```
可以定义自己的集合

```
# 将名为 my_local 的现有变量添加到名为 my_collection_name 的集合中
tf.add_to_collection("my_collection_name", my_local)
# 获取某个集合中的所有变量（或其他对象）的列表
tf.get_collection("my_collection_name")

```

#### 初始化变量
> 初始化器: tf.initializers（constant、zeros）

要在训练开始前一次性初始化所有可训练变量，请调用`tf.global_variables_initializer()`。默认情况下，`tf.global_variables_initializer`不会指定变量的初始化顺序

```
v = tf.get_variable("v", shape=(), initializer=tf.zeros_initializer())
w = tf.get_variable("w", initializer=v.initialized_value() + 1)
```

#### 基本操作
要为变量赋值，请使用 assign、assign_add 方法以及 tf.Variable 类中的友元。

共享变量两种方式：

* 显式传递 `tf.Variable` 对象
* 在 `tf.variable_scope` 对象内隐式封装 tf.Variable 对象。

变量作用域允许您在调用隐式创建和使用变量的函数时控制变量重用。作用域还允许您以分层和可理解的方式命名变量。

### 图和会话

#### tf.Graph 
* 图结构：图的节点和边缘，表示各个操作组合在一起的方式，但不规定它们的使用方式。
* 图集合：TensorFlow 提供了一种在`tf.Graph`中存储元数据集合的通用机制。`tf.add_to_collection` 函数允许您将对象列表与一个键关联（其中 tf.GraphKeys 定义了部分标准键），`tf.get_collection` 允许您查询与某个键关联的所有对象。

大多数 TensorFlow 程序都以数据流图构建阶段开始。在此阶段，您会调用 TensorFlow API 函数，这些函数可构建新的 `tf.Operation（节点）`和 `tf.Tensor（边缘` 对象并将它们添加到 tf.Graph 实例中。

类张量类型:

* tf.Tensor
* tf.Variable
* numpy.ndarray
* list（以及类似于张量的对象的列表）
* 标量 Python 类型：bool、float、int、str

> 注意：在 TensorFlow API 中调用大多数函数时，只是将操作和张量添加到默认图中，并不会执行实际的计算。您应编写这些函数，直到拥有表示整个计算（例如执行梯度下降法的一步）的 tf.Tensor 或 tf.Operation，然后将该对象传递给 tf.Session 以执行计算。


#### tf.Sessoin
`tf.Session` 拥有物理资源（例如 GPU 和网络连接），因此通常（在 with 代码块中）用作上下文管理器，并在您退出代码块时自动关闭会话

```python
# Create a default in-process session.
with tf.Session() as sess:
  # ...

# Create a remote session.
with tf.Session("grpc://example.org:2222"):
  # ...
```

* tf.Session.init 
	* target： 如果将此参数留空（默认设置），会话将仅使用本地机器中的设备。也可以指定 grpc:// 网址，以便指定 TensorFlow 服务器的地址，这使得会话可以访问该服务器控制的机器上的所有设备。
	* graph：默认情况下，新的 tf.Session 将绑定到当前的默认图，并且仅能够在当前的默认图中运行操作。如果您在程序中使用了多个图，则可以在构建会话时指定明确的 `tf.Graph`。
	* config：此参数允许您指定一个控制会话行为的`tf.ConfigProto`
* `tf.Session.run`：运行 `tf.Operation` 或评估 `tf.Tensor` 的主要机制
	* `tf.Session.run` 要求您指定一组 fetch，这些 fetch 可确定返回值，并且可能是 `tf.Operation`、`tf.Tensor` 或`类张量类型`
	* `tf.Session.run` 也可以选择接受 Feed 字典，该字典是从 `tf.Tensor` 对象（通常是 tf.placeholder 张量）到在执行时会被替换为这些张量的值（通常是 Python 标量、列表或 NumPy 数组）的映射
	* `tf.Session.run` 也接受可选的 options 参数（允许您指定与调用有关的选项）和可选的 run_metadata 参数（允许您收集与执行有关的元数据）