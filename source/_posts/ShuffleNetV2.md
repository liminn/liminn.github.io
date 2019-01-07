---
title: ShuffleNetV2
mathjax: true
date: 2018-12-30 22:25:50
categories: 
- 轻量型CNN
tags:
---
 
# 摘要

- 目前，神经网络架构的设计主要由计算复杂度的间接指标(即FLOPs)来指导；
- 但是，直接指标(如速度或延迟)还依赖于其他因素，如内存访问成本(MAC, memory access cost)和平台特点(platform characterics)；
- 因此，本文指出过去在网络架构设计上仅注重间接指标 FLOPs 的不足，并提出两个基本原则(principles)和四个实用准则(practical guidelines)来指导网络架构设计；
- 提出针对移动端深度学习的第二代卷积神经网络ShuffleNetV2，在综合实验评估中，ShuffleNetV2在速度和精度的权衡方面达到当前最优水平。
- 注：FLOPs，即the number of float-point operations，定义为the number of multiply-adds.
<!-- more -->

# 创新点

## 间接指标(FLOPs)与直接指标(速度/延迟)

对于FLOPs：

- 当前，FLOPs，即浮点运算数，是度量计算复杂度的常用指标。
- 然而，FLOPs是一种间接指标。它只是本文真正关心的直接指标(如速度或延迟)的一种近似形式，通常无法与直接指标划等号。
- 实验表明：FLOPs近似的网络也会有不同的速度。
- 所以，将FLOPs作为衡量计算复杂度的唯一标准是不够的，这样会导致次优设计。

间接指标(FLOPs)与直接指标(速度/延迟)之间存在差异的原因可以归结为两点：
- 首先，对速度有较大影响的几个重要因素对FLOPs不产生太大作用:
 - 第一个因素是内存访问成本(MAC, memory access cost)。在某些操作(如组卷积)中，MAC占运行时间的很大一部分。对于像GPU这样具备强大计算能力的设备而言，这就是瓶颈。在网络架构设计过程中，MAC不能被简单忽视。
 - 第二个因素是并行度(degree of parallelism)。当FLOPs相同时，高并行度的模型可能比低并行度的模型快得多。
- 其次，取决于平台特点(platform characterics)，FLOPs相同的运算可能有着不同的运行时间
 - 例如，早期研究广泛使用张量分解(tensor decomposition)来加速矩阵相乘。但是，近期研究发现尽管张量分解减少了75%的FLOPs，但在GPU上甚至更慢。本文研究人员对此进行了调查，发现原因在于最新的CUDNN库专为3×3卷积优化。因此，不能简单地认为3×3卷积的速度比1×1卷积慢9倍。

## 两个基本原则(two principles)
基于上述间接指标(FLOPs)与直接指标(速度/延迟)之间存在差异的原因，本文提出了高效网络架构设计应该考虑的两个基本原则：

- 第一，应该用直接指标(速度/延迟)替换间接指标(FLOPs)；
- 第二，这些指标应该在目标平台(target platform)上进行评估。

## ShuffleNetV1和MobileNetV2的运行时间性能(runtime performance)分析
<img src="/images/ShuffleNetV2/1.png"  width = "600" height = "100"/>

如Fig2所示，为ShuffleNetV1和MobileNetV2在ARM和GPU上的运行时间分析：

- 总体的运行时间可以分解为五个部分：卷积、数据I/O、数据重排(data shuffle)、元素级运算(张量加法、ReLU等)及其他；
- 文中注意到FLOPs仅和卷积部分相关，尽管这一部分需要消耗大部分的时间，但其它过程也需要消耗相当数量的时间；
- 因此，FLOPs不是真实运行时间的足够准确地估计。

## 四个实用准则(practical guidelines)
基于上述网络运行时间性能的分析，提出设计高效网络架构需要遵循的四个实用准则(practical guidelines)。
### G1. 相同的通道宽度可最小化内存访问成本(MAC）

- 现代网络大多会采用depthwise separable convolutions，pointwise convolution($1\times1$卷积)占据了其中大部分复杂度。
- $1\times1$卷积的FLOPs为：$B=h w c_1 c_2$
 - 其中，$h$和$w$为特征图的空间尺寸，$c_1$为输入通道数，$c_2$为输出通道数
- 为了简化计算，假设计算设备的缓存足够大，能够储存完整的特征图和参数。那么，MAC或内存访问操作的数量是：$MAC = hw(c_1+c_2)+c_1c_2$
 - 其中，第一项为输入/输出特征图的内存访问，第二项为卷积核权重参数的内存访问
