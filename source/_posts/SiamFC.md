---
title: SiamFC
mathjax: true
date: 2019-04-10 22:26:50
categories: 
- [视频目标跟踪]
tags:
---

# 摘要
- 首次提出用全卷积的孪生网络(fully-convolutional Siamese network)，来处理视频目标跟踪问题。
- 该方法在视频目标跟踪任务上的速度能够达到实时。
- 根本思想：通过相似度学习(similarity learning)的方式，来处理对任意目标进行跟踪的问题。

<!-- more -->

# 创新点
## SiamFC总体结构

<img src="/images/SiamFC/1.png"  width = "800" height = "300"/>
如上图所示，为SiamFC的总体结构示意图：
- $z$ 为exemplar image，尺寸为$127\times127\times3$；$x$ 为search image，尺寸为$255\times255\times3$。
- $\varphi$ 代表卷积神经网络(文中采用AlexNet)，用于对exemplar image以及search image进行特征提取；$z$ 对应的输出特征图的尺寸为$6\times6 \times 128$，$x$ 对应的输出特征图的尺寸为$22\times 22 \times 128$。
- “*”代表对上述两个输出特征，进行相似度计算。具体地，计算两特征之间的互相关性(cross-correlation)。
- 经相似度计算之后，得到score map，尺寸为$17\times 17 \times 1$。其中，score map中的红色像素点和蓝色像素点，即对应于$x$ 中红色区域窗口和蓝色区域窗口。score map中红色像素点和蓝色像素点所包含的值，即代表了$x$ 中红色区域窗口和蓝色区域窗口与$z$的相似度。


## 相似度计算
xx

# 训练流程相关
## training pairs(exemplar image与search image)的制作
- 首先，每段视频标注数据包含：每一帧的原图以及每一帧中物体的包围框($x,y,w,h$)。
- 对某一段视频，选取其中不超过最大距离$T$的任意两帧，一帧用于制作exemplar image，一帧用于制作search image。
- 对于选出的两帧图片，以任意一帧为例，继续进行如下操作：
 - 在标注的包围框的基础上，对其四周进行外扩，以包含进上下文信息(context)。外扩的宽度为$p = (w+h)/4$；
 - 然后，计算出外扩后的新的面积$s = \frac{255}{127} \times (w+2p)\times(h+2p)$；
 - 然后，计算出新的方形区域的宽，即$w = \sqrt{s}$；
 - 然后，计算出新的方形区域的左上角坐标和右下角坐标；
 - 然后，依据计算出的方形区域的左上角坐标和右下角坐标，判断是否需要对原图像进行填充；若需要填充，则采用原图的RGB均值来进行填充；
 - 然后，从填充后的原图的方形区域中，CenterCrop出$127\times127$的区域(若该帧是exemplar image)，或$255\times255$的区域(若该帧是search image)。其中，可以进行随机裁剪，即中心点可以略微有些偏移，具体解释见[issue](https://github.com/huanglianghua/siamfc-pytorch/issues/12)。
- exemplar image与search image的示例如下：
<img src="/images/SiamFC/2.png"  width = "700" height = "300"/>

## label的制作


## 损失函数
x

## 训练流程
x

# 推理流程相关

# 实验结果

# 总结

# 参考文献
- [论文：Fully-Convolutional Siamese Networks for Object Tracking](https://arxiv.org/pdf/1606.09549.pdf)
- [代码及注释：huanglianghua/siamfc-pytorch](https://github.com/liminn/siamfc-pytorch)



