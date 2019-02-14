---
title: ENet
mathjax: true
date: 2019-01-10 22:26:50
categories: 
- 轻量型语义分割
tags:
---

# 摘要

- 为了实现实时的语义分割，提出ENet(efficient neural network)
- 相比于当时最优的SegNet，ENet比它快约18倍，FLOPs少约75倍，参数量少约79倍；并且，ENet有着和SegNet相似或更好的准确率。
- 针对Cityscapes/CamVid/SUN数据集，在NVIDIA Jetson TX1 Embedded Systems Module和NVIDIA Titan X GPU上，进行了实验验证

<!-- more -->

# 创新点

## 初始模块(initial block)/常规瓶颈模块(bottleneck module)/下采样瓶颈模块/上采样瓶颈模块
<img src="/images/ENet/1.png"  width = "900" height = "200"/>

如图Fig2所示，Fig2(a)为ENet的初始模块，Fig2(b)为ENet的瓶颈模块(非下采样/下采样)。
对于初始模块(initial block)，**其由InceptionV3启发**，如Fig2(a)所示：
- 并行执行常规卷积和MaxPooling
 - 卷积操作以核为$3 \times 3$，步长为2的方式执行,卷积核个数为13；输出特征图的通道数为13
 - MaxPooling操作以不重叠的$2 \times 2$窗口执行；输出特征图的通道数为3
- 串联两者的输出；输出特征图的通道数为16
- 初始模块(initial block)对应代码：见源码中的`InitialBlock`类

