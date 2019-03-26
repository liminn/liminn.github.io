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

- 为了实现实例分割，Mask R-CNN在Faster R-CNN的基础上，添加一个并行的分支，用来预测目标的mask。因此，`Mask R-CNN` = `Faster R-CNN` + `Mask Head(FCN)`。
- Mask R-CNN在单GPU上的推理速度约为200ms，即5FPS。
- Mask R-CNN的技术要点：
 - 要点1——更强的基础网络：基础网络采用`ResNet/ResNeXt`+`FPN(Feature Pyramid Networks)`。
 - 要点2——RoIAlign：RoIPool适用于目标检测任务，但不适用于分割任务。因此，对RoIPool进行改进，提出RoIAlign。
 - 要点3——解耦mask预测与类别预测：Mask Head分支输出$K$个binary mask，只有RoI所属的GT类别所对应的第$k$个binary mask对loss做出贡献。因此，通过此种loss计算方式，避免了类间竞争。
<!-- more -->

# 创新点
## Mask R-CNN总体结构

<img src="/images/Mask R-CNN/1.png"  width = "1000" height = "300"/>
如上图所示，为Mask R-CNN的总体结构示意图。

## Backbone Architecture
backbone architecture用于提取整张图片的特征。在Mask R-CNN中，使用了如下几种backbone architecture：
- `ResNet-C4`：Faster R-CNN即是采用ResNet中4-th阶段的最后卷积层的特征来作为提取到的输出特征，C4即是此含义。
 - 疑问：Faster R-CNN为什么用C4？？？
- `ResNet-FPN`/`ResNeXt-FPN`：采用FPN作为backbone，使得Mask R-CNN在精度和速度上均得到显著受益。

## RoIAlign
RoIPool缺点：
- RoIPool在`坐标映射`以及`区域分块`两步骤，均在一次量化(quantization)。这两次量化，造成了RoI与提取出的特征之间的不对齐(misalignments)。
- 分类性能对此种不对齐现象，表现得很鲁棒，并未造成大的影响。因此，RoIPool在Fast/Faster R-CNN中工作的不错。
- 但在分割任务中，需要像素级别的特征。因此，RoIPool导致的不对齐现象，对分割任务有非常负面的影响。

<img src="/images/Mask R-CNN/2.png"  width = "600" height = "300">
RoIAligh做法，如Fig3所示：
- 在`坐标映射`步骤，不进行量化，保留浮点数级别的坐标映射。
- 在`区域分块`步骤，不进行量化，保留浮点数级别的矩形区域划分。
- 在每一个划分出的矩形区域中，采样四个点；然后通过双线性插值，计算出该四个采样点的准确值；然后，取四个采样点的最大值，为该矩形区域的输出值。
 - 实验发现：只要不使用任何量化，采样点的位置以及采样点的个数对结果的影响不大。
 - 疑问：那到底是如何采样的？随机采样？到底是如何通过双线性差值的方式计算出采样点的值的？

下图为RoIPool与RoIAlign的对比图：
<img src="/images/Mask R-CNN/3.png"  width = "1000" height = "300"/>

## Mask Head
<img src="/images/Mask R-CNN/4.png"  width = "500" height = "300"/>
如Fig4所示，Mask R-CNN在两种现有Faster R-CNN head的基础上，进行了拓展：
- 图我没看懂！
- 论文中该段相关的文字我也没看懂！已用蓝色标注出！

## Multi-task loss
在训练过程中，对于每一个RoI，其多任务loss为：
- $L = L_{cls} +  L_{box} + L_{mask}$
- 其中，分类损失$L_{cls}$和包围边框损失$L_{box}$与Fast R-CNN中定义的一样。

对于$L_{mask}$：
- 对于每一个RoI，Mask Head分支的输出通道数为$Km^2$。其中，$K$为类别数，$m \times m$为binary mask的分辨率，即该$K$个$m \times m$的binary mask分别对应了$K$个类别中的每一个类别。
- $L_{mask}$的计算方式为：对输出特征图施加逐像素的sigmoid，$L_{mask}$即等于`average binary cross-entropy loss`。
- 并且，只有该RoI所属的GT类别的第$k$个通道的binary mask对loss做出贡献，其余通道的binary mask均不做出贡献。
- 这种$L_{mask}$的定义方式，使得网络为每个类别生成mask，而没有类别之间的竞争。Mask R-CNN依靠分类分支去预测类别，然后再选择出输出的mask。这解耦了mask预测和类别预测。这与语义分割中常规的FCN的做法不同，它们使用逐像素的softmax(a per-pixel softamx)和多项式的交叉熵(a multinomial cross-entropy loss)，在这种情况里，mask通过了类别竞争。
- 实验证明了本文的做法对示例分割取得好结果很关键。

# 实施细节(超参数设置)
见文中3.1Implementation Details，关于训练和推理的超参数设置，均有详细的细节解释，还未看！

# 训练流程
补训练流程

# 实验结果
从文中4. Experiments: Instance Segmentation开始，还未看！

# 总结
x

# 参考文献
- [论文：Mask R-CNN](https://arxiv.org/pdf/1703.06870.pdf)
- [博客：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Image segmentation with Mask R-CNN](https://medium.com/@jonathan_hui/image-segmentation-with-mask-r-cnn-ebe6d793272)
- [视频：Paper Review : Mask RCNN - Instance Aware Semantic Segmentation](https://www.youtube.com/playlist?list=PLkRkKTC6HZMxZrxnHUDYSLiPZxiUUFD2C)
 


