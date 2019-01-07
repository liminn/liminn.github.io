---
title: MobileNetV2
mathjax: true
date: 2018-12-29 22:22:50
categories: 
- 轻量型CNN
tags:
---

# 摘要

- 提出新的移动端模型——MobileNetV2。相比于MobileNetV1，其改进之处即在于基础模块——线性瓶颈层的转置残差模块(the inverted residual with linear bottleneck)。
- 并且，对MobileNetV2应用于目标检测和语义分割进行了应用及实验比较(见原文)。

 <!-- more -->

# 创新点

## 线性瓶颈层的转置残差模块(the inverted residual with linear bottleneck)

<img src="/images/MobileNetV2/1.png"  width = "400" height = "100"/>

- 如Table1所示，该模块将输入特征看作低维度的压缩特征：
 - 首先，扩展到高维度；
 - 然后，通过轻量的depthwise卷积进行滤波；
 - 然后，将特征映射回低维度并且采用线性卷积；
 - 最后，输入层和瓶颈层建立残差连接。

<img src="/images/MobileNetV2/2.png"  width = "400" height = "100"/>
- 如Fig3所示，为典型残差模块和转置残差模块的区别：
 - 典型的残差模块是将输入层和高通道数的层建立残差连接，而转置残差模块是将输入层和瓶颈层建立残差连接。 
 - 注：但都是输入和输出做残差连接，没什么特殊的地方

## MobileNetV1与MobileNetV2的基础模块比较
<img src="/images/MobileNetV2/3.png"  width = "600" height = "100"/>

**相同点**：
- **都采用depthwise(DW)卷积搭配pointwise(PW)卷积的方式来提特征**
 - 这两个操作合起来也被称为深度可分离卷积(depthwise separable convolution)，之前在Xception中被广泛使用。其优点是：理论上可以成倍的减少卷积层的时间复杂度和空间复杂度。
 
 - 下式为深度可分离卷积和常规卷积的复杂度比率(参数个数比率或乘加次数比率都如下式所示)：
 - <img src="/images/MobileNetV2/4.png"  width = "500" height = "100"/>
 - 可以看出：由于卷积核的尺寸$K$通常远小于输出通道数$C_{out}$，因此标准卷积的计算复杂度近似为深度可分离卷积的$K^2$倍。

**不同点**：
- **MobileNetV2在DW卷积之前新加了一个PW卷积**
 - 这么做的原因，**是因为DW卷积由于本身的计算特性决定它自己没有改变通道数的能力，上一层给它多少通道，它就只能输出多少通道。所以如果上一层给的通道数本身很少的话，DW卷积也只能很委屈的在低维空间提特征，因此效果不够好**。
 - 现在MobileNetV2为了改善这个问题，给每个DW卷积之前都配备了一个PW卷积，专门用来升维。定义升维系数$t = 6$，这样不管输入通道数$C_{in}$是多是少，经过第一个PW卷积升维之后，DW卷积都是在相对的更高维$( t \cdot C_{in})$进行工作。
 - 该段解释了出现**转置残差模块**的原因：即需先增加一个PW卷积来升通道数，使得DW卷积更加有效，然后再通过PW卷积降通道数(当然也有通道重组的功能)。因此，便出现了所谓转置残差模块的现象。
- **MobileNetV2去掉了第二个PW卷积的激活函数**(论文作者称其为**Linear Bottleneck**)
 - 这么做的原因，是因为作者认为激活函数在高维空间能够有效的增加非线性，而在低维空间时则会破坏特征，不如线性的效果好。
 - 由于第二个PW卷积的主要功能就是降维，因此按照上面的理论，降维之后就不宜再使用ReLU6了。
 - 该段解释了所谓**线性瓶颈层**的原因。
 - 注意：Xception中是DW卷积之后不加非线性激活函数，因为Xception文中认为DW卷积只有1通道数，非线性激活函数变得有害，导致信息丢失。
 - 疑问：接上，那么，为什么MobileNetV2中DW卷积后还进行非线性激活？

## ResNet与MobileNetV2的构造模块对比
 <img src="/images/MobileNetV2/5.png"  width = "600" height = "100"/>

