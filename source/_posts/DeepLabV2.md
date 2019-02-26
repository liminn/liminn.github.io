---
title: DeepLabV2
mathjax: true
date: 2019-01-13 22:26:50
categories: 
- 语义分割
tags:
---

# 摘要
- DeepLabV2结构：dilated VGG16/ResNet-101 + ASPP(atrous spatial pyramid pooling) + CRF
 - dilated VGG16/ResNet-101：作为密集特征提取器(Dense Feature Extraction)，用于获得输出步长(output stride)为8的特征图。其中膨胀卷积在不增加参数数量和计算量的情况下，有效增大了感受野。
 - ASPP：提出ASPP用于分割多种尺度的目标，ASPP由并行的不同采样率的膨胀卷积构成，不同采样率的膨胀卷积代表着不同大小的感受野，因此可以捕捉不同尺度的图片上下文信息。
 - CRF：用于进一步优化边缘结果，提升目标边缘的定位精度(localization accuracy)。

<!-- more -->

# 创新点

## ASPP(atrous spatial pyramid pooling)
<img src="/images/DeepLabV2/1.png"  width = "500" height = "200"/>
如Fig4所示，为ASPP结构示意图：
- ASPP由[SPP(spatial pyramid pooling)](https://arxiv.org/pdf/1406.4729.pdf)启发，是SPP的变体。
- 通过四个并行的层级：各层级含不同采样率的膨胀卷积，其中$K = 3$，采样率分别为6/12/18/24；各分支上单独进行的后续处理。
- 最后，融合四个分支的结果，产生最终的结果。

# 总结
- DeepLabV2结构：dilated VGG16/ResNet-101 + ASPP(atrous spatial pyramid pooling) + CRF
- DeepLabV2相比于DeepLabV1的改进点为增加了ASPP，用于分割多种尺度的目标。
- DeepLabV1/V2/V3都只会简述，对于DeepLabV3+再做详细阐述。

# 参考文献
- [论文：DeepLab: Semantic Image Segmentation with Deep Convolutional Nets, Atrous Convolution, and Fully Connected CRFs](https://arxiv.org/pdf/1606.00915.pdf)


