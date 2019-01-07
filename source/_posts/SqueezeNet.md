---
title: SqueezeNet
mathjax: true
date: 2018-12-28 22:22:50
categories: 
- 轻量型CNN
tags:
---

# 摘要

- 提出小CNN结构——SqueezeNet。
- 在ImageNet上，SqueezeNet能够得到和AlexNet一样的精度，同时参数为其$1/50$。
- 借助于模型压缩技术(model compression techniques)，可将SqueezeNet压缩到小于0.5MB(比AlexNet小510倍)。

 <!-- more -->

# 创新点

## 针对较少参数CNN结构的设计策略提纲

为了设计出含有较少参数且维持较高精度的CNN结构，主要采用如下三点策略：

- 策略1:用$1\times 1$卷积替换$3\times 3$卷积
 - $1\times 1$卷积的参数比$3\times 3$卷积少9倍 
- 策略2:减少$3\times 3$卷积的输入通道数
 - 对于$3\times 3$卷积层，总的参数量是`(number of input channels) * (number of filters)*(3*3)`。
 - 策略1是为了减少$3\times 3$卷积的数量，策略2是为了减少$3\times 3$卷积的输入通道数。
- 策略3:在网络中较晚地进行下采样，使得卷积层有更大尺寸的激活特征图
 - 激活特征图的尺寸由如下两个尺寸控制：1.输入数据的尺寸；2.选择在CNN结构中的哪个层进行下采样(即什么时候进行下采样，是早点还是晚点)
 - 在CNN结构中，下采样操作通常由步长大于1的卷积层或步长大于1的池化层完成。
 - 若网络前端的层用大的步长，那么大多数层将有小的特征图；相反地，若步长大于1的层集中在网络的末端，那么网络中的许多层将拥有较大的激活特征图。
 - 在保持其他设置相等的情况下，较大的特征图(由于较晚地下采样)可以得到更高的分类精度。

 
## Fire模块(Fire module)

<img src="/images/SqueezeNet/1.png"  width = "600" height = "100"/>

如Fig1所示，即为Fire模块：

- Fire模块由squeeze卷积层和expand卷积层构成：
 - squeeze卷积层：由$1\times 1$卷积构成。
 - expand卷积层：由$1\times 1$卷积和$3\times 3$卷积混合构成。
 - squeeze卷积层和expand卷积层两部分，分别压缩和扩展特征的通道数。expand卷积层中，两个不同尺寸核的输出经串联后作为最终输出。
- Fire模块有三个可调参数： 
 - $s_1$: squeeze层中的1×1卷积的通道数 
 - $e_1$: expand层中的1×1卷积的通道数 
 - $e_3$: expand层中的3×3卷积的通道数
- Fire模块中$1\times 1$卷积的大量使用，即遵循上述策略1。
- 文中设置$s_1 <(e_1 + e_3)$，那么squeeze层即帮助限制了传给$3\times 3$卷积的通道数，即遵循上述策略2。
- 在SqueezeNet中，Fire模块采用的压缩比率(squeeze ratio)为0.125，意味着每一个squeeze层的输出通道数比expand层的输出通道数少8倍。
- 输入输出尺寸相同。输入通道数不限，输出通道数为$e1+e3$。 
- 在本文提出的SqueezeNet结构中，$e1=e3=4s1$。

## SqueezeNet结构 
<img src="/images/SqueezeNet/2.png"  width = "700" height = "100"/>

如Fig2所示，为SqueezeNet结构：

- SqueezeNet起始是一个常规卷积(conv1)，接着跟随8个Fire模块(fire2-9)，最后为一个常规卷积(conv10)。
- 在SqueezeNet中，每隔一个Fire模块，逐步地增加卷积核的个数。
- 在SqueezeNet中，在conv1、fire4、fire8和conv10后，采用步长为2的max-pooling。
 - 这些相对延后放置的池化层遵循策略3。

<img src="/images/SqueezeNet/3.png"  width = "700" height = "100"/>

- 如Table1所示，为SqueezeNet的详细信息。

其他细节：
- 在expand层中，对$3\times3$卷积的输入增加了1像素边界的零填充，以使得$3\times3$卷积与$1\times1$卷积的输出激活函数的宽高相等。
- 在Fire模块中，对squeeze层和expand层的激活函数均采用了ReLU。
- 在fire9模块后，采用了比率为50%的Dropout
- SqueezeNet未采用全连接层，由NIN启发。

# 模型比较
<img src="/images/SqueezeNet/4.png"  width = "650" height = "100"/>

如Table2所示，比较了AlexNet和SqueezeNet，以及在各模型压缩方法下的结果：

- 对于AlexNet：
 - 基于SVD的模型压缩方法(SVD-based approach)，将AlexNet压缩了5倍，减少Top1准确率到56%。
 - 网络修剪(Network Pruning)的模型压缩方法，将AlexNet压缩了9倍，并维持Top1和Top5准确率。
 - 深度压缩(Deep Compression)将AlexNet压缩了35倍，并维持Top1和Top5准确率。
- 相比于AlexNet的模型尺寸，SqueezeNet本身就比AlexNet小50倍，并且或维持或超过了AlexNet的Top1和Top5准确率。
- 对SqueezeNet采用深度压缩(Deep Compression)，其中用33%的稀疏性(33% sparsity)和8比特的量化(8-bit quantization)：
 - 得到了0.66MB大小的模型(比32比特表示的AlexNet小363倍)，且相比于基准SqueezeNet没有精度损失，且与AlexNet精度相当。
-  对SqueezeNet采用深度压缩，其中用33%的稀疏性和6比特的量化：
 - 得到了0.47MB大小的模型(比32比特表示的AlexNet小510倍)，且相比于基准SqueezeNet没有精度损失，且与AlexNet精度相当。
- 深度压缩(Deep Compression)将SqueezeNet压缩了10倍，且维持基准精度。

# 参考文献

- [论文：SqueezeNet: AlexNet-Level Accuracy with 50x Fewer Parameters and <0.5MB Model Size](https://arxiv.org/pdf/1602.07360.pdf)
- [博客：超轻量级网络SqueezeNet算法详解](https://blog.csdn.net/shenxiaolu1984/article/details/51444525)
- [代码：论文中给出的源码地址](https://github.com/DeepScale/SqueezeNet)

