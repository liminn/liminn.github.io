---
title: R-CNN
mathjax: true
date: 2019-01-16 22:26:50
categories: 
- 目标检测
tags:
---

# 摘要

- R-CNN是目标检测领域的里程碑方法，第一次提出“区域建议(region proposals)+CNN”的两阶段目标检测思路。
- R-CNN由四个模块构成：`区域建议生成算法`+`CNN`+`SVMs`+`边框回归(bounding-box regression)`。

<!-- more -->

# 创新点
## R-CNN总体结构

<img src="/images/R-CNN/1.png"  width = "700" height = "200"/>
如上图所示，为R-CNN的总体结构示意图，R-CNN由四个模块构成：
- 模块1：生成区域建议的算法
 - Selective Search算法，对一张图片，生成约2k个区域建议/RoI。
- 模块2：CNN
 - R-CNN通过该CNN分别为每个区域建议提取出固定长度的特征向量；CNN为AlexNet，因此提取出的特征向量的维度为4096维。
- 模块3：一系列SVM
 - SVM为2分类器，因此每一目标类别需要一个SVM。
- 模块4：边框回归
 - 将每一个区域建议通过CNN提取出的特征，通过特定类别(class-specific)的SVM进行分类后，再通过特定类别的边框回归器(bounding-box regressor)，预测出一个新的包围边框，改善定位表现。

# 总结
- 了解R-CNN的结构即可，R-CNN的所有环节都在后续被改进。

# 参考文献
- [论文：Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/pdf/1311.2524v3.pdf)
- [博客(极好)：Object Detection for Dummies Part 3: R-CNN Family](https://lilianweng.github.io/lil-log/2017/12/31/object-recognition-for-dummies-part-3.html#fast-r-cnn)
- [博客：边框回归(Bounding Box Regression)详解](https://blog.csdn.net/zijin0802034/article/details/77685438)
- [博客：Bounding box regression详解](https://blog.csdn.net/u011534057/article/details/51235964)