对于常规瓶颈模块，**其由ResNet启发，类似于残差模块**，如Fig2(b)所示：
- 主干为捷径连接，即忽略图中的虚线部分
- 分支由3个卷积层构成：
 - $1\times 1 $的卷积来降维，压缩通道数(压缩比为4)
 - 主卷积操作`conv`，`conv`或是常规卷积(核为$3 \times 3$)，或是膨胀卷积，或是不对称卷积(核为$5 \times 5$的常规卷积分解出来的$1 \times 5$卷积和$5 \times 1$卷积)
 - $1\times 1 $的卷积来升维(注意，此处只是恢复输入的通道数）
 - 采用Spatial Dropout进行正则化；Spatial Dropout对应于torch中的`torch.nn.Dropout2d`
 - 注1：其中，对于$1\times1$卷积，不使用偏置项(bias terms)。原因：减少计算；结果：不会对精度产生任何影响。
 - 注2：其中，在每两个卷积层之间，放置了`BN`和`PReLU`
 - 注3：对于Spatial Dropout，在`bottleneck2.0`之前，采用$p=0.01$；之后，采用$p=0.1$；
- 最后，主干和分支进行元素级加法(element-wise addition)，然后进行`PReLU`
- 常规瓶颈模块对应代码：见[源码](https://github.com/liminn/PyTorch-ENet/blob/master/models/enet.py)中的`RegularBottleneck`类

对于下采样瓶颈模块，如Fig2(b)所示：
- 主干，图中虚线部分生效：
 - MaxPooling，使得特征图的空间尺寸减半，通道数不变；同时，保存最大池化索引，以供后续对应的unpooling层使用。
 - 进行零填充，使得主干的输出通道数与分支的输出通道数(分支的通道数需要变大)相等，从而支持最后的元素级加法
- 分支由3个卷积层构成：
 - 第一个$1\times1$卷积用核为$2\times 2$且步长为2的卷积替代，同时压缩通道数(压缩比为4)
 - 主卷积操作`conv`，`conv`为常规卷积(核为$3 \times 3$)
 - $1\times 1 $的卷积来升维（注意，区别于常规瓶颈模块，此处是提升通道数，而不仅仅是恢复通道数；下采样，特征图空间尺寸变小，则通道数要变大）
 - 采用Spatial Dropout进行正则化；
- 最后，主干和分支进行元素级加法(element-wise addition)，然后进行`PReLU`
- 下采样瓶颈模块对应代码：见[源码](https://github.com/liminn/PyTorch-ENet/blob/master/models/enet.py)中的`DownsamplingBottleneck`类

对于上采样瓶颈模块：
- 主干：
 - $1\times 1$卷积，来减少通道数(上采样，特征图的空间尺寸变大，则通道数减小)
 - max pooling操作用max unpooling操作替代(同SegNet)
- 分支：
 - $1\times 1 $的卷积来降维，压缩通道数(压缩比为4)
 - 主卷积操作`conv`，`conv`为转置卷积(核为$3 \times 3$)(注意，此处有待商榷，论文中没有进行交待，源码中是这么写的；并且，论文中强调最后的上采样操作使用了反卷积，是整个网络中唯一的反卷积)
 - $1\times 1 $的卷积来升维（注意，区别于常规瓶颈模块，此处其实减少了通道数；上采样，特征图的空间尺寸变大，则通道数减小）
 - 采用Spatial Dropout进行正则化；
- 最后，主干和分支进行元素级加法(element-wise addition)，然后进行`PReLU`
- 上采样瓶颈模块对应代码：见[源码](https://github.com/liminn/PyTorch-ENet/blob/master/models/enet.py)中的`UpsamplingBottleneck`类

## 网络结构
<img src="/images/ENet/2.png"  width = "400" height = "200"/>

如Table1所示，为ENet的总体网络结构。其中，以$512 \times 512 \times 3 $的输入尺寸为例：

- 初始阶段(initial stage):
 - 包含一个单独的模块，即Fig2(a)所示的初始化模块。输出尺寸为：$256\times 256 \times 16$
- 阶段1、阶段2以及阶段3属于编码器：
 - 阶段1(stage 1)包含1个下采样瓶颈模块和4个常规瓶颈模块。输出尺寸为：$128\times 128 \times 64$
 - 阶段2(stage 2)包含9个常规瓶颈模块。输出尺寸为：$64\times 64 \times 128$
 - 阶段3(stage 3)是阶段2的重复，其中阶段3移除了阶段2中的第0个下采样的瓶颈模块，即阶段3一开始不进行下采样。输出尺寸为：$64\times 64 \times 128$
- 阶段4和阶段5属于解码器：
 - 阶段4(stage 4)包含1个上采样瓶颈模块和2个常规瓶颈模块，其中`bottleneck4.0`的unpooling操作，通过采用阶段2中`bottleneck2.0`的max-pooling索引来实现。输出尺寸为：$128\times 128 \times 64$
 - 阶段5(stage 4)包含1个上采样瓶颈模块和1个常规瓶颈模块，其中`bottleneck5.0`的unpooling操作，通过采用阶段2中`bottleneck1.0`的max-pooling索引来实现。输出尺寸为：$256\times 256 \times 16$
- 最后的上采样步骤：
 - 没有采用unpooling操作。因为与其相对应的初始阶段(initial stage)，是对$513 \times 512 \times 3$的输入尺寸上进行上采样，其通道数仅仅是3，而最终的输出需要$C$个特征图(即目标类别的个数)。通道数不匹配，故无法进行unpooling操作
 - 采用反卷积来进行上采样。这也是ENet中唯一的一个反卷积，这也占用了编码器中的很大一部分处理时间。

## 网络结构中各设计细节的原因
### 特征图分辨率(Feature map resolution)
- 思考1：下采样操作在语义分割任务中有两个主要缺点：
 - 第一，下采样减小了特征图的分辨率，意味着损失了如边缘形状等空间信息
 - 第二，语义分割任务需要恢复出与输入同等的分辨率。施加越多的下采样，即意味着需要施加同等强度的上采样，这会增加模型大小以及计算代价。
- ENet措施1：对于上采样，ENet沿用SegNet中的unpooling操作
 - 相比于FCN中将编码器的特征图直接加给末端的方式，unpooling操作可以减少所需内存

- 思考2：下采样操作有一个主要优点：
 - 经下采样的特征图，有这更大的感受野，可以聚集更多上下文信息，即利于分类
- ENet措施2：ENet中使用了空洞卷积

### 较早地下采样(Early downsampling)
- 思考1：对于想要实时地得到较好的表现，需要意识到：处理大的特征图是非常昂贵的。
 - 尽管这很明显，但在很多主流模型结构中并没有意识到这点，去在网络的较早阶段进行优化
- ENet措施1：一方面，ENet的前两个模块，较强地减少了输入尺寸；另一方面，其中的特征图只含有较少的通道数
 - 该措施背后的思想是：视觉信息有很高的空间冗余(highly spatially redundant)，因此它可以被压缩成一个更有效的表征。因此，本文认为网络的初始层不需要直接对分类做出贡献，它们只需要预处理输入来供网络后续的部分使用。
 - 实验证明：将初始模块中的通道数从16增加到32，对精度没有任何改善

### 解码器尺寸(Decoder size)
- 思考1：SegNet是一个非常对称的结构，解码器严格镜像了编码器。但本文认为:
 - 编码器应该同原始的分类结构类似，编码特征，提供分类能力
 - 解码器的角色只是去上采样编码器的输出，只需要微调细节信息(fine-tuning the details)
- ENet措施1：ENet采用大的编码器，小的解码器。

### 非线性激活函数(Nonlinear operations)
看了几遍没看懂

### Information-preserving dimensionality changes
- 思考1：如上所述，较早地对输入进行下采样是很有必要的。但是，过度的维度减少会阻碍信息流动。
 - 如在VGG中：在maxpooling之后，跟随一个扩张通道数的卷积。这样的做法虽然相对较便宜，一方面引入了表达瓶颈，另一方面强迫使用更多的通道数，使得计算更昂贵。
- ENet措施1：沿用InveptionV3中处理该问题的方法，即将pooling操作和步长为2的卷积操作并行执行，然后串联起结果特征图。该技术使得ENet中初始模块(initial block)的推理时间加速了10倍。

- 思考2：文中发现原始的ResNet结构的一项问题：在下采样时，ResNet的残差模块的第一个1x1卷积的步长为2，忽略了75%的输入。
- ENet措施2：将该1x1卷积的卷积核扩大到2x2，使得全部的输入被考虑进来，因此改善信息流动和最终的精度。
 - 当然，会使得这些层的计算代价贵4倍，但在ENet中，用于下采样的这些层很少，因此这些开销的增加并不明显。

### 分解卷积核(Factorizing filters)
- 思考：同InceptionV3中指出的，卷积权重有很大程度地冗余，每一个nxn卷积可以被分解成为两个更小的卷积：nx1和1xn。沿用InceptionV3中对于此种卷积的命名-不对称卷积(asymmetric convolutions)
- ENet措施：采用n=5的不对称卷积，即5x1和1x5，相当于一个单独的3x3卷积。这鞥家了模块学习到的函数的多样性，并且增加了感受野。

### 空洞卷积(Dilated convolutions)
- 思考：如上所述，拥有大的感受野对于网络来说很重要，通过采用更大的感受野，可以提升模型的分类性能。但本文想避免过度的下采样特征图，所以决定采用空洞卷积来改善模型。
- ENet措施：在几个瓶颈模块中采用空洞卷积。具体地，是在其他瓶颈模块(常规卷积/非对称卷积)中插入空洞卷积的瓶颈模块，而不是序列化的施加空洞卷积的瓶颈模块(`Multi-scale context aggregation by dilated convolutions`指出)，这样的做法可以得到最好的准确率。
 - 该做法提高了Citycaps数据集上的IoU约4个百分点，并且没有任何其他额外的代价。

### 正则化(Regularization)
- 思考：大多数语义分割任务的数据集相对较小(10^3张图片)。因此，强大的神经网络快速地开始过拟合。
- ENet做法：
 - 采用L2权重衰减只小有成果
 - 最终采用Spatial Dropout，在卷积分支的末端采用Spatial Dropout，在相加之前。

# 训练策略
整体训练策略：

- 首先，只训练编码器，来分类输入图片的下采样的区域
- 然后，添加解码器，训练整个网络，来执行上采样和像素级分类

最优超参数：
 - 输入尺寸：480x360
 - 优化器：Adam
 - 初始学习率：5e-4
 - L2权重衰减：2e-4
 - 批量大小：10
 - 自定义的类别权重方案：$W_{class} = \frac{1}{ln(c+p_{class})}$
  - $c$设为1.02
 - 训练耗时：在4个Titan X GPUs上，训练耗时约3-6个小时。

# 模型比较
- 为了验证ENet在实际应用中的实时性和表现，在三个数据集上进行了实验：道路场景的CamVid和Cityscapes，室内场景的SUN RGB-D。
- 在NVIDIA Titan X GPU和NVIDIA TX1 embedded system module上进行实验
- ENet是为了在640x360的尺度下，在NVIDIA TX1上，获得10fps而设计的；这样的fps对于实际的道路场景解析应用中是足够的。
- 在推理阶段，融合BN和dropout层进入卷积核之中，加速了整个网络。

<img src="/images/ENet/3.png"  width = "900" height = "200"/>
如Table2所示，为SegNet和ENet的性能比较：

- ENet显著地快于SegNet，提供了可用于实时语义分割的高帧率。

<img src="/images/ENet/4.png"  width = "900" height = "200"/>
如Table3所示，为SegNet和ENet的硬件需求比较。

<img src="/images/ENet/5.png"  width = "700" height = "200"/>
<img src="/images/ENet/6.png"  width = "700" height = "200"/>
<img src="/images/ENet/7.png"  width = "700" height = "200"/>

# 总结

# 参考文献

- [论文：ENet: A Deep Neural Network Architecture for Real-Time Semantic Segmentation](https://arxiv.org/pdf/1606.02147.pdf)
- [代码及注释：davidtvs/PyTorch-ENet](https://github.com/liminn/PyTorch-ENet)


