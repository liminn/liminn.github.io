---
title: Mask R-CNN
mathjax: true
date: 2019-01-19 22:26:50
categories: 
- [目标检测]
- [实例分割]
tags:
---

# 摘要

- 为了实现实例分割，Mask R-CNN在Faster R-CNN的基础上，添加一个并行的分支，用来预测目标的mask。因此，`Mask R-CNN` = `Faster R-CNN` + `FCN`。
- Mask R-CNN的技术要点：
 - 要点1——更强的基础网络：基础网络采用`ResNet/ResNeXt`+`FPN(Feature Pyramid Networks)`。
 - 要点2——RoIAlign：RoIPool适用于目标检测任务，但不适用于分割任务。因此，对RoIPool进行改进，提出RoIAlign。
 - 要点3——解耦mask预测与类别预测：FCN分支输出$K$个binary mask，只有所属类别的第$k$个binary mask对loss做出贡献。因此，通过此种loss计算方式，避免了类间竞争。
<!-- more -->

# 创新点
## Mask R-CNN总体结构

<img src="/images/Mask R-CNN/1.png"  width = "1000" height = "300"/>
如上图所示，为Mask R-CNN的总体结构示意图。


## FPN
x

## RoIAlign
RoIPool缺点：
- RoIPool在`坐标映射`以及`区域分块`两步骤，均在一次量化(quantization)，因此打破了像素到像素的对齐(breaks pixel-topixel alignment)。
- RoIPool是为检测任务而设计，包围边框不需要像素级别的精度。因此，RoIPool在目标检测任务中可以表现的不错。
- 但在分割任务中，需要像素级别的特征。因此，RoIPool导致的选择出的特征图区域与原图中对应的图像区域不对齐(misaligned)，对分割任务不利。

RoIAligh做法：
- 在`坐标映射`步骤，不进行量化，保留浮点数级别的坐标映射。
- 在`区域分块`步骤，不进行量化，保留浮点数级别的区域划分。在每一个划分出的矩形单元(bin)中，将其划分成四部分，每部分中的中点为一个采样点。各采样点的值通过双线性差值获得。然后，取四个采样点的最大值，为该bin的输出值。

下图为RoIPool与RoIAlign的对比图：
<img src="/images/Mask R-CNN/2.png"  width = "1000" height = "300"/>


## FCN
FCN分支，解耦了mask和类别预测，避免了类间竞争：
- 在FCN分支中，每个RoI特征的输出维度为$K$，$K$即类别个数。
- 输出特征的每个维度均为binary mask，其loss计算方式如下：$L_{mask}(Cls_{k}) = \frac{1}{N} Sum(sigmoid(Cls_{k}))$，即平均二值交叉熵损失(average binary cross-entrop loss)。
- 只有第$k$个通道的binary mask对loss做出贡献，其他通道的binary mask不贡献loss。
- 因此，通过此种loss计算方式，避免了类间竞争。

## 总体训练流程
补训练流程，推理流程

# 实验结果
<img src="/images/Mask R-CNN/3.png"  width = "900" height = "300"/>
如Table1所示，为xx。

# 总结
x

# 参考文献
- [论文：Mask R-CNN](https://arxiv.org/pdf/1703.06870.pdf)
- [博客：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Image segmentation with Mask R-CNN](https://medium.com/@jonathan_hui/image-segmentation-with-mask-r-cnn-ebe6d793272)
- [视频：Paper Review : Mask RCNN - Instance Aware Semantic Segmentation](https://www.youtube.com/playlist?list=PLkRkKTC6HZMxZrxnHUDYSLiPZxiUUFD2C)
 