- 依据均值不等式(mean value inequality)，即$\sqrt{c_1 c_2} \leq \frac{c_1+c_2}{2}$，可得：$MAC \geq 2\sqrt{hwB}+\frac{B}{hw}$
- **因此，在给定FLOPs时，即可得到MAC的下界(lower bound)。并且，当输入的通道数和输出的通道数相等时，达到MAC的下界。**
- 注意，该结论只是理论性的。因为在实际中，许多设备的缓存并不足够大，并且现代计算库通常采用复杂的模块策略来充分利用其缓存机制。


<img src="/images/ShuffleNetV2/2.png"  width = "600" height = "100"/>

如Table1所示，为G1的验证试验：

- 该实验固定总体的FLOPs，变化比率$c_1：c_2$，来比较运行速度；
- 可以发现：当$c_1:c_2$接近$1:1$时，MAC开始变小，网络评估速度更快；
- 因此，G1得到实际验证。

### G2. 过度的组卷积会增加MAC
 
- 分组卷积通过改变所有通道之间的密集卷积(dense convolution)，成为只和组内通道进行的稀疏卷积(sparse convolution)，来减少计算复杂度(FLOPs)；
- 一方面，在固定的FLOPs下，它允许了更多的通道数；并且，它增加了网络的容量(capacity)，也因此得到更高的准确率；
- 另一方面，然而，增加的通道数量导致了更多的MAC。

- 正式的讲，对于$1\times1$分组卷积，MAC和FLOPs的关系为：$MAC = hw(c_1+c_2)+\frac{c_1c_2}{g} = hwc_1 + \frac{Bg}{c_1}+ \frac{B}{hw} $
 - 其中，$g$为分组的组数；B为$1\times1$分组卷积的FLOPs，$B = \frac{hwc_1c_2}{g}$
 - 容易看出，在给定固定的输入尺寸$c_1 \times h \times w$以及计算代价$B$时，MAC随着$g$的增长而增加。

<img src="/images/ShuffleNetV2/3.png"  width = "600" height = "100"/>

如Table2所示，为G2的验证试验：

- 该实验固定总体的FLOPs，采用不同的组数，来比较运行速度；
- 明显地，采用大的分组数，显著地降低了运行速度。
 - 例如：在GPU上，当采用分组数为8时，比分组数为1(标准密集卷积)时，慢2两倍；在ARM上，慢30%。
 - 这主要是由于MAC的增加。
- 因此，我们建议：
 - 对于分组数，基于目标平台和任务，来谨慎地选择；
 - 简单地因为大的分组数可以允许更大的通道数(获得更高的准确率)，而选择大的分组数，是不明智的。

### G3. 网络碎片化(network fragmentation)会降低并行度(degree of parallelism)

- 例如GoogLeNet的多路径结构会降低并行度
- 尽管这种分散的结构(fragmented structure)已经展示出对准确率有益，但是它可能降低效率，因为它对像GPU这种带有强大并行计算能力(parallel
computing powers)的设备很不友好。
 - 疑问：为啥不友好？
- 这种分散的结构也引起了诸如卷积核加载(kernel launching)和同步化(synchronization)等开销。

<img src="/images/ShuffleNetV2/4.png"  width = "600" height = "100"/>

如Table3所示，为G3的验证试验：

- 可以发现：
 - 在GPU上，分散化显著地减少了运行速度；
 - 在ARM上，速度的减少相对较小。

### G4. 元素级运算(element-wise operations)不可忽视(non-negligible)

- 如Fig2所示，对于轻量化的模型ShuffleNetV1和MobileNetV2，元素级运算占据了相当大的时间，尤其是在GPU上。
- 在这里，元素级运算包括：ReLU, AddTensor, AddBias等。它们有较小的FLOPs，但是有相对较大的MAC。
- 特殊地，我们将depthwise convolution考虑为元素级运算，因为它也有较高的MAC/FLOPs比率。

<img src="/images/ShuffleNetV2/5.png"  width = "600" height = "100"/>

如Table4所示，为G4的验证试验：
- 该实验以ResNet中的“bottleneck”单元为实验对象，探究在ReLU和捷径连接在分别被去除后，运行时间的变化；
- 可以发现：在ReLU和捷径连接被移除之后，在GPU和ARM上，均取得了20%的加速。

### 结论和建议

基于上述四点指导准则和实证研究，对于高效网络结构的设计，我们给出以下结论：

- 1.应该使用平衡的卷积("balanced" convolutions)，即相等的通道宽度
- 2.意识到使用分组卷积的代价
- 3.减少分散程度(the degree of fragmentation)
- 4.减少元素级运算
- 并且，这些期望特性(desirable properties)取决于平台特性(如内存操作和代码优化)，超越了理论的FLOPs。它们都应该在实际的网络设计中被考虑到。

## 近期轻量型网络分析