- 相同点：
 - MobileNetV2借鉴ResNet，都采用了$1 \times 1 \to 3 \times 3 \to 1 \times 1$的模式。
 - MobileNetV2借鉴ResNet，同样使用Shortcut将输出与输入相加(未在上图画出)。

- 不同点：
 - ResNet使用**标准卷积**提取特征，MobileNet始终使用**DW卷积**提取特征。
 - ResNet**先降维** (0.25倍)、卷积、再升维，而 MobileNetV2则是**先升维**(6倍)、卷积、再降维。直观的形象上来看，ResNet的微结构是**沙漏形**，而MobileNetV2则是**纺锤形**，刚好相反。因此论文作者将MobileNetV2的结构称为Inverted Residual Block。这么做也是因为使用DW卷积而作的适配，希望特征提取能够在高维进行。

##  MobileNetV2总体结构
<img src="/images/MobileNetV2/6.png"  width = "400" height = "100"/>

- 如Table2所示，为MobileNetV2结构，其中：
 - $t$是转置残差模块的扩张比率(expansion rate)
 - $n$是该模块重复次数
 - $c$是输出通道数
 - $s$是该阶段第一次重复时的第一个模块的DW卷积的步长(后面重复都是步长为1)（疑问：为啥不直接使用第一个PW卷积实现下采样？）
- MobileNetV2主要由初始的常规卷积层和17个线性瓶颈层的转置残差模块构成。
- 采用ReLU6作为非线性函数，因为在对于低精度计算时其更鲁棒。
 - 首先说明一下ReLU6，卷积之后通常会接一个ReLU非线性激活，在MobileV1里面使用ReLU6，ReLU6就是普通的ReLU但是限制最大输出值为6（对输出值做clip），这是为了在移动端设备float16的低精度的时候，也能有很好的数值分辨率，如果对ReLU的激活范围不加限制，输出范围为0到正无穷，如果激活值非常大，分布在一个很大的范围内，则低精度的float16无法很好地精确描述如此大范围的数值，带来精度损失。
- 只使用$3 \times 3$的卷积核。
- 除了第一层，在网络中采用恒等的扩张比率$t$：
 - 实验中发现，扩张比率$t$在5～10之间，可以得到几乎相等的表现曲线。
 - 较小的网络更适合采用较小的扩张比率;较大的网络更适合采用较大的扩张比率。
 - 文中实验均采用扩张比率为$t=6$。例如，对于一个线性瓶颈层的转置残差模块，输入为64通道的张量，输出为128通道的张量，其中间的扩张层的张量的通道数即为$64\times 6 = 384$。
 
<img src="/images/MobileNetV2/7.png"  width = "400" height = "100"/>

- 如Fig6(a)所示，实验结果表明：验证了线性瓶颈层的重要性，提供了对于非线性激活函数会破坏低维空间的信息的支持。
- 如Fig6(b)所示，实验结果表明：对于残差连接，与瓶颈层进行残差连接的表现，优于与扩张层进行残差连接以及不使用残差连接。

<img src="/images/MobileNetV2/8.png"  width = "400" height = "100"/>

- 如Fig4所示，为几种不同结构之间的构建模块比较。

# 训练策略
- 采用RMSProp优化器，动量系数设为0.9
- 在每一层后面施加BN
- 标准的权重衰减系数设为0.00004
- 同MobileNetV1一样，采用初始学习率为0.045，学习速率衰减速率为每一opoch变为0.98
- 采用16个GPU异步工作器，batch size为96

# 模型比较
<img src="/images/MobileNetV2/9.png"  width = "400" height = "100"/>

- 如Table4所示，比较了MobileNetV1、ShuffleNet和NASNet-A模型。

此外，文中有将MobileNetV2用于目标检测任务和语义分割任务的详细对比实验。

# 参考文献

- [论文：MobileNetV2: Inverted Residuals and Linear Bottlenecks](https://arxiv.org/pdf/1801.04381.pdf)
- [博客(推荐)：MobileNet V2 论文初读](https://zhuanlan.zhihu.com/p/33075914)
- [博客：轻量化网络：MobileNet-V2](https://blog.csdn.net/u011995719/article/details/79135818)
- [代码](https://github.com/tensorflow/models/tree/master/research/slim/nets/mobilenet)



