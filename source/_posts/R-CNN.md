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
- R-CNN由三个模块构成：`区域建议生成算法`+`CNN`+`SVMs`。

<!-- more -->

# 创新点
## R-CNN总体结构

<img src="/images/R-CNN/1.png"  width = "700" height = "200"/>
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
 


