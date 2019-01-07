---
title: ShuffleNetV1
mathjax: true
date: 2018-12-30 22:22:50
categories: 
- 轻量型CNN
tags:
---

# 摘要

- 提出CNN结构ShuffleNet，专门针对计算力非常有限的移动设备(10-150MFLOPs)而设计。
- ShuffleNet利用两种新的操作：逐点分组卷积(pointwise group convolution)和通道重排(channel shuffle)，在保持准确率的同时，显著地减少了计算代价。
- 在给定的计算复杂度开销下，ShuffleNet允许更多的特征图通道数，这一点可以帮助编码更多的信息，这对于小型网络的表现是尤其关键的。
- 在ImageNet分类任务中，在计算开销为40MFLOPs时，ShuffleNet的top-1错误率，比MobileNetV1要低7.8%。
- 在基于ARM的移动设备上，ShuffleNet相比于AlexNet得到了大约13倍的实际加速，同时保持了类似的精度。

<!-- more -->

# 创新点

## 逐点分组卷积(pointwise group convolution)

背景：

- 现代卷积神经网络的绝大多数计算量集中在卷积操作上，因此高效的卷积层设计是减少网络复杂度的关键。
- 其中，稀疏连接(sparse connection)是提高卷积运算效率的有效途径，当前不少优秀的卷积模型均沿用了这一思路：
 - 谷歌的”Xception“网络引入了”深度可分离卷积”的概念，将普通的卷积运算拆分成逐通道卷积（depthwise convolution）和逐点卷积（pointwise convolution）两部进行，有效地减少了计算量和参数量；
 - 而 Facebook 的“ResNeXt”网络[2]则首先使用逐点卷积减少输入特征的通道数，再利用计算量较小的分组卷积（group convolution）结构取代原有的卷积运算，同样可以减少整体的计算复杂度。

具体改进：

- ShuffleNet 网络结构同样沿袭了稀疏连接的设计理念。
- 作者通过分析Xception和ResNeXt模型，发现这两种结构通过卷积核拆分虽然计算复杂度均较原始卷积运算有所下降，然而拆分所产生的逐点卷积计算量却相当可观，成为了新的瓶颈。
- 即分析Xception和ResNeXt，它们在$1 \times1$卷积上的代价过大。因此提出逐点分组卷积来减少$1 \times1$卷积的复杂度:
 - 例如：在ResNeXt中，只有$3 \times3$卷积配备了分组卷积。因此，在ResNeXt的残差单元中，逐点卷积占据了93.4%的multiplication-adds。
 - 在小型网络中，昂贵的逐点卷积会限制通道的数目，以应对复杂度限制。但这也显著地伤害了准确率。
 - 为了解决这个问题，一个直截了当的解决方案便是施加通道稀疏连接，如对$1 \times1$卷积进行分组卷积。
 - 分组卷积通过保证在对应的输入通道分组上进行卷积，因此显著地减少了计算代价。

## 通道重排(channel shuffle)

<img src="/images/ShuffleNetV1/1.png"  width = "400" height = "100"/>

原因：

- 在多层逐点卷积堆叠时，模型的信息流被分割在各个组内，组与组之间没有信息交换（如图 1(a) 所示）。这将可能影响到模型的表示能力和识别精度。
- 因此，在使用分组逐点卷积的同时，需要引入组间信息交换的机制；也就是说，对于第二层卷积而言，每个卷积核需要同时接收各组的特征作为输入，如图 1(b) 所示。
- 作者指出，通过引入“通道重排”（channel shuffle，见图 1(c) ）可以很方便地实现这一机制；并且由于通道重排操作是可导的，因此可以嵌在网络结构中实现端到端的学习。

## ShuffleNet单元(ShuffleNet Unit)
<img src="/images/ShuffleNetV1/2.png"  width = "700" height = "100"/>

基于分组逐点卷积和通道重排操作，作者提出了全新的ShuffleNet结构单元，如Fig2所示。该结构继承了“残差网络”（ResNet）的设计思想，在此基础上做出了一系列改进来提升模型的效率：

- 首先，如Fig2(a)所示，使用逐通道卷积替换原有的 3x3 卷积，降低卷积操作抽取空间特征的复杂度；
- 然后，如Fig2(b)所示，将原先结构中前后两个 1x1 逐点卷积分组化，并在两层之间添加通道重排操作，进一步降低卷积运算的跨通道计算量。最终的结构单元如图 2(b) 所示。
 - 第二个逐点分组卷积的目的是恢复通道的维度，来与捷径相匹配。
 - 为了简洁，没有在第二个逐点分组卷积层之后施加通道重排，因为这样的网络结构已经可以得到有力的结果。
-  BN和非线性激活函数的使用同ResNet和ResNeXt类似；但在depthwise卷积之后，不适用ReLU。这一点由Xception建议。
- 如Fig2(c)所示，为ShuffleNet专门用于特征图的降采样的单元，做了两点修改：
 - 在捷径中增加$3\times 3$的步长为2的平均池化操作
 - 用通道串联(channel concatenation)替代逐像素加(element-wise addition)，使得通过较少的额外的计算代价，来简单地实现通道维度的扩大
- 由于伴有通道重排的逐点分组卷积，ShuffleNet单元中的所有组件均能够有效地进行计算。给定一个计算开销，ShuffleNet能够得到更宽的特征图，这一点对于小型网络至关重要，因为小型网络通常没有足够数量的通道数目来处理信息。

