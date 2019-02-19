---
title: DeepLabV1
mathjax: true
date: 2019-01-12 22:26:50
categories: 
- 语义分割
tags:
---

# 摘要

- DeepLabV1结构：dilated VGG16 + CRF(Conditional Random Field)
- dilated VGG16用于获得输出步长(output stride)为8的特征图，CRF用于进一步优化边缘结果。

<!-- more -->

# 创新点

## dilated VGG16
对于dilated VGG16，其相较于原始VGG16的改动，表述如下：
- 原始的VGG16：有五个maxpool层，即进行了5次下采样，即输出特征图的空间尺寸为原图的$1/32$
- 注：此处的输出特征图是指全连接层之前的输出特征图
- dilated VGG16：去除最后两个maxpool层，因此下采样次数减少2次，为3次，输出特征图为输入尺寸的$1/8$；同时，施加膨胀卷积，扩大特征图的感受野，弥补去除的两次下采样本可以带来的扩大感受野的机会。

## CRF
<img src="/images/DeepLabV1/1.png"  width = "700" height = "200"/>
CRF用于优化边缘，效果如上图所示。

# 总结
- DeepLabV1结构：dilated VGG16 + CRF(Conditional Random Field)
- DeepLabV1/V2/V3都只会简述，对于DeepLabV3+再做详细阐述。

# 参考文献
- [论文：Semantic Image Segmentation with Deep Convolutional Nets and Fully Connected CRFs](https://arxiv.org/pdf/1412.7062.pdf)


