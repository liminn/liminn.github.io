---
title: UNet
mathjax: true
date: 2019-01-08 22:26:50
categories: 
- 语义分割
tags:
---

## 摘要

- 提出UNet，包含收缩路径(contracting path)和扩张路径(expanding path):
 - 收缩路径来捕捉语义信息；
 - 对称的扩张路径能够得到更精确的定位。
- 提出了基于数据增广(data augmentation)的训练策略，更有效地利用起现有的标注样本。

<!-- more -->

## 创新点

### U-Net结构

<img src="/images/UNet/1.png"  width = "700" height = "200"/>

如Fig1所示，为U-Net的总体结构示意图：

- 左侧为收缩路径(contracting path)，右侧为扩张路径(expanding path)
- 收缩路径为典型的CNN结构：
 - 包含对两个堆叠$3\times3$卷积(不填充)的重复使用；
 - 每一次重复都跟随ReLU和步长为2的$2\times2$的最大池化操作；
 - 每一次下采样，都加倍特征的通道数。
- 对于扩张路径：
 - 扩张路径中的每一阶层都包含上采样特征图的操作，由$2\times2$的转置卷积实现，同时减半特征通道数；
 - 然后和对应的收缩路径中的剪裁过的特征图进行串联(concatenate)；剪裁是必要的，因为在每一次卷积中，边缘像素会丢失（注意该句话）；
 - 然后跟随两个$3\times3$卷积，跟随ReLU。
- 最后为$1\times1$卷积层，用来将64通道数的特征映射到想要的分类数目维度。
- U-Net总计有23个卷积层。

注：

 - 疑问1：这里的上采样操作，是反卷积不？看源码解决疑问
 - 答：这里的上采样操作，是转置卷积/反卷积，是有参数的；不是简单的双线性插值操作。

### 数据增广(Data Augmentation)

当只有较少的训练样本时，数据增广至关重要，可以教导网络学习到我们想要的不变性和鲁棒特征：

- 对训练数据施加弹性变形(elastic deformations):
 - 原因：这对于生物医学图片尤其重要，因为变形是组织结构中最常见的变化，并且这些变形可以被有效的仿真。
 - 效果：该数据增强操作可使得网络学习到对于这些变形的不变性，而不需要在标注数据集中看到这些变换。

<img src="/images/UNet/2.png"  width = "500" height = "100"/>
 
- 施加带有权重的损失函数：$E =\sum_{x\in \Omega} w(x)log(p_{l(x)}(x))$ 
 - 如Fig3所示，对于细胞分割任务，许多相互接触的同类细胞个体需要被分离
 - 预先定义每一个GT的权重图，来强迫网络去学习接触细胞之间的小的分离边界
 - 对于互相接触的细胞的待分离背景，在损失函数中，会得到较大的权重
 
## 模型比较

<img src="/images/UNet/3.png"  width = "700" height = "100"/>

## 参考文献

- [论文：U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/pdf/1505.04597.pdf)
- [视频：U-Net](https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/)
- [代码及注释：dhkim0225/keras-image-segmentation](https://github.com/liminn/keras-image-segmentation/blob/master/model/unet.py)