## ShuffleNet结构
<img src="/images/ShuffleNetV1/3.png"  width = "800" height = "100"/>

如Table1所示，为ShuffleNet总体结构：

- 借助ShuffleNet结构单元，作者构建了完整的ShuffeNet网络模型：
 - 它主要由16个ShuffleNet结构单元堆叠而成，分属网络的三个阶段；
 - 每经过一个阶段特征图的空间尺寸减半，而通道数翻倍；
 - 即在每一个阶段中的第一个building block中施加步长为2；在每一阶段中的其他超参数保持一致；下一个阶段的输出通道数double；
 - 整个模型的总计算量约为 140 MFLOPs。通过简单地将各层通道数进行放缩，可以得到其他任意复杂度的模型。
- 对于每一个ShuffleNet单元，与ResNet相似，设置bottleneck的通道数为输出通道数的$1/4$。
- 在ShuffleNet中，分组数目$g$控制着逐点卷积的连接稀疏性。Table1中探究了不同的分组数目，文中通过调整输出通道数来保证了总体的计算代价大体不变(~140MFLOPs)：
 - 可以发现，当卷积运算的分组数越多，模型的计算量就越低；
 - 这就意味着当总计算量一定时，较大的分组数可以允许较多的通道数，作者认为这将有利于网络编码更多的信息，提升模型的识别能力。
- 为了定制模型到想要的复杂度，简单地在通道数目上施加一个缩放因子$s$即可。Table1中的模型称为"ShuffleNet 1x"，"ShuffleNet sx"意为缩放"ShuffleNet 1x"中的通道数目$s$倍，因此，总体地复杂度大约是"ShuffleNet 1x"的$s^2$倍。
 - 疑问：为什么是$s^2$倍？

# 训练策略

大多数的训练设置和超参数选择和ResNeXt一致，有两点例外：

- 将权重衰减设置为$4e^{-5}$而不是$1e^{-4}$；用线性衰减学习率策略(从0.5减少到0)
- 在预处理时，用更少的尺寸增强(scale augmentation)
- 在MobileNet中，也有上述两点相似的修改，因为小网络通常更易欠拟合而不是过拟合。

# 消融实验
<img src="/images/ShuffleNetV1/4.png"  width = "600" height = "100"/>

 如Table2所示，为逐点分组卷积的组数对分类错误率的影响：

- 从结果可以看出，带有分组卷积($g>1$)的模型，总是比对应的不带有分组卷积($g=1$)的模型要好。
- 更小的网络从分组卷积中受益更多：
 - 如ShuffleNet1x($g=8$)比ShuffleNet1x($g=1$)的错误率低1.2%;
 - ShuffleNet0.5x($g=4$)比ShuffleNet0.5x($g=1$)的错误率低3.5%;
 - ShuffleNet0.25x($g=8$)比ShuffleNet0.25x($g=1$)的错误率低4.4%。
- 注意，在给定的复杂度限制下，分组卷积允许了更多的特征图通道数，因此本文假设：更宽的特征图帮助编码更多信息，可以获得更好的表现。

<img src="/images/ShuffleNetV1/5.png"  width = "600" height = "100"/>

  如Table3所示，为通道重排的消融实验结果：
  
  - 通道重排的目的是使得组间信息能够互相交流。在实验中，有通道重排的网络始终优于没有通道重排的网络，错误率降低 0.9%~4.0%。
  - 尤其，当分组数目较大(如$g=8$)时，通道重排对于模型的提升更大，如对于ShuffleNet1x($g=8$)，通道重排对于模型的提升达5.2%；这也说明了跨组之间的信息交换的重要性。

# 模型比较
<img src="/images/ShuffleNetV1/6.png"  width = "600" height = "100"/>

如Table5所示，为在不同复杂度下，ShuffleNet和MobileNet的对比结果：

- 结果表明，在所有复杂度下，ShuffleNet都优于MobileNet：
 - 尽管ShuffleNet专为小型网络设计(<150MFLOPs)，在增大到 MobileNet 的 500~600 MFLOPs 量级，仍优于MobileNet，如在500MFLOPs的计算代价上，ShuffleNet比MobileNet1x高3.1%。
 - 在40MFLOPs的计算代价上，ShuffleNet比MobileNet错误率低 6.7%。
 
<img src="/images/ShuffleNetV1/7.png"  width = "600" height = "100"/>

如Table6所示，为ShuffleNet与不同模型的计算复杂度对比。

<img src="/images/ShuffleNetV1/8.png"  width = "700" height = "100"/>
 
 如Table8所示，为在基于ARM平台的移动设备上，ShuffleNet与不同模型的真实推理速度评估结果：

- 对于ShuffleNet，尽管更大的分组卷积(如$g=4$或如$g=8$)通常有更好的表现，但文中发现在实际应用中地效率较低；因此，文中经验性地选择$g=3$来作为精度和实际推理速度之间的合适的权衡值。
- 在实际应用中，由于内存访问和其他开销，文中发现：每4倍的理论复杂度减少通常会导致约2.6倍的实际加速。
- 相比于AlexNet，ShuffleNet0.5x在得到相似准确率的同时，得到约13倍的实际加速(理论加速为18倍)。

# 参考文献

- [论文：ShuffleNet: An Extremely Efficient Convolutional Neural Network for Mobile Devices, 2017](https://arxiv.org/pdf/1707.01083.pdf)
- [博客(旷视官方)：为移动AI而生——旷视(Face++)最新成果ShuffleNet全面解读](https://www.sohu.com/a/156321743_418390)





