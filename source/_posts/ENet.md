---
title: ENet
mathjax: true
date: 2019-01-10 22:26:50
categories: 
- 语义分割
tags:
---

# 摘要

- 为了实现实时的语义分割，提出ENet(efficient neural network)
- 相比于当时最优的SegNet，ENet比它快约18倍，FLOPs少约75倍，参数量少约79倍；并且，ENet有着和SegNet相似或更好的准确率。
 - 针对Cityscapes/CamVid/SUN数据集，在NVIDIA Jetson TX1 Embedded Systems Module和NVIDIA Titan X GPU上，进行了实验验证

<!-- more -->

# 创新点

## 初始模块(initial block)以及瓶颈模块(bottleneck module)
<img src="/images/ENet/1.png"  width = "900" height = "200"/>

如图Fig2所示，Fig2(a)为ENet的初始模块，Fig2(b)为ENet的瓶颈模块(非下采样/下采样)。
对于初始模块，**由InceptionV3启发**，如Fig2(a)所示：
 - MaxPooling操作以不重叠的$2 \times 2$窗口执行
 - 卷积操作以核为$3 \times 3$，步长为2的方式执行,卷积核个数为13
 - 经串联后，特征图的通道数为16

对于瓶颈模块(非下采样)，**由ResNet启发，类似于残差模块**，如Fig2(b)所示：
- 主干为输出特征图
- 分支由3个卷积层构成：一个$1\times 1 $的卷积来降维，一个主要的卷积操作，一个$1\times 1 $的卷积来升维
 - 其中，在每两个卷积层之间，放置了`BN`和`PReLU`
 - 其中，卷积操作或是核为$3 \times 3$的常规卷积/膨胀卷积/反卷积，或是核为$5 \times 5$的常规卷积分解出来的两个$1 \times 5$和$5 \times 1$的不对称卷积
 - 其中，对于$1\times1$卷积，不使用偏置项(bias terms)。原因：减少计算；结果：不会对精度产生任何影响。
- 对分支进行正则化，采用Spatial Dropout
 - 在`bottleneck2.0`之前，采用$p=0.01$；之后，采用$p=0.1$     <- 疑问
- 最后，主干和分支进行元素级相加(element-wise addition)，后施加`PReLU`(待确认)

对于瓶颈模块(下采样)，如Fig2(b)所示：
- 主干中增加max pooling层，并进行零填充，来使得特征图的通道数和分支的输出特征图的通道数相等(待确认)，使得支持元素级相加
- 分支中的第一个$1\times1$卷积用核为$2\times 2$且步长为2的卷积替代


对于瓶颈模块(上采样)：
- 主干中的max pooling操作用max unpooling操作替代(同SegNet)，零填充操作用不含偏置项的空间卷积替代 <- 疑问：啥意思
- 分支中做出什么改变？论文中没说，服了


## 网络结构
<img src="/images/ENet/2.png"  width = "400" height = "200"/>

如Table1所示，为ENet的总体网络结构。其中，以$512 \times 512 \times 3 $的输入尺寸为例：

- 初始阶段(initial stage)包含一个单独的模块，即Fig2(a)所示的初始化模块。输出尺寸为：$256\times 256 \times 16$
- 阶段1、阶段2以及阶段3属于编码器：
 - 阶段1(stage 1)包含5个瓶颈模块。输出尺寸为：$128\times 128 \times 64$
 - 阶段2(stage 2)包含9个瓶颈模块。输出尺寸为：$64\times 64 \times 128$
 - 阶段3(stage 3)是阶段2的重复，除了阶段3移除了阶段2中的第0个下采样的瓶颈模块以外，即阶段3一开始不进行下采样。输出尺寸为：$64\times 64 \times 128$
- 阶段4和阶段5属于解码器：
 - 阶段4(stage 4)包含3个瓶颈模块，其中`bottleneck4.0`的unpooling操作，通过采用阶段2中`bottleneck2.0`的max-pooling索引来实现。输出尺寸为：$128\times 128 \times 64$
 - 阶段5(stage 4)包含2个瓶颈模块，其中`bottleneck5.0`的unpooling操作，通过采用阶段2中`bottleneck1.0`的max-pooling索引来实现。输出尺寸为：$256\times 256 \times 16$
- 在最后的上采样模块中：
 - 没有采用unpooling操作。因为与其相对应的初始阶段(initial stage)，是对$513 \times 512 \times 3$的输入尺寸上进行上采样，其通道数仅仅是3，而最终的输出需要$C$个特征图(即目标类别的个数)。通道数不匹配，故无法进行unpooling操作
 - 采用反卷积来进行上采样。这也是ENet中唯一的一个反卷积，这也占用了编码器中的很大一部分处理时间。

## 网络结构中各设计细节的原因


# 训练策略
待补，class balancing等

# 模型比较

# 总结



# 参考文献

- [论文：SegNet: A Deep Convolutional Encoder-Decoder Architecture for Image Segmentation](https://arxiv.org/pdf/1511.00561.pdf)
- [官方代码：alexgkendall/caffe-segnet](https://github.com/alexgkendall/caffe-segnet)
- [代码及注释：meetshah1995/pytorch-semseg](https://github.com/liminn/pytorch-semseg/blob/master/ptsemseg/models/segnet.py)


