---
title: Faster R-CNN
mathjax: false
date: 2019-01-18 22:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- 提出RPN(Region Proposal Network)来提供区域建议。其中，RPN与检测网络共享CNN，因此RPN可以几乎无代价的提供区域建议(3~10ms)。
- RPN网络提供区域建议，Fast R-CNN作为检测网络，因此，`Faster R-CNN` = `RPN` + `Fast R-CNN`。

<!-- more -->

# 创新点
## Faster R-CNN总体结构

<img src="/images/Faster R-CNN/1.jpeg"  width = "400" height = "400"/>
如上图所示，为Faster R-CNN的总体结构示意图。

## RPN
准确的说，**RPN的目的是为了预测一个anchor是前景/背景的概率，以及精修anchor的位置**。
想要理解RPN，需要理解以下几个方面：`Anchors的定义`->`RPN的训练数据制作`->`anchors的特征如何表达`->`RPN的损失函数`

### Anchors的定义
<img src="/images/Faster R-CNN/2.png"  width = "500" height = "400"/>
Anchors是Faster R-CNN中一个重要的概念，一个anchor就是一个包围框。在Faster R-CNN中，对于输入图像中的某个像素位置，其默认设置9个anchors。
如上图所示，为尺寸为(600,800)的输入图像，在(320,320)位置处的9个anchors：
- 定义三种尺度：$128 \times 128$、$256 \times 256$、$512 \times 512$；每个尺度采用三种比率：$1:1$、$1:2$、$2:1$
- 因此，产生9个不同尺寸的anchors，分别为$128 \times 128$、$\frac{128}{\sqrt{2}} \times (128 \times \sqrt{2})$、$(128 \times \sqrt{2}) \times \frac{128}{\sqrt{2}}$，其余6个依此类推。

因此，对于尺寸为(600,800)的输入图像，步长为16，则有$\frac{600}{16}\times \frac{800}{16} \times 9= 38 \times 50 \times 9 = 17100$个anchors。

注意：
- Faster R-CNN的anchors定义方式，对于Pascal VOC以及COCO数据集可以很好的适用。
- 但是，也可以自行定义不同种类的anchors。例如，对于行人检测，可以考虑非常窄且长的anchors。
- 整洁的anchors设置可以提高速度及精度。

### RPN的训练数据制作
RPN的目的是为了预测一个anchor是前景/背景的概率，以及精修anchor的位置。因此，RPN需要训练一个前景/背景的分类器以及一个包围边框的回归器。**因此，对于每一个anchor，需要标注该anchor是前景/背景（为了分类器），以及边框位置坐标（为了回归器）。**
对于一个anchor的类别标注，通过图像中原有的GT边框来进行标注：
- 将与GT边框有最大重叠率的anchor标注为前景，其他重叠率较低的anchors皆为背景。

对于一个anchor的位置坐标标注：
- 若anchor为前景，则对应的原图中的GT边框即为位置坐标标注，若anchor为背景，则无位置坐标标注。
- 该点存疑，待验证。是坐标偏移？？

### anchors的特征如何表达
以尺寸为(600,800)的输入图像为例，经CNNs后，输出步长为16，因此，得到尺寸为$39\times51$的特征图。特征图中的每一个位置，拥有9个anchors，每个anchor有着两个标签，即是前景/背景以及坐标位置。anchors的特征通过如下方式表达：
- 对于分类器，让特征图的深度为18($9\times2$)。因此，每个anchor拥有了两个values/logits来表征前景和背景。
- 对于回归器，让特征图的深度为36($9\times4$)。因此，每个anchor拥有了四个values来表征xx(坐标偏移？待验证)。

注意：
- **要注意所使用的CNNs的感受野，要确保特征图中每个位置的感受野能够cover住它所需表达的所有anchors**。否则，anchors的特征向量没有足够的信息来做出预测。
- 对于感受野，阅读参考文献中的[A guide to receptive field arithmetic for Convolutional Neural Networks](https://medium.com/mlreview/a-guide-to-receptive-field-arithmetic-for-convolutional-neural-networks-e0f514068807)。

### RPN的损失函数
对于RPN的损失函数，即有两部分构成：`分类损失`+`边框回归损失`
待补。

## Faster R-CNN训练流程
RPN在和Fast R-CNN共享CNN的情况下的训练流程，推理流程
待补。

# R-CNN家族对比
<img src="/images/Faster R-CNN/3.png"  width = "800" height = "400"/>

# 总结
待补。

# 参考文献
- [论文：Faster R-CNN](http://openaccess.thecvf.com/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
- [博客：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Faster R-CNN Explained](https://medium.com/@smallfishbigsea/faster-r-cnn-explained-864d4fb7e3f8)
- [博客：A guide to receptive field arithmetic for Convolutional Neural Networks](https://medium.com/mlreview/a-guide-to-receptive-field-arithmetic-for-convolutional-neural-networks-e0f514068807)
- [视频：Paper Review : Faster R-CNN for Real-time Object Detection](https://www.youtube.com/playlist?list=PLkRkKTC6HZMzp28TxR_fJYZ-K8Yu3EQw0)
- [x](https://blog.csdn.net/u014380165/article/details/80379812)
- [x](https://towardsdatascience.com/fasterrcnn-explained-part-1-with-code-599c16568cff)
- [x](https://blog.csdn.net/lanran2/article/details/54376126)
 


