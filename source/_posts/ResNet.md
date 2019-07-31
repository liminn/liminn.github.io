---
title: ResNet
mathjax: true
date: 2019-07-25 13:45:00
categories: 
- CNN
tags:
---

# 摘要

- 为了解决随着网络层数的加深，随之带来的**退化问题(degradation problem)**，提出ResNet。
- 对于ResNet，其带有**捷径连接(shortcut connections)**的**构建模块(BasicBlock或Bottleneck)**，为其最大的创新点。
- ResNet的思想是将神经网络的学习重新定义为学习关于层输入的残差函数(residual functions)，而不是学习无参考的函数。
- 实验结果证明：残差网络有效地解决了退化问题，并且随着网络深度的增加可以获得精度提升。
<!-- more -->

# 关键点

## 退化问题(degradation problem)
<img src="/images/ResNet/1.png"  width = "500" height = "100"/>
如上图所示：
- 56层网络的trainning error和test error都比20层网络高(不是由于过拟合导致的)，此现象即为退化问题。

## 构建模块
<img src="/images/ResNet/2.png"  width = "500" height = "100"/>
如上图所示，为ResNet构建模块：
- 左侧为ResNet-18和ResNet-34中所使用的构建模块，右侧为ResNet-50,ResNet-101,ResNet-152中所使用的构建模块，称为"bottleneck building block"。 
- 其中，随着网络的加深，bottleneck的设计是为了节省参数。
- 在上述构建模块中，在维度增加时，捷径连接采用映射捷径(projection shortcuts)，其他皆为恒等捷径连接。
 - 注：对于捷径连接，也可以全部采用映射捷径(projection shortcuts)，但效果差不多，且会引入很多参数，没必要。

## ResNet结构 
<img src="/images/ResNet/3.png"  width = "900" height = "500"/>
如上图所示，为ResNet结构：
- 分别为ResNet-18, ResNet-34, ResNet-50, ResNet-101以及ResNet-152。

# 实现细节
对于ImageNet：
- 预处理：
    - 图像的较短边被随机缩放到[256,480]之间
    - 随机水平翻转
    - 随机剪裁224x224的图像块
    - 减去均值
    - 标准颜色增强(standard color augmentation)？？？

- 训练相关：
 - SGD，batch size为256
 - 初始学习率为0.1， 达到error plateaus时，学习率除以10，训练60x10^4次迭代
 - 权重衰减为0.0001, 动量为0.9

# 实验结果
<img src="/images/ResNet/4.png"  width = "450" height = "100"/>
如上图所示，为ResNet单模型在ImageNet验证集上的结果。

# Diss and Respect
Diss: 
- 没有diss，只有respect

Respect:
- ResNet解决了随着网络层数加深时，出现的退化问题。
- 这也意味着，当构建的网络较深时，捷径连接是必须要有的操作了。


# 参考文献

- [论文：Deep Residual Learning for Image Recognition](https://arxiv.org/pdf/1512.03385.pdf)
- [代码及注释：vision/torchvision/models/resnet.py](https://github.com/liminn/vision/blob/master/torchvision/models/resnet.py)

