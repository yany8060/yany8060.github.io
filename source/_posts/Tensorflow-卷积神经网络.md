---
title: Tensorflow-卷积神经网络
date: 2019-12-06 16:55:42
tags: tensorflow
categories : tensorflow
---

### 神经网络
具体过程就是：神经信号x乘上权重向量w，经过输入函数（Net input function）求和后，由激活函数（Activation function）输出。监督学习过程中，输出结果将会对比数据集样本结果（label），使用损失函数（cost function）计算损失，并且经过优化器迭代后更新权重。

![avatar](http://wx3.sinaimg.cn/mw690/007h1WTYly1fyvx74sscdj30lz0asn4a.jpg)


#### 输入层
该层只起到输入信号的扇出作用.所以在计算网络的层数时不被记入。该层负责接收来自网络外部的信息。
#### 输出层
它是网络的最后一层，具有该网络的最大层号，负责输出网络的计算结果。
#### 隐藏层
除输入层和输出层以外的其他各层叫做隐藏层。隐藏层不直接接受外界的信号,也不直接向外界发送信号。

#### 计算过程

![avatar](http://wx3.sinaimg.cn/mw690/007h1WTYly1fyvxj3nd8nj30ib0aggp7.jpg
)



### 卷积神经网络
卷积神经网络包含了一个由卷积层和子采样层构成的特征抽取器。

卷积神经网络分为了三部分，第一部分为输入层，第二部分由若干个卷积层和池化层 组成，第三部分为一个全连接的多层感知分类器构成。
![avatar](http://wx2.sinaimg.cn/mw690/007h1WTYly1fywy9j09g3j30va08mgnj.jpg)



#### 卷积层
卷积层通常包含若干个特征平面（featureMap），每一个特征平面由一些矩形排列的神经元组成，同一特征平面的神经元共享权值，这个共享权值就是卷积核。卷积核一般以随机小数矩阵的形式初始化，在网络的训练过程中卷积核将学习得到合理的权值。
![avatar](http://wx1.sinaimg.cn/mw690/007h1WTYly1fywy9mg3h9g30dj04owh8.gif)


#### 池化层
子采样也叫做池化（pooling），目的是为了减少特征图。通常有**均值采样（mean poooling）和 最大值采样（max pooling）**两种形式。

池化操作将保存深度大小不变。

如果池化层的输入单元大小不是二的整数倍，一般采取边缘补零（zero-padding）的方式补成2的倍数，然后再池化。

池化操作对每个深度切片独立，规模一般为 2＊2，相对于卷积层进行卷积运算
![avatar](http://wx4.sinaimg.cn/mw690/007h1WTYly1fywy419ylnj30lv0a8n0i.jpg)





#### 全连接层
全连接层的每一个节点都与上一层的所有节点相连，用来把前边提取到的特征综合起来。

### CNN卷积神经网络
#### 卷积核
带着一组固定权重的神经元，通常是n\*m二维的矩阵，n和m也是神经元的感受视野（local receptive fields）。n\*m矩阵中存的是对感受视野中数据处理的系数。一个卷积核的滤波可以用来提取特定的特征（例如提取物体轮廓，颜色深浅等）。通过卷积层从原始数据中提取出的新的特征过程又成为feature map（特征映射）。将感受视野对输入的扫描间隔称为步长（stride）；

#### tf.nn.conv2d
tf.nn.conv2d\(input, filter, strides, padding, use\_cudnn\_on\_gpu=None, name=None\) 

* **参数input**：指需要做卷积的输入图像，它要求是一个Tensor，具有[batch, in_height, in_width, in_channels]这样的shape，具体含义是[训练时一个batch的图片数量, 图片高度, 图片宽度, 图像通道数]，注意这是一个4维的Tensor，要求类型为float32和float64其中之一

* **参数filter**：相当于CNN中的卷积核，它要求是一个Tensor，具有[filter_height, filter_width, in_channels, out_channels]这样的shape，具体含义是[卷积核的高度，卷积核的宽度，图像通道数，卷积核个数]，要求类型与参数input相同，有一个地方需要注意，第三维in_channels，就是参数input的第四维
*  **参数strides**：卷积时在图像每一维的步长，这是一个一维的向量，长度4
*  **参数padding**：string类型的量，只能是"SAME","VALID"其中之一，这个值决定了不同的卷积方式。
	*  当其为"VALID"时，输出的size总比原图的size小，有时不会覆盖原图所有元素(既，可能舍弃边上的某些元素)。
	*  当其为"SAME"时，表示卷积核可以停留在图像边缘，输出并不一定和原图size一致，但会保证覆盖原图所有像素，不会舍弃边上的莫些元素;
* **参数use\_cudnn\_on\_gpu**：bool类型，是否使用cudnn加速，默认为true
* **结果返回一个Tensor**，这个输出，就是我们常说的feature map，shape仍然是[batch, height, width, channels]这种形式。

#### 网络案例：
LeNet、AlexNet、ZF Net、GoogLeNet、VGGNet、ResNet


