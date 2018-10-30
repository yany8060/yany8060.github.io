---
title: TensorFlow-损失函数
date: 2018-10-16 20:11:11
tags: tensorflow
categories : tensorflow
---

<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [ ['$','$']],
        displayMath: [ ['$$','$$']]
    }
});
</script>

<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>



### 损失函数定义
损失函数（loss function）,量化了分类器输出地结果（预测值）和我们期望的结果（标签）之间的差距。  
在机器学习中，**损失函数（loss function）**也称cost function（代价函数），是用来**计算预测值和真实值的差距**。然后以loss function的最小值作为目标函数进行反向传播迭代计算模型中的参数，这个让loss function的值不断变小的过程称为优化。

损失函数分为**经验风险损失函数**和**结构风险损失函数**

* 经验风险损失函数指预测结果和实际结果的差别
* 结构风险损失函数是指经验风险损失函数加上正则项

设总有N个样本的样本集为$(X,Y)=(x_i,y_i)$，那么总的损失函数为$$L = \sum_{i=1}^n l(y_i,f(x_i)) $$
其中 $y_i,i∈[1,N]$为样本$i$的真实值，$f(x_i),i∈[1,N]$为样本$i$的预测值， $f()$为分类或者回归函数。

一般来说，对于分类或者回归模型进行评估时，需要使得模型在训练数据上似的损失函数值最小，即使得经验风险函数(Empirical risk)最小化，但是如果只考虑经验风险，容易出现过拟合，因此还需要考虑模型的泛化性，一般常用的方法就是在目标函数中加上正则项，有损失项（loss term）加上正则项（regularization term）构成结构风险（Structural risk），那么损失函数变为：$$L = \sum_{i=1}^n l(y_i,f(x_i))+\lambda R(w) $$

R(w)就是评价模型复杂度的指标，一般只是权重w的函数，$\lambda$表示复杂损失在总损失中的比例。



#### 常见的损失函数
1. 0-1损失函数（0-1 loss function）$$L(y,f(x)) = \begin{cases}
1, y = f(x) \\
0, y \neq  f(x) \\
\end{cases}$$ 
2. 平方损失函数（quadratic loss function） $$L(Y,f(x)) = (Y-f(x))^2$$
3. L1正则损失函数（绝对损失函数 absolute loss function）$$L(Y,f(x)) = \left| Y-f(x) \right|$$
4. L2正则损失函数（即欧拉损失函数）$$L(Y,f(x)) = \sum_{i=1}^n(Y-f(x))^2$$
当对L2取平均值，就变成均方误差（MSE, mean squared error） $$ MSE(y,{y}') = \frac{1}{n}\sum_{i=1}^n (y^i - {y}')^2 $$
4. 对数损失函数(logarithmic loss function)或对数似然损失函数(log-likelihood loss function)$$L(Y,P(Y|X)) = -logP(Y|X)$$

#### 交叉熵损失函数
交叉熵（Cross Entropy）是Loss函数的一种（也称为损失函数或代价函数），用于描述模型预测值与真实值的差距大小（用来评估当前训练得到的概率分布与真实分布的差异情况），常见的Loss函数就是均方平方差（Mean Squared Error）

$$  loss =\frac{1}{n}\sum_{i=0}^n{(y_{i} \cdot log(y\_predicted_{i})
+(1-y_{i}) \cdot log(1-y\_predicted_{i})
)}  $$

熵：用来表示所有信息量的期望 
$$H(X)=-\sum_{i=1}^n p(x_i)log(p(x_i))$$
信息量定义：$h(x) = -log(p(x))$，概率分布函数p(x)

### 损失函数Api
#### tf.nn.softmax\_cross\_entropy\_with\_logits

对于每个**独立的分类任务**，这个函数是去度量概率误差。比如，在 CIFAR-10 数据集上面，每张图片只有唯一一个分类标签：一张图可能是一只狗或者一辆卡车， 但绝对不可能两者都在一张图中。

* 输入API的数据 logits 不能进行缩放，因为在这个API的执行中会进行 softmax 计算，如果 logits 进行了缩放，那么会影响计算正确率。
* logits 和 labels 必须有相同的数据维度 [batch_size, num_classes]，和相同的数据类型 float32 或者 float64 。
* 它适用于每个类别相互独立且排斥的情况，一幅图只能属于一类，而不能同时包含一条狗和一只大象.

```
softmax_cross_entropy_with_logits(
    _sentinel=None,
    labels=None,
    logits=None,
    dim=-1,
    name=None
)
## softmax_cross_entropy_with_logits_v2 (现在)
```
流程：

1. 首先是对网络最后一层的输出做一个softmax（公式详见激活函数）
2. 对softmax输出的向量$[Y1,Y2,Y3,...]$和样本的时机标签做一个交叉熵

```
import tensorflow as tf  

# our NN's output  
logits=tf.constant([[1.0,2.0,3.0],[1.0,2.0,3.0],[1.0,2.0,3.0]])  

# step1:do softmax  
y=tf.nn.softmax(logits)  

# true label  
y_=tf.constant([[0.0,0.0,1.0],[0.0,0.0,1.0],[0.0,0.0,1.0]])  

# step2:do cross_entropy  
cross_entropy = tf.reduce_sum(-tf.reduce_sum(y_*tf.log(y), axis=1))

# do cross_entropy just one step  
cross_entropy2 = tf.reduce_sum(tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=y_))

with tf.Session() as sess:  
	print(sess.run(cross_entropy))
	print(sess.run(cross_entropy2))

# 1.222818
# 1.222818
```




#### tf.nn.sparse\_softmax\_cross\_entropy\_with\_logits
计算logits 和 labels 之间的稀疏softmax 交叉熵

