---
title: Fast R-CNN
mathjax: true
date: 2019-01-17 22:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- Fast R-CNN基于R-CNN做出改进，联合了[R-CNN](http://cvnotes.cn/2019/01/16/R-CNN/)中的模块2与模块3：
 - Fast R-CNN通过CNN首先得到整张图像的特征图，所有的区域建议/ROI共享该特征图，这归功于RoI pooling层； 而R-CNN为图像中的每一个区域建议/ROI，通过CNN独立地提取特征；
 - Fast R-CNN通过多任务loss(a multi-task loss)，同时训练目标分类器(object classifier)和包围边框回归器(bounding-box regressor)；
- 综上，Fast R-CNN通过`RoI pooling layer`和`multi-task loss`，成功联合了R-CNN中的模块2与模块3；通过共享计算，无论训练阶段或推理阶段，均加速了R-CNN。

<!-- more -->

# 创新点
## RoI pooling layer
RoI pooling layer目的：对输入的任意尺寸的RoI，输出固定尺寸的特征图(如$7\times7$)
RoI pooling layer工作流程简述：

## Fast R-CNN总体结构

<img src="/images/Fast R-CNN/1.png"  width = "600" height = "200"/>
如上图所示，为R-CNN的总体结构示意图，R-CNN由三个模块构成：
- 模块1：生成区域建议的算法
 - Selective Search算法，对一张图片，生成约2k个区域建议/感兴趣区域。
- 模块2：CNN，分别为每个区域提取出固定长度的特征向量
 - CNN为AlexNet，因此提取出的特征向量的维度为4096维。
- 模块3：一系列SVM
 - SVM为2分类器，因此每一目标类别需要一个SVM。

# 总结
- 了解R-CNN的结构即可。

# 参考文献
- [论文：Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524v3.pdf)
- [博客(极好)：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：Region of interest pooling explained](https://deepsense.ai/region-of-interest-pooling-explained/)
- [博客：ROI Pooling层解析](https://blog.csdn.net/lanran2/article/details/60143861)
 


