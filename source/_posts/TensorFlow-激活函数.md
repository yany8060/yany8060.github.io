---
title: TensorFlow-激活函数
date: 2018-09-22 17:16:17
tags: tensorflow
categories : tensorflow
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

### 激活函数

激活函数（Activation Function）运行时激活神经网络中**某一部分神经元**，将**激活信息**向后传入下一层的**神经网络**。
激活函数不会改变数据的维度，也就是**输入和输出的维度是相同的**。

#### why (为什么要用激活函数)
因为线性模型的表达能力不够，引入激活函数是为了添加非线性因素，解决线性模型所不能解决的问题
> 参见：https://www.zhihu.com/question/22334626/answer/21036590

#### what (激活函数是什么)
激活函数就是一个普通函数。通过函数把特征保留并映射出来，这是神经网络能解决非线性问题关键。
特性：

* **非线性**：当激活函数是线性的时候，一个两层的神经网络就可以逼近基本上所有的函数了。如果使用的是恒等激活函数，那么其实整个网络跟单层神经网络是等价的。
* **可微性**：当优化方法是基于梯度的时候，这个性质是必须的。
* **单调性**：当激活函数是单调的时候，单层网络能够保证是凸函数。
* **f(x)≈x**：当激活函数满足这个性质的时候，如果参数的初始化是random的很小的值，那么神经网络的训练将会很高效；如果不满足这个性质，那么就需要很用心的去设置初始值。
* **输出值范围**：当激活函数输出值有限时，基于梯度的优化方法会更加稳定，因为特征的表示受有限权值的影响更显著；当激活函数的输出是无限时，模型的训练会更加高效，不过在这种情况小，一般需要更小的learning rate。

#### 分类：

* Traditional：`sigmoid(logistic)`、`tanh`
* RELU Family：`RELU`、`Leaky RELU`、`PRELU`、`RRELU`
* Exponential Family：`ELU`、`SELU`

饱和性:

* 软饱和：函数的导数趋近于0
* 硬饱和：函数的导数等于0

##### tf.nn.sigmoid

$$f(x) = \frac{1}{1+e^{-x}}$$

![avatar](http://wx1.sinaimg.cn/mw690/007h1WTYly1fvrdsjbbehj30hs0d5t8x.jpg)


优点：

* sigmoid函数的输出映射在(0,1)之间，单调连续，输出范围有限，优化稳定，可以用作输出层
* 求导容易

缺点：

* sigmoid神经元有一个不好的特性，就是**当神经元的激活在接近0或1处时会饱和：在这些区域，梯度几乎为0**。因此在反向传播时，这个局部梯度会与整个损失函数关于该单元输出的梯度相乘，结果也会接近为 0 。**因此这时梯度就对模型的更新没有任何贡献**。
* sigmoid函数的输出不是零中心的，即sigmoid函数关于原点中心不对称

> 非饱和神经元：反向传播经过这个神经元时，它的梯度不接近0，还能继续往前传

```python
import tensorflow as tf

a = tf.constant([[-1.0, -2.0], [1.0, 2.0], [0.0, 0.0]])
sess = tf.Session()
print(sess.run(tf.sigmoid(a)))

# [[ 0.26894143  0.11920292]
#  [ 0.7310586   0.88079703]
#  [ 0.5         0.5       ]]
```


##### tf.nn.tanh

$$f(x) = \tanh(x) = \frac{\sinh(x)}{\cosh(x)} = \frac{e^{x} - e^{-x}}{e^{x} + e^{-x}} = \frac{1 - e^{-2x}}{1 + e^{-2x}} $$

![avatar](http://wx4.sinaimg.cn/mw690/007h1WTYly1fvrdsu5jjlj30hs0d5mxe.jpg)

tf.nn.relu函数是将大于0的数保持不变，小于0的数置为0

tanh函数也具有软饱和性。**因为它的输出是以0为中心，收敛速度比sigmoid函数要快**。但是仍然无法解决梯度消失问题。

```
import tensorflow as tf

a = tf.constant([[-1.0, -2.0], [1.0, 2.0], [0.0, 0.0]])
sess = tf.Session()
print(sess.run(tf.tanh(a)))
# [[-0.76159418 -0.96402758]
#  [ 0.76159418  0.96402758]
#  [ 0.          0.        ]]
```

##### tf.nn.relu
$$ f(x) = max(0, x) $$
![avatar](http://wx4.sinaimg.cn/mw690/007h1WTYly1fvrdrpcyyxj30hs0d50sx.jpg)

这个函数的作用是计算激活函数relu，即max(features, 0)。 
所有负数都会归一化为0，所以的正值保留为原值不变


由上图的函数图像可以知道，relu在x<0时是硬饱和。由于当x>0时一阶导数为1。所以，relu函数在x>0时可以保持梯度不衰减，从而缓解梯度消失问题，还可以更快的去收敛。但是，随着训练的进行，部分输入会落到硬饱和区，导致对应的权重无法更新。我们称之为“神经元死亡”。

* 优点在于不收”梯度消失”的影响,且取值范围在[0,+oo)
* 缺点在于使用了较大的学习速率时,易受达到饱和的神经元的影响


```pyton
import tensorflow as tf
 
a = tf.constant([-2,-1,0,2,3])
with tf.Session() as sess:
 	print(sess.run(tf.nn.relu(a)))

# 结果 [0 0 0 2 3]
```

优化：leakrelu函数、ELU函数、SELU函数

##### tf.nn.softmax

$$softmax(x)_i = \frac {e^{x_i}}{\sum _j e^{x_j}}$$

softmax = tf.exp(logits) / tf.reduce_sum(tf.exp(logits), axis)

softmax它将多个神经元的输出，映射到（0,1）区间内，可以看成概率来理解，从而来进行多分类！



```
import tensorflow as tf

a = tf.constant([1.0,2.0,3.0,4.0,5.0,6.0])
with tf.Session() as sess: 
print sess.run(tf.nn.softmax(a))
# [0.00426978  0.01160646  0.03154963  0.08576079  0.23312201  0.63369131]            
```

