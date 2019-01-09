---
title: FCN
mathjax: true
date: 2019-01-08 22:25:50
categories: 
- 语义分割
tags:
---

## 摘要

- 针对语义分割任务，提出了全卷积网络(FCN, fully convolutional network)，使得对任意尺寸的输入，网络能够产生对应尺寸的输出。
- 首先，修改当前的分类模型(如AlexNet、VGG及GoogleNet)成为全卷积网络，然后通过微调(fine-tuning)来对分割任务进行迁移学习。
- 其中，定义了跳跃结构(skip architecture)，即将深层的粗糙的语义信息(deep,coarse,semantic information)和浅层的精细的轮廓信息(shallow,fine,appearance information)相结合(元素级相加)。

<!-- more -->

## 创新点

### 全卷积化分类网络
<img src="/images/FCN/1.png"  width = "700" height = "100"/>

如上图所示，修改分类网络来进行密集预测(Adapting classifiers for dense prediction)：

- 1.将分类网络最后的分类层抛弃，将全连接层卷积化：
 - 分类网络(如AlexNet、VGG及GoogleNet)中存在全连接层，而全连接层有着固定的维度，并且抛弃了空间坐标信息，故将全连接层进行卷积化
- 2.增加$1\times 1$卷积，使得通道数为分类数目，即21(20前景和1背景)
- 3.双线性插值来上采样粗糙的输出特征(coarse outputs)成为像素级密集的输出特征(pixel-dense outputs)
- 3.增加逐像素的loss(a pixelwise loss)

注：

- 疑问1：如何将全连接层卷积化的（需看源码）
- 解答：见源码:
 - 以VGG16为例：Block5的输出尺寸为$7 \times 7 \times 512$，后接4096的全连接层；原VGG16中，是将$7 \times 7 \times 512$拉长成25088，然后和4096个神经元进行全连接。故原参数个数为：$25088 \times 4096+4096 =  102764544$。
 - 在FCN中，以卷积核为$7\times7$，步长为1，卷积核个数为4096，同时采用same卷积，使得该全连接层卷积化，该层的输出为$7 \times7 \times 4096$，即空间尺寸不变。
- 疑问2：如何实现上采样（需看源码）
- 解答：见源码，只是双线性插值 
- 疑问3：上采样、双线性插值、转置卷积到底有什么区别，如何实现(概念性问题)
- 解答：上采样的说法高于双线性插值/转置卷积；双线性插值不需学习；转置卷积带有参数，需要学习。
    
### 跳跃结构(skip architecture)
<img src="/images/FCN/2.png"  width = "700" height = "100"/>

- 设计跳跃结构的原因：
 - 语义分割面临一个内在的语义和定位之间张力：全局信息解决了是什么，有利于语义；局部信息解决了在哪儿，有利于定位。
 - DCNN将位置信息和语义信息进行编码，成为一个局部到全局的金字塔。
 - 因此，定义一个跳跃结构来将深层的粗糙的语义信息(deep,coarse,semantic information)和浅层的精细的轮廓信息(shallow,fine,appearance information)相结合。
- 跳跃结构是端到端地进行学习，来改善输出的语义和定位精度
- 如上图所示：
 - FCN-32s：基础的FCN称为FCN-32s。因为是从`pool5`输出的分辨率为$7 \times 7$的特征图直接插值到分辨率为$224 \times 224$的特征图，插值前后的分辨率比率为$1/32$。
 - FCN-16s：FCN-32s与`pool4`输出的特征图进行跳跃连接后的模型，称为FCN-16s。
 - FCN-8s：FCN-16s与`pool3`输出的特征图进行跳跃连接后的模型，称为FCN-8s。

<img src="/images/FCN/3.png"  width = "500" height = "100"/>

如上图所示，为增加跳跃结构之后的语义分割结果对比。

## 模型比较
<img src="/images/FCN/4.png"  width = "500" height = "100"/>

## 训练策略及各超参数

- 整体训练策略：加载VGG16模型在ImageNet上预训练的权重，基于VOC2011微调整个FCN网络：
 - 注1：加载VGG16的全连接层之前的层的权重
 - 注2：若只微调FCN的分类层，只能获得微调整个FCN网络70%的表现

- 预处理及数据增强：无

各超参数参考官方[`voc-fcn8s/solver.prototxt`](https://github.com/shelhamer/fcn.berkeleyvision.org/blob/master/siftflow-fcn8s/solver.prototxt)及[voc-fcn8s/deploy.prototxt](https://github.com/shelhamer/fcn.berkeleyvision.org/blob/master/voc-fcn8s/deploy.prototxt)：

- 输入尺寸：$500\times 500 \times 3 $
- 优化器：SGD
- 动量：0.99
- 初始学习率：1e-12
- 学习率策略：采用固定学习率
- 权重衰减：0.0005
- 批量尺寸：1
- 最大迭代次数：300000 

## 参考文献

- [论文：Fully Convolutional Networks for Semantic Segmentation,2015](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf)
- [视频：How FCN Fully Convolutional Networks Works for Semantic Segmentation](https://www.youtube.com/watch?v=UdZnhZrM2vQ)
- [代码及注释：dhkim0225/keras-image-segmentation](https://github.com/liminn/keras-image-segmentation/blob/master/model/fcn.py)
- [官方Caffe代码：shelhamer/fcn.berkeleyvision.org](https://github.com/shelhamer/fcn.berkeleyvision.org)