- 近期，轻量级神经网络架构上的研究进展主要基于间接指标FLOPs，并且没有考虑上述四个准则
- 例如，ShuffleNetV1严重依赖组卷积(违反G2)和瓶颈形态的构造块(违反G1)
- MobileNetV2使用倒置的瓶颈结构，违反了G1。它在「厚」特征图上("thick" feature maps)使用了depthwise convolution和ReLU，违反了G4
- 自动生成结构(the auto-generated structures)的高度碎片化，违反了G3

## ShuffleNetV1回顾与分析

<img src="/images/ShuffleNetV2/6.png"  width = "600" height = "100"/>

- ShuﬄeNetV1是一种先进的网络架构，广泛应用于手机等低配设备。它启发了本文工作，因此首先对其进行回顾与分析。
- 根据ShuﬄeNetV1，轻量级网络的主要挑战是在给定计算预算(FLOPs)时，只能获得有限数量的特征通道。
- 为了在不显著增加FLOPs情况下增加通道数量，ShuﬄeNetV1采用了两种技术：逐点分组卷积(pointwise group convolutions)和类瓶颈(bottleneck-like)结构；
- 然后引入通道重排(channel shuﬄe)操作，令不同组的通道之间能够进行信息交流，提高精度。
- 其构建模块如Fig3中的(a)(b)所示。

- 如第二部分所述，逐点分组卷积和瓶颈结构都增加了MAC(G2和G1)。这个成本不可忽视，特别是对于轻量级模型。
- 另外，使用太多分组也违背了G3。
 - 疑问：应该是违背了G2吧
- 捷径连接（shortcut connection）中的元素级「加法」操作也不可取 (G4)。
- 因此，为了实现较高的模型容量和效率，关键问题是如何保持大量且同样宽的通道，既没有密集卷积也没有太多的分组。

## 通道分割(channel split)

为此，本文引入一个简单的操作——通道分割(channel split)。

- 如Fig3(c)所示:
 - 在每个单元的开始，$c$特征通道的输入被分为两支，分别带有$c−c'$ 和$c'$个通道。
 - 按照G3，一个分支仍然保持不变。
 - 另一个分支由三个卷积组成，为满足 G1，令输入和输出通道相同。
 - 与ShuffleNetV1不同的是，两个$1\times 1$卷积不再是分组卷积。这样做的部分原因是为了遵循G2，部分原因是因为通道分割操作已经产生了两个组。
 - 卷积之后，把两个分支拼接起来，从而通道数量保持不变(G1)。然后进行与ShuffleNetV1相同的通道重排(channel shuﬄe)操作来保证两个分支间能进行信息交流。
 - 通道重排(Channel Shuﬄe)之后，下一个单元开始运算。
 - 注意，ShuﬄeNetV1中的「加法」操作不再存在。像ReLU 和epth- wise convolutions这样的操作只存在一个分支中。
 - 另外，三个连续的操作拼接(concat)、通道重排(channel shuﬄe)和通道分割(channel shuﬄe)合并成一个元素级操作。根据G4，这些变化是有利的。
 - 疑问：三个连续操作时如何合并成一个的？？

- 如Fig3(d)所示，对于空间下采样，该单元经过稍微修改:
 - 通道分割运算被移除。因此，输出通道数量翻了一倍。

## ShuffleNetV2 

<img src="/images/ShuffleNetV2/7.png"  width = "600" height = "100"/>

- 基于本文提出的构造模块Fig3(c)(d)而构成的网络，被称之为ShuﬄeNetV2。
- 基于上述对构造模块Fig3(c)(d)分析，本文得出结论：由于遵循四个准则的遵循，该架构设计异常高效。
- 上述基础构建模块被重复堆叠以构建整个网络。为了简洁，本文令$c' = c/2$。
- 如Table5所示，为ShufﬂeNetV2的整体结构：
 - 整个网络结构类似于ShufﬂeNetV1，二者之间只有一个区别：前者在全局平均池化层之前添加了一个额外的$1\times 1$卷积层来混合特征，ShuﬄeNetV1中没有该层。
 - 与ShuﬄeNetV1类似，每个构建块中的通道数量可以扩展以生成不同复杂度的网络，标记为`0.5×`、`1×`等。

# 模型比较
<img src="/images/ShuffleNetV2/8.png"  width = "600" height = "100"/>

- 如Table8所示，为各网络在四个级别的计算复杂度(40/140/300/500+ MFLOPs)级别上的FLOPs、Top-1错误率以及在GPU/ARM上的速度对比。
 - 可以发现：在同等的FLOPs下，ShuffleNetV2的各项表现均为最佳。

# 参考文献
- [论文：ShuffleNet V2: Practical Guidelines for Efficient CNN Architecture Design](https://arxiv.org/pdf/1807.11164.pdf)
- [博客(旷视官方)：ECCV 2018 | 旷视科技提出新型轻量架构ShuffleNet V2：从理论复杂度到实用设计准则](http://www.sohu.com/a/244491616_418390)





