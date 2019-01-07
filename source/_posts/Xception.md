---
title: Xception
mathjax: true
date: 2018-12-29 22:20:51
categories: 
- 轻量型CNN
tags:
---

# 摘要

- 首先，提出对Inception模块的一种解释：Inception模块是介于常规卷积(regular convolution)和深度可分离卷积(depthwise separable convolution)的中间步骤。
- 然后，引入了”深度可分离卷积”的概念，将普通的卷积运算拆分成逐通道卷积(depthwise convolution)和逐点卷积(pointwise convolution)两部进行，有效地减少了计算量和参数量。
- 然后，提出用深度可分离卷积来替代Inception模块的想法。
- 最后，基于上述想法，提出Xception，意为"Extreme Inception"。
- 在ImageNet数据集上，Xception的表现比Inception V3稍好；在更大的JFT数据集上，Xception的表现比Inception V3的优势扩大。
- 注：Xception和Inception V3有这几乎相同的参数，因此，模型表现的提升不是因为参数增加，而是对于模型参数的更有效利用。
- 本质：Xception是采用depthwise separable convolution来改进InceptionV3。

 <!-- more -->

# 创新点

## 对于Inception模块的假设
<img src="/images/Xception/1.png"  width = "400" height = "100"/>

- 独立地看待跨通道相关性(cross-channel correlations)和空间相关性(spatial correlations)，如Fig1所示:
 - Inception模块的$1\times 1$卷积即是关于跨通道相关性
 - Inception模块的$3\times 3$或$5\times 5$卷积即是关于跨通道相关性和空间相关性 
 - 提出假设：**在Inception模块中，跨通道相关性和空间相关性被充分地解耦(decoupled)了，跨通道相关性和空间相关性更适合不被共同映射**。
 - 因此，进一步提出一个比Inception模块假设**更强的假设**：**让跨通道相关性和空间相关性的映射完全分离**。

## Inception模块的终极版本
<img src="/images/Xception/2.png"  width = "400" height = "100"/>

- 基于**上述的更强假设**，设计出Inception模块的终极版本，如Fig4所示：
 - 首先，用$1\times 1$卷积对跨通道相关性进行映射；
 - 然后，分别单独对每一个通道的空间相关性进行映射。
- 注意到，这个Inception的终极形式，和深度可分离卷积的含义几乎相等，只有两点微小区别：
 - 1.操作的顺序：深度可分离卷积通常先进行逐通道的空间卷积，然后进行$1\times 1$卷积。而Inception模块是先进行$1\times 1$卷积。
 - 注：文中认为这第一个不同点不重要，尤其是这些操作会以堆叠的方式被使用。
 - 2.存不存在非线形激活函数：在Inception模块中，两个操作之后都会接一个ReLU非线性激活函数；然而，执行深度可分离卷积时，通常不用非线性激活函数。
 - 注：第二个不同点可能有影响，后续进行了实验验证（见下一小节，**文中实验验证：depthwise卷积之后不加非线性激活函数，可以更快地拟合且得到更好的最终表现**）。
- 因此，产生新的想法：**可以用深度可分离卷积来替换Inception模块，以此来改善Inception家族的表现**。

## Xception结构
<img src="/images/Xception/3.png"  width = "700" >

Xception如Fig5所示：

- Xception即代表"Extreme Inception"。
- Xception结构含有36个卷积层构成，由14个模块构成，除了第一个和最后一个模块，所有模块均包含线性残差连接。
- **对于深度可分离卷积**（先进行depthwise卷积，再进行pointwise卷积，中间没有非线性激活函数）：
 - **文中实验验证：depthwise卷积之后不加非线性激活函数，可以更快地拟合且得到更好的最终表现。**
 - 对此，文中猜测：depthwise卷积的所施加在的中间特征空间的深度对于非线性激活函数的有效性至关重要：对于很深的特征空间(如Inception模块中的)，非线性激活函数很有帮助；但对于较浅的特征空间(如深度可分离卷积中的1通道深的特征空间)，非线性激活函数变得有害，可能是由于信息丢失导致。
- **对于残差连接**：
 - 文中实验验证，残差连接在帮助Xception方面至关重要，不仅提升了拟合速度，而且提升了最终的表现。
- 简而言之，Xception结构是带有残差连接的深度可分离卷积的线性堆叠。
- 在Keras或TensorFlow-Slim上，只需要30至40行代码，即可完成Xception架构。
- 基于Keras和TensorFlow的Xception开源实现，已经被提供为Keras应用模块的一部分，见[链接](https://keras.io/applications/#xception)。
 
# 训练策略
选择比较Xception和Inception V3，因为它们的尺寸相似，两者有几乎相同的参数。
对两个分类任务进行比较，ImageNet和JFT。

## 数据集
- 1.1000类别的单标签的ImageNet数据集
- 2.17000类别的多标签的JFT数据集
 -  谷歌内部数据集，由17000类别的超过350百万张高分辨率带标签图像构成

## 最优化配置
对于ImageNet和JFT采用不同的优化配置：

- 对于ImageNet：
 - 优化器: SGD
 - 动量: 0.9
 - 初始学习率: 0.045
 - 学习率衰减: decay of rate 0.94 every 2 epochs
- 对于JFT：
 - 优化器: RMSprop
 - 动量: 0.9
 - 初始学习率: 0.001
 - 学习率衰减: decay of rate 0.9 every 3,000,000 samples
- 对Xception和Inception V3都采用如上的相同优化配置，如上的优化配置是为Inception V3调整出最佳表现的，没有尝试为Xception调整最优超参数。

## 正则化配置

- 权重衰减
 - Inception V3模型采用L2正则化系数为$4e^{-5}$(该参数是Inception V3针对ImageNet表现进行调整)
 - 经实验，上述系数对于Xception不适用，Xception调整为$1e^{-5}$
- Dropout
 - 对于ImageNet，两个模型均在逻辑回归层之前，加上0.5的dropout层
 - 对于JFT，由于该数据集较大，模型在合理时间内不可能出现过拟合现象
- 辅助loss
 - Inception V3中，会在网络前端施加辅助分类器，来反向传播分类器loss，做为另外的正则化机制。但为了简洁性，两模型均不采用辅助loss 

# 模型比较
<img src="/images/Xception/4.png"  width = "400" height = "100"/>

- 如上图所示，在ImageNet上，Xception比Inception V3稍好，也比ResNet-152更好。

<img src="/images/Xception/5.png"  width = "400" height = "100"/>

- 如上图所示，在JFT上，Xception相比于Inception V3的优势，比在ImageNet上更大了：
 - 文中猜测，这是由于Inception V3是专注于ImageNet而设计的，因此，对该特定任务有一些过拟合。
 - 另一方面，两个模型都没有在JFT上调整过，因此，对于Xception，在ImageNet上可能会有更好的超参数(尤其是最优化参数和正则化参数)，进一步让网络表现明显提升。

# 参考文献
- [论文：Xception: Deep Learning with Depthwise Separable Convolutions, 2017](https://arxiv.org/pdf/1610.02357.pdf)