度量在离散分类任务中的错误率，这些类之间是相互排斥的（每个输入只能对应唯一确定的一个类）。举例来说，每个CIFAR-10 图片只能被标记为唯一的一个标签：一张图片可能是一只狗或一辆卡车，而不能两者都是。

区别：**sparse\_softmax\_cross\_entropy\_with\_logits** 直接用标签计算交叉熵，而 **softmax\_cross\_entropy\_with\_logits** 是标签的onehot向量参与计算。**softmax\_cross\_entropy\_with\_logits** 的 labels 是 **sparse\_softmax\_cross\_entropy\_with\_logits** 的 labels 的一个独热版本。
**tf.nn.sparse\_softmax\_cross\_entropy\_with\_logits** 比 **tf.nn.softmax\_cross\_entropy\_with\_logits** 多了一步将labels稀疏化的操作

> onehot标签则是顾名思义，一个长度为n的数组，只有一个元素是1.0，其他元素是0.0

> 稀疏化：例如表示一个3分类的一个样本的标签，稀疏表示的形式为[0,0,1]（这个表示这个样本为第3个分类），而非稀疏表示就表示为2（因为从0开始算，0,1,2,就能表示三类）

```
import tensorflow as tf

input_data = tf.Variable([[0.2, 0.1, 0.9], [0.3, 0.4, 0.6]], dtype=tf.float32)

output = tf.nn.sparse_softmax_cross_entropy_with_logits(logits=input_data, 

labels=[0, 2])

with tf.Session() as sess:
    init = tf.global_variables_initializer()
    sess.run(init)
    print(sess.run(output))

# [ 1.36573195  0.93983102]
```


#### tf.nn.sigmoid\_cross\_entropy\_with\_logits
函数意义：

* 衡量的是分类任务中的概率误差，他也是试用于每一个类别都是相不排斥的
* 这个函数的作用是计算经**sigmoid**函数激活之后的交叉熵。 
* 与sigmoid搭配使用的交叉熵损失函数，输入不需要额外加一层sigmoid，**tf.nn.sigmoid\_cross\_entropy\_with\_logits**中会集成有sigmoid并进行了计算优化；它适用于分类的类别之间**不是相互排斥**的场景，即多个标签（如图片中包含狗和猫）。

公式：$$targets \* -log(sigmoid(logits)) + (1 - targets) \* -log(1 - sigmoid(logits))$$

现在我们使用 x = logits, z = targets

$$
\begin {aligned}
sigmod交叉熵 &=z \* -log(sigmoid(x)) + (1 - z) \* -log(1 - sigmoid(x)) \\\
&= z \* -log(1 / (1 + exp(-x))) + (1 - z) \* -log(exp(-x) / (1 + exp(-x))) \\\
&= z \* log(1 + exp(-x)) + (1 - z) \* (-log(exp(-x)) + log(1 + exp(-x)))\\\
&= z \* log(1 + exp(-x)) + (1 - z) \* (x + log(1 + exp(-x))\\\
&= (1 - z) \* x + log(1 + exp(-x))\\\
&= x - x \* z + log(1 + exp(-x))
\end {aligned}
$$

当$x<0$时，$ e^{-x} \rightarrow \infty$ 溢出，所以使用计算式：
$$ -x*z + log(1+e^{x}) $$

$$ 
\begin {aligned}
推到过程： &= x - x \* z + log(1 + exp(-x))\\\
    &= log(exp(x)) - x \* z + log(1 + exp(-x)) \\\
    &= - x \* z + log(1 + exp(x)) 
\end {aligned}
$$


为了确保计算稳定，避免溢出，综合x>0和x<0的情况,我们使用以下函数式 ： 
$$ max(x,0) - x\*z + log(1+e^{−|x|})  $$


#### tf.nn.weighted\_cross\_entropy\_with\_logits
功能以及计算方式基本与**tf\_nn\_sigmoid\_cross\_entropy\_with\_logits**差不多,但是加上了权重的功能,是计算具有权重的sigmoid交叉熵函数
$$pos_weight \* targets \* -log(sigmoid(logits)) + (1 - targets) \* -log(1 - sigmoid(logits))$$


现在我们使用 x = logits, z = targets, q = pos_weight的代数式

$$
\begin {aligned}
  weighted交叉熵 &= qz \* -log(sigmoid(x)) + (1 - z) \* -log(1 - sigmoid(x))\\\
  &= qz \* -log(1 / (1 + exp(-x))) + (1 - z) \* -log(exp(-x) / (1 + exp(-x)))\\\
  &= qz \* log(1 + exp(-x)) + (1 - z) \* (-log(exp(-x)) + log(1 + exp(-x)))\\\
  &= qz \* log(1 + exp(-x)) + (1 - z) \* (x + log(1 + exp(-x))\\\
  &= (1 - z) \* x + (qz +  1 - z) \* log(1 + exp(-x))\\\
  &= (1 - z) \* x + (1 + (q - 1) \* z) \* log(1 + exp(-x))\\\
\end {aligned}
$$

我们把$l = (1 + (q - 1) \* z)$, 来确保稳定性并且避免溢出,公式为：
$$(1 - z) \* x + l \* (log(1 + exp(-abs(x))) + max(-x, 0))$$


### 了解

**正则化**：防止过拟合，提高泛化能力；正则化的原理就是在损失函数中加入评价模型复杂度的指标。R(w)就是评价模型复杂度的指标，一般只是权重w的函数，$\lambda$表示复杂损失在总损失中的比例。

**下溢出（underflow）和上溢出（overflow）**:实数在计算机内用二进制表示，所以不是一个精确值，当数值过小的时候，被四舍五入为0，这就是下溢出。此时如果对这个数再做某些运算（例如除以它）就会出问题。反之当数值过大的时候，情况就变成上溢出。