---
title: FPN
mathjax: false
date: 2019-01-18 23:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- FPN(Feature Pyramid Networks)的目的是通过特征金字塔来解决检测任务中的多尺度目标问题。
- `FPN` = `bottom-up pathway` + `top-down pathway` + `lateral connections `
 - `bottom-up pathway`：任意backbone ConvNet，文中通过ResNet，构成自下而上的路径。
 - `top-down pathway`：通过对`bottom-up pathway`中各金字塔层级的特征进行逐步上采样，构成自上而下的路径。
 - `lateral connections`：横向连接将`bottom-up pathway`与`top-down pathway`中对应层级的特征相加，弥补丢失的空间信息，增强定位能力。
- 文中将FPN与RPN以及Faster R-CNN相结合，大幅提升了原有效果。

<!-- more -->

# 创新点
## FPN总体结构

<img src="/images/FPN/2.png"  width = "450" height = "400"/>
如上图所示，为FPN的总体结构示意图：
首先，见总体结构示意图中的"bottom-up"部分所示：
- Bottom-up pathway即为主干网络，文中选用ResNet；ResNet由`conv1`、`conv2_x`、`conv3_x`、`conv4_x`、`conv5_x`构成；
- 文中选用`conv2_x`、`conv3_x`、`conv4_x`、`conv5_x`的输出特征$\{C_2,C_3,C_4,C_5\}$为四个金字塔层级，其输出步长分别为$\{4,8,16,32\}$。
- 文中未将`conv1`的输出特征包含进金字塔之中，是因为其需要大量的内存占用。

然后，见总体结构示意图中的"top-down"部分所示：
- 首先，对$C_5$施加$1\times1$卷积(通道数固定为$d=256$)，得到$M_5$
- 然后，进行横向连接，将$M_5$进行上采样(最近邻插值)后的特征图，与$C_4$进行$1\times1$卷积(通道数固定为$d=256$)后的特征图，进行像素级加法(element-wise addition)，即得到$M_4$；依此类推，得到$M_3$和$M_2$
- **为什么要横向连接**：将$M_5$进行上采样(最近邻插值)后的特征图记为$M_{4before}$。$M_{4before}$相比于$C_4$，$M_{4before}$的**语义信息更丰富(分类能力强)，但空间信息更粗糙(定位能力弱)**。因此，将$C_4$于$M_{4before}$进行连接，补充$M_{4before}$的空间信息，弥补其定位能力的不足。

最后，见总体结构示意图中的最右侧一列：
- **为了减少上采样带来的混叠影响(reduce the aliasing effect of upsampling)**：对$M_4$进行$3\times 3$卷积，得到$P_4$。依此类推，得到$P_3$以及$P_2$。
- FPN各金字塔层级的最终特征图即为${P_5,P_4,P_3,P_2}$，分别对应于${C_5,C_4,C_3,C_2}$，它们的空间尺寸，对应相等。


## FPN with RPN

## FPN with Fast R-CNN or Faster R-CNN

## Segmentation

# 总结


# 参考文献

- [论文：Feature Pyramid Networks for Object Detection](https://arxiv.org/pdf/1612.03144.pdf)
- [博客：Understanding Feature Pyramid Networks for object detection (FPN)](https://medium.com/@jonathan_hui/understanding-feature-pyramid-networks-for-object-detection-fpn-45b227b9106c)
- [x](https://www.jiqizhixin.com/articles/2017-07-25-2)
- [x](https://blog.csdn.net/u014380165/article/details/72890275)
 


